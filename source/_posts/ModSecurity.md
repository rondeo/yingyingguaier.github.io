---
title: ModSecurity
date: 2018-12-28 15:24:32
tags: [环境搭建]
categories: 
---

# ModSecurity

---

## 配置环境(ubuntu18.04)

```shell
#备份源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
#添加阿里云源
sudo vim /etc/apt/sources.list

#把以下内容覆盖
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

#保存退出后更新软件列表
sudo apt-get update
#更新软件包
sudo apt-get upgrade
```

## 安装apache

```shell
sudo apt install apache2
```

![001](/img/modsecurity/001.png)

## 安装MYSQL

```shell
sudo apt install mysql-server
```

### 设置root密码

```shell
sudo su
mysql
mysql> select user, plugin from mysql.user;
+------------------+-----------------------+
| user             | plugin                |
+------------------+-----------------------+
| root             | auth_socket           |
| mysql.session    | mysql_native_password |
| mysql.sys        | mysql_native_password |
| debian-sys-maint | mysql_native_password |
+------------------+-----------------------+

mysql> update mysql.user set authentication_string=PASSWORD('密码'), plugin='mysql_native_password' where user='root';
mysql> flush privileges;
mysql> exit

#重启mysql
sudo /etc/init.d/mysql restart
mysql -uroot -p
```

## 安装PHP

```shell
sudo apt install php libapache2-mod-php php-mysql
#添加info.php文件来测试
<?php
phpinfo();
?>
```

![002](/img/modsecurity/002.png)



```shell
#默认版本不符合实验，安装PHP5.6
sudo add-apt-repository -y ppa:ondrej/php
sudo apt update
sudo apt install php5.6

#首先使用命令禁用 PHP 7.2 模块
sudo a2dismod php7.2
#启用 PHP 5.6 模块
sudo a2enmod php5.6

sudo apt install libapache2-mod-php5.6 php-mysql5.6
#重启apache2
sudo systemctl restart apache2
```

其他配置

```shell
#将 PHP 5.6 设置为默认版本
sudo update-alternatives --set php /usr/bin/php5.6
#设置默认情况下要使用的全局 PHP 版本
sudo update-alternatives --config php
```



## 配置phpmyadmin

```shell
#安装phpmyadmin,安装中会要求输入数据库连接密码
sudo apt install phpmyadmin

#创建软链接到web根目录下（我的是/var/www/html/)
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin

sudo apt-get install php5.6-mbstring
sudo apt-get install php5.6-gettext
```

如果出现`The mbstring extension is missing. Please check your PHP configuration.`的错误

```shell
#确保php_mbstring安装，且配置文件打开(下面是我的目录)
/etc/php/5.6/apache2/php.ini
```

![003](/img/modsecurity/003.png)

## modsecurity

```shell
#添加源
sudo vim /etc/apt/sources.list

deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse

sudo apt-get update
```

### 安装modsecurity

```shell
sudo apt-get install libxml2 libxml2-dev libxml2-utils libaprutil1 libaprutil1-dev libapache2-modsecurity
```

### 开启ModSecurity功能

```shell
cd /etc/modsecurity/
sudo mv modsecurity.conf-recommended modsecurity.conf
sudo vim /etc/modsecurity/modsecurity.conf

查找SecRuleEngine，并将其修改为On：
SecRuleEngine On
```
### 安装OWASP Rule Set

```shell
cd ~
sudo wget https://github.com/root25/MODSEC/raw/master/modsecurity-crs_2.2.5.tar.gz
sudo tar -zxvf modsecurity-crs_2.2.5.tar.gz
sudo mkdir -p /usr/share/modsecurity-crs
sudo cp -R modsecurity-crs_2.2.5/* /usr/share/modsecurity-crs/
```

在`/usr/share/modsecurity-crs/`的目录下，有主要的几个规则目录，activated_rules、slr_rules、optional_rules和base_rules.

将srl_rules、base_rules和optional_rules目录下的所有conf文件，拷贝到activated_rules目录下。

```shell
sudo cp /usr/share/modsecurity-crs/modsecurity-crs_10_setup.conf.example /usr/share/modsecurity-crs/modsecurity-crs_10_setup.conf

cd /usr/share/modsecurity-crs/activated_rules

sudo cp /usr/share/modsecurity-crs/base_rules/* .

sudo cp /usr/share/modsecurity-crs/optional_rules/* .

sudo cp /usr/share/modsecurity-crs/slr_rules/* .
```

### 在apache中启用modsecurity模块

```shell
sudo vim /etc/apache2/mods-available/security2.conf
#在 … 中加入以下内容，保存退出。
includeOptional /etc/modsecurity/*.conf
include /usr/share/modsecurity-crs/*.conf
include /usr/share/modsecurity-crs/activated_rules/*.conf

#因为要用自己的规则，修改如下
<IfModule security2_module>
        # Default Debian dir for modsecurity's persistent data
        SecDataDir /var/cache/modsecurity

        # Include all the *.conf files in /etc/modsecurity.
        # Keeping your local configuration in that directory
        # will allow for an easy upgrade of THIS file and
        # make your life easier
        IncludeOptional /etc/modsecurity/*.conf

        # Include OWASP ModSecurity CRS rules if installed
        #IncludeOptional /usr/share/modsecurity-crs/owasp-crs.load
        include /usr/share/modsecurity-crs/activated_rules/my.conf
</IfModule>
```

### 启用headers和Modsecurity

```shell
sudo a2enmod headers
sudo a2enmod security2
```
在中`/usr/share/modsecurity-crs/activated_rules/my.conf`编写规则测试即可

```shell
SecRule ARGS "@contains select" "id:00001,msg:'SQL Injection maybe',phase:1,t:compressWhitespace,t:htmlEntityDecode,deny"

SecRule ARGS "@contains union" "id:00002,msg:'SQL Injection maybe',phase:1,t:compressWhitespace,t:htmlEntityDecode,block" 
```

