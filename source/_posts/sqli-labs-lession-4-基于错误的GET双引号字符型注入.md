---
title: sqli-labs lession-4 基于错误的GET双引号字符型注入
date: 2018-09-20 09:08:58
tags: [sqli-labs]
categories: sql注入
---
# sqli-labs lession-4 基于错误的GET双引号字符型注入 #

## 登录界面 ##

![1](https://i.imgur.com/J6A7U4R.png)

## 注入过程 ##

题目提示是基于错误的,根据提示应该先判断错误类型。前三课分别使用了

* 1	`$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";`
* 2 `$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";`
* 3	`$sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1";`

所以不可能是上面的3种。尝试下双引号。

`http://10.60.250.214/less/Less-4/?id=1%22`

![2](https://i.imgur.com/DVv5q5W.png)

返回报错信息:

`You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near '"1"") LIMIT 0,1' at line 1`

根据返回的错误信息猜测到SQL语句的源码应该为:

`$sql="SELECT * FROM users WHERE id=("$id") LIMIT 0,1";`

查看SQL语句,果然如此。

![3](https://i.imgur.com/Z0Eor43.png)

`http://10.60.250.214/less/Less-4/?id=1") order by 100%23`

![4](https://i.imgur.com/89fae8M.png)

成功！！！

剩下的操作无论是手注还是使用SQLMAP都和Lesson-1一样。

具体流程可以查看这里。

{% post_link sqli-labs-lession-1-基于错误的GET单引号字符型注入 点击查看Lesson-1 %}