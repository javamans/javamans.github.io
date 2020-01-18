---
layout: article
titles:
  # @start locale config
  zh-Hans : &ZH_HANS  Dubbo
  # @end locale config
key: page-about
sidebar:
  nav: dubbo
---

### Dubbo简介

Dubbo是一个分布式、高性能、透明化的RPC服务框架；

Dubbo提供服务自动注册、自动发现等高效服务治理方案；

Dubbo主要功能包括高性能NIO通讯及协议集成，服务动态寻址与路由，软负载均衡与容错，依赖分析与降级等。

### Dubbo结构及功能

- container负责启动、加载、运行provider
- provider启动时，向registry注册自己的服务
- consumer启动时，向registry订阅自己的服务
- registry提供provider列表给consumer，实时推送变动情况
- consumer根据provider列表，按照负载算法选一台provider调用
- monitor统计rpc的调用频次

### Dubbo标签

主要标签：

- <dubbo:application >：应用信息配置，对应配置类：` org.apache.dubbo.config.ApplicationConfig `
- <dubbo:protocol >：服务提供者协议配置，对应配置类：` org.apache.dubbo.config.ProtocolConfig `
- <dubbo:registry >：注册中心配置，对应的配置类：`org.apache.dubbo.config.RegistryConfig`
- <dubbo:provider >： 服务提供者缺省值配置，对应的配置类： `org.apache.dubbo.config.ProviderConfig` 
- <dubbo:consumer >： 服务消费者缺省值配置，配置类： `org.apache.dubbo.config.ConsumerConfig` 
- <dubbo:service >： 服务提供者暴露服务配置，对应的配置类：`org.apache.dubbo.config.ServiceConfig` 
- <dubbo:reference >： 服务消费者引用服务配置，对应的配置类： `org.apache.dubbo.config.ReferenceConfig` 

