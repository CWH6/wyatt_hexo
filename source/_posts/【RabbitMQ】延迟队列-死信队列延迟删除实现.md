---
title: 【RabbitMQ】延迟队列+死信队列延迟删除实现
date: 2024-10-11 22:11:18
tags:
  - MQ
  - RabbitMQ
category: 后端
---



## 场景

当一个订单在删除时，状态修改为锁定，当30分钟没有解锁后，那么将订单的的状态变成删除，并且执行真正的删除逻辑，将订单相关的删除。

解决：订单被误删的问题，如果在删除的延迟时间中（30分钟内），将状态修改为正正常则不会删除

[![image-20241011200928497](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241011200928497.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241011200928497.png)

## 实现

### 常量定义

```java
public class MQConst {
    public static final String EXCHANGE_DELAY_NAME_DELETE_ORDER ="topicExchange.delay.delete.order";

    public static final String QUEUE_DELAY_NAME_DELETE_ORDER ="queue.delay.delete.order";

    public static final String ROUTINGKEY_DELAY_NAME_DELETE_ORDER ="routingKey.delay.delete.order";

    public static final String EXCHANGE_DEADLETTER_NAME_DELETE_ORDER ="topicExchange.deadLetter.delete.order";

    public static final String QUEUE_DEADLETTER_NAME_DELETE_ORDER ="queue.deadLetter.delete.order";

    public static final String ROUTINGKEY_DEADLETTER_NAME_DELETE_ORDER ="routingKey.deadLetter.delete.order";
}
```

### 绑定关系

springboot 启动是动态创建死信队列，交换机，路由键。 延迟队列，交换机，路由键，已经绑定

以及延迟队列消息过期根据 死信的路由键以及队列找到对应的死信队列，将消费放到里面

```java
/**
 * RabbitMQ运行应用
 * 在springboot应用启动时，动态创建队列、交换机和绑定
 */
@Component
@Slf4j
public class RabbitMQApplicationRunner implements ApplicationRunner {

    @Resource
    private RabbitAdmin rabbitAdmin;


    @Override
    public void run(ApplicationArguments args) throws Exception {

        // 创建 删除订单死信队列
        createExchangeQueueBinding(EXCHANGE_DEADLETTER_NAME_DELETE_ORDER, QUEUE_DEADLETTER_NAME_DELETE_ORDER, ROUTINGKEY_DEADLETTER_NAME_DELETE_ORDER;

        // 创建 删除订单延迟队列
        createExchangeDelayedQueueBinding(EXCHANGE_DELAY_NAME_DELETE_ORDER, QUEUE_DELAY_NAME_DELETE_ORDER, ROUTINGKEY_DELAY_NAME_DELETE_ORDER, EXCHANGE_DEADLETTER_NAME_DELETE_ORDER,ROUTINGKEY_DEADLETTER_NAME_DELETE_ORDER);

    }

    /**
     * 动态创建交换机、队列和绑定
     *
     * @param exchangeName 交换机名称
     * @param queueName    队列名称
     * @param routingKey   路由键
     */
    public void createExchangeQueueBinding(String exchangeName, String queueName, String routingKey) {
        // 创建交换机
        TopicExchange exchange = new TopicExchange(exchangeName);
        rabbitAdmin.declareExchange(exchange);

        // 创建队列
        Queue queue = new Queue(queueName);
        rabbitAdmin.declareQueue(queue);

        // 创建绑定关系
        Binding binding = BindingBuilder.bind(queue).to(exchange).with(routingKey);
        rabbitAdmin.declareBinding(binding);

        log.info("Exchange, Queue, and Binding created successfully: Exchange [{}], Queue [{}], Routing Key [{}]", exchangeName, queueName, routingKey);
    }


    /**
     * 动态创建交换机、延迟队列和绑定
     *
     * @param exchangeName           交换机名称
     * @param queueName              队列名称
     * @param routingKey             路由键
     * @param deadLetterExchangeName 死信交换机名称
     * @param deadLetterRoutingKey 死信路由键名称
     */
    public void createExchangeDelayedQueueBinding(String exchangeName, String queueName, String routingKey, String deadLetterExchangeName,String deadLetterRoutingKey) {
        // 创建交换机
        TopicExchange exchange = new TopicExchange(exchangeName);
        rabbitAdmin.declareExchange(exchange);

        // 创建队列，绑定死信交换机
        Map<String, Object> arguments = new HashMap<>();
        arguments.put("x-dead-letter-exchange", deadLetterExchangeName); // 指定死信交换机
        arguments.put("x-dead-letter-routing-key", deadLetterRoutingKey);
        //arguments.put("x-message-ttl", 2 * 60 * 1000);
        arguments.put("x-message-ttl", 6 * 60 * 60 * 1000); // 设置消息 TTL 为 6 小时（单位：毫秒）

        Queue queue = new Queue(queueName, true, false, false, arguments);
        rabbitAdmin.declareQueue(queue);

        // 绑定延迟队列与交换机
        Binding binding = BindingBuilder.bind(queue).to(exchange).with(routingKey);
        rabbitAdmin.declareBinding(binding);

        log.info("Delayed Queue created: [{}]", queueName);
    }


}
```

### 死信消费者

最好先注释（因为执行顺序问题，可能队列未有找到，然后报错）

用上面的代码先创建死信队列，延迟队列...到服务器的Rabbitmq上

```java
@Component
@DependsOn("rabbitMQApplicationRunner")
@Slf4j
public class BusinessConsumer {

    @Resource
    private OrderService orderService;

    @RabbitListener(queues = QUEUE_DEADLETTER_NAME_DELETE_ORDER)
    public void processTenantDelete(Long orderId) {
        log.info("\n ==== 收到删除订单消息,orderId: {} =====", orderId);
        try {
            // 在这里执行订单删除操作
            orderService.deleteRealOrder(orderId);
            log.info("订单删除成功，orderId: {}", orderId);
        } catch (Exception e) {
            log.error("删除订单失败: {} - orderId: {}", e, orderId);
        }
    }

}
```

### 生产者

```java
@Api(tags = "订单管理 - V2")
@RestController
@Slf4j
@RequestMapping("/v2/order")
public class TenantControllerV2 {

    @Autowired
    OrderService orderService;

    @DeleteMapping(value = "/{id}")
    @ApiOperation(value = "管理员删除订单", notes = "仅供管理员操作")
    @ApiImplicitParam(name = "id", value = "订单id", required = true, dataType = SwaggerDataType.LONG)
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        orderService.deleteV2(id);
        return new ResponseEntity<>(HttpStatus.OK);
    }

}
```

服务

```java
@Override
public void deleteV2(Long id) {
    Order tenant = this.orderRepo.findById(id)
            .orElseThrow(() -> new RestBadRequestException("order not found!"));
    order.setStatus(OrderStatusEnum.LOCKED);
    orderRepo.save(tenant);
    // 发送到延迟队列
    rabbitTemplate.convertAndSend(
            MQConst.EXCHANGE_DELAY_NAME_DELETE_ORDER,
            MQConst.ROUTINGKEY_DELAY_NAME_DELETE_ORDER,
            id
    );
}
```

真正删除的方法

```java
@Override
@Transactional
public void deleteRealOrder(Long id) {
    order order = this.orderRepo.findById(id)
            .orElseThrow(() -> new RestBadRequestException("order not found!"));
    log.info("开始删除订单:{}, 状态为:{}, 是否是锁定:{}",id,order.getUserStatus(),order.getUserStatus().equals(orderStatusEnum.LOCKED));

    if(order.getUserStatus().equals(orderStatusEnum.DELETED)){
        return;
    }
    // 如何没有在规定的时间解锁就真正删除了
    if(order.getUserStatus().equals(orderStatusEnum.LOCKED)){
        order.setUserStatus(orderStatusEnum.DELETED);
        orderRepo.save(order);
        Map<Long, SimpleorderVO> orderIdToSimpleorderMap = (Map<Long, SimpleorderVO>) RedisUtils.getMappingMap("orderIdToSimpleorderMap", redisTemplate);
        if (null != orderIdToSimpleorderMap) {
            orderIdToSimpleorderMap.remove(order.getId());
            redisTemplate.opsForHash().delete("orderIdToSimpleorderMap", order.getId());
        }
        deviceService.unbindByorderIdV2(order);
        orderRelationService.delete(id);
    }
}
```

## 验证

打开rabbitmq的控制台，进入到队列里面，点击 “queue.delay.delete.order”

发起后就能看到里面有一条消息，等半小时后就会消失，在去数据库检查状态是否变成DELETE,是就删除成功。

如果测试的话，最好将延迟队列设置为2分钟，如果需要设置 就需要去rabbitmq控制台先手动删除之前的订单延迟，死信队列。注释监听死信队列的消费者代码，重启应用，去创建这些队列。

延迟队列创建成功，去控制台查看ttl真的是2分钟吗？如果是解开死信队列的消费者代码，重启应用。
