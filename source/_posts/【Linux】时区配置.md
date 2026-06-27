---
title: 【Linux】时区配置
date: 2024-09-02 23:52:32
tags:
  - Linux
categories:
  - 运维
---



### 场景

今日看日志的时候，发现日志的时间跟当前时间不一样，后面发现就是当前Linux系统的时区不对，java的logback-spring 是用linux的时区的时间

### 操作

**1、检查服务器当前时区**

```shell
timedatectl
```

结果显示时区为`Etc/UTC (UTC, +0000)` , 应该采用 `Asia/Shanghai (CST, +0800)`

```shell
               Local time: Mon 2024-09-02 01:42:20 UTC
           Universal time: Mon 2024-09-02 01:42:20 UTC
                 RTC time: Mon 2024-09-02 01:42:04    
                Time zone: Etc/UTC (UTC, +0000)       
System clock synchronized: no                         
              NTP service: inactive                   
          RTC in local TZ: no 
```

**2、检查服务器当前时间**

```shell
date
```

结果显示时间为`01:41:06`, 实际时间应该是 `09:42:50`

```shell
Mon Sep  2 01:41:06 CST 2024
```

**3、设置服务器时区**

```shell
sudo timedatectl set-timezone Asia/Shanghai
```

**4、检查时区是否生效**

```shell
timedatectl
```

结果显示为 `Asia/Shanghai (CST, +0800)`, 说明配置成功了

```shell
               Local time: Mon 2024-09-02 09:34:59 CST
           Universal time: Mon 2024-09-02 01:34:59 UTC
                 RTC time: Mon 2024-09-02 01:34:59    
                Time zone: Asia/Shanghai (CST, +0800) 
System clock synchronized: yes                        
              NTP service: active                     
          RTC in local TZ: no  
```
