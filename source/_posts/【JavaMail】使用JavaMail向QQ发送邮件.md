---
title: 【JavaMail】使用JavaMail向QQ发送邮件
date: 2024-07-25 00:01:07
tags:
  - JavaMail
category: 
  - 后端
---



## 概述

JavaMail 是一个用于发送和接收电子邮件的 Java API。它提供了一个平台无关和协议无关的框架，允许开发人员通过标准电子邮件协议（如 SMTP、POP3 和 IMAP）来创建、发送和读取电子邮件。以下是 JavaMail 的一些关键概念和功能介绍：

> 基本概念

**Session**: JavaMail 的 `Session` 对象表示邮件会话。它存储了配置信息，如邮件服务器地址和认证信息。

**Store**: `Store` 对象用于与邮件服务器通信，特别是接收邮件时。它支持协议如 IMAP 和 POP3。

**Transport**: `Transport` 对象用于发送邮件。它支持协议如 SMTP。

**Message**： `Message` 对象代表一封电子邮件。JavaMail 提供了 `MimeMessage` 类，用于创建和解析 MIME 类型的电子邮件。

**Folder**：`Folder` 对象代表邮件文件夹，如收件箱、发件箱、草稿等。通过 `Folder` 对象可以操作邮件。

## 实现

### 步骤

发送电子邮件的基本步骤包括

1、创建一个 `Session` 对象。

2、使用 `MimeMessage` 创建电子邮件内容。

3、使用 `Transport` 对象发送邮件。

### 具体

#### 引入依赖

```maven
<dependency>
     <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

#### 获取qq邮箱授权码

账号-> 安全设置 -> 生成授权码， 得到一串字符串编码

[![image-20240725184342857](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240725184342857.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240725184342857.png)

#### 代码

```java
    public static void main(String[] args) {
        // 配置邮件服务器属性
        Properties props = new Properties();
        props.put("mail.smtp.host", "smtp.qq.com");
        props.put("mail.smtp.ssl.protocols", "TLSv1.2");
        props.put("mail.smtp.port", "587"); // 或者使用 465 端口，并启用 SSL
        props.put("mail.smtp.auth", "true");
        props.put("mail.smtp.starttls.enable", "true"); // 启用 TLS

        // QQ 邮箱账户信息
        final String username = "xxxxx@qq.com"; // 您的QQ邮箱
        final String password = "xxxxxx"; // 您的QQ邮箱授权码

        // 创建会话
        Session session = Session.getInstance(props, new Authenticator() {
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(username, password);
            }
        });

        try {
            // 创建消息对象
            Message message = new MimeMessage(session);
            message.setFrom(new InternetAddress(username));
            message.setRecipients(Message.RecipientType.TO, InternetAddress.parse("xxxx@qq.com")); // 收件人邮箱地址
            message.setSubject("Test Email from QQ");

            // 创建富文本内容
            String htmlContent = "<h1>Hello,</h1>" +
                    "<p>This is a test email sent from <b>QQ Mail</b> using <i>JavaMail</i>!</p>" +
                    "<p><a href='https://www.example.com'>Visit Example</a></p>";

            // 设置邮件内容为 HTML 格式
            message.setContent(htmlContent, "text/html; charset=utf-8");

            // 发送邮件
            Transport.send(message);
            System.out.println("Email sent successfully!");
        } catch (MessagingException e) {
            e.printStackTrace();
        }
    }
}
```

## 测试

[![image-20240725185346835](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240725185346835.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240725185346835.png)
