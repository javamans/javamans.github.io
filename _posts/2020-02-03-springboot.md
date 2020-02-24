---
title: 3.springboot整合jpa
tags: springboot
---

jpa也是一个非常优秀的ORM框架，使用springboot整合jpa

#### jar包导入

```java
<!--JPA配置-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

#### application.properties配置

```properties
# jpa配置
# create为创建语句；update为更新表结构
spring.jpa.hibernate.ddl-auto=update
# 打印sql语句
spring.jpa.show-sql=true
```

#### 实例bean

配置`spring.jpa.hibernate.ddl-auto=create`，当启动项目的时候回自动生成t_student表

```java
@Data
@Entity
@Table(name = "t_student")
public class Student implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "name")
    private String name;

    @Column(name = "card_num")
    private String cardNum;

    @Column(name = "age")
    private String age;

    @Column(name = "sex")
    private String sex;
}
```

#### dao配置

泛型中指定了操作的实体bean，JpaRepository中定义了该实体的增删改查的所有方法，用studentDao对象调用即可

```java
public interface StudentDao extends JpaRepository<Student,Integer> {
}
```

#### 业务代码中使用

```java
@Service
public class CommonServiceImpl implements CommonService {

    @Autowired
    private StudentDao studentDao;

    @Override
    public Student findById(Integer id){
        Optional<Student> optional = studentDao.findById(id);
        if (optional.isPresent()){
            return optional.get();
        }
        return null;
    }
}
```

