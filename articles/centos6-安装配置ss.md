title: centos6 安装配置ss
author: 染洛凉
date: 2018-08-14 10:13:05
tags:
---
下述操作在2018.08.14 自测有效

操作环境：CentOS 6

服务器地址：东京

 

1.准备VPS（亚马逊，谷歌云都可以免费申请一年的）

2.安装sshd，设置sshd开机启动，允许root远程登录（可忽略，一般服务商会帮你预配置好）

3.控制台设置防火墙，入站开启所有端口

4.使用root用户远程登录主机，推荐使用xshell,SecureCRT

5.进行系统更新
```
yum -y update
```
6.安装wget下载器
```
yum -y install wget
```
7.下载ss安装脚本
```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
```
8.使上述脚本属性可执行
```
chmod +x shadowsocks.sh
```
9.执行脚本
```
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```
10.控制台会依次询问ss访问密码，访问端口，加密方式（选择端口后不需要你另外开放防火墙规则）

11.请尽情使用吧！

 

 

注:

服务启停命令：

启动
```
/etc/init.d/shadowsocks start
```
停止
```
/etc/init.d/shadowsocks stop
```
重启
```
/etc/init.d/shadowsocks restart
```
状态查询
```
/etc/init.d/shadowsocks status
```
编辑配置文件：
```
nano /etc/shadowsocks.json
```
（如果没有装nano编辑器请先下载 yum -y install nano）