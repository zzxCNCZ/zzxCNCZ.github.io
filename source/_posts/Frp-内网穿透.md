---
title: Frp 内网穿透
date: 2019-04-02 20:41:48
tags:
- frp
categories:
- 内网穿透
- frp
---
## frp内网穿透基本使用
### 在没用到ipv6之前，因为公网ip资源限制，为了能像使用公网ip直接访问内网机器的服务，而产生了很多开源的内网穿透工具。
### 内网穿透工具使用场景为比较复杂的网络环境，例如被NAT的家庭宽带，若没有固定ip，或者没有动态固定ip，就很难远程家里的电脑，或者远程管理家庭路由器等等。
### frp内网穿透使用起来比ngrok方便，功能相对来说也更全一些，最新版本的ngrok已经闭源，很遗憾。因此在服务器上换了一种内网穿透方式，原理相同，使用过程有些区别。个人看下来，认为frp配置更人性化。
#### 准备
- 一台有公网IP的服务器（本例子使用aliyun的服务器，系统 Ubuntu 16.0，配置 1c1t 1g ram 25g rom，30 Mbps 带宽）
#### 服务器配置
- 下载服务器对应版本的frp
```html
// 下载
wget https://github.com/fatedier/frp/releases/download/v0.25.3/frp_0.25.3_linux_amd64.tar.gz
// 解压
tar zxvf frp_0.25.3_linux_amd64.tar.gz
// 移动到frp 文件夹
mv frp_0.25.3_linux_amd64 frp
// 进入frp文件夹
cd frp
```
- frps为服务端 frps.ini为简略配置文件 frps_full.ini 为全配置文件，可以按需求修改
```html
// frp对外暴露的端口
bind_port = 7000
// frp给内网服务器代理web网页时使用的http端口（可自行修改）
vhost_http_port = 8888
// frp给内网服务器代理web网页时使用的https端口（可自行修改）
vhost_https_port = 4443
// web管理页面配置
dashboard_addr = 0.0.0.0
dashboard_port = 7500
// web管理页面账号密码配置
dashboard_user = admin
dashboard_pwd = admin 
// 客户端与服务器端认证密码
token = admin
// 客户端使用的远程端口，端口对应客户端本地端口。如2222 对应本地22 ssh端口
allow_ports = 2222,3307,11521
// web域名代理，将下面域名泛解析，例如*.frp.zzx.com 解析到本服务器，内网web使用subdomain 来区分并解析带对应服务
subdomain_host = frp.zzx.com
```
- 开启服务端
```html
nohup ./frps -c ./frps_full.ini &
```
#### 客户端配置
- 解压的文件夹中 frpc 即为frp客户端， frpc.ini 为简略配置 frpc_full.ini 为全配置
```html
// 服务地址
server_addr = *.*.*.*
server_port = 7000
// 认证密码
token = admin
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 2222
[mysql]
type = tcp
local_ip = 127.0.0.1
local_port = 3306
remote_port = 3307
[oracle]
type = tcp
local_ip = 192.168.192.128
local_port = 1521
remote_port = 11521
[web]
type = http
local_port = 8080 
// web服务的域名，此例子的外网方式即为 test.frp.zzx.com:8888
subdomain = test
```
- 客户端启动
```html
nohup ./frpc -c ./frpc.ini &
```
#### 以上

