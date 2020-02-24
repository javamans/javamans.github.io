---
title: 8.springboot整合actuator
tags: springboot
---

actuator监控是一个用于监控springboot健康状况的工具，可以实时的工程的健康和调用情况

#### jar包导入

```java
<!--集成actuator-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### application.properties配置

```properties
# actutor配置(默认只有info/health) 
# 链接地址:http://localhost:8080/actuator/
management.endpoints.web.exposure.include=*
```

1 /health/{component}/{instance} GET
报告程序的健康指标，这些数据由 HealthIndicator 实现类提供
2 /info GET
获取程序指定发布的信息，这些信息由配置文件中 info 打头的属性提供
3 /configprops GET
描述配置属性（包含默认值）如何注入到 bean
4 /beans GET
描述程序中的 bean，及之间的依赖关系
5 /env GET
获取全部环境属性
6 /env/{name} GET
根据名称获取指定的环境属性值
7 /mappings GET
描述全部的 URI 路径，及和控制器的映射关系

8 /metrics/{requiredMetricName} GET
统计程序的各种度量信息，如内存用量和请求数
9 /httptrace GET
提供基本的 http 请求跟踪信息，如请求头等
10 /threaddump GET
获取线程活动的快照
11 /conditions GET
提供自动配置报告，记录哪些自动配置通过，哪些没有通过
12 /loggers/{name} GET
查看日志配置信息
13 /auditevents GET
查看系统发布的事件信息
14 /caches/{cache} GET/DELETE
查看系统的缓存管理器，另可根据缓存管理器名称查询；另 DELETE 操作
可清除缓存
15 /scheduledtasks GET
查看系统发布的定时任务信息
16 /features GET
查看 Springcloud 全家桶组件信息
17 /refresh POST
重启应用程序，慎用
18 /shutdown POST
关闭应用程序，慎用