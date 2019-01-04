---
title: sqli-labs-lession 12 基于错误的POST双引号字符型注入
date: 2018-09-27 13:38:47
tags: [sqli-labs]
categories: sql注入
---
# sqli-labs-lession 12 基于错误的POST双引号字符型注入 #
---
## 登录界面 ##

![1](/img/sql/Lesson-12/1.png)

## 注入过程 ##

根据标题知道这一课和上一课区别只在引号,来判断一下类型。

![2](/img/sql/Lesson-12/2.png)

没有反应,尝试`"`

![3](/img/sql/Lesson-12/3.png)

获得报错信息

`You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '1") LIMIT 0,1' at line 1`

根据报错信息得知它是双引号加括号闭合。在hackbar中添加`)`和`#`注释。

`uname=1") order by 100#&passwd=1&submit=Submit`

![4](/img/sql/Lesson-12/4.png)

接下来操作跟Lesson-11一样,详细的看Lesson-11。

{% post_link sqli-labs-lession-11-基于错误的POST单引号字符型注入 查看Lesson-11%}