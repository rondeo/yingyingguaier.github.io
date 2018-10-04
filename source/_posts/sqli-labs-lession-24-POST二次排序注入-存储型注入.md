---
title: sqli-labs lession-24 POST二次排序注入-存储型注入
date: 2018-10-03 19:25:09
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-24 POST二次排序注入-存储型注入

---

## 登录界面

![001](/img/sql/Lesson-24/001.png)

## 分析

主要有3个php文件:`login.php`负责查询数据库用户存在和验证登录,`login_create.php`创建新用户,`pass_change.php`修改老用户密码。

![002](/img/sql/Lesson-24/002.png)

本次的过滤在账户登录和密码处,使用了`mysql_escape_string`函数

Note: mysql_escape_string() 并不转义 % 和 _。

输入的特殊字符会被它转义而不奏效,但是之后存储进数据库时依然会还原成原来的样子。

经过代码审计发现`pass_change.php`中直接把session中的用户名直接拿来用了,结合上面的过滤函数,如果知道切确的用户,可以构造单引号闭合来修改其密码。

![003](/img/sql/Lesson-24/003.png)

假如知道了个用户`Dumb`,可以创建一个新用户`Dumb'#`来修改用户`Dumb`密码。

可以看见我已经创建了`Dumb'#`用户。

![004](/img/sql/Lesson-24/004.png)

用该用户进入修改密码的界面修改密码。

![005](/img/sql/Lesson-24/005.png)

可以发现确实把密码改了。

![006](/img/sql/Lesson-24/006.png)

这里下面没有报错的地方,没有办法利用报错注入,利用时间注入的话需要的语句很长,这里的数据库账户长度只支持到20个字符,感觉只能到此为止了。如果这里的字符长度允许的话,可以构造类似`admin' and if(8=(length(database())),sleep(60),null)#`的SQL语句来注入。

但是本课的思想已经体会到了:

对于存储型的注入,可以先将导致SQL注入的字符预先存到数据库中,当再次调用到这个恶意构造的字符时就可以触发注入。