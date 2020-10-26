---
title: Spring Security入门
date: 2020-10-16 14:27:02
tags: Spring Security
categories: spring
---

Spring Security 是 Spring 家族中的一个安全管理框架，结合Spring Boot，能够零配置使用 Spring Security。

<!--more-->



# 配置类 WebSecurityConfigurerAdapter 详解

配置 Spring Security 主要通过继承 WebSecurityConfigurerAdapter 实现：

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
        http.authorizeRequests()
        .anyRequest().authenticated()
        ...
    }
}
```

待完善



#  项目体验

使用IDEA创建 Spring Boot 项目并添加 `spirng-security` 相关依赖，之后什么都不用做，Spring Security 能够自动保护所有接口。

创建一个Congroller:

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

访问定义的 "/hello" 接口，会跳转到 spirng security 自带的登录界面，默认用户名是 user，默认密码是在控制台打印的一串 UUID 密码。

## 用户配置



配置有三种配置方式：

* 通过配置文件配置
* 通过配置类配置
* 使用数据库

### 配置文件配置

可以在 `application.yaml` 配置文件中添加：

```yaml
spring:
  security:
    user:
      name: day
      password: 36987
```

重启项目，就能使用自定义的用户和密码了

### 配置类配置

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("boy").roles("admin")
                .password("$2a$10$OR3VSksVAmCzc.7WeaRPR.t0wyCsIj24k0Bne8iKWV1o.V9wsP8Xe");
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```



1. 首先定义 `SecurityConfig` 继承 `WebSecurityConfigurerAdapter` ，重写 `configure` 方法
2. 提供 `PasswordEncoder` 实例
3. configure 方法中，`PasswordEncoder` 开启内存中定义用户
4. 使用 `and()` 配置多个用户

还有一种方式重写 WebSecurityConfigurerAdapter 中的 userDetailsService 方法来提供一个 UserDetailService 实例进而配置多个用户：

```java
@Override
@Bean
protected UserDetailsService userDetailsService() {
    InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
    manager.createUser(User.withUsername("admin").password("123").roles("admin").build());
    manager.createUser(User.withUsername("test").password("123").roles("test").build());
    return manager;
}
```

## 登录使用JSON交互



```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .anyRequest().authenticated()
        .and()
        .formLogin()
        .loginProcessingUrl("/doLogin")
        .successHandler((req, resp, authentication) -> {
            Object principal = authentication.getPrincipal();
            resp.setContentType("application/json;charset=utf-8");
            PrintWriter out = resp.getWriter();
            out.write(new ObjectMapper().writeValueAsString(principal));
            out.flush();
            out.close();
        })
        .failureHandler((req, resp, e) -> {
            resp.setContentType("application/json;charset=utf-8");
            PrintWriter out = resp.getWriter();
            out.write(e.getMessage());
            out.flush();
            out.close();
        })
        .and()
        .logout()
        .logoutSuccessHandler((req, resp, authentication) -> {
            resp.setContentType("application/json;charset=utf-8");
            PrintWriter out = resp.getWriter();
            out.write("注销成功");
            out.flush();
            out.close();
        })
        .and()
        .csrf().disable().exceptionHandling()
        .authenticationEntryPoint((req, resp, authException) -> {
            resp.setContentType("application/json;charset=utf-8");
            PrintWriter out = resp.getWriter();
            out.write("尚未登录，请先登录");
            out.flush();
            out.close();
        });
}
```

`successHandler` 中定义登录成功的操作，返回JSON给前端；`successHandler` 中利用 `req` 参数做服务端跳转 ，使用 `resp` 做客户端跳转，返回JSON数据； 第三个参数 `authentication` 保存了登录成功的用户信息。

`failureHandler` 定义登录失败的操作，返回JSON提示给前端；第三个参数 保存了登录失败的原因，使用JSON返回给前端。

```java
.logout()
    .logoutSuccessHandler((req, resp, authentication) -> {
        resp.setContentType("application/json;charset=utf-8");
        PrintWriter out = resp.getWriter();
        out.write("注销成功");
        out.flush();
        out.close();
    })
```



`logoutSuccessHandler` 注销登录的方案



```java
.csrf().disable().exceptionHandling()
    .authenticationEntryPoint((req, resp, authException) -> {
        resp.setContentType("application/json;charset=utf-8");
        PrintWriter out = resp.getWriter();
        out.write("尚未登录，请先登录");
        out.flush();
        out.close();
    });
```

`authenticationEntryPoint` 未认证处理方案



 

## 授权操作

### 配置用户

使用基于内存的方式提供 UserDetailService 实例来配置。

### 测试接口



```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @GetMapping("/admin/hello")
    public String admin() {
        return "admin hello";
    }

    @GetMapping("/test/hello")
    public String test() {
        return "test hello";
    }
}
```



### 配置

配置拦截规则，在 `WebSecurityConfigurerAdapter` 重写 `configure(HttpSecurity http)` 方法：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/admin/**").hasRole("admin")
        .antMatchers("/test/**").hasRole("text")
        .anyRequest().authenticated()
        .and()
        .formLogin() // 要有这一项，不然无法跳转至登录页面
        .and()
        .csrf().disable();
}
```

注意，`anyRequest` 一定要在所有的 `antMatchers` 后面。

antMatchers 中的配置为 Ant 风格，规则如下：

| 通配符 | 含义             |
| :----- | :--------------- |
| **     | 匹配多层路径     |
| *      | 匹配一层路径     |
| ?      | 匹配任意单个字符 |

1. 如果请求路径满足 `/admin/**` 格式，则用户需要具备 admin 角色。
2. 如果请求路径满足 `/user/**` 格式，则用户需要具备 user 角色。
3. 剩余的其他格式的请求路径，只需要认证（登录）后就可以访问。

通过以上配置，admin 角色不能访问 `/usr/**` ，实现了授权配置。

### 角色继承

实现 admin 具有 test 的所有权限，在 `securityConfig` 中添加以下配置：

```java
@Bean
RoleHierarchy roleHierarchy() {
    RoleHierarchyImpl hierarchy = new RoleHierarchyImpl();
    hierarchy.setHierarchy("ROLE_admin > ROLE_text");
    return hierarchy;
}
```

给相应的角色手动添加 `ROLE_` 前缀，上面的配置表示 `ROLE_admin` 自动具备 `ROLE_user` 的权限。



# 参考

* [手把手带你入门 Spring Security！](https://juejin.im/post/6844903896687575047#heading-0)

  















