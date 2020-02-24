---
title: 2.springboot整合mybatis
tags: springboot
---

mybatis是一个非常优秀的ORM框架，使用springboot整合mybatis

#### jar包导入

```java
<!-- 把mybatis的启动器引入 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>RELEASE</version>
</dependency>
```

#### application.properties配置

```properties
# mybatis配置
# 把该包下的bean生成别名
mybatis.type-aliases-package=com.gaochaojin.model
# mybaits会把这个路径下的xml解析出来建立接口的映射关系
mybatis.mapper-locations=classpath:xml/*Mapper.xml
```

#### 启动了添加扫描注解

```java
@SpringBootApplication(scanBasePackages = {"com.gaochaojin"})
@MapperScan("com.gaochaojin.dao")
public class SpringbootWebApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootWebApplication.class, args);
    }

}
```

@MapperScan注解回扫描到包下的接口，把接口生成代理对象并加入到spring容器中 ，在业务代码中按照类型注入就可以使用了，如下代码

```java
@Service
public class CommonServiceImpl implements CommonService {

    @Autowired
    private UsersMapper usersMapper;
    
}
```

