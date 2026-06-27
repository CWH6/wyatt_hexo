---
title: 【MQ】RabbitMQ订单未支付x分钟自动取消
date: 2023-05-14 12:53:20
tags:
  - RabbitMQ
category: 
  - 后端
---



## 技术点

RabbitMQ: 死信队列，脑瓜子空空就点 [传送门](http://cwh6.gitee.io/bk/2023/05/05/【RabbitMQ】消息队列/#死信队列) 进行回忆呗！

## 实现原理

1、用户下单之后，投递一个订单消息存放在订单队列里，该消息过期时间为x分钟，一直未被订单消费者消费，消息会转移到死信交换机路由到死信队列中，被我们的死信消费者x分钟后消费

2、死信消费者在根据订单号码查询支付订单状态，如果是未支付情况下，则将该订单设置未超时。

对筛选出来的订单号码进行核对校验

- 订单中是否存在
- 携带订单号码调用支付宝查询订单支付状态是否为待支付
- 更新该订单号码状态

## 超时消费流程图

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230514115759717.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230514115759717.png)

## 代码

**定义配置类**

1、定义订单交换机，订单队列，设置订单交换机绑定订单队列

2、定义死信订单交换机，死信订单队列，设置死信订单交换机绑定死信订单队列

3、设置订单队列的死信订单交换机

```java
@Configuration
public class QueueConfig {

    // 订单交换机名称
    public static final String ORDER_EXCHANGE="ORDER_EXCHANGE";
    // 死信订单交换机名称
    public static final String DEAD_ORDER_EXCHANGE="DEAD_ORDER_EXCHANGE";
    // 订单队列名称
    public static final String ORDER_QUEUE="ORDER_QUEUE";
    // 死信订单队列名称
    public static final String DEAD_ORDER_QUEUE="DEAD_ORDER_QUEUE";
    // 订单交换机路由key
    public static final String ORDER_EXCHANGE_ROUTING_KEY="ORDER_EXCHANGE_ROUTING_KEY";
    // 死信订单交换机路由key
    public static final String DEAD_ORDER_EXCHANGE_ROUTING_KEY="DEAD_ORDER_EXCHANGE_ROUTING_KEY";
    
    //声明订单交换机
    @Bean("orderExchange")
    public DirectExchange orderExchange(){
        return new DirectExchange(ORDER_EXCHANGE);
    }

    //订单死信订单交换机
    @Bean("orderDeadExchange")
    public DirectExchange orderDdadExchange(){
        return new DirectExchange(DEAD_ORDER_EXCHANGE);
    }

    //声明订单队列
    @Bean("orderQueue")
    public Queue orderQueue(){
        Map<String,Object> arguments = new HashMap<>(2);
        //设置死信订单交换机
        arguments.put("x-dead-letter-exchange",DEAD_ORDER_EXCHANGE);
        //设置死信订单交换机RoutingKey
        arguments.put("x-dead-letter-routing-key",DEAD_ORDER_EXCHANGE_ROUTING_KEY);
        return QueueBuilder.durable(ORDER_QUEUE).withArguments(arguments).build();
    }

    //声明死信订单队列
    @Bean("deadOrderQueue")
    public Queue deadOrderQueue(){
        return QueueBuilder.durable(DEAD_ORDER_QUEUE).build();
    }

    //声明订单队列 orderQueue 绑定 orderExchange 订单交换机
    @Bean
    public Binding queueABindingX(@Qualifier("orderQueue") Queue orderQueue,
                                  @Qualifier("orderExchange") DirectExchange orderExchange){
        return BindingBuilder.bind(orderQueue).to(orderExchange).with(ORDER_EXCHANGE_ROUTING_KEY);
    }


    //声明死信订单队列 deadOrderQueue 绑定 orderDeadExchange 死信订单交换机
    @Bean
    public Binding queueDBindingY(@Qualifier("deadOrderQueue") Queue deadOrderQueue,
                                  @Qualifier("deadorderExchange") DirectExchange deadorderExchange){
        return BindingBuilder.bind(deadOrderQueue).to(deadorderExchange).with(DEAD_ORDER_EXCHANGE_ROUTING_KEY);
    }

}
```

**生产者**

```JAVA
@Resource
private OrderTimeoutManager orderTimeoutManager;    

@Override
public BaseResponse<String> toPayResultToken(PayOrderTokenDto payOrderTokenDto) {
    // 添加支付信息(状态未设置)
    PaymentInfoEntity paymentChannelEntity = dtoToDo(payOrderTokenDto, PaymentInfoEntity.class);
    int result = paymentInfoMapper.insert(paymentChannelEntity);
    if (result <= 0) {
         return setResultError("插入支付记录失败!");
     }
     // 获取记录id
     Long id = paymentChannelEntity.getId();

     // 向mq投递一条msg消息 设置过期时间 30分钟
     orderTimeoutManager.sendOrderTimeoutMsg(id + "");
     return setResultSuccess(payToken);
}
```

指定消息的过期时间

```java
@Component
public OrderTimeoutManager {
    
    @Resource
    private RabbitTemplate rabbitTemplate;
    
    public void sendOrderTimeoutMsg(String msg) {
        
        MessagePostProcessor messagePostProcessor = message -> {
            message.getMessageProperties().setExpiration("30000");
            return message;
        };
        
        rabbitTemplate.convertAndSend(ConfirmConfig.ORDER_EXCHANGE,
        ConfirmConfig.CONFIRM_ROUTING_KEY,msg,messagePostProcessor);
        log.info("发送消息内容:{}",msg);
    }
    
}
```

也可以定义多个消费者，进行轮训消费消息

**消费者**

```java
@Slf4j
@Component
public class OrderConsumer {

    @RabbitListener(queues = ConfirmConfig.ORDER_QUEUE)
    public void receiveConfirmMessage(Message message){
        String msg = new String(message.getBody());
        log.info("接受到的订单队列orderqueue消息:{}",msg);
    }
}
```

**死信队列消费者**

```java
@Slf4j
@Component
public class DeadOrderConsumer {
    

    @Autowired
    private PayOrderTimeoutService payOrderTimeoutService;

    /**
    * 订单死信队列监听队列的回调方法
    */
    @RabbitListener(queues = ConfirmConfig.DEAD_ORDER_QUEUE)
    public void receiveConfirmMessage(Message message){
    String msg = new String(message.getBody());
    log.info("接受到的队列deadOrderqueue消息:{}",msg);
    payOrderTimeoutService.orderTimeout(payId);

}

}
```

**校验封装**

```java
@Service
public class PayOrderTimeoutService {
    @Autowired
    private PaymentInfoMapper paymentInfoMapper;

    public boolean orderTimeout(Long payId) {
        // 1.根据该支付id 查询支付订单信息的状态
        PaymentInfoEntity paymentInfoEntity = paymentInfoMapper.selectById(Integer.paseInt(payId));
        if (paymentInfoEntity == null) {
            return false;
        }

        // 2.如果支付状态是为未支付的话，则将该状态该已超时
        if (!PaymentConstant.PAYMENT_STATUS_NOT.equals(paymentInfoEntity.getPaymentStatus())) {
            return false;
        }
        // 3. 主动调用支付宝接口,根据订单状态查询,支付宝这边是否已经支付了31、32分钟 调用支付宝接口查询
       
        // 4.用户跳转到支付页面支付超时30分钟,调用库存接口,修改该状态为超时
        paymentInfoEntity.setPaymentStatus(PaymentConstant.PAYMENT_STATUS_TIMEOUT);
        paymentInfoMapper.updateById(paymentInfoEntity);
        return true;
    }
}
```

后续更新要解决消息队列消息丢失，持久化设置
