# Swagger简介

Swagger就是一个API文档工具



# Swagger快速开始

1. 新建一个spring boot项目
2. 导入相关依赖

```xml
<dependency>    
    <groupId>io.springfox</groupId>     
    <artifactId>springfox-swagger2</artifactId>     
    <version>2.9.2</version> 
</dependency> 
<dependency> 
    <groupId>io.springfox</groupId> 
    <artifactId>springfox-swagger-ui</artifactId>     
    <version>2.9.2</version> 
</dependency>
```

编写一个helloword项目

![image-20231116155753371](assets\image-20231116155753371.png)

然后编写Swagger的配置类

![image-20231116155814285](assets\image-20231116155814285.png)

在网页访问：http://localhost:8080/swagger-ui.html

![image-20231116155834951](assets\image-20231116155834951.png)



## 配置Swagger

首先，在Swagger的配置文件中配置swagger的docket的bean实例

![image-20231116155901171](assets\image-20231116155901171.png)

然后根据源码中的默认Docket修改配置

![image-20231116155917031](assets\image-20231116155917031.png)

首先修改的是apiInfo，可以看到Docket对象有设置apiinfo的方法，我们只需要传一个apiInfo的实例对象进去,在配置文件中编写如下函数：

![image-20231116155931613](assets\image-20231116155931613.png)

重新编译项目后的ui网页显示：

![image-20231116155952504](assets\image-20231116155952504.png)



## Swagger配置扫描接口

Docket.select();



### 修改Docket的生成函数：

![image-20231116160025307](assets\image-20231116160025307.png)

注意：select之后就只有apis和paths两个函数，要进行其他操作就要在select函数之前。



### 配置是否启动Swagger：

![image-20231116160106140](assets\image-20231116160106140.png)

如何让Swagger在生产环境中使用，在发布的时候不使用？

- 判断是不是生产环境 flag=false
- 注入enable()

![image-20231116160124535](assets\image-20231116160124535.png)



### 分组以及接口注释

​	关于分组:

​	![image-20231116160202670](assets\image-20231116160202670.png)



### 接口注释

Model：

![image-20231116160305916](assets\image-20231116160305916.png)

![image-20231116160316614](assets\image-20231116160316614.png)

方法注解和参数注解：

![image-20231116160335219](assets\image-20231116160335219.png)

以下文件可直接使用:

```java
package com.zmy.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.core.env.Profiles;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;

@Configuration
@EnableSwagger2  //开启Swagger2
public class SwaggerConfig {

    //配置了swagger的docket的bean实例
    @Bean
    public Docket docket(Environment environment){
        //设置要显示的Swagger环境
        Profiles profiles = Profiles.of("dev","test");
        //获取项目环境：通过environment.acceptsProfiles判断是否存储在自己设定的环境当中
        boolean flag = environment.acceptsProfiles(profiles);


        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .enable(flag)//配置是否开启
                .groupName("周谋远")//配置组名，若有多个组 再去配多个docket
                .select()
                //RequestHandlerSelectors:配置要扫描接口的方式，然后指定要扫描的包
                /*RequestHandlerSelectors.any()：扫描全部
                  RequestHandlerSelectors.none()：不扫描
                  RequestHandlerSelectors.withClassAnnotation()：扫描类上的注解
                  RequestHandlerSelectors.withMethodAnnotation()：扫描方法上的注解
                  一般都是用basepackage
                * */
                //.apis(RequestHandlerSelectors.basePackage("com.zmy.controller"))
                /*paths：过滤路径
                * */
                //.paths(PathSelectors.ant("/zmy/**"))
                .build();   //build
    }

    //配置Swagger信息=apiInfo
    private ApiInfo apiInfo(){
        return  new ApiInfo("周某的Swagger日记",
                "周某的项目API信息",
                "1.0",
                "https://baidu.com",
                new Contact("周谋远","" ,"1140267690@qq.com" ),  //作者信息,url可填自己的博客地址什么的
                "Apache 2.0",
                "",
                new ArrayList<>()
                );
    }

}

```

