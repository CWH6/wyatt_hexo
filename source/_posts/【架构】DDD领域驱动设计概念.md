---
title: 【架构】DDD领域驱动设计概念
date: 2024-12-18 21:29:11
tags:
  - 架构
  - 设计
category: 架构
---

## 概念

**DDD**，全程是Domain-Driven Design，翻译过来就是`领域驱动设计`。

DDD是一个`软件设计理念`，通过`深入理解业务领域`，`来指导软件设计、和开发`。

DDD 强调`与业务专家紧密合作`，`将复杂的业务问题`，`转化为可管理的软件模型`。

## 作用

**1、满足业务需求**

通过与业务专家密切合作，DDD 确保软件模型准确反映业务需求，`减少了软件与业务之间的脱节`。

**2、增强设计**

将复杂业务逻辑，分解成领域模型、和限界上下文，使得系统设计更加模块化、和灵活。

**3、改善团队协作**

使用通用语言，使得业务专家、和开发团队，能够有效沟通、和协作，减少了误解、和沟通成本。

**4、支持业务变化**

DDD 强调领域模型的演化、和适应，允许系统随着业务需求的变化，而不断调整和优化。

## DDD四层架构

总共有四层：

1、用户界面层（UI/Presentation Layer）

2、应用层（Application Layer）

3、领域层（Domain Layer）

3、基础设施层（Infrastructure Layer）

这四层架构提供了一个清晰的层次结构，确保业务逻辑和技术细节之间的分离。

### 1、用户界面层

**「用户界面层」**的主要职责是`处理系统与用户的交互`。无论是`Web 页面`、`API 接口`还是`桌面应用`，所有用户请求和系统响应都通过这一层进行。

它通常不会包含任何业务逻辑，而是负责将用户的输入数据传递给应用层，并展示应用层返回的结果。用户界面层与应用层的交互通过 DTO（数据传输对象）来实现，确保两层之间的通信解耦。

```Java
@RestController
public class OrderController {
    private final OrderApplicationService orderService;

    public OrderController(OrderApplicationService orderService) {
        this.orderService = orderService;
    }

    @PostMapping("/orders")
    public ResponseEntity<String> createOrder(@RequestBody OrderDTO orderDTO) {
        orderService.createOrder(orderDTO);
        return ResponseEntity.ok("Order created successfully.");
    }
}
```

在这里，`OrderController` 负责接收用户输入，并将数据传递给 `OrderApplicationService` 处理。

### 2、应用层

**「应用层」**的职责是`协调用户界面层和领域层`。它`不直接处理业务逻辑，而是负责管理应用的流程`。应用层通过调用领域层中的对象来完成具体的业务操作，同时确保事务性和流程控制。

应用层是面向用例的，它知道如何组合领域层的各个部分以完成特定的业务需求。例如，`在一个订单系统中，应用层负责下单、取消订单等流程`。

```Java
@Service
public class OrderApplicationService {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;

    public OrderApplicationService(OrderRepository orderRepository, PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
    }

    public void createOrder(OrderDTO orderDTO) {
        Order order = new Order(orderDTO.getCustomerId(), orderDTO.getItems());
        orderRepository.save(order);
    }

    public void cancelOrder(Long orderId) {
        Order order = orderRepository.findById(orderId);
        order.cancel();
        orderRepository.save(order);
    }

    public void payOrder(Long orderId, PaymentDTO paymentDTO) {
        Order order = orderRepository.findById(orderId);
        paymentService.processPayment(order, paymentDTO);
        orderRepository.save(order);
    }
}
```

这里的 `OrderApplicationService` 不直接处理业务逻辑，而是调用 `Order` 聚合根或 `PaymentService` 等领域服务来处理核心业务逻辑。

### 3、领域层

**「领域层」**是整个系统的核心，负责处理业务逻辑。它包含了所有的实体、值对象、领域服务以及聚合根。领域层定义了系统的业务规则和行为，确保数据的一致性，并且完全独立于其他层。

在领域层中，最重要的概念是**「聚合根」**。它是领域模型的一个重要组成部分，负责管理聚合内部的一致性和业务操作。每个聚合根都会拥有相关的实体和值对象，聚合根通过仓储（Repository）进行持久化操作。

```Java
public class Order {
    private Long orderId;
    private List<OrderItem> items;
    private OrderStatus status;
    private Customer customer;

    public Order(Long customerId, List<OrderItem> items) {
        this.customer = new Customer(customerId);
        this.items = items;
        this.status = OrderStatus.CREATED;
    }

    public void cancel() {
        if (this.status == OrderStatus.PAID) {
            throw new IllegalStateException("Cannot cancel a paid order");
        }
        this.status = OrderStatus.CANCELED;
    }

    public void pay(Money payment) {
        if (this.status != OrderStatus.CREATED) {
            throw new IllegalStateException("Cannot pay for an order that is not created");
        }
        this.status = OrderStatus.PAID;
    }
}
```

`Order` 作为聚合根，负责订单的状态管理和操作逻辑。领域层的业务逻辑在这里被封装和执行。

### 4、基础设施层

**「基础设施层」**的主要职责是为其他层提供技术支持，包括数据库访问、消息队列、缓存、外部系统集成等。它实现了对底层技术的抽象，确保领域层和应用层不直接依赖具体的技术实现。

在 DDD 架构中，领域层通过仓储（Repository）模式与基础设施层进行交互，仓储负责将聚合根的持久化抽象化，隐藏了具体的存储实现细节。

```Java
@Repository
public class JpaOrderRepository implements OrderRepository {
    private final JpaRepository<Order, Long> jpaRepository;

    public JpaOrderRepository(JpaRepository<Order, Long> jpaRepository) {
        this.jpaRepository = jpaRepository;
    }

    @Override
    public void save(Order order) {
        jpaRepository.save(order);
    }

    @Override
    public Order findById(Long orderId) {
        return jpaRepository.findById(orderId).orElseThrow(() -> new EntityNotFoundException("Order not found"));
    }
}
```

`JpaOrderRepository` 实现了 `OrderRepository` 接口，使用 JPA 进行数据库操作。这样，领域层的 `Order` 和 `OrderRepository` 不会直接依赖于数据库的具体实现。

## 领域事件与事件驱动设计

在 DDD 中，**「领域事件」**（Domain Events）是另一个关键的设计理念。通过领域事件，我们可以实现系统内部的松耦合，尤其是在多个聚合或限界上下文之间进行通信时，事件驱动的方式能够很好地解耦复杂的业务交互。

例如，当订单支付完成后，可以发布一个“订单支付成功”的事件通知其他子系统（如库存、物流等）进行后续处理。

```java
public class Order {
    // 支付订单
    public void pay(Money amount) {
        this.status = OrderStatus.PAID;
        // 发布支付完成事件
        DomainEventPublisher.publish(new OrderPaidEvent(this.orderId));
    }
}
```

`OrderPaidEvent` 可以被其他服务或模块订阅，触发相应的处理逻辑。通过这种方式，我们避免了订单模块与其他模块的强耦合。
