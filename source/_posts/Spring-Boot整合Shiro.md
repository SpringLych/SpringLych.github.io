---
title: Spring Boot整合Shiro
date: 2019-05-27 14:54:53
tags: [Spring Boot, Shiro]
categories: Spring Boot
---

整合Shiro

<!--more-->

## 开始

使用IDEA创建项目。

`pom.xml`依赖如下

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

其中，使用Shiro官方支持的ehcache缓存。

> 修改`application.yml`

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/springboot_shiro?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 654321
    driver-class-name: com.mysql.cj.jdbc.Driver
  thymeleaf:
    mode: LEGACYHTML5
    cache: false
```

> 初始化数据库

```mysql
DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `username` varchar(255) DEFAULT NULL COMMENT '用户名',
  `password` varchar(255) DEFAULT NULL COMMENT '密码',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

> 实体类Usre.java

```java
@Data
@ToString
public class User {
    private Long id;
    private String username;
    private String password;
}
```

> mapper，service

```java
// mapper
public interface UserMapper {
    @Select("select * from user where username = #{username}")
    User findByUserName(String username);
}

//service
public interface UserService {
    User findByUsername(String username);
}

// impl
@Service
public class UserServiceImpl implements UserService {
    private final UserMapper userMapper;

    public UserServiceImpl(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public User findByUsername(String name){
        return userMapper.findByUsername(name);
    }
}
```

> Shiro配置

首先要配置的是 ShiroConfig 类，Apache Shiro 核心通过 Filter 来实现，就好像 SpringMvc 通过 DispachServlet 来主控制一样。 既然是使用 Filter 一般也就能猜到，是通过 URL 规则来进行过滤和权限校验，所以我们需要定义一系列关于 URL 的规则和访问权限。

shiroConfig.java

```java
@Configuration
public class ShiroConfig {

    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean filterFactoryBean = new ShiroFilterFactoryBean();
        filterFactoryBean.setSecurityManager(securityManager);
        filterFactoryBean.setLoginUrl("/login");

        // 拦截器
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        // 配置退出
        filterChainDefinitionMap.put("/logout", "logout");
        // 如果不设置默认会自动寻找Web工程根目录下的"/login.jsp"页面;
        filterChainDefinitionMap.put("/login", "anon");


        // 静态资源
        filterChainDefinitionMap.put("/css/**", "anon");
        filterChainDefinitionMap.put("/js/**", "anon");
        filterChainDefinitionMap.put("/lib/**", "anon");

        // 其他请求都拦截，
        filterChainDefinitionMap.put("/**", "user");

        filterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return filterFactoryBean;

    }

    @Bean
    public MyShiroRealm myShiroRealm() {
        return new MyShiroRealm();
    }

    @Bean
    public SecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(myShiroRealm());
        securityManager.setSessionManager(sessionManager());
        return securityManager;
    }

    @Bean
    public SessionManager sessionManager() {
        DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
        // 10min
        sessionManager.setGlobalSessionTimeout(60 * 60 * 10);
        sessionManager.setSessionDAO(new EnterpriseCacheSessionDAO());
        return sessionManager;
    }
}
```

Filter Chain 定义说明：

* 1、一个URL可以配置多个 Filter，使用逗号分隔
* 2、当设置多个过滤器时，全部验证通过，才视为通过
* 3、部分过滤器可指定参数，如 perms，roles

> 自定义Realm实现

在认证、授权内部实现机制中都有提到，最终处理都将交给Real进行处理。因为在 Shiro 中，最终是通过 Realm 来获取应用程序中的用户、角色及权限信息的。通常情况下，在 Realm 中会直接从我们的数据源中获取 Shiro 需要的验证信息。可以说，Realm 是专用于安全框架的 DAO. Shiro 的认证过程最终会交由 Realm 执行，这时会调用 Realm 的`getAuthenticationInfo(token)`方法。

该方法主要执行以下操作:

* 1、检查提交的进行认证的令牌信息
* 2、根据令牌信息从数据源(通常为数据库)中获取用户信息
* 3、对用户信息进行匹配验证。
* 4、验证通过将返回一个封装了用户信息的`AuthenticationInfo`实例。
* 5、验证失败则抛出`AuthenticationException`异常信息。

而在我们的应用程序中要做的就是自定义一个 Realm 类，继承AuthorizingRealm 抽象类，重载 doGetAuthenticationInfo()，重写获取用户信息的方法。

MyShiroRealm.java

```java
public class MyShiroRealm extends AuthorizingRealm {
    @Resource
    private UserService userService;

    /**
     * 权限校检
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    /**
     * 主要是用来进行身份认证的，也就是说验证用户输入的账号和密码是否正确。
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken)
            throws AuthenticationException {
        /*
         * 1. 从Token中获取输入的用户名密码
         * 2. 通过输入的用户名查询数据库得到密码
         * 3. 调用Authentication进行密码校验
         */
        // 获取用户名和密码
        String username = (String) authenticationToken.getPrincipal();
        String password = new String((char[]) authenticationToken.getCredentials());

        User user = userService.findByUsername(username);
        if (user == null) {
            throw new UnknownAccountException();
        }
        if (!password.equals(user.getPassword())) {
            throw new IncorrectCredentialsException();
        }
        return new SimpleAuthenticationInfo(user, password, getName());
    }
}
```

* `AuthorizationInfo`用于权限校验
* `AuthenticationInfo`用于身份验证

以上，shiro配置便完成了

## 测试

> 创建`LoginController.java`

```java
@Controller
public class LoginController {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    /**
     * 首页
     */
    @GetMapping(value = {"/", "/index"})
    public String index() {
        return "index";
    }

    /**
     * 登录地址
     */
    @GetMapping("/login")
    public String login() {
        return "login";
    }

    /**
     * 登录接口
     *
     * @return 状态信息，或成功页面视图
     */
    @PostMapping("/login")
    public String login(String username, String password, Model model) {
        String info = null;

        // 封装Token信息 = 用户名+密码
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
        // shiro subject 实例
        Subject subject = SecurityUtils.getSubject();
        try {
            subject.login(token);
            info = String.valueOf(subject.isAuthenticated());
            model.addAttribute("info", "登录状态 ==>" + info);
            return "/index";
        } catch (UnknownAccountException e) {
            e.printStackTrace();
            info = "未知账户异常";
        } catch (AuthenticationException e) {
            e.printStackTrace();
            info = "账户名密码错误";
        } catch (Exception e) {
            e.printStackTrace();
            info = "其他异常";
        }
        model.addAttribute("info", "登录状态 ==>" + info);
        logger.info("登录状态 ==>{}" + info);
        return "login";
    }
}
```

* `@Controller`用来告诉Spring这是个处理HTTP请求的控制器。
* `@RestController`是`@ResponseBody`和`@Controller`的组合，被标记的控制器类所有`return`数据都自动封装为JSON格式。
* `@GetMapping`标记该请求是Get请求，如果用Post请求则会报错no support

对于出现的两个`login`接口，`@GetMapping`和`@PostMapping`利用Java的方法重载创建了两个名称相同的接口，但是根据HTTP请求方法的不同（Get还是Post）会自动寻找对应的映射方法。

> 登录页面

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>login</title>
</head>
<body>

<h1>登录</h1>

<form method="post" action="/login">
    <label>
        <input type="text" name="username">
    </label> <br>
    <label>
        <input type="password" name="password">
    </label> <br>
    <input type="submit" value="登录">
</form>

<div style="color: blue;" th:text="${info}"></div>
</body>
</html>
```

如上，前台form表单中的`action="/login"`和`method="post"`决定了请求走这个地址，通过调用`subject.login(token)`，Shiro自动查询Realm实现，于是找到我们自定义的Realm实现：`AuthRealm`，进而通过`SimpleAuthenticationInfo`方法验证了登录用户的身份，如果身份认证成功，就`return "/index"`，否则就`return "/login"`。



## 参考

* [纯洁的微笑 - Spring Boot (十四)： Spring Boot 整合 Shiro-登录认证和权限管理](<http://www.ityouknow.com/springboot/2017/06/26/spring-boot-shiro.html>)
* [TyCoding](<https://github.com/TyCoding/spring-learn/tree/master/boot-shiro>)