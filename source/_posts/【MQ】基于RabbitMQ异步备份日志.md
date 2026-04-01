---
title: 【MQ】基于RabbitMQ异步备份日志
date: 2026-04-01 11:43:48
categories:
  - 技术实战
  - 消息队列
tags:
  - RabbitMQ
  - 日志系统
  - 异步
---



## 一、场景

根据业务采用`RabbitMQ`去异步将消息 (实体) 发送到redis



##  二、操作

###  2.1 依赖

```xml
// pom.xml
<dependency>
       <groupId>org.springframework.amqp</groupId>
       <artifactId>spring-rabbit</artifactId>
</dependency>
```

### 2.2 配置yaml

```yml
spring:
  rabbitmq:
    host: xxxx
    port: 5672
    username: xxx
    password: xxx
```


###  2.3 声明MQ组件

```java
import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class AnswerBackupMQConfig {

    // 答案备份交换机名称
    public static final String ANSWER_BACKUP_EXCHANGE = "ANSWER_BACKUP_EXCHANGE";
    // 死信答案备份交换机名称
    public static final String DEAD_ANSWER_BACKUP_EXCHANGE = "DEAD_ANSWER_BACKUP_EXCHANGE";

    // 答案备份队列名称
    public static final String ANSWER_BACKUP_QUEUE = "ANSWER_BACKUP_QUEUE";
    // 死信答案备份队列名称
    public static final String DEAD_ANSWER_BACKUP_QUEUE = "DEAD_ANSWER_BACKUP_QUEUE";

    // 答案备份交换机路由key
    public static final String ANSWER_BACKUP_EXCHANGE_ROUTING_KEY = "ANSWER_BACKUP_EXCHANGE_ROUTING_KEY";
    // 死信答案备份交换机路由key
    public static final String DEAD_ANSWER_BACKUP_EXCHANGE_ROUTING_KEY = "DEAD_ANSWER_BACKUP_EXCHANGE_ROUTING_KEY";


    // 声明答案备份交换机
    @Bean("answerBackupExchange")
    public DirectExchange answerBackupExchange() {
        return new DirectExchange(ANSWER_BACKUP_EXCHANGE);
    }

    // 声明死信答案备份交换机
    @Bean("deadAnswerBackupExchange")
    public DirectExchange deadAnswerBackupExchange() {
        return new DirectExchange(DEAD_ANSWER_BACKUP_EXCHANGE);
    }

    // 声明答案备份队列
    @Bean("answerBackupQueue")
    public Queue answerBackupQueue() {
        Map<String, Object> arguments = new HashMap<>(2);
        // 设置死信答案备份交换机
        arguments.put("x-dead-letter-exchange", DEAD_ANSWER_BACKUP_EXCHANGE);
        // 设置死信答案备份交换机RoutingKey
        arguments.put("x-dead-letter-routing-key", DEAD_ANSWER_BACKUP_EXCHANGE_ROUTING_KEY);
        return QueueBuilder.durable(ANSWER_BACKUP_QUEUE).withArguments(arguments).build();
    }

    // 声明死信答案备份队列
    @Bean("deadAnswerBackupQueue")
    public Queue deadAnswerBackupQueue() {
        return QueueBuilder.durable(DEAD_ANSWER_BACKUP_QUEUE).build();
    }

    // 声明答案备份队列 answerBackupQueue 绑定 answerBackupExchange 交换机
    @Bean
    public Binding answerBackupQueueBinding(
            @Qualifier("answerBackupQueue") Queue answerBackupQueue,
            @Qualifier("answerBackupExchange") DirectExchange answerBackupExchange) {
        return BindingBuilder.bind(answerBackupQueue)
                .to(answerBackupExchange)
                .with(ANSWER_BACKUP_EXCHANGE_ROUTING_KEY);
    }

    // 声明死信答案备份队列 deadAnswerBackupQueue 绑定 deadAnswerBackupExchange 死信交换机
    @Bean
    public Binding deadAnswerBackupQueueBinding(
            @Qualifier("deadAnswerBackupQueue") Queue deadAnswerBackupQueue,
            @Qualifier("deadAnswerBackupExchange") DirectExchange deadAnswerBackupExchange) {
        return BindingBuilder.bind(deadAnswerBackupQueue)
                .to(deadAnswerBackupExchange)
                .with(DEAD_ANSWER_BACKUP_EXCHANGE_ROUTING_KEY);
    }
}
```

### 2.4 配置MQ消息序列化

设置mqjson配置类后，mq在传递消息种可以使用实体类进行传递

```java

/**
 * RabbitMQ JSON配置类: 
 * 用于配置RabbitMQ的JSON转换器，将对象转换为JSON字符串，再发送到队列。
 */
@Configuration
public class RabbitMQJsonConfig {

    @Bean
    public MessageConverter rabbitMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory, MessageConverter rabbitMessageConverter) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(rabbitMessageConverter);
        return template;
    }

    @Bean(name = "rabbitListenerContainerFactory")
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(ConnectionFactory connectionFactory,
                                                                              MessageConverter rabbitMessageConverter) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setMessageConverter(rabbitMessageConverter);
        return factory;
    }
}
```

### 2.5 编写生产者

```java
/**
 * 答案备份-生产者
 */
@Component
@Slf4j
public class AnswerBackupMQProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendBackupMsg(ProjectsAnswerSingleDTO answer) {
        // 将处于队列30s唯有消费的丢到死信队列
        MessagePostProcessor messagePostProcessor = message -> {
            message.getMessageProperties().setExpiration("30000");
            return message;
        };
        log.info("发送消息到消息队列:{}",answer);
        rabbitTemplate.convertAndSend(AnswerBackupMQConfig.ANSWER_BACKUP_EXCHANGE,
                AnswerBackupMQConfig.ANSWER_BACKUP_EXCHANGE_ROUTING_KEY,answer,messagePostProcessor);
    }
}

```


### 2.6 编写消费者

消费队列消息，执行业务操作

```java
/**
 * 答案备份-消费者
 */
@Slf4j
@Component
public class AnswerBackupMQConsumer {

    @Resource
    RedisCache redisCache;

    final String baseKey = "answerBackup:%s:%s:%s";

    /**
     * 监听回调函数
     */
    @RabbitListener(queues = AnswerBackupMQConfig.ANSWER_BACKUP_QUEUE)
    public void receiveConfirmMessage(ProjectsAnswerSingleDTO answer) {
        log.info("已收到，消息队列信息:{}",answer);
        // 接收队列里面消息，处理消息
        // key-value 设计:  "answerBackup:项目id:answerId:题号"   answer
        String answerId = answer.getAnswerId();
        Long projectId = answer.getProjectId();
        String questionNo = "";
        Answer detailAnswer = answer.getAnswer();
        try {
            questionNo = answer.getAnswer().getQuestionNo();
            // 检查是否有重复key
            //String questionNo = detailAnswer.getQuestionNo();
            String finalKey = String.format(baseKey, projectId, answerId, questionNo);
            redisCache.setCacheObject(finalKey, detailAnswer);
        }catch (Exception e){
            log.error("消费消息失败:answerId={},productId={},questionNo={},eMsg:={}",answerId,projectId,questionNo,e.getMessage());
        }
    }

}
```


### 2.7 编写死信消费者

```java

/**
 * 答案备份-死信-消费者
 */
@Slf4j
@Component
public class AnswerBackupMQDeadConsumer {

    /**
     * 监听回调函数
     */
    @RabbitListener(queues = AnswerBackupMQConfig.DEAD_ANSWER_BACKUP_QUEUE)
    public void receiveConfirmMessage(ProjectsAnswerSingleDTO answer){
        log.error("已收到，死信队列信息:{}",answer);
        // todo 处理过期的消息
    }



}

```