---
title: 'CentOS7'
date: 2018-02-27 22:28:52
tags: CentOS
---
CentOS 7 安装配置LAMP服务器方法(Apache+PHP+MariaDB)
摘要: CentOS 7 安装配置LAMP服务器方法(Apache+PHP+MariaDB)，CentOS 7后Mysql改为MariaDB
<!--more-->
准备篇：

一、配置防火墙，开启80端口、3306端口

CentOS 7 默认使用的是firewall作为防火墙，这里改为iptables防火墙。

1、关闭firewall：
```
systemctl stop firewalld.service #停止firewall 
systemctl disable firewalld.service #禁止firewall开机启动
```
2、安装iptables防火墙

```
yum install iptables-services #安装 
vi /etc/sysconfig/iptables #编辑防火墙配置文件
```
```
# Firewall configuration written by system-config-firewall 
# Manual customization of this file is not recommended. 
*filter 
:INPUT ACCEPT [0:0] 
:FORWARD ACCEPT [0:0] 
:OUTPUT ACCEPT [0:0] 
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT 
-A INPUT -p icmp -j ACCEPT 
-A INPUT -i lo -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT 
-A INPUT -j REJECT --reject-with icmp-host-prohibited 
-A FORWARD -j REJECT --reject-with icmp-host-prohibited 
COMMIT 
:wq! #保存退出
```
```
systemctl restart iptables.service #最后重启防火墙使配置生效 
systemctl enable iptables.service #设置防火墙开机启动
```
二、关闭SELINUX
```
vi /etc/selinux/config
#SELINUX=enforcing #注释掉
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加
:wq! #保存退出
setenforce 0 #使配置立即生效
```
安装篇：

一、安装Apache
```
yum install httpd #根据提示，输入Y安装即可成功安装
systemctl start httpd.service #启动apache
systemctl stop httpd.service #停止apache
systemctl restart httpd.service #重启apache
systemctl enable httpd.service #设置apache开机启动
```
二、安装MariaDB

CentOS 7 中，已经使用MariaDB替代了MySQL数据库

1、安装MariaDB
```
yum install mariadb mariadb-server #询问是否要安装，输入Y即可自动安装,直到安装完成
systemctl start mariadb.service #启动MariaDB
systemctl stop mariadb.service #停止MariaDB
systemctl restart mariadb.service #重启MariaDB
systemctl enable mariadb.service #设置开机启动
cp /usr/share/mysql/my-huge.cnf /etc/my.cnf #拷贝配置文件（注意：如果/etc目录下面默认有一个my.cnf，直接覆盖即可）
```
2、为root账户设置密码
```
mysql_secure_installation
```
回车，根据提示输入Y

输入2次密码，回车

根据提示一路输入Y

最后出现：Thanks for using MySQL!

MySql密码设置完成，重新启动 MySQL：
```
systemctl restart mariadb.service #重启MariaDB
```
三、安装PHP

1、安装PHP 和 php-devel工具 方便日后使用 phpize 做扩展编译
```
yum install php php-devel #根据提示输入Y直到安装完成
```
2、安装PHP组件，使PHP支持 MariaDB
```
yum install php-mysql php-gd libjpeg* php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-bcmath php-mhash
```
这里选择以上安装包进行安装，根据提示输入Y回车
```
systemctl restart mariadb.service #重启MariaDB

systemctl restart httpd.service #重启apache
```
注意：apache默认的程序目录是/var/www/html

权限设置：
```
chown -R apache:apache  /var/www/html
```
至此，CentOS 7.0安装配置LAMP服务器(Apache+PHP+MariaDB)教程完成！
原文链接：https://my.oschina.net/sallency/blog/467647