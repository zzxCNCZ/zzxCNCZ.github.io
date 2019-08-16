---
title: MyBatis Generator使用及自定义Oracle和Mysql分页插件
date: 2018-04-26 14:20:33
categories:
- Java
- SpringMVC
tags:
- MyBatis
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
## 自定义分页插件com.zzxcn.plugin.OraclePaginationPlugin,继承PluginAdapter类
```javascript

//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package com.zzxcn.plugin;

import java.util.List;
import org.mybatis.generator.api.CommentGenerator;
import org.mybatis.generator.api.IntrospectedTable;
import org.mybatis.generator.api.PluginAdapter;
import org.mybatis.generator.api.dom.java.Field;
import org.mybatis.generator.api.dom.java.FullyQualifiedJavaType;
import org.mybatis.generator.api.dom.java.JavaVisibility;
import org.mybatis.generator.api.dom.java.Method;
import org.mybatis.generator.api.dom.java.Parameter;
import org.mybatis.generator.api.dom.java.TopLevelClass;
import org.mybatis.generator.api.dom.xml.Attribute;
import org.mybatis.generator.api.dom.xml.Document;
import org.mybatis.generator.api.dom.xml.TextElement;
import org.mybatis.generator.api.dom.xml.XmlElement;

public class OraclePaginationPlugin extends PluginAdapter {
    public OraclePaginationPlugin() {
    }

    public boolean validate(List<String> list) {
        return true;
    }

    public boolean modelExampleClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
        topLevelClass.addImportedType(new FullyQualifiedJavaType("com.zzxcn.pojo.Page"));
        String name = "Page";
        CommentGenerator commentGenerator = this.context.getCommentGenerator();
        Field field = new Field();
        field.setVisibility(JavaVisibility.PROTECTED);
        field.setType(new FullyQualifiedJavaType("com.zzxcn.pojo.Page"));
        field.setName("Page");
        commentGenerator.addFieldComment(field, introspectedTable);
        topLevelClass.addField(field);
        char c = name.charAt(0);
        String camel = Character.toUpperCase(c) + name.substring(1);
        Method method = new Method();
        method.setVisibility(JavaVisibility.PUBLIC);
        method.setName("set" + camel);
        method.addParameter(new Parameter(new FullyQualifiedJavaType("com.zzxcn.pojo.Page"), name));
        method.addBodyLine("this." + name + "=" + name + ";");
        commentGenerator.addGeneralMethodComment(method, introspectedTable);
        topLevelClass.addMethod(method);
        method = new Method();
        method.setVisibility(JavaVisibility.PUBLIC);
        method.setReturnType(new FullyQualifiedJavaType("com.zzxcn.pojo.Page"));
        method.setName("get" + camel);
        method.addBodyLine("return " + name + ";");
        commentGenerator.addGeneralMethodComment(method, introspectedTable);
        topLevelClass.addMethod(method);
        return true;
    }

    public boolean sqlMapDocumentGenerated(Document document, IntrospectedTable introspectedTable) {
        XmlElement parentElement = document.getRootElement();
        XmlElement paginationPrefixElement = new XmlElement("sql");
        paginationPrefixElement.addAttribute(new Attribute("id", "OracleDialectPrefix"));
        XmlElement pageStart = new XmlElement("if");
        pageStart.addAttribute(new Attribute("test", "Page != null"));
        pageStart.addElement(new TextElement("select * from ( select row_.*, rownum rownum_ from ( "));
        paginationPrefixElement.addElement(pageStart);
        parentElement.addElement(paginationPrefixElement);
        XmlElement paginationSuffixElement = new XmlElement("sql");
        paginationSuffixElement.addAttribute(new Attribute("id", "OracleDialectSuffix"));
        XmlElement pageEnd = new XmlElement("if");
        pageEnd.addAttribute(new Attribute("test", "Page != null"));
        pageEnd.addElement(new TextElement("<![CDATA[ ) row_ ) where rownum_ > #{Page.begin} and rownum_ <= #{Page.end} ]]>"));
        paginationSuffixElement.addElement(pageEnd);
        parentElement.addElement(paginationSuffixElement);
        return super.sqlMapDocumentGenerated(document, introspectedTable);
    }

    public boolean sqlMapSelectByExampleWithoutBLOBsElementGenerated(XmlElement element, IntrospectedTable introspectedTable) {
        XmlElement pageStart = new XmlElement("include");
        pageStart.addAttribute(new Attribute("refid", "OracleDialectPrefix"));
        element.getElements().add(0, pageStart);
        XmlElement isNotNullElement = new XmlElement("include");
        isNotNullElement.addAttribute(new Attribute("refid", "OracleDialectSuffix"));
        element.getElements().add(isNotNullElement);
        return true;
    }
}

```
## Page类
```javascript
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package com.zzxcn.pojo;

public class Page {
    private int begin;
    private int end;
    private int length;
    private int count;
    private int current;
    private int total;

    public Page() {
    }

    public Page(int begin, int length) {
        this.begin = begin;
        this.length = length;
        this.end = this.begin + this.length;
        this.current = (int)Math.floor((double)this.begin * 1.0D / (double)this.length) + 1;
    }

    public Page(int begin, int length, int count) {
        this(begin, length);
        this.count = count;
    }

    public int getBegin() {
        return this.begin;
    }

    public int getEnd() {
        return this.end;
    }

    public void setEnd(int end) {
        this.end = end;
    }

    public void setBegin(int begin) {
        this.begin = begin;
        if(this.length != 0) {
            this.current = (int)Math.floor((double)this.begin * 1.0D / (double)this.length) + 1;
        }

    }

    public int getLength() {
        return this.length;
    }

    public void setLength(int length) {
        this.length = length;
        if(this.begin != 0) {
            this.current = (int)Math.floor((double)this.begin * 1.0D / (double)this.length) + 1;
        }

    }

    public int getCount() {
        return this.count;
    }

    public void setCount(int count) {
        this.count = count;
        this.total = (int)Math.floor((double)this.count * 1.0D / (double)this.length);
        if(this.count % this.length != 0) {
            ++this.total;
        }

    }

    public int getCurrent() {
        return this.current;
    }

    public void setCurrent(int current) {
        this.current = current;
    }

    public int getTotal() {
        return this.total == 0?1:this.total;
    }

    public void setTotal(int total) {
        this.total = total;
    }
}
```
## 自定义的插件创建方法：新建maven项目,写入方法,maven install,然后再项目中引用
