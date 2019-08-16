---
layout: w
title: SpringBoot注解
date: 2018-03-05 15:17:33
categories:
- Java
- SpringBoot
tags:
- SpringBoot
---

##SpringBoot注解使用
1. @SpringBootApplication
- @SpringBootApplication = (默认属性)@Configuration + @EnableAutoConfiguration + @ComponentScan。
```javascript
@SpringBootApplication  
public class ApplicationMain {  
    public static void main(String[] args) {  
        SpringApplication.run(Application.class, args);  
    }  
}  
```
* @Configuration：提到@Configuration就要提到他的搭档@Bean。使用这两个注解就可以创建一个简单的spring配置类，可以用来替代相应的xml配置文件。
```javascript
Spring配置文件
<beans>  
    <bean id = "car" class="com.test.Car">  
        <property name="wheel" ref = "wheel"></property>  
    </bean>  
    <bean id = "wheel" class="com.test.Wheel"></bean>  
</beans>  
相当于
@Configuration  
public class Conf {  
    @Bean  
    public Car car() {  
        Car car = new Car();  
        car.setWheel(wheel());  
        return car;  
    }  
    @Bean   
    public Wheel wheel() {  
        return new Wheel();  
    }  
}  
```
<!--more-->
* @Configuration的注解类标识这个类可以使用Spring IoC容器作为bean定义的来源。@Bean注解告诉Spring，一个带有@Bean的注解方法将返回一个对象，该对象应该被注册为在Spring应用程序上下文中的bean。
* @EnableAutoConfiguration：能够自动配置spring的上下文，试图猜测和配置你想要的bean类，通常会自动根据你的类路径和你的bean定义自动配置。
* @ComponentScan：会自动扫描指定包下的全部标有@Component的类，并注册成bean，当然包括@Component下的子解@Service,@Repository,@Controlle。

2. @Mapper注解
* 让DemoMapper能够让别的类进行引用
```javascript
@Mapper  
public interface DemoMapper {  
    @Insert("insert into Demo(name) values(#{name})")  
    @Options(keyProperty="id",keyColumn="id",useGeneratedKeys=true)  
    public void save(Demo demo);  
}  
```
3. @MapperScan注解
* 使用@MapperScan可以指定要扫描的Mapper类的包的路径,可以添加多个路径,以及可以用 `*` 来表示子包
```javascript
@SpringBootApplication  
@MapperScan({"com.kfit.*.mapper","org.kfit.*.mapper"})  
public class App {  
    public static void main(String[] args) {  
       SpringApplication.run(App.class, args);  
    }  
}
```
