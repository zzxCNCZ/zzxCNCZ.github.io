---
title: Spring源码分析(2)
date: 2019-08-05 22:54:59
categories:
- Java
- Spring
tags:
- 源码分析
---
### Spring中的BeanDefinition

##### BeanDefinition

- BeanDefinition顾名思义bean的定义，它其实是bean定义的一个顶级接口

  [![image.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/image.png)](http://blog.zhuangzexin.top:8082/image/PRs)

- BeanDefinition描述一个bean的实例，跟Class类中的字段、方法描述一个类不同，一个Class类的字段、方法并不能描述如何实例化这个类。如果说，Class类描述了一块猪肉，那么BeanDefinition就是描述如何做红烧肉。

- Spring如何解析一个bean的（用配置文件<bean/>或者@Bean）
  - Spring首先会扫描解析指定位置的所有的类得到Resources（可以理解为.Class文件）
  - 然后依照TypeFilter和@Conditional注解决定是否将这个类解析为BeanDefinition
  - 稍后再把一个个BeanDefinition取出实例化成Bean

举一个例子：

现有一个UserCOntroller，以及UserServiceImpl implements UserService，并在UserController中注入UserService：

```java
@Autowired
private UserService userService;
```
<!--more-->
以上代码依赖注入的一定是UserServiceImpl实例吗？

事实上并不一定，当例如在UserServiceImpl中加入@Transactional 注解，则通过在UserController中调用：

[![image150fc6c36c598c16.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/image150fc6c36c598c16.png)](http://blog.zhuangzexin.top:8082/image/YdZ)

输出的是：

[![image9f894ffa88222190.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/image9f894ffa88222190.png)](http://blog.zhuangzexin.top:8082/image/ckc)

而不是预想的：

[![image3795445577e2c32e.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/image3795445577e2c32e.png)](http://blog.zhuangzexin.top:8082/image/FHS)

原因是当加入了类似@Transactional 的注解，注入的是CGLib动态代理生成的userServiceImpl的代理对象（动态代理以下描述）。例如加入@Transactional注解，当Spring读取到该注解，便可以开启事务，但UserService中没有包含任何事务的代码，要实现这种功能，则会用到java的动态代理。

##### Spring Aop原理：

[动态代理]: https://www.jianshu.com/p/1712ef4f2717

java动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。而cglib动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

SpringAOP动态代理策略是：

> ```
> 1、如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP 
> 2、如果目标对象实现了接口，可以强制使用CGLIB实现AOP 
> 3、如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换
> ```

##### 后置处理器

以上例子中UserServiceImpl加入了@Transactional，就会返回代理对象，Spring在何时做了此操作？

大部分人把Spring比作容器，其实潜意识里是将Spring完全等同于一个Map了。其实，真正存单例对象的map，只是Spring中很小很小的一部分，仅仅是BeanFactory的一个字段，我更习惯称它为“单例池”。

```java
/** Cache of singleton objects: bean name --> bean instance */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
```

[![imagef85c647c623e6493.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/imagef85c647c623e6493.png)](http://blog.zhuangzexin.top:8082/image/U5u)

这里的ApplicationContext和BeanFactory是接口，实际上都有各自的子类。比如注解驱动开发时，Spring中最关键的就是AnnotationConfigApplicationContext和DefaultListableBeanFactory。

所以，很多人把Spring理解成一个大Map，还是太浅了。就拿ApplicationContext来讲，它也实现了BeanFactory接口，但是作为容器，其实它是用来包含各种各样的组件的，而不是存bean： 

[![imagee10a0239e36358f1.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/imagee10a0239e36358f1.png)](http://blog.zhuangzexin.top:8082/image/i3d)

那么，Spring是如何给咸鱼加佐料（事务代码的织入）的呢？关键就在于后置处理器。

后置处理器其实可以分好多种，属于Spring的扩展点之一。

后置处理器分类：

[![image81c28eff1f0340c5.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/image81c28eff1f0340c5.png)](http://blog.zhuangzexin.top:8082/image/08y)

上面BeanFactory、BeanDefinitionRegistryPostProcessor、BeanPostProcessor都算是后置处理器，这里篇幅有限，只介绍一下BeanPostProcessor。

[![imageac7cf6654e97e3cf.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/imageac7cf6654e97e3cf.png)](http://blog.zhuangzexin.top:8082/image/9Df)

BeanFactoryPostProcessor是用来干预BeanFactory创建的，而BeanPostProcessor是用来干预Bean的实例化。不知道大家有没有试过在普通Bean中注入ApplicationContext实例？你第一时间想到的是：

```java
@Autowired
ApplicationContext annotationConfigApplicationContext;

```

除了利用Spring本身的IOC容器自动注入以外，你还有别的办法吗？

我们可以让Bean实现ApplicationContextAware接口：

[![image49ef19f40b97d93a.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/image49ef19f40b97d93a.png)](http://blog.zhuangzexin.top:8082/image/1UM)

后期，Spring会调用setApplicationContext()方法传入ApplicationContext实例。

> Spring官方文档：一般来说，您应该避免使用它，因为它将代码耦合到Spring中，并且不遵循控制反转样式。

这是我认为Spring最牛逼的地方：代码具有高度的可扩展性，甚至你自己都懵逼，为什么实现了一个接口，这个方法就被莫名其妙调用，还传进了一个对象…

这其实就是后置处理器的工作！

什么意思呢？

就是说啊，明面上我们看得见的地方只要实现一个接口，但是背地里Spring在自己框架的某一处搞了个for循环，遍历所有的BeanPostProcessor，其中就包括处理实现了ApplicationContextAware接口的bean的后置处理器：ApplicationContextAwareProcessor。

上面这句话有点绕，大家停下来多想几遍。

[![imagef4dd8f861ad65757.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/imagef4dd8f861ad65757.png)](http://blog.zhuangzexin.top:8082/image/3K6)

也就是说，要扩展的类是不确定的，但是处理扩展类的流程是写死的。总有一个要定下来吧。也就是说，在这个Bean实例化的某一紧要处，必然要经过很多BeanPostProcessor。但是，BeanPostProcessor也不是谁都处理，有时也会做判断。比如：

```java
if (bean instanceof Aware) {
    if (bean instanceof EnvironmentAware) {
        ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
    }
    if (bean instanceof EmbeddedValueResolverAware) {
        ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
    }
    if (bean instanceof ResourceLoaderAware) {
        ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
    }
    if (bean instanceof ApplicationEventPublisherAware) {
        ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
    }
    if (bean instanceof MessageSourceAware) {
        ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
    }
    if (bean instanceof ApplicationContextAware) {
        ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
    }
}

```

所以，此时此刻一个类实现ApplicationContextAware接口，有两层含义：

- 作为后置处理器的判断依据，只有你实现了该接口我才处理你
- 提供被后置处理器调用的方法

# 利用后置处理器返回代理对象

大致了解Spring Bean的创建流程后，接下来我们尝试着用BeanPostProcessor返回当前Bean的代理对象。

- pom.xml

```markup
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.3.12.RELEASE</version>
    </dependency>
</dependencies>
```

- AppConfig

```java
@Configuration //JavaConfig方式，即当前配置类相当于一个applicationConotext.xml文件
@ComponentScan //默认扫描当前配置类（AppConfig）所在包及其子包
public class AppConfig {

}
```

- Calculator

```java
public interface Calculator {
    public void add(int a, int b);
}
```

- CalCulatorImpl

```java
@Component
public class CalculatorImpl implements Calculator {
    public void add(int a, int b) {
        System.out.println(a+b);
    }
}
```

- 后置处理器MyAspectJAutoProxyCreator

使用步骤：

- 实现BeanPostProcessor
- @Component加入Spring容器

```java
@Component
public class MyAspectJAutoProxyCreator implements BeanPostProcessor {
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        final Object obj = bean;
        //如果当前经过BeanPostProcessors的Bean是Calculator类型，我们就返回它的代理对象
        if (bean instanceof Calculator) {
           Object proxyObj = Proxy.newProxyInstance(
                    this.getClass().getClassLoader(),
                    bean.getClass().getInterfaces(),
                    new InvocationHandler() {
                        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                            System.out.println("开始计算....");
                            Object result = method.invoke(obj, args);
                            System.out.println("结束计算...");
                            return result;
                        }
                    }
            );
           return proxyObj;
        }
        //否则返回本身
        return obj;
    }
}
```

- 测试类

```java
public class TestPostProcessor {
    public static void main(String[] args) {

        System.out.println("容器启动成功！");
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);

        String[] beanDefinitionNames = applicationContext.getBeanDefinitionNames();
        //打印当前容器所有BeanDefinition
        for (String beanDefinitionName : beanDefinitionNames) {
            System.out.println(beanDefinitionName);
        }

        System.out.println("============");

        //取出Calculator类型的实例，调用add方法
        Calculator calculator = (Calculator) applicationContext.getBean(Calculator.class);
        calculator.add(1, 2);
}
```



先把MyAspectJAutoProxyCreator的@Component注释掉，此时Spring中没有我们自定义的后置处理器，那么返回的就是CalculatorImpl：

[![imageb1f7e13b90d1d598.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/imageb1f7e13b90d1d598.png)](http://blog.zhuangzexin.top:8082/image/bdv)

把@Component加上，此时MyAspectJAutoProxyCreator加入到Spring的BeanPostProcessors中，会拦截到CalculatorImpl，并返回代理对象：

[![image7d5d698b2e5f6eb3.png](http://blog.zhuangzexin.top:8082/images/2019/08/05/image7d5d698b2e5f6eb3.png)](http://blog.zhuangzexin.top:8082/image/ekq)

代理对象的add()方法被增强：前后打印日志
