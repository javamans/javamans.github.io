---
title: 4.springboot整合atomikos
tags: springboot
---

atomikos是一个基于XA协议的分布式事务解决管理框架，其核心思想就是两段提交，一般使用在一个业务方法里面涉及到的多个数据源的操作，要保证在这个业务方法操作中，保持对两个数据源的同时提交和同事回滚。

#### jar包导入

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jta-atomikos</artifactId></dependency>
```

#### application.properties配置

```properties
spring.datasource.springboot.driverClassName=com.mysql.jdbc.Driver
spring.datasource.springboot.jdbcUrl=jdbc:mysql://127.0.0.1:3306/spring-boot?serverTimezone=GMT%2B8
spring.datasource.springboot.username=root
spring.datasource.springboot.password=root

spring.datasource.springboot2.driverClassName=com.mysql.jdbc.Driver
spring.datasource.springboot2.jdbcUrl=jdbc:mysql://127.0.0.1:3306/spring-boot2?serverTimezone=GMT%2B8
spring.datasource.springboot2.username=root
spring.datasource.springboot2.password=root
```

#### 数据源创建

数据源1和数据源2类似配置

`DBConfig1.java`

```java
@Data
@Component
@ConfigurationProperties(prefix = "spring.datasource.springboot")
public class DBConfig1 {

    private String driverClassName;
    private String jdbcUrl;
    private String username;
    private String password;
}
```

`Datasource1Config.java`

```java
@Configuration
@MapperScan(basePackages = "com.gaochaojin.dao.users", sqlSessionFactoryRef = "test1SqlSessionFactory")
public class Datasource1Config {

    /**
     * 配置数据源
     *
     * @param config1
     * @return
     */
    @Bean(name = "test1DataSource")
    @Primary
    public DataSource testDataSource(DBConfig1 config1) {
        MysqlXADataSource mysqlXADataSource = new MysqlXADataSource();
        mysqlXADataSource.setUrl(config1.getJdbcUrl());
        mysqlXADataSource.setPassword(config1.getPassword());
        mysqlXADataSource.setUser(config1.getUsername());

        AtomikosDataSourceBean atomikosDataSourceBean = new AtomikosDataSourceBean();
        atomikosDataSourceBean.setXaDataSource(mysqlXADataSource);
        atomikosDataSourceBean.setUniqueResourceName("test1DataSource");
        return atomikosDataSourceBean;
    }

    @Bean(name = "test1SqlSessionFactory")
    @Primary
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("test1DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapping/users/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "test1SqlSessionTemplate")
    @Primary
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("test1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

#### 业务代码的使用

```java
@Service
public class OrderServiceImpl implements IOrderService {

    @Resource
    private UsersMapper usersMapper;
    @Resource
    private OrdersMapper ordersMapper;

    @Override
    @Transactional
    public void addOrder(Users users, Orders orders) {
        usersMapper.insertSelective(users);
        int i = 10 / 0;//测试事务是否一致
        ordersMapper.insertSelective(orders);
    }
}
```

