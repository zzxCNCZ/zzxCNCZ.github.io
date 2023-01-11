---
title: JetBrains Client 远程连接时卡住卡住 trying to connect to remote host
date: 2023-01-11 16:28:32
tags:
categories:
- Tools
---

### 使用JetBrains idea远程开发时遇到的连接问题
![](https://chevereto.zhuangzexin.top/images/2023/01/11/20230111163028.png)

配置ssh远程连接时一直卡在连接的地方，测试本机是可以ssh连接的

#### 解决过程
1. 排查idea日志 Help-> Collect logs and Diagnostic Data 打开日志排查出连接报错日志：
```
woke to: Opening `direct-tcpip` channel failed: open failed
```
2. google得出可能是 remote host sshd客户端 localhost port forwarding配置问题
3. 修改 remote host `/etc/ssh/sshd_config`
```
AllowTcpForwarding yes 
GatewayPorts yes 
```
4. 重启sshd客户端
```
service sshd restart
```

#### 参考
[gateway fails to load and stuck in "trying to connect to remote host"](https://youtrack.jetbrains.com/issue/GTW-1203/gateway-fails-to-load-and-stuck-in-trying-to-connect-to-remote-host)
