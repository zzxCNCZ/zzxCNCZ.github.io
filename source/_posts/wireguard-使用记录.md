---
title: wireguard 使用记录
date: 2022-03-17 10:07:55
tags:
- wireguard 
categories:
- Tools
- wireguard
---

# wireguard使用记录

[官网](https://www.wireguard.com/)

1. 安装
```bash
sudo apt install wireguard
# 安装resolvconf  （服务端安装）
sudo apt install resolvconf
```
2. 生成本机公钥和私钥（客户端和服务段都需要生城）
```bash
# 私钥
umask 077
wg genkey > privatekey
# 公钥
wg pubkey < privatekey > publickey
```
<!--more-->
3. 配置服务端
```bash
vim wg0.conf

# 如下
[Interface]
Address = 10.0.10.1
# 监听端口 udp
ListenPort = 12345
# 服务端私钥
PrivateKey = MOqoNfyl+0i54vFUHEp7Gdv9/Zg6wcU+TA468HvBf0U=
DNS = 8.8.8.8
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[peer]
# 客户端ip
AllowedIPs = 10.0.10.2
# 客户端公钥
PublicKey = QReB/eMdHYnN9qbOE4mFAuxpHRFx39A6G2QAedgBWHM=

```
4. 配置客户端
```bash
[Interface]
PrivateKey = 0OlFV+eu7iWSOIDj/ECbXk5jfLdUtmVzJFQ7+e7DCl8=
Address = 10.0.10.2/24
DNS = 8.8.8.8

[Peer]
PublicKey = 5OCxKLY+Y+au8I4BXJECQmt04JIvpziI44LufLNOZUA=
AllowedIPs = 10.0.10.2/24
Endpoint = domian.com:12345
#当会话存在一端 IP 地址为 NAT 地址或虚假公网 IP 地址时，由该方阶段性每 15 秒发送 keepalive 报文保持会话的可用性，防止被设备终止。
PersistentKeepalive = 15

```
4. 启动服务端(使用wg-quick启动)
```bash
sudo wg-quick up /pathto/wg0.conf

```


### 其他指令
停止
```bash
 wg-quick down /full/path/to/wg0.conf

```

启动与重启
```bash
设置为自动启动 wg0：systemctl enable wg-quick@wg0
禁用服务：systemctl disable wg-quick@wg0
启动服务：systemctl start wg-quick@wg0
重启服务：systemctl restart wg-quick@wg0
查看服务状态：systemctl status wg-quick@wg0
```

静态添加节点peer
```bash
# 静态的方式(wg0.conf) 添加节点后需要重启服务，动态添加不需要重启
wg addconf wg0 <(wg-quick strip ./wg0.conf)

```

动态方式 （推荐）
```bash
wg set wg0 peer $(cat cpublickey1) allowed-ips xxx.xxx.xxx.xxx/32 persistent-keepalive 15
#如果显示正常，那么我们就保存到配置文件
wg-quick save wg0
```

停止与启动
```bash
# 启动/停止 VPN 网络接口
$ ip link set wg0 up
$ ip link set wg0 down

# 注册/注销 VPN 网络接口
$ ip link add dev wg0 type wireguard
$ ip link delete dev wg0

# 注册/注销 本地 VPN 地址
$ ip address add dev wg0 192.0.2.3/32
$ ip address delete dev wg0 192.0.2.3/32

# 添加/删除 VPN 路由
$ ip route add 192.0.2.3/32 dev wg0
$ ip route delete 192.0.2.3/32 dev wg0

```
查看信息
```bash

# 查看系统 VPN 接口信息
$ ip link show wg0

# 查看 VPN 接口详细信息
$ wg show all
$ wg show wg
```

### 问题解决
`ip link add wg0 type wireguard Error: Unknown device type` 问题

系统内核版本过低导致的，WireGuard 的安装和使用条件非常苛刻，对内核版本要求极高，不仅如此，在不同的系统中，内核，内核源码包，内核头文件必须存在且这三者版本要一致。所以一般不建议在生成环境中安装，除非你对自己的操作很有把握。Red Hat、CentOS、Fedora 等系统的内核，内核源码包，内核头文件包名分别为 kernel、kernel-devel、kernel-headers，Debian、Ubuntu 等系统的内核，内核源码包，内核头文件包名分别为 kernel、linux-headers。

果这三者任一条件不满足的话，则不管是从代码编译安装还是从 repository 直接安装，也只是安装了 wireguard-tools 而已。而 WireGuard 真正工作的部分，是 wireguard-dkms，也就是动态内核模块支持(DKMS)，是它将 WireGuard 编译到系统内核中。因此，在某些 VPS 商家，是需要你先自主更换系统内核，并事先将这三者安装好，才有可能不会出现编译或安装失败。

当然，目前 WireGuard 已经被合并到 Linux 5.6 内核中了，如果你的内核版本 >= 5.6，就可以用上原生的 WireGuard 了，只需要安装 wireguard-tools 即可。例如，对于 Ubuntu 20.04 来说，它的内核版本是 5.4，虽然小于 5.6，但经过我的测试发现它已经将 WireGuard 合并到了内核中，我们只需要安装 wireguard-tools 即可


[wireguard教程](https://fuckcloudnative.io/posts/wireguard-docs-practice/)

[WireGuard搭建与使用](https://tengwait.com/2020/07/12/WireGuard%E6%90%AD%E5%BB%BA%E4%B8%8E%E4%BD%BF%E7%94%A8/)

[秋水逸冰一键安装脚本](https://teddysun.com/554.html)

[秋水逸冰升级内核脚本](https://teddysun.com/489.html)

[五分钟内装好 WireGuard及Centos 内核升级](https://cloud.tencent.com/developer/article/1752845)

[elrepo 内核官网](http://elrepo.org/tiki/HomePage)

[How To Install Linux Kernel 5.15 on CentOS 8](https://computingforgeeks.com/how-to-install-latest-kernel-on-centos-linux/)
