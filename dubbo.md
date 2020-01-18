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

![Dubbo结构](/img/dubbo/Dubbo结构.png)

- container负责启动、加载、运行provider
- provider启动时，向registry注册自己的服务
- consumer启动时，向registry订阅自己的服务
- registry提供provider列表给consumer，实时推送变动情况
- consumer根据provider列表，按照负载算法选一台provider调用
- monitor统计rpc的调用频次

### Dubbo标签

![Dubbo标签](/img/dubbo/Dubbo标签.png)

