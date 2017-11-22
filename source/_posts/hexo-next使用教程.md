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
5. 在本地http://CrazyMilk.github.io文件夹下通过Git bash依次执行npm install hexo、hexo init、npm install 和 npm install hexo-deployer-git（此时当前分支应显示为hexo）;
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

[Next主题网站](http://theme-next.iissnan.com/)






