---
title: upload-labs笔记
date: 2018-11-06 17:22:33
tags: [文件上传]
categories: 文件上传
---

# upload-labs笔记

---

## 0x01

### 前端绕过

![001](/img/fileupload/upload-labs/001.png)

文件在前端使用了js函数检测,绕过前端即可。

使用firefox的插件NoScript直接关掉js.

![001_1](/img/fileupload/upload-labs/001_1.png)

![001_2](/img/fileupload/upload-labs/001_2.png)

![001_3](/img/fileupload/upload-labs/001_3.png)

## 0x02

### MIME绕过

观察源码可以分析到只对文件的类型做了白名单检测,burp抓包后修改MIME文件类型即可成功上传。

![002](/img/fileupload/upload-labs/002.png)

![002_1](/img/fileupload/upload-labs/002_1.png)

这里有个坑啊,原来我修改的jpg格式竟然不行。

![002_3](/img/fileupload/upload-labs/002_3.png)

## 0x03

### 特殊可解析文件绕过

尝试直接上传php文件,不予许上传,估计应用了黑名单。

![003](/img/fileupload/upload-labs/003.png)

尝试php1,发现能上传

![003_1](/img/fileupload/upload-labs/003_1.png)

由于没有配置其他php文件后缀,这个思路也不行。

如果Apache的配置文件可以匹配php5的其他配置文件,那这里的黑名单就没有过滤。

![003_2](/img/fileupload/upload-labs/003_2.png)

Apache和php常用的php程序文件后缀有phtml、pht、php3、php4和php5。

这里还暂时没有其他思路。

## 0x04

黑名单检测。可以上传的文件gif ,png, jpeg;大小写无效;

![004_0](/img/fileupload/upload-labs/004_0.png)

### Apache解析漏洞

![004](/img/fileupload/upload-labs/004.png)

### .htaccess重写文件解析规则绕过

上传先上传一个名为`.htaccess`文件，内容如下：

```
<FilesMatch "04.png">
SetHandler application/x-httpd-php
</FilesMatch>
```

![004_1](/img/fileupload/upload-labs/004_1.png)

再上传04.png的文件

![004_2](/img/fileupload/upload-labs/004_2.png)

产生重写文件解析规则绕过绕过的原因是httpd.conf文件中的两处配置文件的原因和过滤不严。

```
#允许重写覆盖相关配置
AllowOverride All 
LoadModule rewrite_module modules/mod_rewrite.so
```

### PHP 和 Windows环境的叠加特性

分析文章：`https://www.ctolib.com/topics-88860.html`

`:截断`产生空文件

![004](/img/fileupload/upload-labs/004_3.png)

然后将文件名改为`04.<`或`04.<<<`或`04.>>>`或`04.>><`后再次上传，重写`4.php`文件内容，Webshell代码就会写入原来的`04.php`空文件中。

![004](/img/fileupload/upload-labs/004_4.png)

## 0x05

### 大小写绕过

黑盒测试发现是黑名单过滤

```
特殊可解析后缀 ×
上传`.htaccess`×
点绕过 × 
windows流绕过 ×
```

最后发现后缀大小写能绕过

![005_1](/img/fileupload/upload-labs/005_1.png)

可以查看源码看看还有什么可以绕过。

![005_2](/img/fileupload/upload-labs/005_2.png)

## 0x06

### 空格绕过

黑盒测试发现是黑名单过滤

```
上传后重命名文件 √
后缀大小写 ×
特殊可解析后缀 ×
::$DATA绕过 ×
空格绕过 √
```

![006_1](/img/fileupload/upload-labs/006_1.png)

查看下源码

![006_2](/img/fileupload/upload-labs/006_2.png)

## 0x07

### .绕过

```
黑名单
上传后没有改名 √
可上传jreg、png、gif、jpg √
可上传php1 pHp1 √
空格绕过  ×
.htaccess ×
.绕过 √
```

![007_1](/img/fileupload/upload-labs/007_1.png)

![007_2](/img/fileupload/upload-labs/007_2.png)

查看源码看看

![007_3](/img/fileupload/upload-labs/007_3.png)

没有过滤`.`的话应该可以利用windows和php叠加特性创建php空文件。

### windows和php叠加特性绕过

![007_4](/img/fileupload/upload-labs/007_4.png)

再利用`<`进行重写。

![007_5](/img/fileupload/upload-labs/007_5.png)

写入成功

![007_6](/img/fileupload/upload-labs/007_6.png)

## 0x08

### ::$DATA绕过

```
黑名单
上传后没有改名		√
可上传jreg、png、gif、jpg		√
可上传::$DATA		√
可上传php1,pHp1,php.xxx,php. .		√
空格绕过  ×
.htaccess ×
.绕过 ×
```

::DATA仅支持Windows,在info.php中写入<?php phpinfo();?> 上传info.php::$DATA会在上传目录上生成一个info.php的文件

![008_1](/img/fileupload/upload-labs/008_1.png)

底下插入了phpinfo函数。

![008_2](/img/fileupload/upload-labs/008_2.png)

![008_3](/img/fileupload/upload-labs/008_3.png)

### 相关特性

```
假设上传个info.php文件,服务器为windows,info.php内容为<?php phpinfo();?>

info.php:a.jpg 生成info.php,内容为空
info.php::$DATA 生成info.php,内容为<?php phpinfo();?>
info.php::$INDEX_ALLOCATION  生成info.php文件夹
info.php::$DATA\0.jpg    生成0.jpg,内容为<?php phpinfo();?>


```

把Pass-07的检查::$DATA的注释掉测试一下

![008_4](/img/fileupload/upload-labs/008_4.png)

以上的测试在本地通过。

## 0x09

### php. .绕过

```
上传后没有改名		√
可上传jreg、png、gif、jpg		√
可上传::$DATA		×
可上传php1,pHp1,php.xxx,php. .		√
.绕过 ×
.htaccess ×
```

根据没有改名和可以上传`php. .`猜测可能存在绕过。

![009_1](/img/fileupload/upload-labs/009_1.png)

![009_2](/img/fileupload/upload-labs/009_2.png)

查看源码

![009_3](/img/fileupload/upload-labs/009_3.png)

产生问题的原因是只删除了一次点号,当上传`info.php. .`时,先删除第一个点得到`info.php. `最后变成了点绕过。

## 0x10

### 双写后缀名绕过

```
发现php替换成了空
上传default.pphphp即可成功,应该是源代码中只替换了一次
```

![010_1](/img/fileupload/upload-labs/010_1.png)

![010_2](/img/fileupload/upload-labs/010_2.png)