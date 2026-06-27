---
title: 【MapStruct】基于MapStruct实现类之间转换
date: 2024-08-14 23:42:54
tags:
  - MapStruct
category: 后端
---



## 介绍

MapStruct 是一个注解处理器，它可以在编译时自动生成实现对象之间映射的代码。它通过使用注解来定义映射关系，减少了手动编写转换代码的繁琐工作

## 使用场景

用于将不同类型的对象相互转换（例如从实体对象到数据传输对象DTO，或者从实体对象到视图对象VO）

## 使用

在 Spring Boot 项目中使用的步骤如下

### 引入 MapStruct

**maven**

使用 Maven 构建工具，可以在 `pom.xml` 中添加以下依赖

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.3.Final</version>
</dependency>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.5.3.Final</version>
    <scope>provided</scope>
</dependency>
```

**gradle**

项目使用 Gradle 构建工具，可以在 `build.gradle` 中添加以下依赖

```shell
implementation 'org.mapstruct:mapstruct:1.5.3.Final'
annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.3.Final'
```

### 编译项目

添加依赖后，重新编译项目，MapStruct 将自动生成相应的映射实现代码

**maven**

```shell
mvn clean compile
```

**gradle**

```shell
./gradlew clean build
```

### 编写mapper

编写映射实体类转换的mapper

```java
@Mapper
public interface AppleMapper {
    AppleMapper INSTANCE = Mappers.getMapper(AppleMapper.class);

    AppleVO entityToVO(Apple entity);

    Apple dtoToEntity(AppleDTO dto);

    List<AppleVO> entityListToVOList(List<Apple> entityList);

    Apple update(@MappingTarget Apple apple, AppleDTO dto);
}
```

### 使用

```java
// 将实体转换为VO
Apple apple = new Apple();    
AppleVO applevo = AppleMapper.INSTANCE.entityToVO(apple);

// 将DTO转换为实体
AppleDTO appledto = new AppleDTO();
Apple apple = AppleMapper.INSTANCE.dtoToEntity(appledto);
```
