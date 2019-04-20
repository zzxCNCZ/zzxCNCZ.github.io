---
title: SpringBoot自动部署服务到Tomcat
date: 2019-04-20 15:23:53
categories:
- Java
- SpringBoot
tags:
- SpringBoot
---
## 安装tomcat7-maven-plugin maven插件

```
<plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <url>http://127.0.0.1:8080/manager/text</url>
                    <username>username</username>
                    <password>password</password>
                    <update>true</update>
                </configuration>
            </plugin>
```
- http://127.0.0.1:8080为tomcat地址 /manager/text为tomcat script角色页面，manager-script为可以用来部署管理项目文件的角色
- 配置Tomcat：修改conf/tomcat-users.xml 添加manager-script角色和账户
```
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="root" password="zzxcncz" roles="manager-gui,manager-script"/>
```
- 以上添加了两种角色，manager-script 访问ip:port/manager/text路径,manager-gui访问ip:port/manager/html 路径，也可以只添加一种角色
- 配置tomcat远程访问
- 在tomcat  目录下/conf/Catalina/localhost/添加文件manager.xml，添加内容：
```
<?xml version="1.0" encoding="UTF-8"?>
 
<Context privileged="true" antiResourceLocking="false"   
         docBase="${catalina.home}/webapps/manager">  
             <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />  
</Context>  
```
- 完成以上步骤即可使用tomcat7-maven-plugin插件 部署项目，各种命令如下
1. deploy：第一次部署，打包并发布项目上去
2. redeploy： 已有项目在运行，重新发布
3. undeploy：停止项目运行


---
- 附：
- 使用profiles，切换测试环境，生产环境等，配置文件添加：
```
    <profiles>
        <profile>
            <!-- 本地开发环境 -->
            <id>dev</id>
            <properties>
                <profileActive>dev</profileActive>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <profile>
            <id>test</id>
            <properties>
                <profileActive>test</profileActive>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <profileActive>prod</profileActive>
            </properties>
        </profile>
    </profiles>
```
- application.yml 添加：
```
spring:
    # 环境 dev|test|prod
    profiles:
        active: @profileActive@
```
