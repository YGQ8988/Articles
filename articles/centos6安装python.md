---
title: CentOS 6 安装 Python3.5，pip3以及配置
---

## 安装python3.5
安装步骤如下 ：
1 准备编译环境(环境如果不对的话，可能遇到各种问题，比如wget无法下载https链接的文件)
```
yum groupinstall 'Development Tools'
yum install zlib-devel bzip2-devel  openssl-devel ncurses-devel
```
2 下载 Python3.5代码包
```
wget  https://www.python.org/ftp/python/3.5.4/Python-3.5.4.tar.xz
```
3 编译
```
tar Jxvf  Python-3.5.4.tar.xz
cd Python-3.5.4
./configure --prefix=/usr/local/python3
make && make install
```
4创建软连
```
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
```
5测试
```
python3
```
如果正常进入Python交互命令行就说明配置成功了，这样就达到Python2和Python3共存
## 安装Python3 Pip
1先下载get-pip.py
```
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```
2建立pip3软连
```
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```
3测试
```
pip3
```
如果屏幕输入pip相关信息就说明安装成功了。
### Python3和pip3到这里就安装完成了。
### 为了更好的使用这些环境，建议大家安装虚拟环境，在虚拟环境下运行相应程序。
### 虚拟环境安装请自行百度！
