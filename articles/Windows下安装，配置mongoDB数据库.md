---
title: Windows下安装，配置mongoDB数据库
date: 2018-08-10 17:09:32
tags:
---
# 一：安装

大家自行安装下载mongoDB安装包（介于需要翻墙访问，后面提供企鹅群号，大家可以加入该群进入群文件查找下载），当然安装时注意下安装目录，后续配置环境变量需要用到。

# 二：配置环境变量

这个就简单了。和Python，JAVA类似的方法进行配置。我这里拿自己电脑的路径做示例。大家给自己电脑配置时请根据自己实际的安装目录配置。例：D:\Program Files\MongoDB\3.4\bin
请注意，该目录需配置在系统环境变量下，请勿配置在用户变量下。
请注意，该目录需配置在系统环境变量下，请勿配置在用户变量下。
请注意，该目录需配置在系统环境变量下，请勿配置在用户变量下。
![系统环境配置截图.png](http://upload-images.jianshu.io/upload_images/5324027-0d584c06b7519ea3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三：创建及配置数据库存储路径

管理员运行cmd

进入安装目录bin目录下运行下面命令，配置数据存储路径。
```
mongod.exe --dbpath D:\data\db
```
（前提需在D盘创建data\db目录）

# 四：将mongo服务加入系统服务组件


管理员运行cmd

进入安装目录bin目录下运行下面命令，配置成服务。
```
mongod.exe --bind_ip 0.0.0.0 --logpath D:\data\logs\mongo.log --logappend --dbpath D:\data\db --port 27017 --serviceName "MongoDB" --serviceDisplayName "MongoDB" --install
```
请注意，D:\data\logs\mongo.log，D:\data\db。这些目录及文件需手动先创建。。
请注意，D:\data\logs\mongo.log，D:\data\db。这些目录及文件需手动先创建。
请注意，D:\data\logs\mongo.log，D:\data\db。这些目录及文件需手动先创建。

# 五：启动服务

打开Windows下服务。找到mongoDB服务，双击打开，点击启动运行。
![启动步骤.png](http://upload-images.jianshu.io/upload_images/5324027-cf4bddc11336bb06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
到这里mongoDB就安装配置成功了，大家可以下载可视化工具进行连接访问了。

后续会写点mongoDB存储python爬取的一些数据。
