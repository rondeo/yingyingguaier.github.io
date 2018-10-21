---
title: sqli-labs lession-42 POST报错注入-字符型-堆叠注入
date: 2018-10-21 16:44:55
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-42 POST报错注入-字符型-堆叠注入

---

## 分析

login.php:

![001](/img/sql/Lesson-42/001.png)

只对username进行了转义，而password没有处理,导致存在万能密码`'or'1'='1`

![002](/img/sql/Lesson-42/002.png)

通过密码填写`' or '1'='1' and id='2` ,还能登入id=2的用户。

![002_1](/img/sql/Lesson-42/002_1.png)

![002_2](/img/sql/Lesson-42/002_2.png)

同时还可发现底下的sql函数是可以执行多个sql语句的，可以利用password来堆叠注入。

![003](/img/sql/Lesson-42/003.png)

logged-in.php：

利用万能密码进入后是一个修改密码的界面。

![004](/img/sql/Lesson-42/004.png)

![005](/img/sql/Lesson-42/005.png)

蓝线的是进行了转义,红线是有可能注入的地方。

## 注入

利用报错注入获取信息(这是基本功,不写过程了)。

利用堆叠注入插入用户。

账号不填，密码填写如下。

`';INSERT INTO users(username, password) VALUES ("secure'#", "123456");`

登录返回如下界面

![006](/img/sql/Lesson-42/006.png)

但是数据库中插入数据成功。

![007](/img/sql/Lesson-42/007.png)

同样还可以进行其他增删改操作。

## 二次注入

因为logged-in.php页面直接拿session中的username来用了,没有进行检查,可以添加一个（用户‘#）来截断SQL语句。

例如用`secure'#`达到修改secure用户密码的目的。

原来数据库。

![008](/img/sql/Lesson-42/008.png)

利用前面的方式添加`secure'#`。正常登录改密界面。

修改密码为233。

![009](/img/sql/Lesson-42/009.png)

![010](/img/sql/Lesson-42/010.png)

返回修改成功页面。

![011](/img/sql/Lesson-42/011.png)

这里的username最大只有20个字符,其他的sql语句不好利用了。

这里只是提供了一种思路,就这一课目的而言有了堆叠注入完全用不到二次注入了。





