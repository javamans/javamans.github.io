---
title: 7.springboot整合swagger2
tags: springboot
---

swagger2是一个可以根据接口定义自动生成接口API文档的框架

#### jar包导入

```java
<!--集成swagger2-->
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

#### 代码配置

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .pathMapping("/")
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.gaochaojin.controller"))
                .paths(PathSelectors.any())
                .build()
                .apiInfo(new ApiInfoBuilder()
                .title("架构师学习网站")
                .description("springboot文档")
                .version("1.0")
                .contact(new Contact("gaochaojin","www.javamans.com","javamans@126.com"))
                .build());
    }
}
```

#### 代码中使用

```java
@Controller
@Api(tags = "springboot学习工程相关接口")
public class DemoController {

    private static final Logger logger = LoggerFactory.getLogger(DemoController.class);

    @Value("${application.field:default value gaochaojin}")
    private String textMessage;

    @ApiOperation("jsp测试接口")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "username", value = "用户名", defaultValue = "jack"),
            @ApiImplicitParam(name = "address", value = "地址", defaultValue = "贵阳")})
    @RequestMapping("/index")
    public ModelAndView index() {
        ModelAndView mv = new ModelAndView();
        mv.setViewName("index");
        mv.addObject("time", new Date());
        mv.addObject("message", textMessage);
        return mv;
    }
}
```

#### ui界面

界面请求url： http://localhost:8080/swagger-ui.html 

