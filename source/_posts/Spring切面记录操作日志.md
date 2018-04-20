---
title: Spring切面记录操作日志
date: 2018-04-18 09:47:44
categories:
- Java
- Spring
tags:
- AOP
---
## 关于Spring AOP的使用--记录系统操作日志
1. AOP：面向切面编程（面向接口的编程(⊙﹏⊙)），通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。
2. 记录数据库操作日志的方法一般有三种：
- 最原始的，每次执行操作语句后执行记录方法，不好管理，效率低
- 拦截器：新建一个拦截器的class继承spring web的HandlerInterceptorAdapter类，重写：
* preHandle：它会在处理方法之前执行，可以用来做一些编码处理、安全限制之类的操作。
* postHandle：它是在方法执行后开始返回前执行，可以进行日志记录、修改ModelView之类的操作。
* afterCompletion：最后执行，无论出错与否都会执行这个方法，可以用来记录异常信息和一些必要的操作记录。
* afterConcurrentHandlingStarted：controller方法异步开始执行时就开始执行这个方法，而postHandle需要等到controller异步执行完成后再执行。
 * 缺点：比较麻烦，不能获取参数
- AOP，通过加入注解@Aspect，创建切入点。加入注解@Pointcut，配置接入点（配置切入入口，模块，如Controller，Dao）,然后在配置文件配置<aop:aspectj-autoproxy>，即aspectj动态代理。
## 具体实现
<!--more-->
1. Maven包：
```javascript

        <!-- AOP begin -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>1.7.4</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.7.4</version>
        </dependency>
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>3.1</version>
            <scope>runtime</scope>
        </dependency>
        <!-- AOP end -->
```
2. Spring配置文件
- DTD：
```javascript
xmlns:aop="http://www.springframework.org/schema/aop"
      xsi:schemaLocation="http://www.springframework.org/schema/aop"
      "http://www.springframework.org/schema/aop/spring-aop.xsd"
```
- 内容
```javascript
<aop:aspectj-autoproxy proxy-target-class="true" />
 <!--aop执行操作的类-->
 <bean id="logAopAction" class="com.example.core.LogAopAction"/>
```
3. 创建日志类
```javascript
public class LogEntity implements Serializable{

    private String userId;
    private String moudle;
    private String method;
    private String reponseData;

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String getMoudle() {
        return moudle;
    }

    public void setMoudle(String moudle) {
        this.moudle = moudle;
    }

    public String getMethod() {
        return method;
    }

    public void setMethod(String method) {
        this.method = method;
    }

    public String getReponseData() {
        return reponseData;
    }

    public void setReponseData(String reponseData) {
        this.reponseData = reponseData;
    }

    public String getIp() {
        return ip;
    }

    public void setIp(String ip) {
        this.ip = ip;
    }

    public String getData() {
        return data;
    }

    public void setData(String data) {
        this.data = data;
    }

    public String getCommite() {
        return commite;
    }

    public void setCommite(String commite) {
        this.commite = commite;
    }

    private String ip;
    private String data;
    private String commite;
}
```
4. 创建切入点方法,方法中调用相关日志操作接口，记录到数据库中
```javascript
package com.example.core;

import com.example.pojo.LogEntity;
import com.example.service.LogService;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Method;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Map;

/**
 * @Author:Zhuang Zx
 * @Description:
 * @Date:Created in 17:28 2018/4/2
 * @Modified By:
 */

@Aspect
public class LogAopAction {
    @Autowired
    private LogService logServiceImpl;

    //配置接入点,如果不知道怎么配置,可以百度一下规则
    @Pointcut("execution(* com.example.controller..*.*(..))")
    private void controllerAspect(){}//定义一个切入点

    @Around("controllerAspect()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        //常见日志实体对象
        LogEntity log = new LogEntity();
        //获取登录用户账户
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        String name = (String) request.getSession().getAttribute("USER_ID");
        Map<String,String[]> param= request.getParameterMap();
        log.setUserId(name);
        //获取系统时间
        String time = new SimpleDateFormat("YYYY-MM-dd HH:mm:ss").format(new Date());
        log.setData(time);

        //获取系统ip,这里用的是我自己的工具类,可自行网上查询获取ip方法
       /* String ip = GetLocalIp.localIp();
        log.setIP(ip);*/

        //方法通知前获取时间,为什么要记录这个时间呢？当然是用来计算模块执行时间的
        long start = System.currentTimeMillis();
        // 拦截的实体类，就是当前正在执行的controller
        Object target = pjp.getTarget();
        // 拦截的方法名称。当前正在执行的方法
        String methodName = pjp.getSignature().getName();
        // 拦截的方法参数
        Object[] args = pjp.getArgs();
        // 拦截的放参数类型
        Signature sig = pjp.getSignature();
        MethodSignature msig = null;
        if (!(sig instanceof MethodSignature)) {
            throw new IllegalArgumentException("该注解只能用于方法");
        }
        msig = (MethodSignature) sig;
        Class[] parameterTypes = msig.getMethod().getParameterTypes();

        Object object = null;
        // 获得被拦截的方法
        Method method = null;
        try {
            method = target.getClass().getMethod(methodName, parameterTypes);
        } catch (NoSuchMethodException e1) {
            // TODO Auto-generated catch block
            e1.printStackTrace();
        } catch (SecurityException e1) {
            // TODO Auto-generated catch block
            e1.printStackTrace();
        }
        if (null != method) {
            // 判断是否包含自定义的注解，说明一下这里的SystemLog就是我自己自定义的注解
            if (method.isAnnotationPresent(SystemLog.class)) {
                SystemLog systemlog = method.getAnnotation(SystemLog.class);
                log.setMoudle(systemlog.module());
                log.setMethod(systemlog.methods());
                try {

                    long end = System.currentTimeMillis();
                    //将计算好的时间保存在实体中
                    log.setReponseData(""+(end-start));
                    log.setCommite("执行成功！");
                    //保存进数据库
                    //logservice.saveLog(log);
                    System.out.print("记录成功");
                    object = pjp.proceed();
                } catch (Throwable e) {
                    // TODO Auto-generated catch block
                    long end = System.currentTimeMillis();
                    log.setReponseData(""+(end-start));
                    log.setCommite("执行失败");
                    //logservice.saveLog(log);
                }
            } else {//没有包含注解
                object = pjp.proceed();
            }
        } else { //不需要拦截直接执行
            object = pjp.proceed();
        }
        return object;
    }

}

```
