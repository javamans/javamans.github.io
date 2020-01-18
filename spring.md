---
layout: article
titles:
  # @start locale config
  zh-Hans : &ZH_HANS  Spring
  # @end locale config
key: page-about
sidebar:
  nav: spring
---

### Spring源码的下载

- `git clone --branch v5.1.3.RELEASE https://gitee.com/Z201/spring-framework.git`
- gradle下载，gradle要JDK8的版本
- 到Spring源码路径执行gradle命令`gradlew:spring-oxm:compileTestjava`
- 用idea打开Spring源码工程，在idea中安装插件kotlin，重启idea
- 把编译好的源码导入工程中