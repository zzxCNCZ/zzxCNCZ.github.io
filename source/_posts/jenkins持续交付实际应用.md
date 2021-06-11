---
title: jenkins持续交付实际应用
date: 2021-06-11 17:20:47
categories:
- Jenkins
tags:
- Jenkins
---

# jenkins持续交付实际应用
> 应用发布指定版本号，指定版本发布，回滚

因为团队规模小的原因，代码发布等没有严格的版本规划，使用Jenkins都是用持续集成的那一套，gitlab上的代码分支也只有master一个分支，测试跟正式环境使用的都是master分支，这样不严格的方式有很多缺点，代码管理不严谨，版本规划混乱。在代码部署上还有一个致命缺点，不是很好回滚，线上出了问题话总不能根据commit来回滚吧，而且很多commit信息写的也不是很详细，都不确定要回滚到哪一个版本，所以这种方式不是很可取，特别是当多人开发的时候，问题就越来越明显。
目前使用的解决方案是新加入一个release分支(gitlab上对该分支添加保护模式，只限管理员能push)，发布版本时对此分支添加tag,例如v1.0、v1.1的版本号，发布版本时可以根据tag版本号，发布指定版本,或者回滚版本。
步骤如下:

![image27ecfe870fe31e16.png](https://chevereto.zhuangzexin.top/images/2021/06/11/image27ecfe870fe31e16.png)
<!--more-->
1. 添加release分支
```bash
git branch release && git checkout release
# 提交到远程仓库
git push origin release
```
2. 添加tag
```bash
git tag v1.0
```
3. 将v1.0推送到远程仓库
```bash
git push origin v1.0
```
![image.png](https://chevereto.zhuangzexin.top/images/2021/06/11/image.png)

4. jenkins安装Git Paramter插件,并添加构建步骤
   ![imagef5920eabc2d22729.png](https://chevereto.zhuangzexin.top/images/2021/06/11/imagef5920eabc2d22729.png)
   name为tag，在构建过程中可以当参数使用，在构建docker镜像时，可以为镜像添加tag，使用方式为${tag}
   Parameter Type设置为tag
   Default Value 设置任意
5. 设置git拉取指定tag的代码
   ![image91dc995afb968c85.png](https://chevereto.zhuangzexin.top/images/2021/06/11/image91dc995afb968c85.png)
6. 之后就可以进行编译，打包等操作，并使用docker构建镜像
   ![imageede21c0002cf188d.png](https://chevereto.zhuangzexin.top/images/2021/06/11/imageede21c0002cf188d.png)
7. 此时jenkins build选项已经消失，变成了build width parameters,同时可以选择tag版本进行build
   ![image795bed4d16fca500.png](https://chevereto.zhuangzexin.top/images/2021/06/11/image795bed4d16fca500.png)

- 参考
  [Jenkins 中以构建 Tag 来实现版本管理](https://cloud.tencent.com/developer/article/1626574)
  [不断进化的分支和需求管理](https://cloud.tencent.com/developer/article/1467694)
