title: linux上用jmeter压测时出现很多异常java.net.NoRouteToHostException_Cannot assign requested address
author: 染洛凉
date: 2019-03-05 13:59:49
tags:
---
经网上查资料， 是由于linux分配的客户端连接端口用尽， 无法建立socket连接所致，虽然socket正常关闭，但是端口不是立即释放， 而是处于TIME_WAIT状态， 默认等待60s后才释放。

查看linux支持的客户端连接端口范围, 也就是28232个端口： 
32768 - 61000
```
cat /proc/sys/net/ipv4/ip_local_port_range
```

解决方法：
1\. 调低端口释放后的等待时间， 默认为60s， 修改为15~30s
```
echo 20 > /proc/sys/net/ipv4/tcp_fin_timeout
```


2\. 修改tcp/ip协议配置， 通过配置/proc/sys/net/ipv4/tcp_tw_resue, 默认为0， 修改为1， 释放TIME_WAIT端口给新连接使用。
```
echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse
```


3\. 修改tcp/ip协议配置，快速回收socket资源， 默认为0， 修改为1.
```
echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle
```



通过上面3项调整， 后续压力测试不会出现以上错误。
