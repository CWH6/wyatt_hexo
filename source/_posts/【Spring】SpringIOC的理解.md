---
title: 【Spring】SpringIOC的理解
date: 2026-06-26 23:22:33
tags:
  - Spring
category: 
  - 后端
---



## Spring IOC的理解

`spring ioc`是 spring 两大核心之一，`spring` 为我们提供了一个ioc容器，也就是`beanFactory`。

同时，ioc有个非常强大的功能，叫做di，也就是依赖注入，我们可以通过配置或者xml文件的方式将bean所依赖的对象通过name（名字）或者type（类别）注入进这个beanFactory中，正因为这个依赖注入，`实现类与依赖类之间的解耦`。

如果在一个复杂的系统中，类之间的依赖关系特别复杂，首先，这非常不利于后期代码的维护，ioc就很好的帮助我们解决了这个问题，它帮助我们维护了类与类之间的依赖关系，降低了耦合性，使我们的类不需要强依赖于某个类，而且，在spring容器启动的时候，spring容器会帮助我们自动的创建好所有的bean，这样，我们程序运行的过程中就不需要花费时间去创建这些bean，速度就快了许多。

#### 案例

服务类 UserService 依赖于 UserRepository

**不采用依赖注入的方式**

```java
// UserRepository.java
public class UserRepository {
    public void saveUser(String username) {
        System.out.println("Saving user: " + username);
    }
}

// UserService.java
public class UserService {
    private UserRepository userRepository;

    // 通过构造函数注入依赖
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public void createUser(String username) {
        // 业务逻辑
        userRepository.saveUser(username);
    }
}
```

**通过XML配置文件定义下面两个类之间关系**

```
<!-- applicationContext.xml -->
<beans>
    <!-- 配置 UserRepository bean -->
    <bean id="userRepository" class="com.example.UserRepository" />

    <!-- 配置 UserService bean，并注入 UserRepository -->
    <bean id="userService" class="com.example.UserService">
        <constructor-arg ref="userRepository" />
    </bean>
</beans>
```

使用Spring框架来加载这个XML配置文件，获取并获取 `UserService` 实例

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        // 加载 Spring 容器
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        // 获取 UserService 实例
        UserService userService = (UserService) context.getBean("userService");

        // 使用 UserService
        userService.createUser("john_doe");
    }
}
```
