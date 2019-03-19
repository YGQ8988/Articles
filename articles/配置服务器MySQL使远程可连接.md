---
title: 配置服务器MySQL使远程可连接.
date: 2018-08-10 17:07:16
tags:
---
# 安装
##  首先检查系统中是否已经安装了MySQL
在终端里面输入 
```
sudo netstat -tap | grep mysql
```
若没有反映，没有显示已安装结果，则没有安装。若如下显示，则表示已经安装.
![img.png](http://img.blog.csdn.net/20140117232502859?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl6dXFpbmdibG9n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
## 如果没有安装，则安装MySQL
在终端输入 
```
sudo apt-get install mysql-server mysql-client
```
![img.png](http://img.blog.csdn.net/20140117232651296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl6dXFpbmdibG9n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
在此安装过程中会让你输入root用户(管理MySQL数据库用户，非Linux系统用户)密码，按照要求输入即可。如下所示：
![img.png](http://img.blog.csdn.net/20140117232854781?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl6dXFpbmdibG9n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
## 测试安装是否成功：
在终端输入  
```
sudo netstat -tap | grep mysql
```
出现如下结果则安装成功：
![img.png](http://img.blog.csdn.net/20140117232502859?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl6dXFpbmdibG9n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
## 也可通过登录MySQL测试
在终端输入 
```
mysql -uroot -p
```
接下来会提示你输入密码，输入正确密码，即可进入。如下所示：
![img.png](http://img.blog.csdn.net/20140117233308984?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl6dXFpbmdibG9n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
## MySQL的一些简单管理：
启动MySQL服务：                       
```
sudo start mysql
```

       停止MySQL服务：                       
```
sudo stop mysql
```

       修改 MySQL 的管理员密码：     
```
sudo mysqladmin -u root password newpassword
```
# 配置成可远程登录
## 修改配置文件
修改 /etc/mysql/my.cnf
```
bind-address            = 127.0.0.1
```
修改为
```
bind-address            = 0.0.0.0
```
设置允许所有IP可访问
## 设置登录权限
先进入数据库。
在MySQL终端输入;
```
GRANT ALL PRIVILEGES ON *.* TO 'user'@'%' IDENTIFIED BY 'passwd' WITH GRANT OPTION;
```
注意：上面user为你允许远程连接的用户（一般默认为root）
passwd 为你允许远程用户通过user账户及该密码进行登录。（请自行修改并记住）

## 如遇到问题，请点击页面底部，加入QQ群求助！
