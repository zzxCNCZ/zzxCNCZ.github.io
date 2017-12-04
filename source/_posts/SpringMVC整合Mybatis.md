---
title: SpringMVC整合Mybatis
date: 2017-12-04 17:05:09
categories:
- Java
- SpringMVC
- MyBatis
tags:
- MyBatis
---
## Maven依赖(mybatis,mysql,oracle,druid连接池)
```javascript
<!-- mybatis/spring包 -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.1</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.0</version>
        </dependency>

        <!--mysql驱动包-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.35</version>
        </dependency>

        <!-- Druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.26</version>
        </dependency>


        <!-- oracle driver -->
        <dependency>
            <groupId>com.oracle.driver</groupId>
            <artifactId>jdbc</artifactId>
            <version>14</version>
            <scope>runtime</scope>
        </dependency>
```
<!--more-->
## 配置文件
* web.xml--`可以在spring-mvc文件中引入`
```javascript
<init-param>
            <!--Sources标注的文件夹下需要新建一个spring文件夹-->
            <param-name>contextConfigLocation</param-name>
            <param-value>
                classpath:spring/spring-mvc.xml
                classpath:spring/spring-mybatis-beans.xml
            </param-value>
        </init-param>
```
* spring-mvc.xml文件
```javascript
<!--SpringTool用于工具类使用接口功能-->
  <bean class="com.example.util.SpringTool"/>
  <!--载入配置文件-->
  <context:property-placeholder location="classpath:db/jdbc.properties"/>
```

* spring-mybatis-beans.xml配置
```javascript
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <!-- 配置数据源 -->
    <bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <property name="driverClassName" value="${jdbc_driverClassName}" />
        <property name="url" value="${jdbc_url}"/>
        <property name="username" value="${jdbc_user}"/>
        <property name="password" value="${jdbc_pwd}"/>

        <!-- 初始化连接大小 -->
        <property name="initialSize" value="0"/>
        <!-- 连接池最大使用连接数量 -->
        <property name="maxActive" value="20"/>
        <!-- 连接池最小空闲 -->
        <property name="minIdle" value="0"/>
        <!-- 获取连接最大等待时间 -->
        <property name="maxWait" value="60000"/>
        <property name="validationQuery" value="${validationQuery}"/>
        <property name="testOnBorrow" value="false"/>
        <property name="testOnReturn" value="false"/>
        <property name="testWhileIdle" value="true"/>
        <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
        <property name="timeBetweenEvictionRunsMillis" value="60000"/>
        <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
        <property name="minEvictableIdleTimeMillis" value="25200000"/>
        <!-- 打开removeAbandoned功能 -->
        <property name="removeAbandoned" value="true"/>
        <!-- 1800秒，也就是30分钟 -->
        <property name="removeAbandonedTimeout" value="1800"/>
        <!-- 关闭abanded连接时输出错误日志 -->
        <property name="logAbandoned" value="true"/>
        <!-- 监控数据库 -->
        <!-- <property name="filters" value="stat" /> -->
        <property name="filters" value="mergeStat"/>
    </bean>

    <!-- 配置 mybatis session工厂,需添加 spring-orm -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis/mybatis-config.xml"/>
    </bean>

    <!-- 注册 Mapper方式二：也可不指定特定 mapper，而使用自动扫描包的方式来注册各种M apper ，配置如下： -->
    <!--<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.rambo.sdm.dao.inter"/>
        <property name="sqlSessionFactoryBeanName" value="sessionFactory"/>
    </bean>-->
</beans>
```
* mybatis-config.xml-`MyBatis入口文件`
```javascript
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<settings>
	<!-- 设置sql参数 如果java属性为空需要这样配置进行转换成sql的null，因为mybatis没有提供java中null类型的转换器-->
		<setting name="jdbcTypeForNull" value="NULL" />
	</settings>
	<typeAliases>
		<!-- admin,配置别名 -->
        <typeAlias alias="LoginUser" type="com.example.pojo.LoginUser" />

	</typeAliases>

	<mappers>
		<!-- base,映射文件 -->
		<mapper resource="mybatis/admin/LoginUser.xml" />
	</mappers>
</configuration>
```
* db.properties
```javascript
jdbc_driverClassName=oracle.jdbc.driver.OracleDriver
jdbc_url=jdbc:oracle:thin:@localhost:1521:orcl
jdbc_user=pm
jdbc_pwd=cyber
validationQuery = select 1  from dual
```

## 附录
1.springtool--工具类使用接口方法(例如使用userServiceImpl时为null的情况)
```javascript
package com.example.util;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

/**
 * @Author:Zhuang zexin
 * @Description:
 * @Date:Created in 上午 9:43 2017-12-1 0001
 * @Modified By:
 */
public final class SpringTool implements ApplicationContextAware {
    private static ApplicationContext applicationContext = null;

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        if (SpringTool.applicationContext == null) {
            SpringTool.applicationContext = applicationContext;
            System.out.println(
                    "========ApplicationContext配置成功,在普通类可以通过调用ToolSpring.getAppContext()获取applicationContext对象,applicationContext="
                            + applicationContext + "========");
        }
    }

    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    public static Object getBean(String name) {
        return getApplicationContext().getBean(name);
    }
}

"使用时方法:"
 UserService userServiceImpl=(UserService) SpringTool.getBean("userServiceImpl");
```
2.MyBatis使用
-参考[resultmap和resulttype的区别](http://blog.csdn.net/woshixuye/article/details/27521071)
-参考[MyBatis根据数组集合查询](https://www.cnblogs.com/Eilen/p/6744923.html)

3.SqlSessionFactory接口源码
```javascript
public interface SqlSession extends Closeable {

  <T> T selectOne(String statement);

  <T> T selectOne(String statement, Object parameter);

  <E> List<E> selectList(String statement);

  <E> List<E> selectList(String statement, Object parameter);

  <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds);

  <K, V> Map<K, V> selectMap(String statement, String mapKey);

  <K, V> Map<K, V> selectMap(String statement, Object parameter, String mapKey);

  <K, V> Map<K, V> selectMap(String statement, Object parameter, String mapKey, RowBounds rowBounds);

  void select(String statement, Object parameter, ResultHandler handler);

  void select(String statement, ResultHandler handler);

  void select(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler);

  int insert(String statement);

  int insert(String statement, Object parameter);

  int update(String statement);

  int update(String statement, Object parameter);

  int delete(String statement);

  int delete(String statement, Object parameter);

  void commit();

  void commit(boolean force);

  void rollback();

  void rollback(boolean force);

  List<BatchResult> flushStatements();

  void close();

  void clearCache();

  Configuration getConfiguration();

  <T> T getMapper(Class<T> type);

  Connection getConnection();
}
```
