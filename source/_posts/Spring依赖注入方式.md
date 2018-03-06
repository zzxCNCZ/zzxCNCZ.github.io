---
title: Spring依赖注入方式
date: 2018-03-06 10:48:31
categories:
- Java
- Spring
tags:
---

## Spring依赖注入方式
1. @Autowired与@Resource都可以用来装配bean. 都可以写在字段上,或写在setter方法上。
2. @Autowired默认按类型装配（这个注解是属业spring的），默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false,如：@Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualifier注解进行使用，如下：
```javascript
@Autowired() @Qualifier("baseDao")    
private BaseDao baseDao;
```
3. @Resource 是JDK1.6支持的注解，默认按照名称进行装配，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名，按照名称查找，如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。只不过注解处理器我们使用的是Spring提供的，是一样的，无所谓解耦不解耦的说法，两个在便利程度上是等同的。
```javascript
@Resource(name="baseDao")    
private BaseDao baseDao;
```
- 他们的主要区别就是@Autowired是默认按照类型装配的 @Resource默认是按照名称装配的byName 通过参数名 自动装配，如果一个bean的name 和另外一个bean的 property 相同，就自动装配。byType 通过参数的数据类型自动自动装配，如果一个bean的数据类型和另外一个bean的property属性的数据类型兼容，就自动装配
<!--more-->
* Spring注解@Component、@Repository、@Service、@Controller区别
-  Spring 2.5 中除了提供 @Component 注释外，还定义了几个拥有特殊语义的注释，它们分别是：@Repository、@Service和 @Controller。在目前的 Spring版本中，这 3 个注释和 @Component 是等效的，但是从注释类的命名上，很容易看出这 3个注释分别和持久层、业务层和控制层（Web层）相对应。虽然目前这 3 个注释和 @Component 相比没有什么新意，但 Spring 将在以后的版本中为它们添加特殊的功能。所以，如果 Web应用程序采用了经典的三层分层结构的话，最好在持久层、业务层和控制层分别采用 @Repository、@Service和 @Controller 对分层中的类进行注释，而用 @Component对那些比较中立的类进行注释。
