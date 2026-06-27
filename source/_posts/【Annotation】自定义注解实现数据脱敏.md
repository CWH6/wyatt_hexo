---
title: 【Annotation】自定义注解实现数据脱敏
date: 2023-09-24 23:59:15
tags:
  - Annotation
  - 注解
  - SpringBoot
category: 
  - 后端
---



### 数据脱敏

数据脱敏，指对某些敏感信息通过脱敏规则进行数据的变形，实现`敏感隐私数据的可靠保护`。

需要进行数据脱敏的地方：如身份证号、手机号、卡号、客户号等个人信息

此处通过利用 springboot 自带的 `jackson 自定义序列化`实现。它的实现原理其实就是在`json 进行序列化渲染给前端时，进行脱敏`

### 导入hutool

```maven
<dependency>
      <groupId>cn.hutool</groupId>
       <artifactId>hutool-all</artifactId>
       <version>5.8.0</version>
</dependency>
```

现阶段版本`Hutool支持的脱敏数据类型`如下，基本覆盖了常见的敏感信息。

具体如：用户id 、中文姓名、身份证号、座机号、手机号、地址、电子邮件、密码、中国大陆车牌，包含普通车辆、新能源车辆、银行卡

#### 测试

```java
/**
 *
 * @description: Hutool实现数据脱敏
 */
@SpringBootTest
public class HuToolDesensitizationTest {


    @Test
    public void testPhoneDesensitization(){
        String phone="13723231234";
        System.out.println(DesensitizedUtil.mobilePhone(phone)); //输出：137****1234
    }
    @Test
    public void testBankCardDesensitization(){
        String bankCard="6217000130008255666";
        System.out.println(DesensitizedUtil.bankCard(bankCard)); //输出：6217 **** **** *** 5666
    }

    @Test
    public void testIdCardNumDesensitization(){
        String idCardNum="411021199901102321";
        //只显示前4位和后2位
        System.out.println(DesensitizedUtil.idCardNum(idCardNum,4,2)); //输出：4110************21
    }
    @Test
    public void testPasswordDesensitization(){
        String password="www.jd.com_35711";
        System.out.println(DesensitizedUtil.password(password)); //输出：****************
    }


}
```

### 脱敏枚举

```java
/**
 * 脱敏类型枚举
 */
public enum DesensitizationTypeEnum {
    //自定义
    MY_RULE,
    //用户id
    USER_ID,
    //中文名
    CHINESE_NAME,
    //身份证号
    ID_CARD,
    //座机号
    FIXED_PHONE,
    //手机号
    MOBILE_PHONE,
    //地址
    ADDRESS,
    //电子邮件
    EMAIL,
    //密码
    PASSWORD,
    //中国大陆车牌，包含普通车辆、新能源车辆
    CAR_LICENSE,
    //银行卡
    BANK_CARD
}
```

### 脱敏注解

> @Retention(RetentionPolicy.RUNTIME)：运行时生效。
>
> @Target(ElementType.FIELD)：可用在字段上。
>
> @JacksonAnnotationsInside：此注解可以点进去看一下是一个元注解，主要是用户打包其他注解一起使用。
>
> @JsonSerialize：上面说到过，该注解的作用就是可自定义序列化，可以用在注解上，方法上，字段上，类上，运行时生效等等，根据提供的序列化类里面的重写方法实现自定义序列化。

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside
@JsonSerialize(using = DesensitizationSerialize.class)
public @interface Desensitization {
    /**
     * 脱敏数据类型，在MY_RULE的时候，startInclude和endExclude生效
     */
    DesensitizationTypeEnum type() default DesensitizationTypeEnum.MY_RULE;

    /**
     * 脱敏开始位置（包含）
     */
    int startInclude() default 0;

    /**
     * 脱敏结束位置（不包含）
     */
    int endExclude() default 0;

}
```

### 自定义序列化类

这一步是我们实现数据脱敏的关键。自定义序列化类继承 JsonSerializer，实现ContextualSerializer接口，并重写两个方法。

```java
/**
 * @author
 * @description: 自定义序列化类
 */
@AllArgsConstructor
@NoArgsConstructor
public class DesensitizationSerialize extends JsonSerializer<String> implements ContextualSerializer {
    private DesensitizationTypeEnum type;

    private Integer startInclude;

    private Integer endExclude;

    @Override
    public void serialize(String str, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        switch (type) {
            // 自定义类型脱敏
            case MY_RULE:
                jsonGenerator.writeString(CharSequenceUtil.hide(str, startInclude, endExclude));
                break;
            // userId脱敏
            case USER_ID:
                jsonGenerator.writeString(String.valueOf(DesensitizedUtil.userId()));
                break;
            // 中文姓名脱敏
            case CHINESE_NAME:
                jsonGenerator.writeString(DesensitizedUtil.chineseName(String.valueOf(str)));
                break;
            // 身份证脱敏
            case ID_CARD:
                jsonGenerator.writeString(DesensitizedUtil.idCardNum(String.valueOf(str), 1, 2));
                break;
            // 固定电话脱敏
            case FIXED_PHONE:
                jsonGenerator.writeString(DesensitizedUtil.fixedPhone(String.valueOf(str)));
                break;
            // 手机号脱敏
            case MOBILE_PHONE:
                jsonGenerator.writeString(DesensitizedUtil.mobilePhone(String.valueOf(str)));
                break;
            // 地址脱敏
            case ADDRESS:
                jsonGenerator.writeString(DesensitizedUtil.address(String.valueOf(str), 8));
                break;
            // 邮箱脱敏
            case EMAIL:
                jsonGenerator.writeString(DesensitizedUtil.email(String.valueOf(str)));
                break;
            // 密码脱敏
            case PASSWORD:
                jsonGenerator.writeString(DesensitizedUtil.password(String.valueOf(str)));
                break;
            // 中国车牌脱敏
            case CAR_LICENSE:
                jsonGenerator.writeString(DesensitizedUtil.carLicense(String.valueOf(str)));
                break;
            // 银行卡脱敏
            case BANK_CARD:
                jsonGenerator.writeString(DesensitizedUtil.bankCard(String.valueOf(str)));
                break;
            default:
        }

    }

    @Override
    public JsonSerializer<?> createContextual(SerializerProvider serializerProvider, BeanProperty beanProperty) throws JsonMappingException {
        if (beanProperty != null) {
            // 判断数据类型是否为String类型
            if (Objects.equals(beanProperty.getType().getRawClass(), String.class)) {
                // 获取定义的注解
                Desensitization desensitization = beanProperty.getAnnotation(Desensitization.class);
                // 为null
                if (desensitization == null) {
                    desensitization = beanProperty.getContextAnnotation(Desensitization.class);
                }
                // 不为null
                if (desensitization != null) {
                    // 创建定义的序列化类的实例并且返回，入参为注解定义的type,开始位置，结束位置。
                    return new DesensitizationSerialize(desensitization.type(), desensitization.startInclude(),
                            desensitization.endExclude());
                }
            }

            return serializerProvider.findValueSerializer(beanProperty.getType(), beanProperty);
        }
        return serializerProvider.findNullValueSerializer(null);
    }
}
```

### 测试

#### 接口

```java
@GetMapping("desensitization")
@ApiOperation(value = "数据脱敏")
@PreAuthorize("hasAnyAuthority('system:test:list')")
public R<TestVo> desensitization(){
        return debugManager.desensitization();
}
```

#### 业务层

```java
public R<TestVo> desensitization() {
   TestVo testVo = new TestVo();
   testVo.setUserName("我是用户名");
   testVo.setAddress("地球中国-北京市通州区京东总部2号楼");
   testVo.setPhone("13782946666");
   testVo.setPassword("sunyangwei123123123.");
   return success(testVo);
}
```

#### 实体类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class TestVo {

    private String userName;

    @Desensitization(type = DesensitizationTypeEnum.MOBILE_PHONE)
    private String phone;

    @Desensitization(type = DesensitizationTypeEnum.PASSWORD)
    private String password;

    @Desensitization(type = DesensitizationTypeEnum.MY_RULE, startInclude = 0, endExclude = 2)
    private String address;

}
```

### 响应

```json
{
    "code": 0,
    "message": "",
    "data": {
        "userName": "我是用户名",
        "phone": "137****6666",
        "password": "********************",
        "address": "**中国-北京市通州区京东总部2号楼"
    }
}
```
