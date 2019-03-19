title: 服务器使用Nginx搭建HTTP爬虫代理
author: 染洛凉
date: 2018-11-01 10:24:00
tags:
---
#### Linux系统（CentOS为例）
应用场景： 
		编写爬虫时经常需要用到代理，然而免费的代理又很不稳定，最理想的就是在自己服务器搭建nginx正向代理服务。

1、 在官网下载最新稳定版本的 [Nginx安装包](http://nginx.org/download/nginx-1.10.3.tar.gz)

2、 默认配置编译安装 ( 需要先安装nginx所需的依赖库 )

① 依赖库安装

```
rpm -qa | grep zlib //查看是否安装了zlib，其他类推.
sudo yum install openssl-devel  //安装openssl
sudo yum install pcre-devel  //安装pcre
sudo yum install zlib-devel  //安装zlib
```

②nginx编译

Nginx编译参数详解(自行百度，不懂不影响后续操作)。

③ nginx安装
```
tar zxvf nginx-1.10.3.tar.gz 
cd nginx-1.10.3 
./configure 
make 
make install
```

3、查看服务器可用DNS，下面配置nginx需要用到
   远程连上自己服务器，执行以下命令
```
cat /etc/resolv.conf
```
如存在多个，任选一个就行了。

4、 修改nginx运行配置文件 [ nginx 默认安装在/usr/local/nginx/下 ]
```
vim /usr/local/nginx/conf/nginx.conf
```
如果没有Vim，自行安装
```
sudo yum install vim
```
```
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    #代理服务设置
    server {
        resolver 192.168.99.100;  #可用的DNS
        listen 81;   #监听的端口
        location / { 
            proxy_pass http://$http_host$request_uri;
        }   
    }
} 
```
编辑完成后保存退出，重启Nginx。

###### nginx启动：
```
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

###### nginx停止：
```
/usr/local/nginx/sbin/nginx -s stop //优雅的停止nginx 
/usr/local/nginx/sbin/nginx -s quit //立即停止nginx
```

###### nginx重新加载配置：
```
/usr/local/nginx/sbin/nginx -s reload //平滑的重启nginx并重新加载nginx的配置文件; 
/usr/local/nginx/sbin/nginx -s reopen //重新打开日志文件 
/usr/local/nginx/sbin/nginx -t //可以用来修改配置文件之后，测试配置文件是否有语法错误
```
5、Python代码测试
```
import requests

url = 'http://icanhazip.com/'
ip = "124.235.181.175"	#自行替换成自己服务器公网IP，非DNS地址
port = 80				#自行替换成nginx设置的端口，上文设置为：81
proxy = {"http":"http://{}:{}".format(ip,port),"https:":"https://{}:{}".format(ip,port)}
r = requests.get(url,proxies=proxy)
print(r.text)
```
如下图，打印出的IP地址如果为你服务器公网IP，那么恭喜你，搭建成功了。

终于可以愉快的spider了。

![测试结果](/images/pasted-0.png)