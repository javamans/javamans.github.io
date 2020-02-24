---
title: 1.springboot整合druid
tags: springboot
---

Druid是一个非常优秀的连接池，很好的管理了数据库连接，可以实时监控数据库连接对象和应用程序的数据库操作记录

#### jar包导入

```java
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.0.26</version>
</dependency>
```

#### druid配置

##### application.properties文件配置

```properties
spring.druid.jdbc-url=jdbc:mysql://127.0.0.1:3306/springboot1
spring.druid.jdbc-url1=jdbc:mysql://127.0.0.1:3306/springboot2
spring.druid.username=root
spring.druid.password=root
spring.druid.driver-class-name=org.gjt.mm.mysql.Driver
spring.druid.initialSize=2
spring.druid.minIdle=2
spring.druid.maxActive=2
# 配置获取链接等待超时的时间
spring.druid.maxWait=60000
# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
spring.druid.timeBetweenEvictionRunsMillis=60000
# 配置一个连接在池中最小生存的时间，单位是毫秒
spring.druid.minEvictableIdleTimeMillis=300000
spring.druid.validationQuery=SELECT 1 FROM DUAL
spring.druid.testWhileIdle=true
spring.druid.testOnBorrow=false
spring.druid.poolPreparedStatement=false
spring.druid.maxPoolPreparedStatementPerConnectionSize=20
# 配置监控统计拦截的filters，去掉后监控见面sql无法统计，‘wall’用于防火墙
spring.druid.filters=stat
# 通过connectionProperties属性来打开mergeSql功能；慢sql记录
spring.druid.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
```

##### 数据库连接配置

```java
@Data
@Configuration
@ConfigurationProperties(prefix = "spring.druid", ignoreInvalidFields = true)
public class DruidConfig {

    private String driverClassName;
    private String jdbcUrl;
    private String jdbcUrl1;
    private String username;
    private String password;
    private int maxActive;
    private int minIdle;
    private int initialSize;
    private Long timeBetweenEvictionRunsMillis;
    private Long minEvictableIdleTimeMillis;
    private String validationQuery;
    private boolean testWhileIdle;
    private boolean testOnBorrow;
    private boolean testOnReturn;
    private boolean poolPreparedStatements;
    private Integer maxPoolPreparedStatementPerConnectionSize;
    private String filters;
    private String connectionProperties;

    // 数据源对象创建并加入到spring容器中
    @Bean
    public DataSource getDs1() {
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setDriverClassName(driverClassName);
        druidDataSource.setUrl(jdbcUrl);
        druidDataSource.setUsername(username);
        druidDataSource.setPassword(password);
        druidDataSource.setMaxActive(maxActive);
        druidDataSource.setInitialSize(initialSize);
        druidDataSource.setTimeBetweenConnectErrorMillis(timeBetweenEvictionRunsMillis);
        druidDataSource.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
        druidDataSource.setValidationQuery(validationQuery);
        druidDataSource.setTestWhileIdle(testWhileIdle);
        druidDataSource.setTestOnBorrow(testOnBorrow);
        druidDataSource.setTestOnReturn(testOnReturn);
        druidDataSource.setPoolPreparedStatements(poolPreparedStatements);
        druidDataSource.setMaxPoolPreparedStatementPerConnectionSize(maxPoolPreparedStatementPerConnectionSize);

        try {
            druidDataSource.setFilters(filters);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return druidDataSource;
    }
    
    /**
     * 配置访问 druid监控
     *
     * @return
     */
    @Bean
    public ServletRegistrationBean druidStateViewServlet() {
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        // 初始化参数initParams
        // 添加白名单
        servletRegistrationBean.addInitParameter("allow", "");
        // 添加ip黑名单
        servletRegistrationBean.addInitParameter("deny", "192.168.244.100");
        // 登录查看信息的账号密码
        servletRegistrationBean.addInitParameter("loginUsername", "admin");
        servletRegistrationBean.addInitParameter("loginPassword", "123456");
        // 是否能够重置数据
        servletRegistrationBean.addInitParameter("resetEnable", "false");
        return servletRegistrationBean;
    }
    
    /**
     * 过滤不需要监控的后缀
     *
     * @return
     */
    @Bean
    public FilterRegistrationBean druidStatFilter() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new WebStatFilter());
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
        return filterRegistrationBean;
    }
}
```

