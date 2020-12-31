---
title: 记录一次springboot 循环调用问题
date: 2020-07-09 15:55:40
categories:
- Java
- SpringBoot
tags:
- SpringBoot
---

### 记录一次springboot 循环调用问题

![image.png](https://chevereto.zhuangzexin.top/images/2020/06/23/image.png)

- 在生产环境中运行日志遇到上述问题，securityConfig 为spring security配置文件，该配置文件调用userDetailsServiceImpl 中的查询用户接口,userDetailsServiceImpl 调用了userService接口（此处应该是实现该接口，因为历史原因导致该处不规范），userService又 注入了securityConfig 的bCryptPasswordEncoder bean。因此形成了循环依赖。

#### 解决方案

- [参考自stackoverflow](https://stackoverflow.com/questions/40695893/spring-security-circular-bean-dependency)

![imageb25bb41a2e38710f.png](https://chevereto.zhuangzexin.top/images/2020/06/23/imageb25bb41a2e38710f.png)

- userDetailsServiceImpl实现userService 接口，userDetailsServiceImpl注入到securityConfig，并且加入@lazy注解（如果不加会出现因为securityConfig先启动而userDetailsServiceImpl并没有在Spring容器中的问题）


