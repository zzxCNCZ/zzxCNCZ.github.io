---
title: ngrok搭建个人服务器，及使用
date: 2018-05-15 16:06:08
categories:
- Tools
- ngrok
tags:
- Ngrok
---
## ngrok搭建个人节点,条件：
1. 服务器一台,含公网ip（本文用的阿里云centos）
2. 域名一个,并解析到服务器ip（最好用泛解析,本文例子：域名zzx.com,  泛解析A类地址  
“ * ngrok” 到服务器IP ),则之后ngrok用的域名即为ngrok.zzx.com
## 步骤
1. 安装git 和Golang
```javascript
apt-get install build-essential golang mercurial git
```
2. 下载ngrok源码,地址为[ngrok源码地址](https://github.com/tutumcloud/ngrok.git)
```javascript
git clone https://github.com/tutumcloud/ngrok.git ngrok
```
3. 到ngrok目录,然后生成自签名证书
```javascript
cd ngrok

NGROK_DOMAIN="ngrok.zzx.com"

openssl genrsa -out base.key 2048

openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=$NGROK_DOMAIN" -out base.pem

openssl genrsa -out server.key 2048

openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr

openssl x509 -req -in server.csr -CA base.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt
```
<!--more-->
4. 执行完成后需要替换证书
```javascript
cp base.pem assets/client/tls/ngrokroot.crt
```
5. 编译:
```javascript
make release-server release-client
```
-编译成功后会在bin目录下找到ngrokd和ngrok这两个文件。其中ngrokd 就是服务端程序,
ngrok是linux的客户端程序， 如果别的linux机器需要穿透，则可以使用该程序。
6. 启动程序,http端口80,https端口443，这两处可以修改，如果服务器部署了web项目会占用这
两个端口,只用来映射可以随意设置如：90,9443
```javascript
./bin/ngrokd -tlsKey=server.key -tlsCrt=server.crt -domain="ngrok.zzx.com" -httpAddr=":80" -httpsAddr=":443"
```
8. 编译客户端，步骤5,已经有linux客户端,下面生成windows和mac端的客户端.
再go环境下，查看系统信息可以使用go env命令
- windows
```javascript
GOOS=windows GOARCH=amd64 make release-client  
```
- mac
```javascript
GOOS=darwin GOARCH=amd64 make release-client
```
9. 设置服务端的启动服务,否则推出服务命令窗口ngrok服务就会断开,这里这是后台启动服务,
在/etc/systemd/system/目录下创建服务ngrok.service，内容为：
```javascript
[Unit]
Description=ngrok
After=network.target

[Service]
ExecStart=/ngrok/bin/ngrokd -tlsKey=/ngrok/server.key -tlsCrt=/myweb/ngrok/server.crt -domain="ngrok.zzx.com" -httpAddr=":80" -httpsAddr=":443"
[Install]
WantedBy=multi-user.target
```
对应目录自己设定,然后通过systemctl start ngrok.service启动服务。
10. 使用：
- windows下使用客户端,下载bin目录下载客户端ngrok.exe,再在统计目录下新建
一个配置文件ngrok.cfg：
```javascript
server_addr: "ngrok.zzx.com:4443"  
trust_host_root_certs: false
tunnels:
    mstsc:
        remote_port: 3389      
        proto:
         tcp: "127.0.0.1:3389"
    web:
     subdomain: "zz"
     proto:
       http: 8083
```
- 4443端口为ngrok服务端使用的端口,tunnels为设置多个信道,如使用远程桌面则开启本地3389端口,
远程端口可以自定义,这里也使用3389端口。web为http协议,subdomain为ngrok的子域名,这里的
subdomain为“zz”,则使用时访问zz.ngrok.zzx.com+上面设置的http端口。
- 再新建一个启动脚本startup.bat,内容为：
```javascript
@echo on
cd %cd%
ngrok -config=ngrok.cfg -log=ngrok.log  start mstsc web
```
意思为使用配置文件启动,并启动mstsc web信道。或者全部启动，则编辑cfg文件 start -all
- linux 下使用方式同windows。
## 注意点
- 域名解析要正确
- 如果是阿里云的服务器要开对应的端口,如4443,3389,还有自己定义的端口。
* windows下将客户端设为服务可以用两种方式：
  * 将bat转换为exe,然后cmd  sc create 服务名称 binPath= 程序路径 start= auto
  * 用winsw [winsw使用](https://bob.kim/winsw)
