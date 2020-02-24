---
title: 5.springboot整合redis
tags: springboot
---

#### jar包导入

```java
<!--集成redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>   </dependency>
```

#### application.peoperties配置

```properties
# redis配置
# 数据库索引默认为0
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=8
# 连接池中最小空闲凯娜姐
spring.redis.jedis.pool.min-idle=0
# 连接超时时间
spring.redis.timeout=5000
```

#### 将redis整合到spring缓存体系中

包含CacheManager对象创建和创建redis对象创建；使用`@EnableCaching`开启缓存功能

```java
@Configuration
@EnableCaching
public class RedisConfig {

    /**
     * 缓存管理器
     * @param redisConnectionFactory
     * @return
     */
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory){
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofHours(1));// 设置缓存有限期1h
        return RedisCacheManager.builder(RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory))
                .cacheDefaults(redisCacheConfiguration).build();
    }

    @Bean
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<String,Object> template = new RedisTemplate<>();
        // 配置连接工厂
        template.setConnectionFactory(redisConnectionFactory);
        // 使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value（默认使用JDK的序列化方式）
        Jackson2JsonRedisSerializer jsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();
        // 指定要序列化的域，field，get和set，以及修饰符范围，ANY是都有包括private和public
        objectMapper.setVisibility(PropertyAccessor.ALL,JsonAutoDetect.Visibility.ANY);
        // 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String，Integer等会抛出异常
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jsonRedisSerializer.setObjectMapper(objectMapper);

        // 值采用json序列化
        template.setValueSerializer(jsonRedisSerializer);
        // 使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());
        // 设置hash key和value序列化模式
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jsonRedisSerializer);
        template.afterPropertiesSet();

        return template;
    }
}

```

#### 业务代码中的使用

在业务代码中只需要在方法上面添加`@Cacheable`、`@CachePut`、`@CacheEvict`注解就可以了

```java
@Service
public class CommonServiceImpl implements CommonService {

    @Autowired
    private StudentDao studentDao;

    @Cacheable(value = "student",key = "#id")
    @Override
    public Student findById(Integer id){
        Optional<Student> optional = studentDao.findById(id);
        if (optional.isPresent()){
            return optional.get();
        }
        return null;
    }

    @CachePut(value = "student",key = "#student.id")
    @Override
    public Student save(Student student) {
        Student student1 = studentDao.save(student);
        return student1;
    }

    @CacheEvict(value = "student",key = "#id")
    @Override
    public void delete(Integer id) {
        studentDao.deleteById(id);
    }
}
```

