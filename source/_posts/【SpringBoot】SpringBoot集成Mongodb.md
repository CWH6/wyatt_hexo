---
title: 【SpringBoot】SpringBoot集成Mongodb
date: 2026-06-30 11:41:09
tags:
  - SpringBoot
  - Mongodb
category: 
  - 后端
---



> 从依赖引入到实体声明，从 MongoTemplate 基础 CRUD 到 Aggregation Pipeline 更新，覆盖 ys-survey 模块 MongoDB 集成的完整实践。



## 💡 背景

问卷系统的答案数据量大、结构灵活（嵌套数组、动态字段），不适合传统关系型数据库。选择 MongoDB 的理由：

- 答卷数据是典型的文档模型（一次答题一条文档，内含嵌套 answers 数组）
- 项目备份、外部导入数据也是天然的文档型结构
- 无需 DDL，新增字段零成本
- 支持 Aggregation Pipeline，复杂查询不需要 JOIN



## 📦 Maven 依赖

在父 `pom.xml` 中统一管理版本，`ys-common` 模块引入：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

> 版本由 Spring Boot 父 POM 统一管理，无需手动指定。



## ⚙️ 配置

`bootstrap.yml` 中配置连接（最终从 Nacos 拉取）：

```yaml
spring:
  data:
    mongodb:
      host: 192.168.2.177
      port: 27017
      database: test
      username: testuser
      password: Test@123
```

Spring Boot 自动装配 `MongoTemplate` Bean，业务代码直接 `@Autowired` 注入即可。



## 🗂️ 集合（表）设计

ys-survey 模块使用 7 个 MongoDB 集合，以最核心的 `projects_answer` 为例：

| 集合名 | 对应实体 | 用途 |
|--------|---------|------|
| `projects_answer` | `ProjectAnswer` | 答卷数据，一次答题一条文档，内含嵌套 answers 数组 |



## 📝 实体类声明

### 标准实体（以 `ProjectAnswer` 为例）

```java
@Data
@NoArgsConstructor
@Document(collection = "projects_answer")    // ← 指定集合名
public class ProjectAnswer implements Serializable {

    private String id;                        // MongoDB 自动生成的 _id

    private Long projectId;                   // 项目 ID

    @Field("class")                           // ← 字段名与 Java 关键字冲突时显式映射
    private String clazz;                     // 答案类型

    private String ip;                        // IP 地址
    private String deviceType;                // 设备类型 PC/MOBILE
    private String device;                    // 具体设备

    private String ownerId;                   // 答题人 ID
    private String linkType;                  // 链接类型（test/prod1/prod2...）
    private Long version;                     // 版本号

    private String status;                    // 状态（active/done/out/stop）
    private String statusDesc;                // 状态描述

    @Field("createTime")
    private Date createTime;                  // 创建时间
    @Field("updateTime")
    private Date updateTime;                  // 修改时间
    @Field("lastAnswerTime")
    private Date lastAnswerTime;              // 最后答题时间

    private List<Answer> answers;             // ← 嵌套数组：每道题的答案
    private String currentQuestionNo;         // 当前答题进度
}
```

### 嵌套子文档（`Answer`）

```java
@Data
public class Answer {
    private String questionNo;     // 题号
    private String questionId;     // 题目 ID
    private String type;           // 题型
    private Object answer;         // 答案值（可以是 String / List / Map）
    private Date answerTime;       // 答题时间
}
```

### 简洁实体（以 `ProjectsExternalAnswer` 为例）

```java
@Data
@Document(collection = "projects_external_answer")
public class ProjectsExternalAnswer implements Serializable {

    private String id;

    @Field("projectId")
    private Long projectId;

    @Field("responseId")
    private String responseId;

    @Field("answer")
    private Map<String, String> answer;     // ← Map 直接映射为 BSON 子文档

    @Field("createTime")
    private Date createTime;
}
```

> 实体声明要点：`@Document` 指定集合名、`@Field` 处理特殊字段名、嵌套字段直接用 POJO 或 `Map`。



## 🔧 常量设计

`MongoTableConstant` 集中管理所有集合名和字段名的字符串常量，避免硬编码：

```java
public class MongoTableConstant {

    public static final String ID = "_id";

    // === projects_answer ===
    public static final String PROJECTS_ANSWER_COLLECTION = "projects_answer";
    public static final String PROJECTS_ANSWER_PROJECTID = "projectId";
    public static final String PROJECTS_OWNER_ID = "ownerId";
    public static final String PROJECTS_ANSWER = "answers";
    public static final String PROJECTS_ANSWER_STATUS = "status";
    public static final String PROJECTS_ANSWER_TYPE = "linkType";
    public static final String PROJECTS_ANSWER_VERSION = "version";
    public static final String PROJECTS_ANSWER_CREATE_TIME = "createTime";

    // === projects_external_answer ===
    public static final String PROJECTS_EXTERNAL_ANSWER_COLLECTION = "projects_external_answer";
    public static final String PROJECTS_EXTERNAL_ANSWER_PROJECT_ID = "projectId";
    public static final String PROJECTS_EXTERNAL_ANSWER_RESPONSE_ID = "responseId";

    // === projects_backup ===
    public static final String PROJECTS_BACKUP_COLLECTION = "projects_backup";
    public static final String PROJECTS_BACKUP_PROJECTID = "projectId";
    public static final String PROJECTS_BACKUP_VERSION = "version";

    // === projects_data_backup ===
    public static final String PROJECTS_DATA_BACKUP_COLLECTION = "projects_data_backup";

    // === projects_respondent ===
    public static final String PROJECTS_RESPONDENT_GUID = "guid";
    public static final String PROJECTS_RESPONDENT_PROJECTID = "projectId";
    public static final String PROJECTS_RESPONDENT_PANELID = "panelId";
    public static final String PROJECTS_RESPONDENT_STATUS = "status";
}
```

Service 层全部 `import static` 这些常量，代码中看不到裸字符串：

```java
import static com.ys.constant.MongoTableConstant.*;

mongoTemplate.find(query, ProjectAnswer.class, PROJECTS_ANSWER_COLLECTION);
```



## 🔑 基础 CRUD 操作

统一通过 `@Autowired MongoTemplate` 操作，不使用 Spring Data Repository 接口。

### 查 — `find` / `findOne`

```java
// 按条件查询列表
Query query = new Query(Criteria.where(PROJECTS_ANSWER_PROJECTID).is(projectId));
query.with(Sort.by(Sort.Direction.DESC, PROJECTS_ANSWER_CREATE_TIME));
List<ProjectAnswer> list = mongoTemplate.find(
    query, ProjectAnswer.class, PROJECTS_ANSWER_COLLECTION);

// 按 ID 查单条
Query query = new Query(Criteria.where(ID).is(id));
ProjectAnswer answer = mongoTemplate.findOne(
    query, ProjectAnswer.class, PROJECTS_ANSWER_COLLECTION);
```

### 增 — `save` / `insert`

```java
// 单条保存（upsert）
mongoTemplate.save(dto, PROJECTS_ANSWER_COLLECTION);

// 批量插入
mongoTemplate.insert(list, PROJECTS_EXTERNAL_ANSWER_COLLECTION);
```

### 删 — `remove` / `findAllAndRemove`

```java
// 按条件删除
Query query = new Query(Criteria.where(PROJECTS_EXTERNAL_ANSWER_PROJECT_ID).is(projectId));
mongoTemplate.remove(query, ProjectsExternalAnswer.class, PROJECTS_EXTERNAL_ANSWER_COLLECTION);

// 删除并返回（逻辑删除场景）
mongoTemplate.findAllAndRemove(buildQuery(params), PROJECTS_ANSWER_COLLECTION);
```

### 计数

```java
Query query = new Query(Criteria.where(PROJECTS_EXTERNAL_ANSWER_PROJECT_ID).is(projectId));
long count = mongoTemplate.count(query, ProjectsExternalAnswer.class,
    PROJECTS_EXTERNAL_ANSWER_COLLECTION);
```

### 分页查询

```java
Query query = new Query(Criteria.where("projectId").is(projectId));
query.with(Sort.by(Sort.Direction.ASC, "createTime"));
query.skip((long) (pageNum - 1) * pageSize);   // ← skip
query.limit(pageSize);                            // ← limit
return mongoTemplate.find(query, ProjectsExternalAnswer.class, COLLECTION);
```



## 🚀 进阶操作

### 动态条件构建

`buildQuery()` 方法根据 DTO 有值字段动态拼 Criteria：

```java
private Query buildQuery(ProjectsAnswerDTO dto) {
    Criteria criteria = new Criteria();
    if (StringUtils.hasText(dto.getId())) {
        criteria.and(ID).is(dto.getId().trim());
    }
    if (Objects.nonNull(dto.getProjectId())) {
        criteria.and(PROJECTS_ANSWER_PROJECTID).is(dto.getProjectId());
    }
    if (Objects.nonNull(dto.getVersion())) {
        criteria.and(PROJECTS_ANSWER_VERSION).is(dto.getVersion());
    }
    if (Objects.nonNull(dto.getStartTime()) && Objects.nonNull(dto.getEndTime())) {
        criteria.and(PROJECTS_ANSWER_CREATE_TIME)
                .gte(dto.getStartTime())         // ← 时间范围查询
                .lte(dto.getEndTime());
    }
    Query query = new Query(criteria);
    query.with(Sort.by(Sort.Direction.DESC, PROJECTS_ANSWER_CREATE_TIME));
    return query;
}
```

### Aggregation Pipeline — 答案数组去重追加

`saveAnswerToMongo()` 用原生 BSON Pipeline 实现"同名题目覆盖、不同题目追加"：

```java
Bson idFilter = buildIdFilter(answerId);

List<Bson> pipeline = Arrays.asList(
    new Document("$set", new Document()
        .append("answers",
            new Document("$concatArrays", Arrays.asList(
                // 1. 过滤掉 answers 数组中已存在的同名题目
                new Document("$filter", new Document()
                    .append("input", new Document("$ifNull", Arrays.asList("$answers", List.of())))
                    .append("as", "item")
                    .append("cond", new Document("$ne", Arrays.asList("$$item.questionNo", newAnswer.getQuestionNo())))
                ),
                // 2. 追加新答案
                Collections.singletonList(newAnswerDoc)
            )))
        .append("currentQuestionNo", currentQuestionNo)
        .append("updateTime", new Date())
        .append("createTime", new Document("$ifNull", Arrays.asList("$createTime", new Date())))
    )
);

UpdateResult result = mongoTemplate.getCollection(PROJECTS_ANSWER_COLLECTION)
        .updateOne(idFilter, pipeline, new UpdateOptions().upsert(true));
```

> 一次 DB 操作完成"去重 + 追加 + 更新进度 + upsert"，避免了 read-modify-write 的并发问题。

### 批量写入 + 分批

```java
int inserted = 0;
List<Document> batch = new ArrayList<>(1000);
for (Document doc : answers) {
    batch.add(doc);
    if (batch.size() >= 1000) {
        mongoTemplate.getCollection(BACKUP_COLLECTION).insertMany(batch);
        inserted += batch.size();
        batch.clear();
    }
}
// 最后一批不足 1000 的
if (!batch.isEmpty()) {
    mongoTemplate.getCollection(BACKUP_COLLECTION).insertMany(batch);
    inserted += batch.size();
}
```

> 批量 insert 避免逐条写入的网络开销，1000 条一批是实践中的经验值。



## 📊 典型 Service 结构

以 `ProjectsExternalAnswerServiceImpl` 为范本，展示一个完整的 Mongo Service：

```
@Service
ProjectsExternalAnswerServiceImpl
  │
  ├── @Autowired MongoTemplate mongoTemplate
  │
  ├── listByProjectId(projectId)          → find + sort
  ├── pageListByProjectId(projectId, page, size) → find + skip + limit
  ├── countByProjectId(projectId)         → count
  ├── deleteByProjectId(projectId)        → remove
  └── saveAll(list)                       → insert
```

全部通过 `import static MongoTableConstant.*` 引用集合名和字段名，零硬编码。



## ⭐ 设计要点总结

1. **只用 MongoTemplate，不搞 Repository**：放弃 Spring Data 的魔法接口，所有查询显式可控
2. **常量统一管理**：`MongoTableConstant` 集中维护集合名和字段名，禁止硬编码字符串
3. **Pipeline 解决复杂更新**：答案数组去重追加用单一 Aggregation Pipeline，避免并发问题
4. **实体用 `@Document` + `@Field`**：Java 关键字冲突时用 `@Field` 显式映射，不给 Lombok 添乱
5. **批量操作分批提交**：大量 insert 时分批（1000 条/批），平衡性能和内存
6. **动态查询条件**：`Criteria` 按需拼装，不会查出多余数据

