# SpringBoot-1	初探



## 第一个Spring-Boot程序

​	详见：springboot-quickstart，将程序打包执行，不需要idea即可执行

​	项目创建之后：

![image-20231116160653899](assets\image-20231116160653899.png)



## 原理初探

自动配置：

pom.xml

- spring-boot-dependencies：核心依赖在父工程中
- 我们在写或者引入Springboot依赖的时候，不需要指定版本，就是因为有这些版本仓库

启动器:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter</artifactId>
</dependency>
```

启动器：说白了就是Springboot的启动场景；比如spring-boot-starter-web

里面包含了所有web有关的依赖

Springboot主函数

```java
package com.zmy.demo;


import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


//本身就是Spring的一个组件


@SpringBootApplication    //启动类下的所有资源被加载
public class DemoApplication {


    public static void main(String[] args) {

                //SpringApplication
        SpringApplication.run(DemoApplication.class, args);
    }


}
```

关于主函数中的@SpringBootApplication注解

Spring boot应用标注在某个类上说明这个类是Springboot的主配置类，Springboot就应该运行这个类的main方法来启动Springboot应用；

其中包含两个注解（子目录表示该注解下包含的注解）:

- @SpringBootConfiguration 表明这是Springboot的配置 标注在某个类上，表示这是一个Spring boot的配置类
  - @Configuration 表明这是spring的配置类
    - @Component 表明这是一个spring的组件
- @EnableAutoConfiguration 自动配置
  - @AutoConfigurationPackage 自动配置包
    - @Import({Registrar.class}) 自动配置‘包注册’，将主配置类（@SpringbootApplication）的所在包以及下面所有的子包里面的所有组件扫描到Spring容器
  - @Import({AutoConfigurationImportSelector.class}) 自动配置导入选择 导入哪些组件的选择器
    - List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes); 获取所有的配置 会给容器中导入非常多的自动配置类,有了自动配置类，免去了我们手动编写配置类

```java
//获取所有的配置
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {

    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());

    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");

    return configurations;
}
```

Spring boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置。

详见自动配置思维导图

总结：springboot所有自动配置都是在启动的时候扫描并加载：soring.factories所有的自动配置类都在这里面，但是不一定生效，要判断条件是否成立，只要导入了对应的start，就有对应的启动器了，有了启动器，我们自动装配就会生效，最终配置成功。

- springboot在启动的时候，从类路径下：/META-INF/spring.factories获取指定的值
- 将这些自动配置的类导入容器，自动配置就会生效，帮我进行自动配置
- 以前我们需要自动配置的东西，springboot帮我们做了
- 整合JavaEE，解决方案和自动配置的东西都在spring-boot-autoconfigure-2.2.0.RELEASE.jar包下
- 它会把所有需要导入的组件，以类名的方式返回，这些组件就会被添加到容器中

关于SpringBoot，谈谈你的理解：

- **自动装配原理(自行查看博客并且补充)**
- run（）方法



# SpringBoot-2	conf

官方推荐使用.yaml的配置文件(详见Spring-boot-config项目)

Person实体类

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;


import java.util.Date;
import java.util.List;
import java.util.Map;




@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birthday;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

application.yaml

```yaml
#格式 key: value，中间空格不能省
#对空格的要求十分高


#注入到我们的配置类中


#普通的key-value
name: zhoumouyuan


#对象
student:
  name: zhoumouyuan
  age: 21


#行内写法
student1: {name: zhoumouyuan,age: 3}


#数组
pets:
  - cat
  - dog
  - pig


pets1: [cat,dog,pig]




person:
  name: zhoumouyuan
  age: 21
  happy: false
  birth: 2019/11/02
  maps: {k1: v1,k2: v2}
  lists:
    - code
    - music
    - girl
  dog:
    name: 旺财
    age: 3
```

@ ConfigurationProperties的作用：将配置文件中配置的每一个属性的值，映射到这个组件中，告诉Springboot将本类中的所有属性和配置文件中相关的配置进行绑定，上述代码中 prefix = “person”：将配置文件中的person下面的所有属性一一对应



## JSR303校验

在类上方添加@Validated标签来进行数据校验

```java
@Component
@ConfigurationProperties(prefix = "person")


@Validated      //数据校验
public class Person {

    @Email()  //这里演示要求姓名为邮箱格式
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
    }
```

运行结果:

```
Description:


Binding to target org.springframework.boot.context.properties.bind.BindException: Failed to bind properties under 'person' to com.zmy.domian.Person failed:


    Property: person.name
    Value: zhoumouyuan
    Origin: class path resource [application.yaml]:27:9
    Reason: 不是一个合法的电子邮件地址




Action:


Update your application's configuration
```

JSR303校验常用注解

| @Null                     | 对象必须为null                                              |
| ------------------------- | ----------------------------------------------------------- |
| @NotNull                  | 对象必须不为null，无法检查长度为0的字符串                   |
| @NotBlank                 | 字符串必须不为Null，且去掉前后空格长度必须大于0             |
| @AssertTrue               | 对象必须为true                                              |
| @AssertFalse              | 对象必须为false                                             |
| @Max(Value)               | 必须为数字，且小于或等于Value                               |
| @Min(Value)               | 必须为数字，且大于或等于Value                               |
| @DecimalMax(Value)        | 必须为数字( BigDecimal )，且小于或等于Value。小数存在精度   |
| @DecimalMin(Value)        | 必须为数字( BigDecimal )，且大于或等于Value。小数存在精度   |
| @Digits(integer,fraction) | 必须为数字( BigDecimal )，integer整数精度，fraction小数精度 |
| @Size(min,max)            | 对象(Array、Collection、Map、String)长度必须在给定范围      |
| @Email                    | 字符串必须是合法邮件地址                                    |
| @Past                     | Date和Calendar对象必须在当前时间之前                        |
| @Future                   | Date和Calendar对象必须在当前时间之后                        |
| @Pattern(regexp=“正则”)   | String对象必须符合正则表达式                                |

配置文件的优先级顺序：(排在最上面的优先级最高，file是指项目下的，classpath是指类路径下的)

- file:./config/
- file:./
- classpath:/config/
- classpath:/

可以在一个yaml文件中这样做：

```yaml
server:
  port: 8081
spring:
  profiles:
    active: dev #引用下面的dev端口，没有此标签就是默认的8081端口
---
server:
  port: 8082
spring:
  profiles: dev #对此项配置命名
  
---
server:
  port: 8083
spring:
  profiles: test
```

这样相当于写了三个配置文件



# SpringBoot-3	web



## 静态资源导入

静态资源的存放默认一般存放在

![image-20231116161326990](assets\image-20231116161326990.png)

可以通过：localhost:8080/**直接访问静态资源

若是三个文件夹中有相同文件名的文件夹，则resources文件夹下面的文件优先级最高，static其次，public最低。

一般public文件夹下面放公共资源，static放图片，resources放上传的文件。0

总结：

- 在Springboot中我们可以使用以下方式处理静态资源

  - webjars     localhost:8080/webjars/

  - public，static，/**，resources  localhost:8080/

  - 优先级：resources>static(默认)>public

![image-20231116161436387](assets\image-20231116161436387.png)

//在template目录下的所有页面，只能通过controller来跳转

模板引擎thymeleaf(相当于JSP)

导入依赖：

```xml
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
</dependency>


<dependency>
<groupId>org.thymeleaf.extras</groupId>
<artifactId>thymeleaf-extras-java8time</artifactId>
</dependency>
```

这个相当于用Controller标签标注的类，里面返回页面是在template文件下面寻找.html文件。

```java
@Controller
public class IndexController {


    @RequestMapping("/test")
    public String index(){
        return "test";
    }
}
```

![image-20231116161616007](assets\image-20231116161616007.png)

在Spring boot中，有非常多的 xxxx Configuration帮助我们进行拓展配置，只要看见了这个配置，我们就要注意了

lombok:是一个能让你对实体类对象省去写get，set和tostring方法的插件，需要在pom.xml文件中导入如下依赖

```xml
<!--lombok-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

此外 还需要再IDEA的setting->plugins中下载lombok插件（光导入依赖是不可用的！）

以后的实体类编写：

```java
@Data
@AllArgsConstructor //有参构造
@NoArgsConstructor  //无参构造
public class Department  {
    private Integer id;
    private String departmentName;
}
```

项目

- 首页配置：注意点，所有页面的静态资源都需要使用thymeleaf接管

- 页面国际化

  ![image-20231116161723843](assets\image-20231116161723843.png)

自定义的LocaleResolver

```java
package com.zmy.config;


import org.springframework.web.servlet.LocaleResolver;
import org.thymeleaf.util.StringUtils;


import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;


public class MyLocaleResolver implements LocaleResolver {


    //解析请求
    @Override
    public Locale resolveLocale(HttpServletRequest httpServletRequest) {
        String language = httpServletRequest.getParameter("l");
        Locale locale = Locale.getDefault(); //如果没有就是用默认的


        //如果携带了国际化参数
        if (!StringUtils.isEmpty(language)){
            //zh_CN
            String[] s = language.split("_");
            locale =  new Locale(s[0],s[1]);
        }
        return locale;
    }


    @Override
    public void setLocale(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Locale locale) {


    }
}
```

之后要注入Spring的容器中

```java
package com.zmy.config;


        import org.springframework.context.annotation.Bean;
        import org.springframework.context.annotation.Configuration;
        import org.springframework.web.servlet.LocaleResolver;
        import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
        import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;


        import java.util.Locale;


@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
        registry.addViewController("/index.html").setViewName("index");
    }


    @Bean
    public LocaleResolver localeResolver(){
        return new MyLocaleResolver();
    }
}
```



# SpringBoot-4 	data



## Springboot整合JDBC

写一个JDBCController

```java
package com.zmy.controller;


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


import java.util.List;
import java.util.Map;


@RestController
public class JDBCController {


    @Autowired
    JdbcTemplate template;


    @GetMapping("/userList")
    public List<Map<String,Object>> userList(){
        String sql = "select * from user";
        List<Map<String, Object>> maps = template.queryForList(sql);
        return maps;
    }


    @GetMapping("/addUser")
    public String addUser(){
        String sql = "insert into user value(null,'123456789','987654321','男','1140267690@qq.com','15973226458','nothing',null,1,'客户',null)";
        template.update(sql);
        return "update-ok";
    }


    @GetMapping("/deleteUser")
    public String deleteUser(){
        String sql = "delete from user where id = 13";
        template.update(sql);
        return "update-ok";
    }


    @GetMapping("/updateUser")
    public String updateUser(){
        String sql = "update user set gender = '女' where id = 1";
        template.update(sql);
        return "update-ok";
    }
}
```

在application.yml中配置：

```yaml
spring:
  datasource:
    username: root
    password: 698356
    url: jdbc:mysql://localhost:3306/bookstore?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
```

记得在pom.xml文件中导入相关依赖



## Springboot整合Druid

编写一个Druid的配置类，并在其中注册一个servlet

```java
	package com.zmy.config;


import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


import javax.sql.DataSource;
import java.util.HashMap;


@Configuration
public class DruidConfig {


    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druidDataSource(){
        return  new DruidDataSource();
    }


    //后台监控 web.xml
    /*
    * springboot内置了servlet容器，所以没有web.xml，替代方法就是ServletRegistrationBean ,然后记得注入bean
    * */
    @Bean
    public ServletRegistrationBean StatViewServlet(){
        ServletRegistrationBean<StatViewServlet> bean = new ServletRegistrationBean<>(new StatViewServlet(),"/druid/*");


        //后台需要有人登录，账号密码
        HashMap<String, String> initParams = new HashMap<>();


        //增加配置
        initParams.put("loginUsername", "root");    //此处的key值是固定的
        initParams.put("loginPassword", "698356");  //此处的key值是固定的


        //允许谁可以访问
        initParams.put("allow", "");


        //禁止谁能访问
        //initParams.put("kuangshen", "192.168.1.112");


        bean.setInitParameters(initParams);//设置初始化参数


        return bean;




    }


}
```

更改一下application.yml中的配置

```yaml
spring:
  datasource:
    username: root
    password: 698356
    url: jdbc:mysql://localhost:3306/bookstore?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource


    #Spring Boot 默认是不注入这些属性值的，需要自己绑定
    #druid 数据源专有配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
    #如果允许时报错  java.lang.ClassNotFoundException: org.apache.log4j.Priority
    #则导入 log4j 依赖即可，Maven 地址：https://mvnrepository.com/artifact/log4j/log4j
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500


server:
  port: 8080
```

同样的，filter也和servlet一样在配置类中注册

```java
//filter 过滤器
public FilterRegistrationBean webStatFilter(){
    FilterRegistrationBean bean = new FilterRegistrationBean();


    HashMap<String, String> initParams = new HashMap<>();


    //不过滤什么
    initParams.put("exclusions", "*.js,*.css,/druid/*");
    
    bean.setInitParameters(initParams);


    return bean;
}
```



## Springboot整合mybatis

首先一定要导入mybatis整合spring的依赖

```xml
<dependency>
    <!--mybatis整合-->
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
```

编写mapper接口

```java
package com.zmy.mapper;


import com.zmy.domain.User;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Repository;


import java.util.List;


//这个注解表示这是mybatis的mapepr类
//或者在项目的main方法所在类上加入注解MapperScan("com.zmy.mapper")
@Mapper
@Repository
public interface UserMapper {


    List<User> queryUserList();


    User queryById(int id);


    int addUser(User user);


    int updateUser(User user);


    int deleteUser(int id);


}
```

编写mapper的xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zmy.mapper.UserMapper">


    <select id="queryUserList" resultType="User">SELECT * from USER </select>


</mapper>
```

注意：一定要在application.yml中整合mybatis

```yaml
# 整合mybatis
mybatis:
  type-aliases-package: com.zmy.domain
  mapper-locations: classpath:mybatis/mapper/*.xml
```



## SpringBoot整合Redis

说明：在Springboot2.x之后，原来使用的Jedis替换为了lettuce

jedis：采用的是直接连接，多个线程操作的话，是不安全的，如果想要避免不安全就要使用你jedis pool连接池 BIO

lettuce：采用netty，实例可以再多个线程中进行共享，不存在线程不安全的情况，可以减少线程使用

导入依赖：

![image-20231116162525069](assets\image-20231116162525069.png)

编写配置文件：

![image-20231116162546654](assets\image-20231116162546654.png)

一些简单的操作：

![image-20231116162607794](assets\image-20231116162607794.png)

建一个实体类User

![image-20231116162631393](assets\image-20231116162631393.png)

然后新建一个User对象传入redis中

![image-20231116162645061](assets\image-20231116162645061.png)

此处我们直接将user传进去之后，点击运行。

![image-20231116162702224](assets\image-20231116162702224.png)

此时会发现报了一个未序列化的异常，这就说明我们的实体类需要序列化。

![image-20231116162730722](assets\image-20231116162730722.png)

序列化之后的输出：

![image-20231116162751160](assets\image-20231116162751160.png)

将对象转换为Json字符串之后传入Redis之后的输出

![image-20231116162809298](assets\image-20231116162809298.png)

编写自己的RedisTemplate

固定模板

```java
package com.zmy.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.net.UnknownHostException;

@Configuration
public class RedisConfig {

    //编写自己的RedisTemplate,此处是String和Object的redisTemplate，默认的是两个Object
    @Bean
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException{
        RedisTemplate<String,Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);

        //json序列化配置
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        ObjectMapper mapper = new ObjectMapper();
        mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        mapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        serializer.setObjectMapper(mapper);

        //String的序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();


        //配置具体的序列化方式
        //key采用string的序列化
        template.setKeySerializer(stringRedisSerializer);
        //哈希的key也采用String的序列化
        template.setHashKeySerializer(stringRedisSerializer);
        //value采用jackson的序列化
        template.setValueSerializer(serializer);
        //hash的value也采用jackson的序列化
        template.setHashValueSerializer(serializer);
        template.afterPropertiesSet();

        return template;
    }


}

```



# SpringBoot-5	security

首先在Pom文件中导包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

之后导入相应的静态资源

然后编写一个安全配置类

```java
package com.zmy.config;


import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;


@EnableWebSecurity //一定注意这个标签//与shiro的configuration不一样
public class SecurityConfig extends WebSecurityConfigurerAdapter {


    //授权
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //需求：首页所有人可以访问，但是里面的功能页只能对应有权限的人才能访问
        http.authorizeRequests()
                .antMatchers("/").permitAll()//首页所有人都能访问
                .antMatchers("/level1/*").hasRole("vip1")//level1只能由vip1访问
                .antMatchers("/level2/*").hasRole("vip2")
                .antMatchers("/level3/*").hasRole("vip3");
        http.formLogin().loginPage("/toLogin");//把自带的登录页面换成自己的
        http.formLogin();//没有权限默认会跳到登陆页面




        http.csrf().disable();//关闭csrf功能 解决注销不成功的问题
        http.logout().logoutSuccessUrl("/toLogin");//注销,注销之后会跳到首页
        /*
        * .deleteCookies() 移除所有Cookies
        * .invalidateHttpSession() 移除所有session
        * .logoutSuccessUr("/") 注销成功跳转首页
        * .logoutUrl
        * */


        http.rememberMe().rememberMeParameter("remember");//开启记住我功能 本质cookie






    }


    //认证
    //密码编码错误:PasswordEncoder
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //这些数据应该从数据库里面读
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                .withUser("root").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2","vip3")
        .and()
        .withUser("zmy").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1");


    }
}
```

首页的一些配置index.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <title>首页</title>
    <!--semantic-ui-->
    <link href="https://cdn.bootcss.com/semantic-ui/2.4.1/semantic.min.css" rel="stylesheet">
    <link th:href="@{/qinjiang/css/qinstyle.css}" rel="stylesheet">
</head>
<body>


<!--主容器-->
<div class="ui container">


    <div class="ui segment" id="index-header-nav" th:fragment="nav-menu">
        <div class="ui secondary menu">
            <a class="item"  th:href="@{/index}">首页</a>


            <!--登录注销-->
            <div class="right menu">


                <!--如果未登录显示登录按钮，如果登录就显示用户名和注销按钮-->
                <div sec:authorize="!isAuthenticated()">
                    <!--未登录-->
                    <a class="item" th:href="@{/toLogin}">
                        <i class="address card icon"></i> 登录
                    </a>
                </div>
                <div sec:authorize="isAuthenticated()">
                    <!--一登陆-->
                    <a class="item">
                        用户名：<span sec:authentication="name"></span>


                    </a>
                    <a class="item" th:href="@{/logout}">
                        <i class="address card icon"></i> 注销
                    </a>
                </div>


                <!--已登录
                <a th:href="@{/usr/toUserCenter}">
                    <i class="address card icon"></i> admin
                </a>
                -->
            </div>
        </div>
    </div>


    <div class="ui segment" style="text-align: center">
        <h3>Spring Security Study by 秦疆</h3>
    </div>


    <div>
        

        <div class="ui three column stackable grid">
            <div class="column" sec:authorize="hasRole('vip1')">
                <div class="ui raised segment">
                    <div class="ui">
                        <div class="content">
                            <h5 class="content">Level 1</h5>
                            <hr>
                            <div><a th:href="@{/level1/1}"><i class="bullhorn icon"></i> Level-1-1</a></div>
                            <div><a th:href="@{/level1/2}"><i class="bullhorn icon"></i> Level-1-2</a></div>
                            <div><a th:href="@{/level1/3}"><i class="bullhorn icon"></i> Level-1-3</a></div>
                        </div>
                    </div>
                </div>
            </div>


            <div class="column"  sec:authorize="hasRole('vip2')">
                <div class="ui raised segment">
                    <div class="ui">
                        <div class="content">
                            <h5 class="content">Level 2</h5>
                            <hr>
                            <div><a th:href="@{/level2/1}"><i class="bullhorn icon"></i> Level-2-1</a></div>
                            <div><a th:href="@{/level2/2}"><i class="bullhorn icon"></i> Level-2-2</a></div>
                            <div><a th:href="@{/level2/3}"><i class="bullhorn icon"></i> Level-2-3</a></div>
                        </div>
                    </div>
                </div>
            </div>


            <div class="column"  sec:authorize="hasRole('vip3')">
                <div class="ui raised segment">
                    <div class="ui">
                        <div class="content">
                            <h5 class="content">Level 3</h5>
                            <hr>
                            <div><a th:href="@{/level3/1}"><i class="bullhorn icon"></i> Level-3-1</a></div>
                            <div><a th:href="@{/level3/2}"><i class="bullhorn icon"></i> Level-3-2</a></div>
                            <div><a th:href="@{/level3/3}"><i class="bullhorn icon"></i> Level-3-3</a></div>
                        </div>
                    </div>
                </div>
            </div>


        </div>
    </div>
    
</div>




<script th:src="@{/qinjiang/js/jquery-3.1.1.min.js}"></script>
<script th:src="@{/qinjiang/js/semantic.min.js}"></script>


</body>
</html>
```

登陆页面的一些配置

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <title>登录</title>
    <!--semantic-ui-->
    <link href="https://cdn.bootcss.com/semantic-ui/2.4.1/semantic.min.css" rel="stylesheet">
</head>
<body>


<!--主容器-->
<div class="ui container">


    <div class="ui segment">


        <div style="text-align: center">
            <h1 class="header">登录</h1>
        </div>


        <div class="ui placeholder segment">
            <div class="ui column very relaxed stackable grid">
                <div class="column">
                    <div class="ui form">
                        <form th:action="@{/toLogin}" method="post">
                            <div class="field">
                                <label>Username</label>
                                <div class="ui left icon input">
                                    <input type="text" placeholder="Username" name="username">
                                    <i class="user icon"></i>
                                </div>
                            </div>
                            <div class="field">
                                <label>Password</label>
                                <div class="ui left icon input">
                                    <input type="password" name="password">
                                    <i class="lock icon"></i>
                                </div>
                            </div>
                            <input type="submit" class="ui blue submit button"/>
                        </form>
                    </div>
                </div>
            </div>
        </div>


        <div style="text-align: center">
            <div class="ui label">
                </i>注册
            </div>
            

            <small>blog.kuangstudy.com</small>
        </div>
        <div class="ui segment" style="text-align: center">
            <h3>Spring Security Study by 秦疆</h3>
        </div>
    </div>




</div>


<script th:src="@{/qinjiang/js/jquery-3.1.1.min.js}"></script>
<script th:src="@{/qinjiang/js/semantic.min.js}"></script>


</body>
</html>
```



# SpringBoot-6	任务



## 异步任务

描述：当Service层出现耗时的业务逻辑时，前端界面会停止工作等待底层执行成功

如下：

![image-20231116163018935](assets\image-20231116163018935.png)

此时前端页面访问需要等待三秒才能显示OK。

解决办法，此时我们可以在业务层的hello方法上加上注释@Async,告诉spring这是一个异步方法

![image-20231116163039459](assets\image-20231116163039459.png)

并且要记得在启动类上添加开启异步任务的注解

![image-20231116163056223](assets\image-20231116163056223.png)



## 邮件任务

首先在pom文件中导入依赖

```xml
<dependency>   
    <groupId>org.springframework.boot</groupId>   
    <artifactId>spring-boot-starter-mail</artifactId> 
</dependency>
```

然后在QQ邮箱的设置中开启：（在这里顺带把自己的网易也开了）

![image-20231116163200779](assets\image-20231116163200779.png)

然后会给你一个授权码：acogdebnbgumggdh

然后去spring的配置文件中配置properties

![image-20231116163215281](assets\image-20231116163215281.png)

最后在测试类中测试：

![image-20231116163231245](assets\image-20231116163231245.png)

![image-20231116163254523](assets\image-20231116163254523.png)

可以把上述代码进行封装



## 定时任务

在启动类中开启定时任务的注解

![image-20231116163327539](assets\image-20231116163327539.png)	

编写一个service，定时执行一个方法

![image-20231116163343432](assets\image-20231116163343432.png)

关于cron表达式，可以根据需求去百度上面生成一个

![image-20231116163407181](assets\image-20231116163407181.png)

