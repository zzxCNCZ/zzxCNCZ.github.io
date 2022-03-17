---
title: n2n 使用记录
date: 2022-03-17 10:51:48
tags:
- n2n
- vpn 
categories:
- Tools
- n2n
---

# n2n
> p2p network

### n2n config
- supernode配置
```bash
# 端口
-p 41829
# 允许接入的cummunity
-c /home/zzx/n2n/community.list
# 开启日志模式
-v
```
- edge 配置
```
# community name
-c=community
# 密码(用于edge之间访问的的加密)
-k=123456
# 子网ip
-a=10.10.0.4
# supernode 地址
-l=domain.com:12345
# 开启日志模式
-v

```
<!--more-->

### n2n service
- supernode
```
[Unit]
Description=n2n supernode daemon
After=network.target

[Service]
Type=forking
User=root
ExecStart=/home/zzx/n2n/supernode /home/zzx/n2n/supernode.conf
Restart= on-failure
Restart= always
RestartSec=1min
ExecStop=/usr/bin/kill -9 $(pidof supernode)


[Install]
WantedBy=multi-user.target


```
- edge

**linux安装**

`vim /etc/systemd/system/edge.service`
```shell
[Unit]
Description=n2n edge daemon
After=network.target

[Service]
Type=forking
User=root
ExecStart=/home/xilin/n2n/edge /home/xilin/n2n/edge.conf
Restart= on-failure
Restart= always
RestartSec=1min
ExecStop=/usr/bin/kill -9 $(pidof edge)


[Install]
WantedBy=multi-user.target
```
`systemctl enable edge` 设置开机启动

**windows安装**

### 点对网，网对网组网
e.g. :
点端windows(edge ip: 10.10.0.2)
网端linux(edge ip: 10.0.0.100),私网网段172.16.12.0
需要实现点端 与网端私网联通

1. 网端需要一台机器开启路由转发功能 WIN/LINUX都需要
```bash
# 允许系统转发 ipv4
echo 1 > /proc/sys/net/ipv4/ip_forward
# iptables 添加转发规则， edge0为 edge client 虚拟网卡，ens18为172.16.12.0 网卡
iptables -A FORWARD -i edge0 -o ens18 -j ACCEPT
iptables -A FORWARD -i ens18 -o edge0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -t nat -A POSTROUTING -o ens18 -s 10.10.0.0/24 -j MASQUERADE

```
2. 网端在正常启动的基础上添加参数-r
```bash
edge -d edge0 -a 10.10.0.100 -c cummunity -k 123456 -l ${serverip}:${serverport} -r
```
3. 在点端添加路由表
```bash
edge -d edge0 -a 10.10.0.100 -c cummunity -k 123456 -l ${serverip}:${serverport} -n 172.16.12.0/24:10.10.0.100

```

### 其他指令
windows路由
```bash
# 添加
route add 192.168.123.0 mask 255.255.255.0 10.0.0.10 -p

# 查看
route print -4

# 删除
route delete 192.168.123.0

```
linux 路由
```bash
# 查看
route -n

```

### 其他特性
[客户端认证](https://github.com/ntop/n2n/blob/dev/doc/Authentication.md)
