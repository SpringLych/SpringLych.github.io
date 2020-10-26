---
title: Spring-Security与OAuth2的常用配置
date: 2020-10-19 13:51:21
tags: Spring Security
categories: Spring
---

整理Spring Security以及结合OAuth2的一些配置

<!--more-->



# Spring Security的配置

Spring Security 的配置主要在 WebSecurityConfigurerAdapter 类，一般新建一个 WebSecurityConfig 类继承该类进行配置：

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
        //        return new BCryptPasswordEncoder();
    }

    /**
    * 用于配置用户相关
    */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("admin")
            .password("123")
            .roles("admin");
    }

    /**
    * 配置用户
    */
    @Override
    @Bean
    protected UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withUsername("admin").password("123").roles("admin").build());
        manager.createUser(User.withUsername("test").password("123").roles("test").build());
        return manager;
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    /**
    * 处理http认证相关
    */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
    }

}
```



# OAuth2配置



## AuthorizationServerConfigurerAdapter

配置OAuth2 主要重写 AuthorizationServerConfigurerAdapter 类中的三个 configure 方法，该类中的三个方法都是空的：

```java
public class AuthorizationServerConfigurerAdapter implements AuthorizationServerConfigurer {
    public AuthorizationServerConfigurerAdapter() {
    }

    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
    }

    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    }

    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    }
}
```

### configure(AuthorizationServerSecurityConfigurer security)

配置令牌的端点约束，一个端点谁能访问

### configure(ClientDetailsServiceConfigurer clients)

这里用来配置客户端信息，可以配置信息存在内存或数据库中，示例：

```java
clients.inMemory()
        .withClient("baby")
        .secret(new BCryptPasswordEncoder().encode("123"))
        .resourceIds("res1")
        .authorizedGrantTypes("password", "refresh_token")
        .scopes("all")
        .redirectUris("http://localhost:8082/index.html");
```

在 authorizedGrantTypes 中配置 OAuth2 的模式：授权码、密码等模式

存入数据库

```java
clients.withClientDetails(clientDetailsService());
```



### configure(AuthorizationServerEndpointsConfigurer endpoints)

配置令牌端点约束

```java
endpoints.authenticationManager(authenticationManager)
                .tokenStore(jwtTokenStore())
                .accessTokenConverter(accessTokenConverter());
```





## 其他

### 令牌保存位置 TokenStore

。TokenStore 接口有多种实现类用来保存 access_token：

* JDBCTokenStore：令牌保存到数据库中
* InMemoryTokenStore：令牌保存到内存中
* RedisTokenStore：保存到Redis
* JwtTokenStore：使用jwt就是无状态登录，服务端不保存
* JwkTokenStore：保存到 JSON Web Key 中



### AuthorizationServerTokenServices

生成 access_token 

```java
@Bean
AuthorizationServerTokenServices tokenServices() {
    DefaultTokenServices services = new DefaultTokenServices();
    services.setClientDetailsService(clientDetailsService());
    services.setSupportRefreshToken(true);
    services.setTokenStore(tokenStore());
    services.setAccessTokenValiditySeconds(60 * 60 * 2);
    services.setRefreshTokenValiditySeconds(60 * 60 * 24 * 3);
    return services;
}
```

其中 **DefaultTokenServices**，就是默认生成 access_token 的实例。



## JWT





## 示例



我们可以定义 AuthorizationServer 类实现 AuthorizationServerConfigurerAdapter 类：

```java
@EnableAuthorizationServer
@Configuration
public class AuthorizationServer extends AuthorizationServerConfigurerAdapter {

    /**
     * 密码模式用到
     */
    @Autowired
    AuthenticationManager authenticationManager;

    @Autowired
    RedisConnectionFactory redisConnectionFactory;

    @Autowired
    DataSource dataSource;


    @Bean
    TokenStore tokenStore() {
        return new RedisTokenStore(redisConnectionFactory);
    }

    /**
     * 配置客户端信息存储在 MySQL 中
     */
    @Bean
    ClientDetailsService clientDetailsService() {
        return new JdbcClientDetailsService(dataSource);
    }

    @Bean
    AuthorizationServerTokenServices tokenServices() {
        DefaultTokenServices services = new DefaultTokenServices();
        services.setClientDetailsService(clientDetailsService());
        services.setSupportRefreshToken(true);
        services.setTokenStore(tokenStore());
        services.setAccessTokenValiditySeconds(60 * 60 * 2);
        services.setRefreshTokenValiditySeconds(60 * 60 * 24 * 3);
        return services;
    }


    /**
     * 配置客户端详细信息
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.withClientDetails(clientDetailsService());
    }

    /**
     * 配置令牌点约束
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authenticationManager(authenticationManager)
                .tokenStore(tokenStore());
//                .accessTokenConverter(accessTokenConverter());
    }

}

```



