---
title: Spring-Security结合OAuth2
date: 2020-10-19 13:37:35
tags: Spring Security
categories: Spring
---

Spring Security 与 OAuth2结合

<!--more-->

# Oauth2介绍

OAuth 是一个开放标准，该标准允许用户让第三方应用访问该用户在某一网站上存储的私密资源（如头像、照片、视频等），而在这个过程中无需将用户名和密码提供给第三方应用。

实现这一功能是通过提供一个令牌（token），而不是用户名和密码来访问他们存放在特定服务提供者的数据。采用令牌（token）的方式可以让用户灵活的对第三方应用授权或者收回权限。

互联网应用中最常见的 OAuth2 应该就是各种第三方登录了，例如 QQ 授权登录、微信授权登录、微博授权登录、GitHub 授权登录等

## 授权模式

客户端必须得到用户的授权（authorization grant），才能获得令牌（access token）。OAuth 2.0定义了四种授权方式。

- 授权码模式（authorization code）
- 简化模式（implicit）
- 密码模式（resource owner password credentials）
- 客户端模式（client credentials）

# spring security 结合 oauth2



使用spring security 结合 oauth2授权码模式搭建一个demo来学习。

首先使用IDEA创建一个Spring Boot项目，然后在项目上右键 New - module ，创建三个 module：auth-server（授权服务器）、user-server（资源服务器）、client-app（第三方应用），

| 项目        | 端口 | 备注       |
| :---------- | :--- | :--------- |
| auth-server | 8080 | 授权服务器 |
| user-server | 8081 | 资源服务器 |
| client-app  | 8082 | 第三方应用 |

## 搭建授权服务器

首先提供一个 TokenStore 实例，指生成的 token 存储在哪里

```java
@Configuration
public class AccessTokenConfig {
    @Bean
    TokenStore tokenStore(){
        return new InMemoryTokenStore();
    }
}
```



配置授权服务器

```java
@EnableAuthorizationServer
@Configuration
public class AuthorizationServer extends AuthorizationServerConfigurerAdapter {

    @Autowired
    TokenStore tokenStore;

    @Autowired
    ClientDetailsService clientDetailsService;

    /**
     * 配置 token 的基本信息
     */
    @Bean
    AuthorizationServerTokenServices tokenServices() {
        DefaultTokenServices services = new DefaultTokenServices();
        services.setClientDetailsService(clientDetailsService);
        services.setSupportRefreshToken(true);
        services.setTokenStore(tokenStore);
        services.setAccessTokenValiditySeconds(60 * 60 * 2);
        services.setRefreshTokenValiditySeconds(60 * 60 * 24 * 3);
        return services;
    }

    /**
     * 配置令牌端点约束，即该端点谁能访问
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.checkTokenAccess("permitAll()")
                .allowFormAuthenticationForClients();
    }

    /**
     * 配置客户端详细信息，校验客户端，可以将信息存到数据库，本次存入内存
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("baby")
                .secret(new BCryptPasswordEncoder().encode("123"))
                .resourceIds("res1")
                .authorizedGrantTypes("authorization_code", "refresh_token")
                .scopes("all")
                .redirectUris("http://localhost:8082/index.html");
    }

    /**
     * 配置令牌的访问端点和令牌服务
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authorizationCodeServices(authorizationCodeServices())
                .tokenServices(tokenServices());
    }

    @Bean
    AuthorizationCodeServices authorizationCodeServices() {
        return new InMemoryAuthorizationCodeServices();
    }
}

```

* @EnableAuthorizationServer 注解告诉 Spring 激活 authorization server，开启授权服务器的自动化配置。
* @Configuration 注解表明这是一个 authorization server 配置类。
* AuthorizationServerConfigurerAdapter 是 Spring 提供的默认实现 AuthorizationServerConfigurer 接口的类，里面都是空方法。

* ClientDetailsServiceConfigurer 是配置 authorization server 颁发的 client 的凭证



AuthorizationServer 类记得加上 @EnableAuthorizationServer 注解，表示开启授权服务器的自动化配置。

在 AuthorizationServer 类中，主要重写三个 configure 方法

1.  AuthorizationServerSecurityConfigurer 配置令牌端点安全约束。checkTokenAccess 是校检 token 的端点，当资源服务器收到 token 后，需要校检 token 的合法性，就会访问该端点。
2.  ClientDetailsServiceConfigurer 配置客户端详细信息，此处将客户端信息存在内存中，分别配置了客户端的 id，secret，资源id，授权类型，授权范围以及重定向url
3.  AuthorizationServerEndpointsConfigurer 配置令牌的访问端点和令牌服务。authorizationCodeServices 配置**授权码**的存储，此处存在内存中。tokenServices 配置令牌的存储，即 access_token 的存储位置。
4.  AuthorizationServerTokenServices 配置 token 的基本信息



## 搭建资源服务器

配置代码

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    /**
     * 资源服务器和授权服务器是分开的，配置该项
     */
    @Bean
    RemoteTokenServices tokenServices() {
        RemoteTokenServices services = new RemoteTokenServices();
        services.setCheckTokenEndpointUrl("http://localhost:8080/oauth/check_token");
        services.setClientId("baby");
        services.setClientSecret("123");
        return services;
    }

    
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.resourceId("res1").tokenServices(tokenServices());
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin/**").hasRole("admin")
                .anyRequest().authenticated();
    }
}

```



测试接口

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @GetMapping("/admin/hello")
    public String admin() {
        return "admin";
    }

}
```



项目地址：https://github.com/SpringLych/spring-learning

# 参考

* [这个案例写出来，还怕跟面试官扯不明白 OAuth2 登录流程？](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247488214&idx=1&sn=5601775213285217913c92768d415eca&chksm=e9c340b6deb4c9a01bc383b2c0ab124358663adf22a58ba385f792224ba532079a028ba92a3d&scene=178#rd)

* [理解OAuth 2.0](https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)



