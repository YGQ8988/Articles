---
title: 配置远程访问Jupyter
date: 2018-08-10 17:11:45
tags:
---
## 1.访问清华大学开源软件镜像站(https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/),找到对用版本的Anaconda，复制现在链接，在命令行中输入
```
wget  https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.1.0-Linux-x86_64.sh
```
回车(记得替换成自己系统对应版本的链接）。
## 2.下载完成之后，可以看到有一个Anaconda3-5.1.0-Linux-x86_64.sh的文件，我们用
```
sh Anaconda3-5.1.0-Linux-x86_64.sh
```
安装它，在之后安装过程中一路yes就可以了，没有出现报错，就代表安装成功了。如果报错之简单粗暴的方法就是直接重装30秒后满血复活，或者
```
rm -rf ~/anaconda3
```
，然后重新执行文件。
## 3.安装之后，在腾讯云中设置安全组，并且将当前的实例添加到安全组当中。
## 4.配置好安全组之后，要配置一下环境变量再运行一次Jupyter生成配置文件。(命令行输入：先输入
```
export PATH=/root/anaconda3/bin:$PATH
```
回车，在运行
```
jupyter notebook --generate-config
```
)
## 5.命令来设置密码：
```
jupyter notebook password
```
，生成的密码存储在 jupyter_notebook_config.json。
再命令行执行：
```
vi /root/.jupyter/jupyter_notebook_config.json
```
sha1:67c9e60bb8b6:9ffede0825894254b2e042ea597d771089e11aed 这一串就是要在 jupyter_notebook_config.py 添加的密码。
## 6.进入.jupyter文件夹(cd .jupyter)，编辑jupyter_notebook_config.py文件(
```
vim jupyter_notebook_config.py
```
)，按i进入编辑模式，在文件最后输入(
```
c.NotebookApp.ip='*'
c.NotebookApp.password = u'sha:ce...刚才复制的那个密文'
c.NotebookApp.open_browser = False
c.NotebookApp.port =8888 #可自行指定一个端口, 访问时使用该端口
```
之后按ESC，输入：wq，保存文件并退出。
7.以上设置完以后就可以在服务器上启动 jupyter notebook
，
```
jupyter notebook
```
, root 用户使用 
```
jupyter notebook --allow-root
```
。打开 IP:指定的端口, 输入密码就可以访问了。
