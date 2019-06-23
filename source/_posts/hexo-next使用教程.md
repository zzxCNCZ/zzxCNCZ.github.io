---
title: hexo-next使用教程
date: 2017-11-22 16:58:47
categories:
- Hexo
tags:
- Hexo-next
---
##使用github进行多客户端编辑博客
1. 创建仓库，http://CrazyMilk.github.io；
2. 创建两个分支：master 与 hexo；
3. 设置hexo为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件）；
4. 使用git clone git@github.com:CrazyMilk/CrazyMilk.github.io.git拷贝仓库；
5. 在本地http://CrazyMilk.github.io文件夹下通过Git bash依次执行npm install hexo (npm install -g hexo)`全局安装hexo`、hexo init、npm install 和 npm install hexo-deployer-git（此时当前分支应显示为hexo）;
6. 修改_config.yml中的deploy参数，分支应为master；
7. 依次执行git add .、git commit -m "..."、git push origin hexo提交网站相关的文件；8. 执行hexo g -d生成网站并部署到GitHub上。
<!--more-->
##改动时(另一台电脑使用时)
1. 依次执行git add .、git commit -m "..."、git push origin hexo指令将改动推送到GitHub（此时当前分支应为hexo）；
2. 然后才执行hexo g -d发布网站到master分支上。
3. 另一台电脑使用时首先git clone "git地址",然后'npm install hexo'(加载hexo) （头一次使用）
4. 另一台电脑使用时首先git pull "git地址",然后就可以直接使用.
## hexo命令
* hexo init (空目录下,当要创建hexo是使用，不是第一次建立不要使用)
* hexo clean (清除缓存？)
* hexo generate(hexo g) 重新部署
* hexo server ,启动hexo(本地调试时用)
* hexo deploy 部署hexo ,主配置文件配置好master分支
* hexo new "page" 新建md文件。然后编辑

## 进阶教程——将博客部署到云主机
1. 配置SSH公钥登陆
    * 本机生成ssh key，使用过github的话，就已经生成过了，在根目录下.ssh文件夹内
id_rsa.pub是公钥id_rsa是私钥
    * 远程到主机，将公钥保存到云主机$HOME/.ssh/authorized_keys文件中（把公钥追加
到authorized_keys文件末尾），方法：将公钥文件复制到云主机.ssh目录下，然后输入命令
cat >> ~/.ssh/authorized_keys < ~/.ssh/id_rsa.pub即可
    * 然后就可以直接用命令登陆 ssh root@ip
    * [参考文章](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
2. 安装 git 和 nginx
    ````
    apt-get update
    apt-get install git-core nginx
    ````
3. 配置 Nginx
    ````
    mkdir /var/www/blog
    ````
    - /var/www/blog目录用于放置生成的静态文件
    ````
    vim /etc/nginx/conf.d/blog.conf
    ````
    - 编写 nginx 配置文件
    ````
    server
    {
        listen 80;
        root /var/www/blog;
    }
    ````
    - 重启 nginx
    ````
    systemctl restart nginx
    ````
4. 配置 Git Hooks
    - 创建 Git 裸仓库，blog.git作为远程 Git 仓库，Hexo 在本地生成的博客静态文
件可以通过 push 与其同步。
    ````
    mkdir ~/blog.git && cd ~/blog.git
    git init --bare
    ````
    - 配置 Hooks 脚本,post-receive脚本将在blog.git仓库接收到 push 时执行。
              
    ````
    vim ./hooks/post-receive
    ````
    - 脚本作用：删除原有的/var/www/blog目录，然后从blog.git仓库 
    clone 新的博客静态文件。
    ````
    #!/bin/bash
    rm -rf /var/www/blog
    git clone /root/blog.git /var/www/blog
    ````
    - 给post-receive脚本执行权限
    ````
    chmod +x ./hooks/post-receive
    ````
5. 部署 Hexo 博客             
    - 修改_config.yml
    ````
    deploy:
        type: git
        repo: root@ip:blog.git
    ````
6. 使用命令部署，hexo d即可

[Next主题网站](http://theme-next.iissnan.com/)
