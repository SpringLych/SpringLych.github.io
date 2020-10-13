---
title: Spring Boot整合Thymeleaf
date: 2019-05-27 14:04:53
tags: [Spring Boot]
categories: Spring Boot
---

Thymeleaf是一个跟freemarker等类似的模板引擎，能够替代JSP，它也是springboot的官方推荐方案。

<!--more-->

但是我一般倾向于使用前后端分离的模式，前端使用Vus.js 等框架，这里学习Thymeleaf主要是快速实现一些网页，以便继续学习Spring Boot中需要用到网页的部分。

## 开始

使用IDEA新建Spring Boot项目，依赖中选择 Core-Lombok，Web- web, Template Engines-Thymeleaf。

修改applicatiom.yml

```yaml
server:
  port: 8080

spring:
  thymeleaf:
    mode: LEGACYHTML5
    encoding: UTF-8
    servlet:
      content-type: text/html
    suffix: .html
    cache: false
```

其中`spring.thymeleaf.mode = LEGACYHTML5`配置thymeleaf的模式，不要使用`spring.thymeleaf.mode = HTML5`，因为严格遵循HTML5规范会对非严格的报错，例如`<meta charset="UTF-8">`，`<meta>`标签没有结束`<meta />`就会报错。

在 main/resources/templates新建`index.html`

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Index</title>
</head>
<body>
    <h1>This is index page</h1>
</body>
</html>
```

> 若想要在HTML页面中使用Thymeleaf，需要`<html lang="en" xmlns:th="http://www.thymeleaf.org">`

新建RouterController.java

```java
@Controller
public class RouterController {
    @GetMapping("/index")
    public String index(){
        return "index";
    }
}
```

这里的 `return “index”`中的index 是指sources/template中的 `index.html`文件。如果是`templates/common/main.html`页面，就应该`return "/common/main.html"`

浏览器访问`http://localhost:8080/`便可看到刚才的页面。

## Thymeleaf常用表达式

### 变量

> ${user.username}

修改RouterController.java

```java
@Controller
public class RouterController {
    @GetMapping("/")
    public String index(Model model){
        model.addAttribute("demo", "一个测试");
        return "index";
    }
}
```

修改index.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Index</title>
</head>
<body>
	<h1>This is index page</h1>
	<br>
	<p>表达式：<span th:text="${demo}"></span></p>
</body>
</html>
```



更多Thymeleaf相关可以参考[这里](<http://www.ityouknow.com/springboot/2016/05/01/spring-boot-thymeleaf.html>)

## 参考

* [TyCoding](<https://github.com/TyCoding/spring-learn/tree/master/boot-thymeleaf>)