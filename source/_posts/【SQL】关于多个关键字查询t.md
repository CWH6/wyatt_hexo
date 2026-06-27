---
title: 【SQL】关于多个关键字查询
date: 2023-09-27 23:56:57
tags:
  - SQL
category: 
  - 后端
---



## 业务场景

业务场景： 在一个列表页面中，对**输入框 输入一个到多个关键字**（以逗号隔开），

**再根据这些关键字和多个字段匹配符合的数据**

## sql编写

msql的`模糊查询是可以配合concat函数` 一起使用的

```sql
select
 name,class_name,address 
from 
  info_table 
where 
  concat(name,class_name,address) like oncat('%','天','%')
```

**关键字多个的情况，需要拼接多条**，如下

```sql
select 
 name,class_name,address 
from 
 info_table 
where
  concat(name,class_name,address) like oncat('%','天','%')
  or concat(name,class_name,address) like oncat('%','地','%')
  or concat(name,class_name,address) like oncat('%','一','%')
  or concat(name,class_name,address) like oncat('%','号','%')
```

## form

```java
@Getter
@Setter
@ToString
public class InfoForm implements Serializable {
   /**
   * 关键词
   */
   private List<String> keywords;
}
```

## daoImpl编写

```java
 @Override
    public List<Info> getInfoList(@Param("form") InfoForm form) {
        return this.sqlSessionTemplate.selectList(this.getNamespace() + ".getInfoList", form);
}
```

## xml编写

在xml中采 `\<foreach> 标签`

```sql
<select id="getInfoList" parameterType='com.test.from.InfoForm' resultType='com.test.Info'>
    SELECT 
        name,
        class_name,
        address 
    FROM 
        info_table
    WHERE
        is_delete = 0
    <if test="keywords != null and !keywords.isEmpty()">
         AND
        <foreach collection="keywords" item="keyword" open="" close="" separator=" OR ">
             CONCAT(name, class_name, address) LIKE CONCAT('%', #{keyword}, '%')
        </foreach>
    </if>
</select>    
```
