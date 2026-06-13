---
title: 【Mongodb】设置索引优化检索
date: 2026-04-17 15:21:09
tags:
---



## 索引概念

MongoDB索引是一种特殊数据结构（B-Tree），用于加速查询。它像书的目录，避免全表扫描，将查找时间从 O(n) 降至 O(log n)，大幅提升性能。



## 操作

### **查看索引**

```sql
// 查看所有索引
db.projects_answer.getIndexes()
```



### **添加索引**

#### **默认索引**

MongoDB 自动创建的主键索引，`_id`（文档唯一标识）按ID快速查询

```sql
{ key: { _id: 1 }, name: '_id_' }
```



#### 基础过滤索引

给指定字段添加索引，在基于这些字段查询时，提高检索效率。

 background: true， 设置在后台创建，不阻塞你的 Java 应用数据操作

```sql
use prod #切换数据库
# user_projects: 集合名字
db.user_projects.createIndex(
  { userId: 1, projectId: 1 },
  { background: true, name: "idx_userId_projectId" )
)
```



#### 深度结构索引

给多层数据结构添加索引，比如userInfo.name

```sql
db.projects_answer.createIndex(
  { projectId: 1, status: 1, "answers.questionId": 1, "answers.answer": 1 },
  { name: "projectId_1_status_1_answers.questionId_1_answers.answer_1" }
)
```



### 删除索引

```sql
db.projects_answer.dropIndex("projectId_1_status_1_answers.questionId_1_answers.answer_1")

db.projects_answer.dropIndex("idx_project_status_answers")
```



### 测试索引

```sql
use prod

// 1. 查看索引是否创建成功
print("=== 1. 验证索引存在 ===")
var indexes = db.user_projects.getIndexes();
var targetIndex = indexes.find(idx => idx.name === "idx_userId_projectId");
if (targetIndex) {
  print("✅ 索引存在: " + targetIndex.name);
  print("   字段: " + JSON.stringify(targetIndex.key));
} else {
  print("❌ 索引不存在");
}

// 2. 测试单字段查询(userId)
print("\n=== 2. 测试查询1: 只按 userId 查询 ===");
var result1 = db.user_projects.find({ userId: "test_user_123" }).explain("executionStats");
print("索引名: " + (result1.queryPlanner.winningPlan.inputStage?.indexName || result1.queryPlanner.winningPlan.indexName || "COLLSCAN"));
print("扫描文档数: " + result1.executionStats.totalDocsExamined);
print("返回文档数: " + result1.executionStats.nReturned);
print("执行时间: " + result1.executionStats.executionTimeMillis + "ms"); 
```

