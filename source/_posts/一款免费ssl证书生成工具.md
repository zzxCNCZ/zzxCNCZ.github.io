---
title: 一款免费ssl证书生成工具
date: 2021-06-07 09:33:37
tags:
- acme
categories:
- Tools
- acme
---

## 安装
[仓库地址](https://github.com/acmesh-official/acme.sh)
- 一键安装脚本
```bash
curl  https://get.acme.sh | sh

```
- 设置alias
```bash
alias acme.sh=~/.acme.sh/acme.sh
```

## 生成证书方式
- 推荐dns方式(可以一次生成多个证书)
```bash
export DP_Id="yourid"
export DP_Key="cc23352a5b4b89729f5141cfe80babdb"

acme.sh   --issue   --dns dns_dp   -d v2.940303.xyz

```
## 部署至nginx
```bash

acme.sh --install-cert -d v2.940303.xyz \
--key-file       /etc/nginx/cert/v2.940303.xyz.key  \
--fullchain-file /etc/nginx/cert/fullchain.cer \
--reloadcmd     "service nginx force-reload"

```

- Standalone tls alpn mode
```bash
# 443端口需要暂时关闭
./acme.sh  --issue   -d zhuangzexin.top  --alpn --tlsport 443

```
- Webroot mode
- Standalone mode
