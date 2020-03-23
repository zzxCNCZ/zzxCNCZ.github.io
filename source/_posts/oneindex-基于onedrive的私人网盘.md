---
title: oneindex-基于onedrive的私人网盘
date: 2019-08-16 09:05:22
categories:
- The Cloud
tags:
- hobby
---
## oneindex安装及使用
#### 使用lnmp搭建使用环境
> 简介
- OneIndex是一个类似与PHP目录的程序，其主要功能是将OneDrive的文件目录给列出来，仅仅需要将程序部署在服务器上，不占用太大的空间，索引中的文件并不占用服务器空间，仅仅占用OneDrive容量，流量也不用走服务器流量。支持部分音视频/图片格式在线浏览和下载，本质是一个在线下载网站。
> 环境要求
1. PHP空间，PHP 5.6+ 需打开curl支持 (推荐用5.6版本)
2. OneDrive 账号 (个人、企业版或教育版/工作或学校帐户)
3. OneIndex 程序
> lnmp环境安装
- 安装步骤
```
yum -y install wget screen #for CentOS/Redhat
# apt-get  install  screen #for Debian/Ubuntu
wget http://mirrors.linuxeye.com/lnmp-full.tar.gz
tar xzf lnmp-full.tar.gz
#tar xzf lnmp.tar.gz
cd lnmp # 如果需要修改目录(安装、数据存储、Nginx日志)，请修改options.conf文件
screen -S lnmp # 如果网路出现中断，可以执行命令`screen -R lnmp`重新连接安装窗口
./install.sh
```
<!--more-->
![install_oneinstack.png](http://blog.zhuangzexin.top:8082/images/2019/08/16/install_oneinstack.png)- 添加虚拟主机，虚拟主机需要使用域名访问，本地机器可以通过frp映射，域名如果没有https则在建虚拟主机时，选择use http only
```
./vhost.sh
```
![vhost.png](http://blog.zhuangzexin.top:8082/images/2019/08/16/vhost.png)- 虚拟主机命令
```
./vhost.sh --del  删除
./vhost.sh --list  列表
```
> 安装oneindex
```
#进入域名命名的根目录,如虚拟机的目录为/data/wwwroot/example.com/，然后：
wget https://github.com/donwa/oneindex/releases/download/3.1/oneindex.zip
unzip
mv
#给config和cache两个目录赋予权限
chmod -R 777 config cache

```
- 打开域名即可查看oneindex安装页面
- 安装过程省略，主要为client_id,和client_secret, 域名中转如果失效，去下载最新oneindex安装包
- oneindex登录页为 http(s)://域名/?/login 默认密码oneindex，如果要去掉url中的？号需要配置伪静态（虚拟机nginx配置目录）
```
location / {
    if (!-f $request_filename){ 
        rewrite (.*) /index.php; 
    } 
} 
```
> 设置oneindex
- 安装完，进入设置页面后，会在虚拟机目录下config文件夹生成base.php文件，需要给该文件设置权限
```
chmod -R 777 base.php
```
- 设置缓存刷新时间，修改base.php
```
'cache_expire_time' => 300, //缓存过期时间 /秒
'cache_refresh_time' => 60, //缓存刷新时间 /秒
```
- 设置缓存类型
```
'cache_type'=> 'filecache'  

```
- crontab定时刷新缓存 能极大提高系统访问性能,添加以下命令到crontab
```
crontab -e

*/10 * * * * php /data/wwwroot/example.com/one.php cache:refresh

```
> oneindex 使用
- 可以在设置页面添加分享文件夹，如/share，
- 可以添加images文件夹，在oneindex页面会默认在该文件夹下添加上传界面，即可使用oneindex的图床功能
## 以上


