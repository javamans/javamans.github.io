---
title: 1.springcloud注册中心eureka的搭建
tags: springcloud
---

### 注册中心eureka搭建

#### eureka服务端启动器导入

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

#### application.properties配置文件

```properties
server.port=8763
eureka.instance.hostname=localhost

# 是否注册到eureka
eureka.client.register-with-eureka=false
# 是否从eureka中拉取注册信息
eureka.client.fetch-registry=false
# 暴露eureka服务的地址
eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
```

#### 启动类代码

```java
@SpringBootApplication
// 开启EurekaServer服务注册功能
@EnableEurekaServer
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class);
    }
}
```

#### 注册中心地址

[http://localhost:8763](http://localhost:8763)

### 服务提供方

服务提供方需要将服务注册到eureka服务端，所以服务提供方就是eureka的客户端。

#### eureka客户端启动器导入

```java
<!--集成eureka客户端-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### bootstrap.properties配置文件

```properties
spring.application.name=micro-order
server.port=8084

# Eureka客户端设置用户名和密码
eureka.client.serviceUrl.defaultZone=http://localhost:8763/eureka/
```

#### 启动类代码

```java
@SpringBootApplication
// 开启eureka客户端功能
@EnableEurekaClient
public class MicroOrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(MicroOrderApplication.class);
    }
}
```

### 服务消费方

消费方负责调用服务提供方，需要调用客户端，配置和服务提供方一样

#### 启动类代码

```java
@SpringBootApplication
@EnableEurekaClient
public class MicroWebApplication {

    @Bean
    // 负载均衡注解
    @LoadBalanced
    RestTemplate restTemplate(){
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(MicroWebApplication.class);
    }
}
```

服务提供方和服务调用方启动的时候都会往服务注册中心注册服务，服务调用的时候就根据服务提供方的服务名称来调用，再加上服务提供方的接口名就可以调用了。

```java
@Slf4j
@Service
public class UserServiceImpl implements UserService {

    private static String SERVICE_NAME = "micro-order";

    @Autowired
    private RestTemplate restTemplate;

    @Override
    public String queryContents() {
        String result = restTemplate.getForObject("http://" + SERVICE_NAME + "/queryContent", String.class);
        return result;
    }
}
```

### eureka用户认证

连接到eureka时需要带上连接的用户名和密码

#### eureka服务端中添加依赖

```java
<dependency>           
    <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

#### application.properties配置文件增加

```properties
# 开启basic校验，设置登录用户名和密码
security.basic.enabled=true
spring.security.user.name=admin
spring.security.user.password=admin
```

#### 代码配置关闭cesf验证

```java
@EnableWebSecurity
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 关闭csrf
        http.csrf().disable();
        // 开启认证：URL格式登录必须是httpBasic
        http.authorizeRequests().anyRequest().authenticated().and().httpBasic();
    }
}
```

#### eureka服务端改造（服务提供方和服务消费方）

需要在bootstrap.properties配置文件中修改配置如下：

```properties
# Eureka客户端设置用户名和密码
eureka.client.serviceUrl.defaultZone=http://admin:admin@localhost:8763/eureka/
```

### 服务续约保活

当客户端启动向eureka注册了本身服务列表后，需要隔断时间发送一次心跳给eureka服务端来证明自己还活着，当eureka服务端收到这个心跳请求后才知道客户端还活着，才会维护该客户端的服务列表信息。一旦因为某些原因导致客户端没有按时发送心跳给eureka服务端，这时候eureka服务端可能认为这个客户端已经挂了，它就有可能把服务从服务列表中删除掉。

续约保活的配置如下：

#### 服务端配置

```properties
# 自我保护模式，当出现网络分区，eureka在短时间内丢失过多客户端时，会进入自我保护模式
# 即一个服务长时间没有心跳，eureka也不会将其删除，默认为true
eureka.server.enable-self-preservation=true
# Eureka Server在运行期间会去统计心跳失败比例在15分钟之内是否低于85%，如果低于85%，Eureka Server会将这些实例保护起来
eureka.server.renewal-percent-threshold=0.85
# Eureka Server清理无效节点的时间间隔，默认60000毫秒，即60秒
eureka.server.eviction-interval-timer-in-ms=60000
```

#### 客户端配置

```properties
# 服务续约，心跳的时间间隔
eureka.instance.lease-renewal-interval-in-seconds=30
# 如果从前一次发送心跳时间起，90秒没接收到新的心跳，将剔除服务
eureka.instance.lease-expiration-duration-in-seconds=90
# 表示Eureka客户端间隔多久去拉取服务注册信息，默认为30秒
eureka.client.registry-fetch-interval-seconds=30
```

### eureka健康检测

eureka默认的健康检测是校验服务连接是否是UP还是DOWN状态，然后客户端只会调用状态为UP的服务。但是有的情况下，虽然服务连接是好的，但是有可能这个服务的某些接口不是正常的，可能由于需要来连接DB有问题导致接口调用失败，所以理论上服务虽然能正常调用，但是它不是一个就健康的服务。

#### 健康检测依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId></dependency>
```

#### bootstrap.properties配置文件

```properties
# 健康检查
eureka.client.healthcheck.enabled=true
```

### 服务下线

有些情况是服务主机意外宕机了，也就是服务没办法给eureka心跳信息，但是eureka在没有接收到心跳的情况下依然维护该服务90s，在这90s之内可能会有客户端调用到该服务，这就可能会导致调用失败。所以必须要有一个机制能手动的立马把宕机的服务从eureka服务列表中清除掉，避免被服务调用方调用到。

[服务下线接口](http://localhost:8763/eureka/apps/MICRO-ORDER/localhost:micro-order:8084)

`http://localhost:8763/eureka/apps/MICRO-ORDER/localhost:micro-order:8084`

### eureka高可用

整个微服务中存在多个eureka服务，每个eureka服务都是相互复制的，会把客户端注册进来的服务复制到eureka集群中的其他节点里面来。

通过配置两个配置文件，然后启动两个服务达到高可用

端口为8761的eureka服务把自己注册到8762的eureka服务中；端口为8762的eureka服务把自己注册到8761的eureka服务中

#### application-8761.properties配置文件

```properties
server.port=8761
eureka.instance.hostname=Eureka8761

# 是否注册到eureka
eureka.client.register-with-eureka=true
# 是否从eureka中拉取注册信息
eureka.client.fetch-registry=true
# 暴露eureka服务的地址
eureka.client.serviceUrl.defaultZone=http://admin:admin@Eureka8762.com:8762/eureka/

# 开启basic校验，设置登录用户名和密码
security.basic.enabled=true
spring.security.user.name=admin
spring.security.user.password=admin

# 自我保护模式，当出现网络分区，eureka在短时间内丢失过多客户端时，会进入自我保护模式
# 即一个服务长时间没有心跳，eureka也不会将其删除，默认为true
eureka.server.enable-self-preservation=true
# Eureka Server在运行期间会去统计心跳失败比例在15分钟之内是否低于85%，如果低于85%，Eureka Server会将这些实例保护起来
eureka.server.renewal-percent-threshold=0.85
# Eureka Server清理无效节点的时间间隔，默认60000毫秒，即60秒
eureka.server.eviction-interval-timer-in-ms=60000
```

#### application-8762.properties配置文件

```properties
server.port=8762
eureka.instance.hostname=Eureka8762

# 是否注册到eureka
eureka.client.register-with-eureka=true
# 是否从eureka中拉取注册信息
eureka.client.fetch-registry=true
# 暴露eureka服务的地址
eureka.client.serviceUrl.defaultZone=http://admin:admin@Eureka8761.com:8761/eureka/

# 开启basic校验，设置登录用户名和密码
security.basic.enabled=true
spring.security.user.name=admin
spring.security.user.password=admin

# 自我保护模式，当出现网络分区，eureka在短时间内丢失过多客户端时，会进入自我保护模式
# 即一个服务长时间没有心跳，eureka也不会将其删除，默认为true
eureka.server.enable-self-preservation=true
# Eureka Server在运行期间会去统计心跳失败比例在15分钟之内是否低于85%，如果低于85%，Eureka Server会将这些实例保护起来
eureka.server.renewal-percent-threshold=0.85
# Eureka Server清理无效节点的时间间隔，默认60000毫秒，即60秒
eureka.server.eviction-interval-timer-in-ms=60000
```

#### 启动eureka服务端

```shell
# 启动端口为8761的eureka服务
java -jar springcloud-eureka.jar -Dspring.profiles.active=8761
# 启动端口为8762的eureka服务
java -jar springcloud-eureka.jar -Dspring.profiles.active=8762
```

#### eureka服务端的配置

`bootstrap.properties`

```properties
# Eureka客户端配置
eureka.client.serviceUrl.defaultZone=http://admin:admin@Eureka8761:8761/eureka/,http://admin:admin@Eureka8762:8762/eureka/
```

