---
title: 【RabbitMQ】异步更新数据
date: 2024-09-09 23:46:20
tags:
  - MQ
category: 后端
---



## 场景

需求如下：

有个`玩家基本属性配置表`， 配置了玩家拥有的`基本属性值`，当玩家配置表里面的属性调整:比如：策划规定，这个版本需要对玩家`力量值增加`，全服玩家的基本面板里面的力量值就需要增加，策划想让玩家多一个`抗毒时间属性`，那么玩家需要获得这个抗毒时间属性。注意：玩家因为等级提升力量值就不要用数据配置表的属性，而是使用玩家自己属性+数据配置表的基础数据（追加值的问题这里不是重点）

### 表设计

玩家数据配置表， **player_attribute_config**

| 属性            | 类型      | 备注                            |
| --------------- | --------- | ------------------------------- |
| id              | BIGINT    | 主键ID                          |
| attribute_name  | VARCHAR   | 属性名称 (如：力量, 体力, 抗毒) |
| attribute_value | VARCHAR   | 基础属性值 (策划设定的默认值)   |
| created_time    | TIMESTAMP | 创建时间                        |
| update_time     | TIMESTAMP | 修改时间                        |

玩家信息表, **PlayerInfo**

| 属性             | 类型      | 备注     |
| ---------------- | --------- | -------- |
| id               | BIGINT    | 主键ID   |
| name             | VARCHAR   | 玩家姓名 |
| level            | VARCHAR   | 玩家等级 |
| sex              | BYTE      | 性别     |
| attribute_config | JSON      | 玩家属性 |
| created_time     | TIMESTAMP | 创建时间 |
| update_time      | TIMESTAMP | 修改时间 |

### 思路

上面很多人都会想到去同步`玩家基本属性配置表`的力量值 到 所有玩家上，这个是批量修改，当 `玩家基本属性配置表` 每次一调整，就去修改全服玩家的属性值。 服务器压力山大。

**方案设计**

采用读拉取+异步更新玩家信息表的方式去处理，当玩家在创建的时候，设置属性为null,然后查看详情时去读取配置表的配置，更新到玩家的attribute_config，后续，玩家没有的属性就追加，有的属性就通过复杂的比较（这里简单操作，属性已经有就用自己的，不使用配置的值）

**图解**

异步更新数据图

[![image-20240909233728253](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240909233728253.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240909233728253.png)

## 实战

**玩家1详情代码（生产者代码）**

```java
public PlayerInfoVO findById(Long id, Boolean isGetNewConf) {
    PlayerInfoVO PlayerInfoVO = PlayerInfoMapper.INSTANCE.entityToVO(checkEntityIsExistById(id));
    if (isGetNewConf) { //是否拉取最新的基础属性配置
        PlayerInfoVO.setAttributeConfig(dealAttributeConfig(PlayerInfoVO));
    }
    return PlayerInfoVO;
}

/**
 * 获取最新玩家属性配置表的配置信息,补充玩家属性
 *
 * @param vo
 * @return
 */
private String dealAttributeConfig(PlayerInfoVO vo) {
    // 获取玩家属性配置
    List<PlayerAttributeConfig> playerAttributeConfig = playerAttributeConfigRepo.findAll();
    Map<String, String> defaultMap = playerAttributeConfig.stream()
            .collect(Collectors.toMap(PlayerAttributeConfig::getAttributeKey, PlayerAttributeConfig::AttributeValue, (existing, replacement) -> existing));
    ObjectMapper objectMapper = new ObjectMapper();
    String confData = "";
    Map<String, String> mergedMap = new HashMap<>(defaultMap);
    try {
        String attributeConfig = vo.getAttributeConfig();
        if (VerifyUtils.isNull(configData)) {
            confData = objectMapper.writeValueAsString(mergedMap);
        } else {
            Map<String, String> configMap = null;
            configMap = objectMapper.readValue(configData, Map.class);
            mergedMap.putAll(configMap);
            attributeConfig = objectMapper.writeValueAsString(mergedMap);
        }

    } catch (JsonProcessingException e) {
        log.error("\n ====== 转换失败：{} =====\n", e.getMessage());
    }
    if (VerifyUtils.isNotNull(confData)) {
        log.info("\n ====== 玩家{} 更新配置信息 {} =====\n", vo.getId(), attributeConfig);
        sendUpdatePlayerInfoVOAttributeConfMessage(vo, confData);
    }
    return confData;
}

/**
 * 异步玩家的attribute_config(生产者代码)
 *
 * @param vo
 * @param attributeConf
 */
private void sendUpdatePlayerInfoVOAttributeConfMessage(PlayerInfoVO vo, String attributeConf) {
    PlayerInfoVO playerInfoVO = PlayerInfoMapper.INSTANCE.toEntity(vo);
    playerInfoVO.setAttributeConf(attributeConf);
    rabbitTemplate.convertAndSend(
            MQConst.EXCHANGE_NAME_UPDATE_PLAYER_INFO_ATTRIBUTE_CONF,
            MQConst.ROUTINGKEY_NAME_UPDATE_PLAYER_INFO_ATTRIBUTE_CONF,
            playerInfoVO
    );
}
```

**MQ常量定义**

```java
public class MQConst {

    public static final String EXCHANGE_NAME_UPDATE_PLAYER_INFO_ATTRIBUTE_CONF = "exchange.update.player.info.attribute.conf";

    public static final String QUEUE_NAME_UPDATE_PLAYER_INFO_ATTRIBUTE_CONF = "queue.update.player.info.attribute.conf";

    public static final String ROUTINGKEY_NAME_UPDATE_PLAYER_INFO_ATTRIBUTE_CONF = "routingKey.update.player.info.attribute.conf";

}
```

**交换机**

```java
@Configuration
public class RabbitConfig implements RabbitListenerConfigurer {
    /**
     * 更新玩家属性的交换机
     * @return
     */
      @Bean
    public TopicExchange updatePlayerInfoAttributeConfExchange() {
        return new TopicExchange(EXCHANGE_NAME_UPDATE_PLAYER_INFO_ATTRIBUTE_CONF);
    }
}
```

**队列**

```java
@Configuration
public class RabbitConfig implements RabbitListenerConfigurer {
    /**
     * 更新玩家属性的队列
     * @return
     */
      @Bean
    public Queue updatePlayerInfoAttributeConfQueue() {
        return new Queue(QUEUE_NAME_UPDATE_UPDATE_PLAYER_INFO_ATTRIBUTE_CONF);
    }
}
```

**绑定对象**

```java
@Configuration
public class RabbitConfig implements RabbitListenerConfigurer {
    /**
     *  绑定"更新玩家属性队列,交换机"
     * @return
     */
    @Bean
    public Binding bindingUpdateDeviceDirective(Queue updatePlayerInfoAttributeConfQueue, TopicExchange updatePlayerInfoAttributeConfExchange) {
        return BindingBuilder.bind(updatePlayerInfoAttributeConfQueue)
                .to(updatePlayerInfoAttributeConfExchange)
                .with("routingKey.info.device.attribute.#");// 路由key
    }
}
```

**消费者**

```java
@Component
@Slf4j
public class PlayerInfoAttributeConfConsumer {

    @Resource
    private PlayerInfo playerInfo;

    @RabbitListener(queues = QUEUE_NAME_UPDATE_PLAYER_INFO_ATTRIBUTE_CONF)
    public void handleUpdatePlayerInfoVOAttributeConfMessage(PlayerInfo message) {
        log.info("\n ==== 收到玩家跟新属性消息：{}  =====", message);
        try {
            playerInfo.save(message);
        } catch (Exception e) {
            log.error("异步更新玩家属性信息失败: {} - 原始消息: {}", e.getMessage(), message);
        }
    }


}
```

### 效果

当玩家点击详情拉取基本数据配置表的数据，通过算法计算封装到json作为玩家的属性，响应，并发送消息去异步跟新玩家表的配置属性
