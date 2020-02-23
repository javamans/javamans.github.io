---
title: 1.springboot启动源码
tags: springboot
---

springboot在main方法中启动的代码如下：

```java
@SpringBootApplication
public class SpringbootWebApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootWebApplication.class, args);
    }
}
```

### springboot启动

- 完成spring容器的启动
- 将项目部署到tomcat容器中

#### SpringApplication.run方法

```java
public ConfigurableApplicationContext run(String... args) {
    // 统计时间用的工具类
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    // 获取实现了SpringApplicationRunListener接口的实现类，通过SPI机制加载META-INF/spring.factories文件下的类
    SpringApplicationRunListeners listeners = getRunListeners(args);

    // 首先调用SpringApplicationRunListener.starting方法
    listeners.starting();
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);

        // 处理配置数据
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        configureIgnoreBeanInfo(environment);

        // 启动时打印banner
        Banner printedBanner = printBanner(environment);

        // 创建上下文对象
        context = createApplicationContext();

        // 获取SpringBootExceptionReporter接口的类，异常报告
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                                                         new Class[] { ConfigurableApplicationContext.class }, context);
        prepareContext(context, environment, listeners, applicationArguments, printedBanner);

        // 核心方法，启动spring容器
        refreshContext(context);
        afterRefresh(context, applicationArguments);

        // 统计结束
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }

        // 调用started方法
        listeners.started(context);

        // 获取ApplicationRunner/CommandLineRunner两个接口的实现类，并调用其run方法
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```



#### 通过SPI机制加载配置

SPI在springboot中是去读取META-INF/spring.factories目录的配置问价内容，把配置文件中的类加载到spring容器中。

将一个类加载到spring容器中管理的方式有以下三种：

- 通过xml的bean标签
- 通过加@Component注解被@ComponentScan注解扫描
- 通过在spring.factories中配置该类

前两者是加载本工程的bean，扫描本工程的bean，第三种加载第三方定义的jar包中的bean

#### createApplicationContext创建springboot的上下文对象

```java
public static final String DEFAULT_SERVLET_WEB_CONTEXT_CLASS = "org.springframework.boot."
			+ "web.servlet.context.AnnotationConfigServletWebServerApplicationContext";
```

```java
if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
    RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
    def.setSource(source);
    beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
}
```

在这个上下文对象构造函数中把ConfigurationClassPostProcessor变成beanDefinition对象

#### refreshContext容器的启动

```java
private void refreshContext(ConfigurableApplicationContext context) {
    refresh(context);
    if (this.registerShutdownHook) {
        try {
            context.registerShutdownHook();
        }
        catch (AccessControlException ex) {
            // Not allowed in some environments.
        }
    }
}
```

这里调用了上下文对象AnnotationConfigServletWebServerApplicationContext的refresh方法，该方法在spring源码分析中有详细的讲解，这里需要关注的时一个钩子方法onRefresh()

#### 内置tomcat的启动和部署

tomcat的启动在onRefresh中

```java
protected void onRefresh() {
    super.onRefresh();
    try {
        // 创建web服务器
        createWebServer();
    }
    catch (Throwable ex) {
       
    }
}
```

##### createWebServer方法

```java
private void createWebServer() {
    WebServer webServer = this.webServer;
    ServletContext servletContext = getServletContext();
    if (webServer == null && servletContext == null) {
        ServletWebServerFactory factory = getWebServerFactory();
        // 主要看这个方法
        this.webServer = factory.getWebServer(getSelfInitializer());
    }
    else if (servletContext != null) {
        try {
            getSelfInitializer().onStartup(servletContext);
        }
        catch (ServletException ex) {
            throw new ApplicationContextException("Cannot initialize servlet context", ex);
        }
    }
    initPropertySources();
}
```

##### getWebServer方法

```java
public WebServer getWebServer(ServletContextInitializer... initializers) {
    if (this.disableMBeanRegistry) {
        Registry.disableRegistry();
    }
    Tomcat tomcat = new Tomcat();
    File baseDir = (this.baseDirectory != null) ? this.baseDirectory : createTempDir("tomcat");
    tomcat.setBaseDir(baseDir.getAbsolutePath());
    Connector connector = new Connector(this.protocol);
    connector.setThrowOnFailure(true);
    tomcat.getService().addConnector(connector);
    customizeConnector(connector);
    tomcat.setConnector(connector);
    tomcat.getHost().setAutoDeploy(false);
    configureEngine(tomcat.getEngine());
    for (Connector additionalConnector : this.additionalTomcatConnectors) {
        tomcat.getService().addConnector(additionalConnector);
    }
    prepareContext(tomcat.getHost(), initializers);
    return getTomcatWebServer(tomcat);
}
```

##### getTomcatWebServer方法

```java
// 启动tomcat容器
this.tomcat.start();
// 阻塞tomcat容器
TomcatWebServer.this.tomcat.getServer().await();
```

