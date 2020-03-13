---
title: 2.springcloud组件Ribbon的使用
tags: springcloud
---

Ribbon是一个独立的组件，是用来进行远程接口调用的。

### Ribbon负载均衡算法

- 线性轮询`RoundRobinRule`
- 可以重试的轮询`RetryRule`
- 根据运行情况来计算权重`WeightedResponseTimeRule`
- 过滤掉故障实例，选择请求数最小的实例`BestAvailableRule`
- 随机`RandomRule`

### Ribbon的使用

#### 依赖包的导入

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

#### bootstrap.properties配置文件

```properties
# 使用eureka(从eureka中发现需要负载的机器)
ribbon.eureka.enabled=true
# 开启重试
spring.cloud.loadbalancer.retry.enabled=true
```

#### 代码添加配置

使用`@RibbonClients`注解进行加载配置

`LoadBalanceConfig.java`

```java
@Configuration
@RibbonClients(value = {
        @RibbonClient(name = "micro-order",configuration = RibbonLoadBalanceMicroOrderConfig.class)
})
public class LoadBalanceConfig {
}
```

`RibbonLoadBalanceMicroOrderConfig.java`

```java
@Configuration
public class RibbonLoadBalanceMicroOrderConfig {

    @RibbonClientName
    private String name = "micro-order";

    @Bean
    @ConditionalOnClass
    public IClientConfig defaultClientConfigImpl() {
        DefaultClientConfigImpl config = new DefaultClientConfigImpl();
        config.loadProperties(name);
        // 对当前实例的重试次数，当eureka中可以找到服务，但是服务连不上时将会重试
        config.set(CommonClientConfigKey.MaxAutoRetries, 2);
        // 切换实例的重试次数
        config.set(CommonClientConfigKey.MaxAutoRetriesNextServer, 2);
        // 请求连接超时时间，单位ms
        config.set(CommonClientConfigKey.ConnectTimeout, 2000);
        // 请求处理的超时时间，单位ms
        config.set(CommonClientConfigKey.ReadTimeout, 4000);
        // 是否打开重试操作
        config.set(CommonClientConfigKey.OkToRetryOnAllOperations, true);
        return config;
    }

    @Bean
    public IPing iPing(){
        // 这个实现类会去调用服务是否存活
        PingUrl pingUrl = new PingUrl();
        pingUrl.setPingAppendString("/queryUser");
        return pingUrl;
    }

    @Bean
    public IRule iRule(){
        // 线性轮询
        // new RoundRobinRule();
        // 可以重试的轮询
        // new RetryRule();
        // 根据运行情况来计算权重
        // new WeightedResponseTimeRule();
        // 过滤掉故障实例，选择请求数最小的实例
        // new BestAvailableRule();
        return new RandomRule();
    }
}
```

