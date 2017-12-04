---
title: SpringMVC配置及整合spring security
date: 2017-11-30 08:36:11
categories:
- Java
- SpringMVC
tags:
- SpringMVC
- spring security
---

## SpringMVC
1. pom文件
2. springmvc配置
3. web.xml
4. spring security配置
5. java代码 
<!--more-->
 * pom文件
```javascript
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.zzx</groupId>
    <artifactId>maven-springmvc</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>maven-springmvc Maven Webapp</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>

        <!--logback-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.21</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.12</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.1.3</version>
            <scope>compile</scope>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-api</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.1.3</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                </exclusion>
            </exclusions>
            <scope>compile</scope>
        </dependency>

        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-access</artifactId>
            <version>1.1.3</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                </exclusion>
            </exclusions>
            <scope>compile</scope>
        </dependency>

        <!--j2ee相关包 servlet、jsp、jstl-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>


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

        <!--spring相关包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>4.3.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.3.1.RELEASE</version>
        </dependency>
        <!--spring-security-->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-web</artifactId>
            <version>4.1.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-config</artifactId>
            <version>4.1.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-taglibs</artifactId>
            <version>4.1.2.RELEASE</version>
        </dependency>

        <!--其他需要的包-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.4</version>
        </dependency>
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.1</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>maven-springmvc</finalName>
        <resources>
            <!--表示把java目录下的有关xml文件,properties文件编译/打包的时候放在resource目录下-->
            <resource>
                <directory>${basedir}/src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>${basedir}/src/main/resources</directory>
            </resource>
        </resources>
        <plugins>
            <!--servlet容器 jetty插件-->
            <plugin>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <version>9.3.10.v20160621</version>
            </plugin>

            <!--mybatis 逆向生成插件-->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
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
                        <version>1.3.2</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
</project>

```
 * web.xml
 ```javascript
 <?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
         http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0">
   <!--session-config -->
   <session-config>
       <session-timeout>1</session-timeout>
   </session-config>

   <!--welcome pages-->
   <welcome-file-list>
       <welcome-file>login.jsp</welcome-file>
   </welcome-file-list>

   <!--logback begin-->
   <context-param>
       <param-name>logbackConfigLocation</param-name>
       <param-value>classpath:/logback.xml</param-value>
   </context-param>
   <listener>
       <listener-class>com.example.util.LogbackConfigListener</listener-class>
   </listener>
 <!--logback end-->


   <!-- Creates the Spring Container shared by all Servlets and Filters -->
   <listener>
       <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
   </listener>


   <!--限制用户登录端口数 Spring security 模块-->
   <listener>
       <listener-class>org.springframework.security.web.session.HttpSessionEventPublisher</listener-class>
   </listener>
   <!-- Spring Security begin -->
   <context-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath:spring/spring-security.xml</param-value>
   </context-param>
   <!-- 用户权限模块 -->
   <filter>
       <filter-name>springSecurityFilterChain</filter-name>
       <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
   </filter>
   <filter-mapping>
       <filter-name>springSecurityFilterChain</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
    <!-- Spring Security end -->

   <!--配置springmvc DispatcherServlet-->
   <servlet>
       <servlet-name>springMVC</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <init-param>
           <!--Sources标注的文件夹下需要新建一个spring文件夹-->
           <param-name>contextConfigLocation</param-name>
           <param-value>
               classpath:spring/spring-mvc.xml
               classpath:spring/spring-mybatis-beans.xml
           </param-value>
       </init-param>
       <load-on-startup>1</load-on-startup>
       <async-supported>true</async-supported>
   </servlet>
   <servlet-mapping>
       <servlet-name>springMVC</servlet-name>
       <url-pattern>/</url-pattern>
   </servlet-mapping>
</web-app>
```

## springMVC
  * 配置文件
```javascript
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
                         http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-3.2.xsd
                        http://www.springframework.org/schema/mvc
                        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!--启用spring的一些annotation -->
    <context:annotation-config/>
    <!-- 自动扫描该包，使SpringMVC认为包下用了@controller注解的类是控制器 -->
    <context:component-scan base-package="com.example">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!--HandlerMapping 无需配置，springmvc可以默认启动-->

    <!--静态资源映射-->
    <!--本项目把静态资源放在了WEB-INF的statics目录下，资源映射如下-->
    <mvc:resources mapping="/css/**" location="/WEB-INF/statics/css/"/>
    <mvc:resources mapping="/js/**" location="/WEB-INF/statics/js/"/>
    <mvc:resources mapping="/image/**" location="/WEB-INF/statics/image/"/>

    <!--但是项目部署到linux下发现WEB-INF的静态资源会出现无法解析的情况，但是本地tomcat访问正常，因此建议还是直接把静态资源放在webapp的statics下，映射配置如下-->
    <!--<mvc:resources mapping="/css/**" location="/statics/css/"/>-->
    <!--<mvc:resources mapping="/js/**" location="/statics/js/"/>-->
    <!--<mvc:resources mapping="/image/**" location="/statics/images/"/>-->

    <!-- 配置注解驱动 可以将request参数与绑定到controller参数上 -->
    <mvc:annotation-driven/>

    <!-- 对模型视图名称的解析，即在模型视图名称添加前后缀(如果最后一个还是表示文件夹,则最后的斜杠不要漏了) 使用JSP-->
    <!-- 默认的视图解析器 在上边的解析错误时使用 (默认使用html)- -->
    <bean id="defaultViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/views/"/><!--设置JSP文件的目录位置-->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- springmvc文件上传需要配置的节点-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize" value="20971500"/>
        <property name="defaultEncoding" value="UTF-8"/>
        <property name="resolveLazily" value="true"/>
    </bean>

    <!-- 设置使用注解的类所在的jar包 -->
    <context:component-scan base-package="com.example.security"></context:component-scan>

    <!--载入配置文件-->
    <context:property-placeholder location="classpath:db/jdbc.properties"/>

</beans>
```

## 整合Sping Security
  * 配置文件
```javascript
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:security="http://www.springframework.org/schema/security"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/security
          http://www.springframework.org/schema/security/spring-security.xsd">

    <!--需要过滤不被拦截的请求-->
    <security:http pattern="/user/loginPage" security="none"/>
    <security:http pattern="/user/sessiontimeout" security="none"/>
    <security:http pattern="/sessiontimeout.jsp" security="none"/>

    <!--权限控制 hasRole('ROLE_USER') -->
    <security:http auto-config="true" use-expressions="true">
        <security:intercept-url pattern="/**" access="hasRole('ROLE_USER')"/>
        <!--登陆页面配置，登陆成功页面login-page="/user/loginPage"，error页面"/user/loginPage?error=error"
        -->
        <security:form-login login-page="/user/loginPage" authentication-failure-url="/user/loginPage?error=error"
                             default-target-url="/user/index" login-processing-url="/login"/>
          <!--logout控制-->
        <security:logout invalidate-session="true" logout-success-url="/user/loginPage" logout-url="/logout"/>
          <!--session控制-->
        <security:session-management invalid-session-url="/user/sessiontimeout"
                                     session-authentication-error-url="/sessiontimeout.jsp">
            <!-- session-authentication-error-url当有remember_me功能时使用-->
            <!--限制同一用户在应用中同时允许存在的已经通过认证的session数量。这个值默认是1，可以通过concurrency-control元素的max-sessions属性来指定-->
            <security:concurrency-control error-if-maximum-exceeded="true"
                                          max-sessions="1" expired-url="/sessiontimeout.jsp"/>
        </security:session-management>
        <!-- 退出登录时删除session对应的cookie -->
        <security:logout delete-cookies="JSESSIONID"/>
        <security:csrf disabled="true"/>

    </security:http>

    <!--beans-->
    <bean id="loginUserDetailService" class="com.example.security.impl.LoginUserDetailsServiceImpl"></bean>
    <bean id="loginAuthenticationProvider" class="com.example.security.LoginAuthenticationProvider">
        <property name="userDetailsService" ref="loginUserDetailService"></property>
    </bean>

    <security:authentication-manager alias="myAuthenticationManager">
        <security:authentication-provider ref="loginAuthenticationProvider">
        </security:authentication-provider>
    </security:authentication-manager>
</beans>
```
## java代码
  * LoginAuthenticationProvider
```javascript
package com.example.security;

/**
* @Author:Zhuang zexin
* @Description:
* @Date:Created in 下午 4:53 2017-11-24 0024
* @Modified By:
*/
import org.springframework.security.authentication.AuthenticationServiceException;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.authentication.dao.AbstractUserDetailsAuthenticationProvider;
import org.springframework.security.authentication.dao.SaltSource;
import org.springframework.security.authentication.encoding.PasswordEncoder;
import org.springframework.security.authentication.encoding.PlaintextPasswordEncoder;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.util.Assert;

/**
* 重写DaoAuthenticationProvider的验证方法
*/
public class LoginAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {
  private PasswordEncoder passwordEncoder = new PlaintextPasswordEncoder();

  private SaltSource saltSource;

  private LoginUserDetailsService userDetailsService;

  protected void additionalAuthenticationChecks(UserDetails userDetails,
                                                UsernamePasswordAuthenticationToken authentication)
          throws AuthenticationException {
      Object salt = null;

      if (this.saltSource != null) {
          salt = this.saltSource.getSalt(userDetails);
      }

      if (authentication.getCredentials() == null) {
          logger.debug("Authentication failed: no credentials provided");

          throw new BadCredentialsException("Bad credentials:" + userDetails);
      }

      String presentedPassword = authentication.getCredentials().toString();

      if (!passwordEncoder.isPasswordValid(userDetails.getPassword(), presentedPassword, salt)) {
          logger.debug("Authentication failed: password does not match stored value");

          throw new BadCredentialsException("Bad credentials:" + userDetails);
      }
  }

  protected void doAfterPropertiesSet() throws Exception {
      Assert.notNull(this.userDetailsService, "A UserDetailsService must be set");
  }

  protected PasswordEncoder getPasswordEncoder() {
      return passwordEncoder;
  }

  protected SaltSource getSaltSource() {
      return saltSource;
  }

  protected LoginUserDetailsService getUserDetailsService() {
      return userDetailsService;
  }

  protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication)
          throws AuthenticationException {
      UserDetails loadedUser;

      try {
          String password = (String) authentication.getCredentials();
          /**
           * 区别:这里使用的是自定义的验证方法
           */
          loadedUser = getUserDetailsService().loadUserByUsername(username, password);
      } catch (UsernameNotFoundException notFound) {
          throw notFound;
      } catch (Exception repositoryProblem) {
          throw new AuthenticationServiceException(repositoryProblem.getMessage(), repositoryProblem);
      }

      if (loadedUser == null) {
          throw new AuthenticationServiceException(
                  "UserDetailsService returned null, which is an interface contract violation");
      }
      return loadedUser;
  }

  /**
   * Sets the PasswordEncoder instance to be used to encode and validate
   * passwords. If not set, the password will be compared as plain text.
   * <p>
   * For systems which are already using salted password which are encoded
   * with a previous release, the encoder should be of type
   * {@code org.springframework.security.authentication.encoding.PasswordEncoder}
   * . Otherwise, the recommended approach is to use
   * {@code org.springframework.security.crypto.password.PasswordEncoder}.
   *
   * @param passwordEncoder must be an instance of one of the {@code PasswordEncoder}
   *                        types.
   */
  public void setPasswordEncoder(Object passwordEncoder) {
      Assert.notNull(passwordEncoder, "passwordEncoder cannot be null");

      if (passwordEncoder instanceof PasswordEncoder) {
          this.passwordEncoder = (PasswordEncoder) passwordEncoder;
          return;
      }

      if (passwordEncoder instanceof org.springframework.security.crypto.password.PasswordEncoder) {
          final org.springframework.security.crypto.password.PasswordEncoder delegate = (org.springframework.security.crypto.password.PasswordEncoder) passwordEncoder;
          this.passwordEncoder = new PasswordEncoder() {
              private void checkSalt(Object salt) {
                  Assert.isNull(salt, "Salt value must be null when used with crypto module PasswordEncoder");
              }

              public String encodePassword(String rawPass, Object salt) {
                  checkSalt(salt);
                  return delegate.encode(rawPass);
              }

              public boolean isPasswordValid(String encPass, String rawPass, Object salt) {
                  checkSalt(salt);
                  return delegate.matches(rawPass, encPass);
              }
          };

          return;
      }

      throw new IllegalArgumentException("passwordEncoder must be a PasswordEncoder instance");
  }

  /**
   * The source of salts to use when decoding passwords. <code>null</code> is
   * a valid value, meaning the <code>DaoAuthenticationProvider</code> will
   * present <code>null</code> to the relevant <code>PasswordEncoder</code>.
   * <p>
   * Instead, it is recommended that you use an encoder which uses a random
   * salt and combines it with the password field. This is the default
   * approach taken in the
   * {@code org.springframework.security.crypto.password} package.
   *
   * @param saltSource to use when attempting to decode passwords via the
   *                   <code>PasswordEncoder</code>
   */
  public void setSaltSource(SaltSource saltSource) {
      this.saltSource = saltSource;
  }

  public void setUserDetailsService(LoginUserDetailsService userDetailsService) {
      this.userDetailsService = userDetailsService;
  }
}

```
* LoginUserDetailsImpl
```javascript
package com.example.security;


import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.ArrayList;
import java.util.Collection;
/**
 * @Author:Zhuang zexin
 * @Description:
 * @Date:Created in 下午 4:55 2017-11-24 0024
 * @Modified By:
 */
public class LoginUserDetailsImpl extends User implements UserDetails{
    private static final long serialVersionUID = -5424897749887458053L;

    public LoginUserDetailsImpl(String username, String password, boolean enabled, boolean accountNonExpired,
                                boolean credentialsNonExpired, boolean accountNonLocked,
                                Collection<? extends GrantedAuthority> authorities) {
        super(username, password, enabled, accountNonExpired, credentialsNonExpired, accountNonLocked, authorities);
    }

    public LoginUserDetailsImpl(String username, String password, Collection<? extends GrantedAuthority> authorities) {
        super(username, password, authorities);
    }

    public LoginUserDetailsImpl(String username, String password) {
        super(username, password, new ArrayList<GrantedAuthority>());
    }
}

```
* LoginUserDetailsService
```javascript
package com.example.security;

/**
 * @Author:Zhuang zexin
 * @Description:
 * @Date:Created in 下午 4:56 2017-11-24 0024
 * @Modified By:
 */
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;


public interface LoginUserDetailsService {
    /**
     * 根据用户名密码验证用户信息
     * @param username 用户名
     * @param password 密码
     * @return
     * @throws UsernameNotFoundException
     */
    UserDetails loadUserByUsername(String username, String password) throws UsernameNotFoundException;
}

```
*　LoginUserDetailsServiceImpl
```javascript
package com.example.security.impl;

import com.example.security.LoginUserDetailsImpl;
import com.example.security.LoginUserDetailsService;
import com.example.service.UserService;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.Collection;
/**
 * @Author:Zhuang zexin
 * @Description:
 * @Date:Created in 下午 4:57 2017-11-24 0024
 * @Modified By:
 */
public class LoginUserDetailsServiceImpl implements LoginUserDetailsService {

    @Resource(name="userServiceImpl")
    private UserService userServiceImpl;
    /**
     * 进行登录验证
     */

    public UserDetails loadUserByUsername(String username, String password) throws UsernameNotFoundException {
        boolean result = this.validate(username, password);
        if (!result) {
            return null;
        }
        Collection<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
        authorities.add(new SimpleGrantedAuthority("ROLE_USER"));
        LoginUserDetailsImpl user = new LoginUserDetailsImpl(username, password,authorities);
        return user;
    }

    /**
     * 在此处验证
     * @param username
     * @param password
     * @return
     */
    private boolean validate(String username, String password) {
        /**
         * 此处应该在数据库获取用户信息进行验证
         */
        String checkpassword=userServiceImpl.CheckPassword(username);

        if ("xyc".equals(username) && "123".equals(password)) {
            return true;
        }
        return false;
    }
}

```

[参考](http://blog.csdn.net/xyc_csdn/article/details/52343847)
