---
layout: article
titles:
  # @start locale config
  zh-Hans : &ZH_HANS  Spring Cloud
  # @end locale config
key: page-about
sidebar:
  nav: springcloud
---

## Spring Cloud

微服务工程的特点：

- 扩展灵活
- 每个应用都规模不大
- 服务边界清晰，各司其职
- 打包应用很多，需要借助持续集成工具

springcloud主要从一下两方面进行：

- springcloud组件的使用

  springcloud工程是基于springboot工程的，依赖的版本为：

  ```java
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.2.2.RELEASE</version>
      <relativePath/> <!-- lookup parent from repository -->
  </parent>
  ```

  springcloud的版本为：

  ```java
  <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <maven.compiler.source>1.7</maven.compiler.source>
      <maven.compiler.target>1.7</maven.compiler.target>
      <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
  </properties>
  ```

  springcloud的依赖为：

  ```java
  <dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${spring-cloud.version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
  </dependencyManagement>
  ```

- springcloud源码分析