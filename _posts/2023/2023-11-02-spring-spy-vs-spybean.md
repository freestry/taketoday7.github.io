---
layout: post
title: Spy和SpyBean之间的区别
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在本教程中，我们的目标是解决@Spy和@SpyBean之间的差异，解释它们的功能并就何时使用它们提供指导。

## 2. 基本应用

在本文中，我们将使用一个简单的订单应用程序，其中包含用于创建订单的订单服务，并在处理订单时调用通知服务来发出通知。

OrderService有一个save()方法，它接收Order对象，使用OrderRepository保存它，并调用NotificationService：

```java
@Service
public class OrderService {

    public final OrderRepository orderRepository;

    public final NotificationService notificationService;

    public OrderService(OrderRepository orderRepository, NotificationService notificationService) {
        this.orderRepository = orderRepository;
        this.notificationService = notificationService;
    }

    public Order save(Order order) {
        order = orderRepository.save(order);
        notificationService.notify(order);
        if (!notificationService.raiseAlert(order)) {
            throw new RuntimeException("Alert not raised");
        }
        return order;
    }
}
```

为简单起见，我们假设notify()方法记录订单。实际上，它可能涉及更复杂的操作，例如通过队列向下游应用程序发送电子邮件或消息。

我们还假设创建的每个订单都必须通过调用ExternalAlertService来接收警报，如果警报成功则返回true，如果不引发警报则OrderService将失败：

```java
@Component
public class NotificationService {

    private ExternalAlertService externalAlertService;

    public void notify(Order order){
        System.out.println(order);
    }

    public boolean raiseAlert(Order order){
        return externalAlertService.alert(order);
    }
}
```

OrderRepository中的save()方法使用HashMap将order对象保存在内存中：

```java
public Order save(Order order) {
    UUID orderId = UUID.randomUUID();
    order.setId(orderId);
    orders.put(UUID.randomUUID(), order);
    return order;
}
```

## 3. @Spy和@SpyBean注解的实际应用

现在我们已经有了一个基本的应用程序，让我们看看如何使用@Spy和@SpyBean注解来测试它的不同方面。

### 3.1 Mockito的@Spy注解

**[@Spy](https://www.baeldung.com/mockito-annotations)注解是Mockito测试框架的一部分，它创建真实对象的间谍(部分mock)，通常用于单元测试**。

间谍允许我们跟踪并可选地存根或验证真实对象的特定方法，同时仍然执行其他方法的真实实现。

让我们通过为OrderService编写单元测试并使用@Spy标注NotificationService来理解这一点：

```java
@Spy
OrderRepository orderRepository;
@Spy
NotificationService notificationService;
@InjectMocks
OrderService orderService;

@Test
void givenNotificationServiceIsUsingSpy_whenOrderServiceIsCalled_thenNotificationServiceSpyShouldBeInvoked() {
    UUID orderId = UUID.randomUUID();
    Order orderInput = new Order(orderId, "Test", 1.0, "17 St Andrews Croft, Leeds ,LS17 7TP");
    doReturn(orderInput).when(orderRepository)
        .save(any());
    doReturn(true).when(notificationService)
        .raiseAlert(any(Order.class));
    Order order = orderService.save(orderInput);
    Assertions.assertNotNull(order);
    Assertions.assertEquals(orderId, order.getId());
    verify(notificationService).notify(any(Order.class));
}
```

在这种情况下，NotificationService充当间谍对象，并**在没有定义mock时调用真正的notify()方法**。此外，由于我们为raiseAlert()方法定义了一个mock，所以NotificationService的行为就像一个部分mock。

### 3.2 Spring Boot的@SpyBean注解

**另一方面，@SpyBean注解是Spring Boot特有的，用于与Spring的依赖注入进行集成测试**。

它允许我们创建Spring bean的间谍(部分mock)，同时仍然使用应用程序上下文中的实际bean定义。

让我们使用@SpyBean为NotificationService添加一个集成测试：

```java
@Autowired
OrderRepository orderRepository;
@SpyBean
NotificationService notificationService;
@SpyBean
OrderService orderService;

@Test
void givenNotificationServiceIsUsingSpyBean_whenOrderServiceIsCalled_thenNotificationServiceSpyBeanShouldBeInvoked() {
    Order orderInput = new Order(null, "Test", 1.0, "17 St Andrews Croft, Leeds ,LS17 7TP");
    doReturn(true).when(notificationService)
        .raiseAlert(any(Order.class));
    Order order = orderService.save(orderInput);
    Assertions.assertNotNull(order);
    Assertions.assertNotNull(order.getId());
    verify(notificationService).notify(any(Order.class));
}
```

在这种情况下，Spring应用程序上下文管理NotificationService并将其注入到OrderService中。**在NotificationService中调用notify()会触发真正方法的执行，调用raiseAlert()会触发mock的执行**。

## 4. @Spy和@SpyBean之间的区别

让我们详细了解一下@Spy和@SpyBean之间的区别。

在单元测试中，我们使用@Spy，而在集成测试中，我们使用@SpyBean。

如果@Spy标注的组件包含其他依赖项，我们可以在初始化时声明它们。如果在初始化期间未提供它们，系统将使用零参数构造函数(如果可用)。在@SpyBean测试的情况下，我们必须使用@Autowired注解来注入依赖组件。否则，在运行时，Spring Boot会创建一个新实例。

**如果我们在单元测试示例中使用@SpyBean，则当调用NotificationService时，测试将失败并出现[NullPointerException](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/NullPointerException.html)**，因为OrderService需要mock/spy NotificationService。

同样，**如果在集成测试的示例中使用@Spy，则测试将失败**并显示错误消息“Wanted but not invoked: notificationService.notify(<any cn.tuyucheng.taketoday.spytest.Order\>)”，**因为Spring应用程序上下文不知道@Spy标注的类**。相反，它创建一个新的NotificationService实例并将其注入到OrderService中。

## 5. 总结

在本文中，我们探讨了@Spy和@SpyBean注解以及何时使用它们。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-testing-2)上获得。