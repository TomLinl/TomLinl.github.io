---
title: shadowsocks
date: 2018-02-28 00:30:31
tags: shadowsocks
---
VPS服务器超简单搭建shadowsocks 方法
vultr服务器购买：https://www.vultr.com/
安装Xshell，安装完成后新建会话（Alt+N），协议选择SSH，主机填写之前的IP Address，端口号选择22。
<!--more-->
VPS可以先升级
```
yum -y update
```

有些VPS 没有wget
这种要先装
```
yum -y install wget
```

输入以下命令：
```
1.wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
2.chmod +x shadowsocks.sh
3../shadowsocks.sh 2>&1 | tee shadowsocks.log
```
第一行是下载命令，下载东西，第二行是修改权限，第三行是安装命令
```
配置：
密码：（默认是teddysun.com）
端口：默认是8989
然后按任意键安装，退出按 Ctrl+c
```

安装完成会有一个配置
```
Congratulations, shadowsocks install completed!Your Server IP:  ***** VPS的IP地址Your Server Port:  *****  你刚才设置的端口Your Password:  ****  你刚才设置的密码Your Local IP:  127.0.0.1 Your Local Port:  1080 Your Encryption Method:  aes-256-cfb Welcome to visit:https://teddysun.com/342.htmlEnjoy it!
```
然后就可以用这些东西

VPS主机一键安装Google TCP BBR加速
因为VPS架构系统的不同，我们很多软件的安装、部署也是具有局限性的。比如搬瓦工以前都是OpenVZ架构，是不可以安装类似锐速、Google TCP BBR来给服务器加速的。在上周的时候，我们都有看到新增KVM架构方案，虽然还处于测试就阶段，但是也是可以购买的，数据中心在洛杉矶QN，最为主要的是KVM架构，我们可以安装Google TCP BBR工具。

在这篇文章中，将利用teddysun分享的一键Google TCP BBR安装工具脚本（https://teddysun.com/489.html）来部署到搬瓦工KVM VPS中，看看速度是否提高。这样我们用来建站或者其他用途的速度应该还是可以提升的。

第一、系统支持
我们系统需要是CentOS 6+，Debian 7+，Ubuntu 12+，除了OpenVZ架构之外都支持，比如常规的KVM和XEN。

第二、一键脚本安装
```
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```
自动安装，然后最后提示我们需要重启生效。

第三、检查BBR启动
1、检查核心
```
uname -r
```
检查内核是不是4.10，检测后看到的是"4.10.5-1.el6.elrepo.x86_64"。

2、执行后返回
"net.ipv4.tcp_available_congestion_control = bbr cubic reno"
```
sysctl net.ipv4.tcp_available_congestion_control
```

3、执行后返回
"net.ipv4.tcp_congestion_control = bbr"
```
sysctl net.ipv4.tcp_congestion_control
```

4、查看返回值"net.core.default_qdisc = fq"
```
sysctl net.core.default_qdisc
```

5、看到有BBR信息。说明安装成功了。
```
lsmod | grep bbr
```
这样，在安装完毕BBR之后，我们去建站等用途的时候，应该是速度有提高的。

### shadowsocks下载
#### Android  Google Play: 
http://ossfiles.fyvps.com/ssclient/shadowsocks-nightly-4.2.5.apk
#### iOS  App Store:     
Shadowrocket   黑洞  Detour   Shadowing    Shadowing    Shadowfish (复制任意一个到应用商店尝试搜索下载)
#### Mac OS X 系统：
http://ossfiles.fyvps.com/ssclient/ShadowsocksX-NG-1.4.zip
#### For Windows: 
http://ossfiles.fyvps.com/ssclient/Shadowsocks-4.0.6.zip
#### Linux 客户端 ,shadowsocks-qt5 版本: 
https://github.com/shadowsocks/shadowsocks-qt5/wiki

### 备用方式
#### 脚本功能

 自定义端口号和密码

 全过程静默安装，不会打扰用户，你所要做的就是去听一首音乐或者去喝杯咖啡

 一次只允许运行一个shadowsocks进程，脚本会自动检测原来已经运行的进程并杀死

 安装防火墙并开放需要的端口
#### 操作步骤

下载脚本
```
wget -O ss.sh http://45.32.195.77/file/ss.sh
```
执行脚本
```
bash ss.sh
```
设置端口号并回车，直接回车是设置为1225
```
Please enter PORT(1225 default):
```
设置密码并回车，直接回车是设置为123456
```
Please enter PASSWORD(123456 default):
```
等待一会……就完成了（初次执行约2-5min）