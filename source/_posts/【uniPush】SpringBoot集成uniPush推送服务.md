---
title: 【uniPush】SpringBoot集成uniPush推送服务
date: 2024-08-19 00:55:35
tags:
  - uniPush
category: 
  - 前端
---

## 介绍

**UniPush** 是个推（Getui）推出的一项推送服务。它是一个`多平台推送解决方案`，旨在帮助开发者`统一管理`和`发送推送通知`，无论用户使用的是 Android、iOS 还是其他平台的设备。

**主要特点和功能**

**多平台支持**：UniPush 支持 Android、iOS 等主流平台，开发者只需要集成一次 SDK，就能实现跨平台的消息推送。

**高送达率**：个推的推送服务以高送达率著称，通过精准的推送通道和智能的推送策略，确保消息能够及时送达用户设备。

**精准推送**：支持根据用户标签、地理位置、行为等信息进行精准推送，提高消息的有效性和转化率。

**实时统计**：提供详尽的推送统计报告，开发者可以实时查看推送的送达率、点击率等数据，优化推送策略。

**丰富的消息类型**：支持文本消息、富媒体消息（图片、音频、视频）等多种形式的消息推送，满足不同应用场景的需求。

**开发者友好**：提供简单易用的 API 和 SDK，快速集成到应用中。同时，个推提供详细的文档和技术支持，帮助开发者解决集成过程中的问题。

**多种推送策略**：包括定时推送、地理围栏推送、用户分群推送等，开发者可以根据业务需求灵活选择合适的推送策略。

## 使用场景

**营销推广**：通过 UniPush 向用户发送促销、活动等通知，增加用户的活跃度和参与度。

**系统通知**：实时向用户推送重要的系统通知，如安全警告、账户变更等信息。

**用户唤醒**：向长时间未使用应用的用户发送提醒，唤醒用户再次使用应用

## 集成

**个推SDK**的主要目标是提升开发者在服务端集成个推推送服务的开发效率，满足调用过程中所需的鉴权、组装参数、发送HTTP请求等非功能性要求

下面的例子：`集成个推SDK` ，通过`定时任务` 或 `手动调用`，查询数据库中待推送的消息，并使用个推平台将消息推送到特定用户的设备上。推送成功后，会更新消息的状态，以防`重复推送`

### 依赖

```xml
<dependency>
    <groupId>com.getui.push</groupId>
    <artifactId>restful-sdk</artifactId>
    <version>1.0.0.4</version>
</dependency>
```

### yaml配置

```yaml
push:
  base_url: https://restapi.getui.com/v2/your_app_id (此处可以省略)
  app_id: your_app_id
  app_key: your_app_key
  app_secret: your_app_secret (此处可以省略)
  master_secret: your_master_secret
```

### 表

`T_Sys_Message` 表

```sql
CREATE TABLE T_Sys_Message (
    F_MessageID INT PRIMARY KEY AUTO_INCREMENT,  -- 消息ID，主键，自增
    F_Title VARCHAR(255),                        -- 消息标题
    F_Summary TEXT,                              -- 消息摘要
    F_LinkUrl VARCHAR(255),                      -- 消息链接URL
    F_IsPush TINYINT(1) DEFAULT 0,               -- 是否已经推送，布尔值
    F_Status TINYINT(1) DEFAULT 0,               -- 消息状态，0表示未处理
    F_RecieverUserID INT,                        -- 接收者的用户ID，外键指向用户表
    F_IsDeleted TINYINT(1) DEFAULT 0             -- 是否被删除，布尔值
);
```

`T_Sys_User` 表

```sql
CREATE TABLE T_Sys_User (
    F_UserID INT PRIMARY KEY AUTO_INCREMENT,     -- 用户ID，主键，自增
    F_ClientID VARCHAR(255),                     -- 客户端ID (通常是移动设备的唯一标识)
    F_FullName VARCHAR(255),                     -- 用户全名
    F_IsDeleted TINYINT(1) DEFAULT 0,            -- 用户是否被删除，布尔值
    F_AccountState TINYINT(1) DEFAULT 1,         -- 账号状态，1表示正常
    F_LockState TINYINT(1) DEFAULT 0,            -- 锁定状态，0表示未锁定，3表示已锁定
    F_UserState TINYINT(1) DEFAULT 1             -- 用户状态，1表示正常
);
```

外键约束

在 `T_Sys_Message` 表中，`F_RecieverUserID` 是一个外键，指向 `T_Sys_User` 表的 `F_UserID` 字段

```java
ALTER TABLE T_Sys_Message 
ADD CONSTRAINT fk_reciever_user 
FOREIGN KEY (F_RecieverUserID) 
REFERENCES T_Sys_User(F_UserID);
```

### 相关类

**个推配置类**

```java
@Configuration
public class GTPushConfig {
    @Value("${push.app_id}")
    private String appId;

    @Value("${push.app_key}")
    private String appKey;

    @Value("${push.master_secret}")
    private String masterSecret;

    @Bean(name = "myApiHelper")
    public ApiHelper apiHelper() {
        // 设置httpClient最大连接数，当并发较大时建议调大此参数。或者启动参数加上 -Dhttp.maxConnections=200
        System.setProperty("http.maxConnections", "200");

        GtApiConfiguration apiConfiguration = new GtApiConfiguration();
        // 填写应用配置
        apiConfiguration.setAppId(appId);
        apiConfiguration.setAppKey(appKey);
        apiConfiguration.setMasterSecret(masterSecret);
        // 接口调用前缀，请查看文档: 接口调用规范 -> 接口前缀, 可不填写appId
        // 默认为https://restapi.getui.com/v2
        apiConfiguration.setDomain("https://restapi.getui.com/v2/");
        // 实例化ApiHelper对象，用于创建接口对象
        ApiHelper apiHelper = ApiHelper.build(apiConfiguration);
        return apiHelper;
    }
}
```

**controller**

```java
@RestController
@RequestMapping("/sys-message")
public class SysMessageController {

    @Autowired
    private SysMessageService sysMessageService;

    /**
     * 消息推送服务端接口，测试用
     */
    @NoAuthentication
    @RequestMapping(value = "/pushMess", method = RequestMethod.GET)
    public void forwardNews(){
        sysMessageService.forwardNews();
    }
}
```

**service**

```java
public interface SysMessageService extends IService<SysMessage> {

    void forwardNews();
}
```

**serviceImpl**

```java
@Slf4j
@Service
public class SysMessageServiceImpl extends ServiceBaseImpl<SysMessageMapper, SysMessage> implements SysMessageService {

    @Resource
    private GeTuiUtils geTuiUtils;

    @Autowired
    private SysMessageMapper sysMessageMapper;

    /**
     * 集成个推推送消息
     * 通过定时任务（每2分钟执行一次）或手动调用api触发
     */
    @Override
    @Scheduled(cron = "0 0/2 * * * ?")
    public void forwardNews(){
        List<Map<String, Object>> maps = sysMessageMapper.selectMessages();
        if (maps.size() > 0) {
            maps.forEach(map -> {
                log.info("正在推送：{}", map);
                // System.out.println(map);
                // 获取推送信息
                 String cid = (String) map.get("F_ClientID");
                 String title = (String) map.get("F_Title");
                 String content = (String) map.get("F_Summary");
                 String linkUrl = (String) map.get("F_LinkUrl");
                // type = 1 消息推送
                String type = GeTuiUtils.MESSAGE_PUSH;
                ApiResult<Map<String, Map<String, String>>> mapApiResult = geTuiUtils.pushToSingleByCid(cid, title, content, linkUrl, type);
                // 修改推送消息表
                map.put("F_IsPush", 1);
                SysMessage sysMessage = null;
                try {
                    sysMessage = this.mapToEntity(map, SysMessage.class, Other);
                } catch (IllegalAccessException | InstantiationException e) {
                    e.printStackTrace();
                }
                // 判断消息是否推送成功打印日志更新数据库状态
                if (mapApiResult.isSuccess()) {
                    sysMessageMapper.updateById(sysMessage);
                    log.info("成功推送至：{}", map.get("F_FullName"));
                } else {
                    log.error("无法推送消息至：{},请检查后重试", map.get("F_FullName"));
                }

            });
        } else {
            log.info("没有消息需要推送");
        }
    }
}
```

**mapper**

```java
@Mapper
public interface SysMessageMapper extends MapperBase<SysMessage> {

    // 这个查询的目的是找到符合条件的消息和用户信息，通常用于推送尚未推送的消息给特定用户
    @Select("SELECT\n" +
            "\tm.F_MessageID,\n" +
            "\tm.F_Title,\n" +
            "\tm.F_Summary,\n" +
            "\tm.F_LinkUrl,\n" +
            "\tm.F_IsPush,\n" +
            "\tm.F_Status,\n" +
            "\tu.F_ClientID,\n" +
            "\tu.F_FullName \n" +
            "FROM\n" +
            "\tT_Sys_Message m\n" +
            "\tLEFT JOIN T_Sys_User u ON m.F_RecieverUserID= u.F_UserID \n" +
            "WHERE\n" +
            "\tm.F_IsPush= 0 \n" +
            "\tAND m.F_Status = 0\n" +
            "\tAND m.F_IsDeleted= 0 \n" +
            "\tAND u.F_ClientID IS NOT NULL\n" +
            "\tAND u.F_IsDeleted = 0\n" +
            "\tAND u.F_AccountState = 1\n" +
            "\tAND u.F_LockState != 3\n" +
            "\tAND u.F_UserState = 1")
    List<Map<String, Object>> selectMessages();
}
```

**工具类**

```java
@Slf4j
@Component
public class GeTuiUtils {
    // 消息推送
    public static final String MESSAGE_PUSH = "1";
    // 离线推送
    public static final String OFFLINE_PUSH = "2";

    @Resource(name = "myApiHelper")
    private ApiHelper myApiHelper;

    /**
     * 消息推送（离线推送）单cid推送
     *
     */
    public ApiResult<Map<String, Map<String, String>>> pushToSingleByCid(String cid, String title, String content, String linkUrl, String type) {
        PushDTO<Audience> pushDTO = this.buildPushDTO(title, content, linkUrl, type);
        // 设置接收人信息
        Audience audience = new Audience();
        pushDTO.setAudience(audience);
        audience.addCid(cid); // cid
        // 进行cid单推
        PushApi pushApi = myApiHelper.creatApi(PushApi.class);
        ApiResult<Map<String, Map<String, String>>> apiResult = pushApi.pushToSingleByCid(pushDTO);
        if (apiResult.isSuccess()) {
            // success
            log.info("推送成功");
            System.out.println(apiResult.getData());
        } else {
            // failed
            log.error("推送失败");
            System.out.println("code:" + apiResult.getCode() + ", msg: " + apiResult.getMsg());
        }
        return apiResult;
    }

    /**
     * 消息参数模板
     */
    private PushDTO<Audience> buildPushDTO(String title, String content, String linkUrl, String type) {
        PushDTO<Audience> pushDTO = new PushDTO<>();
        // 设置推送参数
        //requestid需要每次变化唯一
        pushDTO.setRequestId(System.currentTimeMillis() + "");
        pushDTO.setGroupName("wxb-group");

        // 消息通知
        //GTNotification notification = new GTNotification();
        //pushMessage.setNotification(notification);
        //notification.setTitle(title);
        //notification.setBody(content);
        //android8.0以上
        //0：无声音，无振动，不显示；
        //1：无声音，无振动，锁屏不显示，通知栏中被折叠显示，导航栏无logo;
        //2：无声音，无振动，锁屏和通知栏中都显示，通知不唤醒屏幕;
        //3：有声音，无振动，锁屏和通知栏中都显示，通知唤醒屏幕;
        //4：有声音，有振动，亮屏下通知悬浮展示，锁屏通知以默认形式展示且唤醒屏幕;
        // notification.setChannelLevel("3");



        //notification.setClickType("payload");
        //notification.setPayload(payload);
        //notification.setBadgeAddNum("1");
        /* 设置个推通道参数，更多参数请查看文档或对象源码 */

        //配置推送条件
        // 1: 表示该消息在用户在线时推送个推通道，用户离线时推送厂商通道;
        // 2: 表示该消息只通过厂商通道策略下发，不考虑用户是否在线;
        // 3: 表示该消息只通过个推通道下发，不考虑用户是否在线；
        // 4: 表示该消息优先从厂商通道下发，若消息内容在厂商通道代发失败后会从个推通道下发。
        Strategy strategy = new Strategy();
        strategy.setDef(1);
        strategy.setSt(1);
        Settings settings = new Settings();
        settings.setStrategy(strategy);
        pushDTO.setSettings(settings);
        //消息有效期，走厂商消息需要设置该值
        settings.setTtl(3600000);

        PushChannel pushChannel = new PushChannel();

        Map<String, String> map = new HashMap<>();
        map.put("title", title);
        map.put("content", content);
        map.put("linkUrl", linkUrl);
        map.put("type", type);
        // 转json对象
        String payload = JSONUtils.toJSONString(map);

        // ========================= ios离线配置 ==============================
        //推送苹果离线通知标题内容
        Alert alert = new Alert();
        //苹果离线通知栏标题
        alert.setTitle(title);
        //苹果离线通知栏内容
        alert.setBody(content);
        Aps aps = new Aps();
        //1表示静默推送(无通知栏消息)，静默推送时不需要填写其他参数。
        // 苹果建议1小时最多推送3条静默消息
        aps.setContentAvailable(0);
        aps.setSound("default");
        aps.setAlert(alert);
        IosDTO iosDTO = new IosDTO();
        iosDTO.setPayload(payload);
        iosDTO.setAps(aps);
        iosDTO.setType("notify");
        pushChannel.setIos(iosDTO);

        // =================== 安卓离线厂商通道推送消息体 ===========================
        AndroidDTO androidDTO = new AndroidDTO();
        pushDTO.setPushChannel(pushChannel);
        pushChannel.setAndroid(androidDTO);
        Ups ups = new Ups();
        androidDTO.setUps(ups);
        ThirdNotification notification1 = new ThirdNotification();
        ups.setNotification(notification1);
        //安卓离线展示的标题
        notification1.setTitle(title);
        //安卓离线展示的内容
        notification1.setBody(content);
        notification1.setClickType("intent");

        notification1.setIntent("intent:#Intent;action=android.intent.action.oppopush;" +
                "launchFlags=0x14000000;" +
                "component=包名/io.dcloud.PandoraEntry;S.UP-OL-SU=true;" +
                "S.title=" + title + ";" +
                "S.content="+  content + ";" +
                "S.payload=" + payload + ";end");
        // notification1.setPayload(payload);
        //各厂商自有功能单项设置
//        ups.addOption("HW", "/message/android/notification/badge/class", "io.dcloud.PandoraEntry ");
//        ups.addOption("HW", "/message/android/notification/badge/add_num", 1);
//        ups.addOption("HW", "/message/android/notification/importance", "HIGH");
//        ups.addOption("VV","classification",1);

        //设置options 方式一
//        ups.addOption("HW","badgeAddNum",3);
//        ups.addOption("HW","badgeClass","com.getui.demo.GetuiSdkDemoActivity");
//        ups.addOption("OP","app_message_id",11);
//        ups.addOption("VV","message_sort",1);
//        ups.addOptionAll("channel","default");

        // PushMessage在线走个推通道才会起作用的消息体
        PushMessage pushMessage = new PushMessage();
        pushDTO.setPushMessage(pushMessage);
        Map<String, Object> mapTC = new HashMap<>();
        mapTC.put("title", title);
        mapTC.put("content", content);
        mapTC.put("payload", map);
        String jsonTC = JSONUtils.toJSONString(mapTC);
        pushMessage.setTransmission(jsonTC);
        log.info("pushDTO:{}", pushDTO);
        return pushDTO;
    }

    /**
     * 离线通知消息参数模板
     *
     */
    private PushDTO<Audience> offlinePushDTO(String title, String content, String linkUrl, String type) {
        PushDTO<Audience> pushDTO = new PushDTO<>();
        // 设置推送参数
        //requestid需要每次变化唯一
        pushDTO.setRequestId(System.currentTimeMillis() + "");
        pushDTO.setGroupName("wxb-group");
        // PushMessage在线走个推通道才会起作用的消息体

        Map<String, String> map = new HashMap<>();
        map.put("title", title);
        map.put("content", content);
        map.put("linkUrl", linkUrl);
        map.put("type", type);

        PushMessage pushMessage = new PushMessage();
        pushDTO.setPushMessage(pushMessage);
        Map<String, Object> mapTC = new HashMap<>();
        mapTC.put("title", title);
        mapTC.put("content", content);
        mapTC.put("payload", map);
        String jsonTC = JSONUtils.toJSONString(mapTC);
        System.err.println(jsonTC);
        pushMessage.setTransmission(jsonTC);
        log.info("pushDTO:{}", pushDTO);

        return pushDTO;
    }
}
```

## 更多文档

[参考 uniapp官网文档](https://uniapp.dcloud.net.cn/unipush-v2.html)

[参考 uni-app 集成推送](https://juejin.cn/post/7267417057451573304?searchId=20240819201433FA39DC3028554F52D06B)

[参考 SpringBoot项目集成UniPush 推送服务](https://blog.csdn.net/qq_43375080/article/details/125663881)
