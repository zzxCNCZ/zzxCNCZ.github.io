---
title: Spring注解
date: 2018-03-05 15:17:13
categories:
- Java
- Spring
tags:
---
## Spring注解
1. @Autowired
* @Autowired 注释，它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。 通过 @Autowired的使用来消除 set ，get方法。在使用@Autowired之前，我们对一个bean配置起属性时，是这用用的
```javascript
<property name="属性名" value=" 属性值"/>   
```
* 具体例子：
- UserRepository.java
```javascript
1 package com.proc.bean.repository;
2
3 public interface UserRepository {
4     
5     void save();
6 }
```
- 这里定义了一个UserRepository接口，其中定义了一个save方法
- UserRepositoryImps.java
```javascript
package com.proc.bean.repository;

import org.springframework.stereotype.Repository;

@Repository("userRepository")
public class UserRepositoryImps implements UserRepository{

    @Override
    public void save() {
        System.out.println("UserRepositoryImps save");
    }
}
```
<!--more-->
- 定义一个UserRepository接口的实现类，并实现save方法，在这里指定了该bean在IoC中标识符名称为userRepository
- UserService.java
```javascript
package com.proc.bean.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.proc.bean.repository.UserRepository;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public void save(){
        userRepository.save();
    }
}
```
- 这里需要一个UserRepository类型的属性，通过@Autowired自动装配方式，从IoC容器中去查找到，并返回给该属性
- applicationContext.xml配置
```javascript
<context:component-scan base-package="com.proc.bean" />
```
-  测试代码：
```javascript
1 ApplicationContext ctx=new ClassPathXmlApplicationContext("applicationContext.xml");
2
3 UserService userService=(UserService) ctx.getBean("userService");
4 userService.save();
```
- 输出结果：UserRepositoryImps save
- @Autowired的原理
- 其实在启动spring IoC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性
```javascript
<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>  
```

-  注意事项：
-　 在使用@Autowired时，首先在容器中查询对应类型的bean
-　　　如果查询结果刚好为一个，就将该bean装配给@Autowired指定的数据
-　　　如果查询的结果不止一个，那么@Autowired会根据名称来查找。
-　　　如果查询的结果为空，那么会抛出异常。解决方法时，使用required=false
- 举例说明：
- 在上面例子中，我们在定一个类来实现UserRepository接口
```javascript
package com.proc.bean.repository;

import org.springframework.stereotype.Repository;

@Repository
public class UserJdbcImps implements UserRepository {

    @Override
    public void save() {
        System.out.println("UserJdbcImps save");
    }
}
```
-  这时在启动容器后，在容器中有两个UserRepository类型的实例，一个名称为userRepository，另一个为userJdbcImps。在UserService中

```javascript
@Autowired
private UserRepository userRepository;
```
- 输出结果：UserRepositoryImps save
- 这里由于查询到有两个该类型的实例，那么采用名称匹配方式，在容器中查找名称为userRepository的实例，并自动装配给该参数。
- 如果这里想要装载userJdbcImps的实例，除了将字段userRepository名称改成userJdbcImps外，可以提供了一个@Qualifier标记，来指定需要装配bean的名称，代码这样写:
```javascript
1 @Autowired
2 @Qualifier("userJdbcImps")
3 private UserRepository userRepository;
```
- 输出结果：UserJdbcImps save
