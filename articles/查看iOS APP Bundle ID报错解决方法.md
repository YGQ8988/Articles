title: 查看iOS APP Bundle ID报错解决方法
author: 染洛凉
date: 2019-03-19 15:24:00
tags:
---
## 执行ideviceinstaller -l 报错：Could not connect to lockdownd. Exiting.

主要表现于使用ideviceinstaller -l命令查看iOS APP Bundle ID报错原因

## 安装libimobiledevice报错解决方案

```
brew install --HEAD libimobiledevice
```

控制台输入信息：

```

==> Cloning [https://git.libimobiledevice.org/libimobiledevice.git](https://git.libimobiledevice.org/libimobiledevice.git) Updating /Users/rjoiner/Library/Caches/Homebrew/libimobiledevice--git

==> Checking out branch master Already on 'master' Your branch is up to date with 'origin/master'. HEAD is now at b34e343 tools: Remove length check on device UDID arguments to support newer devices

==> ./autogen.sh Last 15 lines from /Users/rjoiner/Library/Logs/Homebrew/libimobiledevice/01.autogen.sh: checking dynamic linker characteristics... darwin16.7.0 dyld checking how to hardcode library paths into programs... immediate checking for pkg-config... /usr/local/opt/pkg-config/bin/pkg-config

checking pkg-config is at least version 0.9.0... yes checking for libusbmuxd >= 1.1.0... no configure: error: Package requirements (libusbmuxd >= 1.1.0) were not met:

Requested 'libusbmuxd >= 1.1.0' but version of libusbmuxd is 1.0.10
```

通过控制台输入内容可知异常所在Requested 'libusbmuxd >= 1.1.0' but version of libusbmuxd is 1.0.10，很显然是由于系统要求的*libusbmuxd *版本和所要安装的版本不一致。

使用以下方式修复，注意先断开所有iOS设备执行以下命令：

```
brew update
brew uninstall --ignore-dependencies libimobiledevice
brew uninstall --ignore-dependencies usbmuxd
brew install --HEAD usbmuxd
brew unlink usbmuxd
brew link usbmuxd
brew install --HEAD libimobiledevice
brew link --overwrite libimobiledevice
```

下面继续安装环境依赖，注意先断开所有iOS设备执行以下命令：

```
brew install --HEAD  ideviceinstaller
brew link --overwrite ideviceinstaller
sudo rm -rf /var/db/lockdown/*
```

然后连接iOS设备，然后在出现提示选择“信任”，之后，执行以下命令：

```
sudo chmod -R 777 /var/db/lockdown/
```

最后测试是否解决could not connect to lockdownd. exiting问题：

```
ideviceinstaller -l
```

![ios_id](https://github.com/YGQ8988/Articles/blob/master/screenshots/ios_id.png)

