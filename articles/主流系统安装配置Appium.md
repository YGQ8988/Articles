title: 主流系统安装配置Appium
author: 染洛凉
tags: []
categories: []
date: 2018-08-15 15:15:00
---
## 一、 Windows下安装与配置
### 1、Java环境安装与配置
1、[JAVA下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，下载对应环境版本完成后默认安装(安装路径最好是盘符根目录，防止带空格路径导致环境变量配置失效)。

2、[环境变量配置教程](https://blog.csdn.net/qq_14994573/article/details/79462365)，切记一定要在系统环境变量下添加JAVA_HOME环境，以及添加Path。

### 2、Android环境安装与配置(1,2安装方式二选一即可)
1、[压缩包下载地址](https://dl.google.com/android/android-sdk_r24.4.1-windows.zip?utm_source=androiddevtools&utm_medium=website)，下载完成后直接解压再配置环境变量即可。

2、[exe安装包下载地址](https://dl.google.com/android/installer_r24.4.1-windows.exe?utm_source=androiddevtools&utm_medium=website)，默认安装，环境变量配置安装路径即可。

3、[详细下载安装配置教程](https://blog.csdn.net/zeternityyt/article/details/79655150)，切记一定要在系统环境变量下添加ANDROID_HOME环境，以及添加Path。

### 3、Appium Desktop

1、[各版本客户端Appium下载地址](https://github.com/appium/appium-desktop/releases)，桌面版下载完成直接安装即可。

2、其他依赖下载：Python，Appium-Python-Client
```
pip install Appium-Python-Client
```
3、到此Windows或Mac下Appium环境基本配置完成，大家挑个心仪的IDE来测试下吧。
### 4、运行调试
1、打开Windows下cmd窗口或Mac终端，运行以下命令，不出意外就能看到以下设备名了
（前提工作，手机打开开发者模式，允许USB调试，插上电脑，手机上允许电脑进行调试)
```
adb devices
 ```
![设备名称](https://upload-images.jianshu.io/upload_images/5324027-05eade878ee0acf2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2、运行Appium，双击打开安装好的Appium Desktop(如正常打开或执行代码过程中出错，请以管理员身份运行该程序)
![Appium启动](https://upload-images.jianshu.io/upload_images/5324027-1868a5e84d5e0960.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3、打开IDE进行小牛试刀
如执行以下代码成功打开你手机微信，那么说明环境一切OK，尽情享受自动化带来的乐趣吧~~~
```
from appium import webdriver

# 作者：染洛凉 QQ交流群：368639036

desired_caps = {
                'platformName': 'Android',
                'platformVersion': '4.2.2',      # 替换为你手机的安卓版本号
                'deviceName': '9050ad0c',     # 替换为adb devices命令得到的设备名
                'appPackage': 'com.tencent.mm',
                'appActivity': '.ui.LauncherUI',
                # 'automationName': 'Uiautomator2',
                'unicodeKeyboard': True,
                'resetKeyboard': True,
                'noReset': True
                }

driver = webdriver.Remote('http://127.0.0.1:4723/wd/hub', desired_caps)
driver.implicitly_wait(10)

# 作者：染洛凉 QQ交流群：368639036
```
##  二、Mac下环境安装与配置
### 1、Java环境安装与配置
1、[JAVA下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，下载对应环境版本后默认安装。

2、[环境变量配置教程](https://www.cnblogs.com/dingzhijie/p/7016397.html)，切记如执行以下命令没看到.bash_profile文件，再执行创建。
```
cd ~/
```
```
ls -a
```
执行以上命令没找到.bash_profile文件，再执行以下命令。
```
touch .bash_profile
```

### 2、Android环境安装与配置
1、[压缩包下载地址](https://dl.google.com/android/android-sdk_r24.4.1-macosx.zip?utm_source=androiddevtools&utm_medium=website)，下载完成后直接解压再配置环境变量即可。

2、编辑.bash_profile
```
cd ~/
```
```
vi .bash_profile
```
添加以下数据
```
export ANDROID_HOME=/Users/dell/Documents/android-sdk
export PATH=${PATH}:${ANDROID_HOME}/tools
export PATH=${PATH}:${ANDROID_HOME}/platform-tools
export PATH=${PATH}:${ANDROID_HOME}/build-tools/28.0.1
```
第一行路径请替换为你本地实际路径，第二行版本号替换为你实际在ANDROID SDK Manager下下载的版本号。例：
![ANDROID SDK Manager界面](https://upload-images.jianshu.io/upload_images/5324027-d36bb26996b47f51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3、退出保存，在英文输入法下先按ESC，在按冒号，输入小写wq，回车，保存。
```
:wq
```
4、刷新配置
```
source .bash_profile
```
### 3、剩余步骤
1、到此Mac下大环境基本配置完成了，后续步骤参数Windows下：
```
3、Appium Desktop
4、运行调试
```
#### 如按照以上教程安装配置仍失败，请谷歌或进群求助。
```
QQ交流群：368639036
```