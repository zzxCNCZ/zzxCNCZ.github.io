---
title: 'shiro中切换数据源失败问题'
date: 2019-09-23 23:26:54
categories:
- Java
- SpringBoot
tags:
- SpringBoot
---
##### shiro中切换数据源失败问题

在SpringBoot shiro认证框架中，认证环节使用动态代理切换数据源时切换失败。上代码：

OAuth2Realm 类部分代码：

```java
   @Component
public class OAuth2Realm extends AuthorizingRealm { 
    @Autowired
    private AccountService accountService;
		/**
     * 认证(登录时调用)
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String accessToken = (String) token.getPrincipal();

        // 根据token获取用户id   redis start
        String userId = JWTUtil.getUsername(accessToken);

        if (userId == null) {
            throw new IncorrectCredentialsException("token无效");
        }
        // 根据userId从redis中获取登陆账号token
        String key = RedisKeys.getAdminTokenKey(String.valueOf(userId));
        String adminToken = redisUtils.get(key);
        if (adminToken == null || !accessToken.equals(adminToken)) {
            throw new IncorrectCredentialsException("token失效，请重新登录");
        }
        
        AccountEntity user = new AccountEntity();
        if (userId != null) {
            user = accountService.queryAccount(userId);
        }
        
        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, accessToken, getName());
        return info;
    }
}
```
<!--more-->
在认证时，accountService切换了数据源，在service层加了切换注解。

```jade
 @DataSource(name = DataSourceNames.SECOND)
    @Override
    public AccountEntity queryAccount(String userId) {
        return  this.baseMapper.queryAccountById(userId);
    }
```

常理来说是能切换成功并查到DataSourceNames.SECOND数据源的数据的。但是事实上是没有切换成功。

一开始完全没有头绪，别的service同样切换没问题，也没有加入事务，但就是没有切换成功。

后来想到，唯独这个service在shiro认证中注入了，会不会是注入出了问题，果然删掉注入后就能切换成功了。

最后得出的结论就是，shiro中注入的service在spring初始化时无法使用切面来动态代理，具体原因未知。

解决方案就是使用 @Lazy注解延迟加载bean，在shiro认证中需要用到accountService时再初始化，就能完美切换成功了。

```java
    @Lazy
    @Autowired
    private AccountService accountService;
```

