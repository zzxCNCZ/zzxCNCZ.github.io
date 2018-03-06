---
title: 关于JavaBean-POJO
date: 2018-03-20 09:16:42
categories:
- Java
tags:
- POJO
---
## POJO实体类属性-private
- 为了安全
- 为什么安全？别的类可以通过直接调用对象.属性名调用或者任意赋值。
- 用get，set可以控制某些属性只读或只写
- POJO本身就是对外开放的接口,通过方法调用