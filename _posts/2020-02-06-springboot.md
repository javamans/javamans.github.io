---
title: 6.springboot整合mongodb
tags: springboot
---

#### jar包导入

```java
<!--集成mongodb-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

#### application.properties配置

```properties
# mongodb配置
spring.data.mongodb.uri=mongodb://127.0.0.1:27017/xx_db
```

#### 业务代码中的使用

MongoTemplate对象是可以直接依赖注入进来的，由启动器创建了该类的对象

```java
@Service
public class MongoServiceImpl implements MongoService<User> {

    @Autowired
    private MongoTemplate mongoTemplate;

    @Override
    public String save(User obj) {
        mongoTemplate.save(obj);
        return "1";
    }

    @Override
    public List<User> findAll() {
        List<User> all = mongoTemplate.findAll(User.class);
        return all;
    }

    @Override
    public User getById(String id) {
        Query query = new Query(Criteria.where("_id").is(id));
        return mongoTemplate.findOne(query, User.class);
    }

    @Override
    public User getByName(String name) {
        Query query = new Query(Criteria.where("username").is(name));
        return mongoTemplate.findOne(query, User.class);
    }

    @Override
    public String updateE(User user) {
        Query query = new Query(Criteria.where("_id").is(user.getId()));
        Update update = new Update().set("username", user.getUsername())
                .set("password", user.getPassword());
        UpdateResult updateResult = mongoTemplate.updateFirst(query, update, User.class);
        return updateResult.toString();
    }

    @Override
    public String deleteE(User user) {
        DeleteResult deleteResult = mongoTemplate.remove(user);
        return deleteResult.toString();
    }

    @Override
    public String deleteById(String id) {
        User user = getById(id);
        String result = deleteE(user);
        return result;
    }

    @Override
    public List<User> findLikes(String reg) {
        Pattern pattern = Pattern.compile("^.*" + reg + ".*$", Pattern.CASE_INSENSITIVE);
        Query query = new Query(Criteria.where(reg).regex(pattern));
        List<User> users = mongoTemplate.find(query, User.class);
        return users;
    }
}
```

