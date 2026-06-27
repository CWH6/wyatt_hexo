---
title: 【Activiti7】工作流使用
date: 2023-12-21 23:29:06
tags:
  - Activiti7
  - 工作流
category: 
  - 后端
---

## 概述

ctiviti是一个工作流引警，activiti可以将**业务系统中复杂的业务流程抽取出来**，使用专门的**建模语言BPMN2.0进行定义**，**业务流程按照预先定义的流程进行执行**，实现了系统的流程由activiti进行管理，减少业务系统由于流程变更进行系统升级改造的工作量，**从而提高系统的健壮性，同时也减少了系统开发维护成本。**

## 表结构

| 表分类       | 表名                  | 解释                                               |
| ------------ | --------------------- | -------------------------------------------------- |
| 一般数据     |                       |                                                    |
|              | [ACT_GE_BYTEARRAY]    | 通用的流程定义和流程资源                           |
|              | [ACT_GE_PROPERTY]     | 系统相关属性                                       |
| 流程历史记录 |                       |                                                    |
|              | [ACT_HI_ACTINST]      | 历史的流程实例                                     |
|              | [ACT_HI_ATTACHMENT]   | 历史的流程附件                                     |
|              | [ACT_HI_COMMENT]      | 历史的说明性信息                                   |
|              | [ACT_HI_DETAIL]       | 历史的流程运行中的细节信息                         |
|              | [ACT_HI_IDENTITYLINKJ | 历史的流程运行过程中用户关系                       |
|              | [ACT_HI_PROCINST]     | 历史的流程实例                                     |
|              | [ACT_HI_TASKINST]     | 历史的任务实例                                     |
|              | [ACT_HI_VARINST]      | 历史的流程运行中的变量信息                         |
| 流程定义表   |                       |                                                    |
|              | [ACT_RE_DEPLOYMENT]   | 部署单元信息                                       |
|              | [ACT_RE_MODEL]        | 模型信息                                           |
|              | [ACT_RE_PROCDEF]      | 已部署的流程定义                                   |
| 运行实例表   |                       |                                                    |
|              | [ACT_RU_EVENT_SUBSCR] | 运行时事件                                         |
|              | [ACT_RU_EXECUTION]    | 运行时流程执行实例                                 |
|              | [ACT_RU_IDENTITYLINKJ | 运行时用户关系信息，存储任务节点与参与者的相关信息 |
|              | [ACT_RU_JOB]          | 运行时作业                                         |
|              | [ACT_RU_TASK]         | 运行时任务                                         |
|              | [ACT_RU_VARIABLE]     | 运行时变量表                                       |

## 类关系

| 类                | 作用                                                 |
| ----------------- | ---------------------------------------------------- |
| ProcessEngine     | 调用上面其他服务： RepositoryService,TaskService.... |
| RepositoryService | 操作跟 部署 相关的表，操作 re 表                     |
| RuntimeService    | 操作跟 运行 相关的表，操作 ru 表                     |
| HistoryService    | 操作跟 历史 相关的表，操作 hi 表                     |

## 流程部署

```
/**
* 测试流程部署
*/
@Test
public void testDeployment() {
    // 1、创建 ProcessEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    // 2、获取 RepositoryService
    RepositoryService repositoryService = processEngine.getRepositoryService();

    // 3、使用 service 进行流程的部署
    Deployment deployment = repositoryService.createDeployment()
            .name("出差申请流程")
            .addClasspathResource("bpmn/evection.bpmn") // 添加 BPMN 文件
            .addClasspathResource("bpmn/evection.png")  // 添加流程图 PNG 文件
            .deploy();

    // 4、输出部署信息
    System.out.println("流程部署ID: " + deployment.getId()); //
    System.out.println("流程部署名称: " + deployment.getName()); //出差申请流程
}
```

## 启动流程实例

```
@Test
public void testStartProcess() {
    // 1. 创建ProcessEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    // 2. 获取RuntimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();

    // 3. 根据流程定义的key启动流程实例
    ProcessInstance instance = runtimeService.startProcessInstanceByKey("myEvection");

    // 4. 输出信息
    System.out.println("流程定义ID: " + instance.getProcessDefinitionId());// myEvection: 1:4
    System.out.println("流程实例ID: " + instance.getId()); // 2501 
    System.out.println("当前活动的ID: " + instance.getActivityId()); // null
}
```

## 查看执行任务

```
@Test
public void testFindPersonalTaskList() {
    // 1. 获取流程引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    // 2. 获取TaskService
    TaskService taskService = processEngine.getTaskService();

    // 3. 根据流程Key和任务的负责人查询任务
    List<Task> taskList = taskService.createTaskQuery()
            .processDefinitionKey("myEvection") // 要查询的流程定义Key
            .taskAssignee("zhangsan") // 要查询的负责人
            .list();

    // 4. 输出任务信息
    for (Task task : taskList) {
        System.out.println("流程实例id=" + task.getProcessInstanceId());
        System.out.println("任务Id=" + task.getId());
        System.out.println("任务负责人=" + task.getAssignee());
        System.out.println("任务名称=" + task.getName());
    }
}
```

## 发起申请

```
@Test
public void completeTask() {
    // 获取流程引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    // 获取TaskService
    TaskService taskService = processEngine.getTaskService();

    // 根据任务id完成任务
    String taskId = "2505"; // 请替换为您要完成的任务ID
    taskService.complete(taskId);
}
```

## 上级审批

```
@Test
public void completeTask() {
    // 获取引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    // 获取操作任务的服务 TaskService
    TaskService taskService = processEngine.getTaskService();

    // 完成任务，参数：任务id，完成zhangsan的任务
    taskService.complete("2505");

    // 获取jerry-myEvection对应的任务
    Task task = taskService.createTaskQuery()
            .processDefinitionKey("myEvection")
            .taskAssignee("jerry")  //经理名字   后续修改这个完成其他人多的审批任务
            .singleResult();

    System.out.println("流程实例id=" + task.getProcessInstanceId());
    System.out.println("任务Id=" + task.getId());
    System.out.println("任务负责人=" + task.getAssignee());
    System.out.println("任务名称=" + task.getName());

    // 完成jerry的任务
    taskService.complete(task.getId());
}
```

