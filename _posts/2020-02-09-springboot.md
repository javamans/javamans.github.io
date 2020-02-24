---
title: 9.springboot整合rabbitmq
tags: springboot
---

使用springboot整合rabbitmq

#### jar包导入

```java
<!--集成rabbitmq-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### application.properties配置

```properties
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=admin
```

#### 代码配置类

```java
@Configuration
public class RabbitmqConfig {

    @Bean(name = "message")
    public Queue queueMessage() {
        return new Queue("jack.message");
    }

    @Bean
    public TopicExchange exchange() {
        return new TopicExchange("exchange.message");
    }

    @Bean
    public Binding bindingExchangeMessage(@Qualifier("message") Queue queueMessage,
                                          TopicExchange exchange) {
        return BindingBuilder.bind(queueMessage)
                .to(exchange)
                .with("jack.message.routeKey");
    }
}
```

#### 消息发送和消费

##### 消息的发送

依赖注入`AmqpTemplate`

```java
@Slf4j
@Component
public class RabbitmqSender {

    @Autowired
    private AmqpTemplate amqpTemplate;

    public void sendMessage(String exchange, String routeKey, Message message) {

        log.info("=======发送消息余额宝=========" + message);
        amqpTemplate.convertAndSend(exchange, routeKey, JSONObject.toJSONString(message));

    }
}
```

##### 消息的消费

使用`@RabbitListener`注解监听

```java
@Slf4j
@Component
public class MessageListener {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    TransactionTemplate transactionTemplate;

    @Autowired
    OrderService orderService;

    @RabbitListener(queues = "jack.message")
    public void process(String strMsg) {
        final Message message = JSONObject.parseObject(strMsg, Message.class);
        log.info("=========开始消费支付宝的消息============" + JSONObject.toJSONString(message));
        transactionTemplate.execute(new TransactionCallback<Object>() {
            @Override
            public Object doInTransaction(TransactionStatus status) {
                int messageId = message.getMessageId();
                int count = orderService.queryMessageCountByMessageId(messageId);
                if (count == 0) {
                    orderService.updateAmount(message.getAmount(), message.getUserId());
                    orderService.insertMessage(message.getUserId(), message.getMessageId(), message.getAmount());
                } else {
                    logger.info("异常转账");
                }
                return null;
            }
        });
    }
}
```

本整合实例模拟基于支付宝向余额宝转账功能，其中还包含分布式事务的解决方案

支付宝工程链接：[https://github.com/gaochaojin/alipay-springboot.git](https://github.com/gaochaojin/alipay-springboot.git)

余额宝工程链接：[https://github.com/gaochaojin/balance-springboot.git](https://github.com/gaochaojin/balance-springboot.git)

