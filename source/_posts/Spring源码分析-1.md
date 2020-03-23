---
title: Spring源码分析(1)
date: 2019-08-04 22:45:40
categories:
- Java
- Spring
tags:
- 源码分析
---
#### 分析spring 中IOC及AOP的理解

IOC控制反转( Inversion of Control )

1. 为何要用IOC控制反转？IOC是什么

   [![image0548234a76d3d557.png](http://blog.zhuangzexin.top:8082/images/2019/08/04/image0548234a76d3d557.png)](http://blog.zhuangzexin.top:8082/image/EV4) （1）

   [^1]: 上图基本上解释了IOC原理，IOC即通过依赖倒置原则（依赖注入），将bean交给spring管理，从而避免了每次使用bean时都需要new 一个对象的痛点。

   [依赖倒置原则]: https://www.zhihu.com/question/23277575/answer/169698662?hb_wx_block=0&amp;utm_source=wechat_session&amp;utm_medium=social&amp;utm_oi=551840056621449216	"依赖倒置原则"

   
<!--more-->
2. AOP 面向切面编程

   合理使用切面可以大幅度解决交叉模块的复用，使交叉业务模块化，比如日志记录，接口文档，事务。

   [![imagedec2c40ef7138621.png](http://blog.zhuangzexin.top:8082/images/2019/08/04/imagedec2c40ef7138621.png)](http://blog.zhuangzexin.top:8082/image/dKi)

   


