---
title: Spring Boot与MyBatis-Plus进阶使用
date: 2019-08-07 20:14:21
tags: Spring Boot
categories: [Spring Boot]
---

几个月前整合完了MBP，用的比较少，所以MBP的高级功能没有详细体验。今天发先MBP有很多强大的功能，可以少写许多代码，就体验了一下。

<!--more-->

## 自动生成代码

不同于mybatis的生成代码，mybatis-plus可以生成controller等代码。AutoGenerator 是 MyBatis-Plus 的代码生成器，通过 AutoGenerator 可以快速生成 Entity、Mapper、Mapper XML、Service、Controller 等各个模块的代码，极大的提升了开发效率。

额外添加依赖：

```xml
<!--        代码生成器依赖-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.1.2</version>
</dependency>
<!--        mybatis-plus代码生成器默认模板-->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.1</version>
</dependency>
```

### 编写SQL

```mysql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` bigint(20) DEFAULT NULL COMMENT '唯一标示',
  `code` varchar(20) DEFAULT NULL COMMENT '编码',
  `name` varchar(64) DEFAULT NULL COMMENT '名称',
  `status` char(1) DEFAULT '1' COMMENT '状态 1启用 0 停用',
  `gmt_create` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 编写配置

```java
public class MysqlGenerator {

    private static final String PACKAGE_NAME = "com.example";

    private static final String OUT_PATH = "/src/main/java";

    private static final String AUTHOR = "spring";

    private static final String MODULE_NAME = "demo";

    /**
     * 数据库相关连接
     */
    private static final String URL = "jdbc:mysql://[]:3306/springboot_mybatis?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false";

    private static final String USERNAME = "";

    private static final String PASSWORD = "";

    private static final String DRIVER = "com.mysql.cj.jdbc.Driver";

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator generator = new AutoGenerator();

        // 全局配置
        String projectPath = System.getProperty("user.dir");
        generator.setGlobalConfig(new GlobalConfig()
                // 输出目录
                .setOutputDir(projectPath + OUT_PATH)
                // 覆盖文件
                .setFileOverride(true)
                .setOpen(false)
                .setAuthor(AUTHOR)
                .setXmlName("%sMapper")
                // 自定义文件命名，注意 %s 会自动填充表实体属性！
                .setMapperName("%sDao")
                .setServiceName("%Service")
                // 开启resultMap
                .setBaseResultMap(true)
                // 开启BaseColumnList
                .setBaseColumnList(true)
        );

        // 数据源配置
        generator.setDataSource(new DataSourceConfig()
                .setUrl(URL)
                .setUsername(USERNAME)
                .setPassword(PASSWORD)
                .setDriverName(DRIVER)
        );

        //包配置
        generator.setPackageInfo(new PackageConfig()
                .setModuleName(MODULE_NAME)
                .setParent(PACKAGE_NAME)
                .setMapper("dao")
        );


        // 如果模板引擎是 velocity
        String templatePath = "/templates/mapper.xml.vm";

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {

            }
        };

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 配置xxxMapper.xml路径和文件名
                return projectPath + "/src/main/resources/mapper/"
                        + "/" + tableInfo.getEntityName() + "Mapper.xml";
            }
        });

        cfg.setFileOutConfigList(focList);
        generator.setCfg(cfg);

        // 配置模板
        generator.setTemplate(new TemplateConfig()
                .setXml(null)
                // 禁止生成Controller
                //.setController(null)
        );

        // 策略配置
        generator.setStrategy(new StrategyConfig()
                .setNaming(NamingStrategy.underline_to_camel)
                .setColumnNaming(NamingStrategy.underline_to_camel)
                // public User setName(String name) {this.name = name; return this;}
                .setEntityColumnConstant(true)
                .setEntityLombokModel(true)
                .setControllerMappingHyphenStyle(true)
        );

        generator.setTemplateEngine(new VelocityTemplateEngine());

        // 执行
        generator.execute();
    }
}
```

![1565161529519](assets/1565161529519.png)

生成后运行如图所示。第一次编写比较麻烦，但是编写完成后可以存入到IDEA的模板中，下次直接生成`MysqlGenerate.java`模板，只需做些小小的修改即可使用。

官网[代码生成器指南](https://mp.baomidou.com/guide/generator.html#代码生成器)

## 通用的CRUD

配置MyBatis-Plus

```java
@Configuration
@MapperScan("com.example.demo.dao")
public class MyBatisPlusConfig {
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

对于常用的CRUD，不用自己写xml，因为MyBatis-Plus内置了一些常用操作。例如：

```
@Test
public void insert() {
	User user = new User();
	user.setName("Jon");
	user.setCode("009");
	user.setStatus("0");
	int res = userDao.insert(user);
	log.info("--插入 {} 条", res);
}
```

批量插入。属于Service CRUD的接口

```java
@Test
public void saveMany() {
    int cap = 15;
    List<User> users = new ArrayList<>(cap);
    for (int i = 0; i < cap; ++i) {
        User user = new User();
        user.setName("Jon + " + i);
        user.setStatus("A");
        user.setCode("007" + i);
        users.add(user);
    }
    userService.saveBatch(users);
    log.info("-----批量插入--------");
}
```

更多使用：[CRUD接口](https://mp.baomidou.com/guide/crud-interface.html#mapper-crud-接口)

## 分页插件

在MyBatisPlusConfig中配置如下：

```java
/**
     * 分页插件
     */
@Bean
public PaginationInterceptor paginationInterceptor() {
    // paginationInterceptor.setLimit(你的最大单页限制数量，默认 500 条，小于 0 如 -1 不受限制);
    return new PaginationInterceptor();
}
```

dao层

```java
public interface UserDao extends BaseMapper<User> {
    /**
     * <p>
     * 查询 : 根据state状态查询用户列表，分页显示
     * 注意!!: 如果入参是有多个,需要加注解指定参数名才能在xml中取值
     * </p>
     *
     * @param page   分页对象,xml中可以从里面进行取值,传递参数 Page 即自动分页,必须放在第一位(你可以继承Page实现自己的分页对象)
     * @param status 状态
     * @return 分页对象
     */
    IPage<User> selectPage(Page page, @Param("status") String status);
}
```

mapper编写

```xml
<!--分页查询-->
<select id="selectPage" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List"/>
    from user
</select>
```

测试

```java
/**
     * 分页查询测试
     */
@Test
public void findPage() {
    Page<User> page = new Page<>(1, 3);
        // 当前页码 每页数
        Page<User> page = new Page<>(1, 3);
        IPage<User> userIPage = userDao.selectPage(page, "A");
        log.info(userIPage.toString());
        log.info("---总条数：{} ---", userIPage.getTotal());
        log.info("---当前页：{} ---", userIPage.getCurrent());
        log.info("---总页码：{} ---", userIPage.getPages());
        log.info("---每页多少条：{} ---", userIPage.getSize());

        List<User> users = userIPage.getRecords();
        users.forEach(user -> log.info(user.toString()));
}
```



## 参考

* [官网](<https://mp.baomidou.com/>)
* [Mybatis-Plus使用全解](https://blog.lqdev.cn/2018/08/06/日常积累/mybatis-plus-guide-one/)