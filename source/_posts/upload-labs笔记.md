---
title: upload-labs笔记
date: 2018-11-06 17:22:33
tags:
---

# upload-labs笔记

---

## 0x01

![001](/img/fileupload/upload-labs/001.png)

文件在前端使用了js函数检测,绕过前端即可。

使用firefox的插件NoScript直接关掉js.

![001_1](/img/fileupload/upload-labs/001_1.png)

![001_2](/img/fileupload/upload-labs/001_2.png)

![001_3](/img/fileupload/upload-labs/001_3.png)

## 0x02

观察源码可以分析到只对文件的类型做了白名单检测,burp抓包后修改MIME文件类型即可成功上传。

![002](/img/fileupload/upload-labs/002.png)

![002_1](/img/fileupload/upload-labs/002_1.png)

这里有个坑啊,原来我修改的jpg格式竟然不行。

![002_3](/img/fileupload/upload-labs/002_3.png)

## 0x03

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

### 重写文件解析规则绕过

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