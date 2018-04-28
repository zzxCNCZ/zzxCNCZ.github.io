---
title: MyBatis Generator使用及自定义Oracle和Mysql分页插件
date: 2018-04-26 14:20:33
categories:
- Java
- SpringMVC
- MyBatis
tags:
- MyBatis Generator
---
### 用MBG插件生成Pojo，PojoExample(条件类)，--Mapper(dao层接口)，--Mapper.xml文件，并重写PluginAdapter类，生成自定义分页代码。
## Maven加入插件，自定义分页插件依赖
```javascript
<!--mybatis 逆向生成插件-->
        <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.3.5</version>
            <!-- 定义配置文件 -->
            <configuration>
                <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
                <verbose>true</verbose>
                <overwrite>true</overwrite>
            </configuration>
            <executions>
                <execution>
                    <id>Generate MyBatis Artifacts</id>
                </execution>
            </executions>
            <dependencies>
                <dependency>
                    <groupId>org.mybatis.generator</groupId>
                    <artifactId>mybatis-generator-core</artifactId>
                    <version>1.3.5</version>
                </dependency>
                 <!-- 自定义插件 -->
                <dependency>
                    <groupId>com.zzxcn</groupId>
                    <artifactId>PaginationPlugin</artifactId>
                    <version>1.0</version>
                </dependency>
            </dependencies>
        </plugin>
```
<!--more-->
## generatorConfig.xml配置文件
```javascript
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration PUBLIC
        "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >
<generatorConfiguration>
    <!-- 数据库配置文件 -->
    <properties resource="db/jdbc.properties"  />
    <!-- 本地数据库驱动程序jar包的全路径 -->
    <classPathEntry location="E:/apache_maven/repository/com/oracle/driver/jdbc/14/jdbc-14.jar"/>
    <!--  MyBatis3Simple(简略模式),MyBatis3普通模式  -->
    <context id="context" targetRuntime="MyBatis3">
<!--
        <plugin type="org.mybatis.generator.plugins.RowBoundsPlugin"></plugin>
-->
<!--
        <plugin type="com.xxg.mybatis.plugins.MySQLLimitPlugin"></plugin>
-->
      <!-- 自定义分页插件 -->
        <plugin type="com.zzxcn.plugin.OraclePaginationPlugin"></plugin>
        <commentGenerator>
            <!--  关闭自动生成的注释  -->
            <property name="suppressAllComments" value="true"/>
            <property name="suppressDate" value="true"/>
        </commentGenerator>

        <jdbcConnection driverClass="${jdbc_driverClassName}"
                        connectionURL="${jdbc_url}"
                        userId="${jdbc_user}"
                        password="${jdbc_pwd}">
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <javaModelGenerator targetPackage="com.zzx.pojo"
                            targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <sqlMapGenerator targetPackage="auto" targetProject="src/main/resources/mybatis">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>

        <javaClientGenerator type="XMLMAPPER" targetPackage="com.zzx.dao.auto"
                             targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>


        <!--   enableCountByExample="false"
              enableUpdateByExample="false"
              enableDeleteByExample="false"
              enableSelectByExample="false"
              selectByExampleQueryId="false" -->
       <!-- <table tableName="TD_ADMIN_ROLE" mapperName="RoleDao" domainObjectName="Role">
            <property name="useActualColumnNames" value="false"/>
        </table>-->
        <!--<table tableName="TE_STRATEGY_OPTION" mapperName="OptionDao" domainObjectName="Option">
            <property name="useActualColumnNames" value="false"/>
        </table>-->
        <table tableName="TE_STRATEGY_FENCE" mapperName="FenceDao" domainObjectName="Fence">
            <property name="useActualColumnNames" value="false"/>
        </table>
    </context>
</generatorConfiguration>
```
