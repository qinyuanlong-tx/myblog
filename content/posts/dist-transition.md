+++
title = 'Spring中的分布式事务解决方案'
date = 2024-08-29T20:37:04+08:00
draft = false
tags = ["分布式"]
+++

## 1. JTA (Java Transaction API) 与 Atomikos

JTA是一种标准的XA事务管理API，Atomikos是一个流行的JTA实现。

### 配置

```java
@Configuration
@EnableTransactionManagement
public class JtaConfig {

    @Bean(initMethod = "init", destroyMethod = "close")
    public AtomikosDataSourceBean dataSourceA() {
        AtomikosDataSourceBean dataSource = new AtomikosDataSourceBean();
        // 配置数据源A
        return dataSource;
    }

    @Bean(initMethod = "init", destroyMethod = "close")
    public AtomikosDataSourceBean dataSourceB() {
        AtomikosDataSourceBean dataSource = new AtomikosDataSourceBean();
        // 配置数据源B
        return dataSource;
    }

    @Bean
    public JtaTransactionManager transactionManager() {
        UserTransactionManager userTransactionManager = new UserTransactionManager();
        UserTransaction userTransaction = new UserTransactionImp();
        return new JtaTransactionManager(userTransaction, userTransactionManager);
    }
}
```


### 使用

```java
@Service
public class UserService {

    @Transactional
    public void createUserWithOrder(User user, Order order) {
        userRepository.save(user);
        orderRepository.save(order);
        // 如果发生异常，两个操作都会回滚
    }
}
```



## 2. Seata 框架

Seata是一个开源的分布式事务解决方案，支持AT、TCC、SAGA和XA模式。

### 配置

```java
@Configuration
public class SeataConfig {

    @Bean
    public DataSourceProxy dataSourceProxy(DataSource dataSource) {
        return new DataSourceProxy(dataSource);
    }

    @Bean
    public GlobalTransactionScanner globalTransactionScanner() {
        return new GlobalTransactionScanner("your-application-name", "your-tx-service-group");
    }
}
```


### 使用 (AT模式)

```java
@Service
public class OrderService {

    @GlobalTransactional
    public void createOrder(Order order) {
        orderRepository.save(order);
        accountService.deduct(order.getUserId(), order.getAmount());
        inventoryService.deduct(order.getProductId(), order.getQuantity());
    }
}
```



## 3. TCC (Try-Confirm-Cancel) 模式

TCC是一种补偿型事务模式，需要为每个操作实现try、confirm和cancel方法。

### 实现

```java
@Service
public class OrderService {

    @Transactional
    @TccTransaction(confirmMethod = "confirmCreate", cancelMethod = "cancelCreate")
    public void tryCreate(Order order) {
        // 预留资源，创建订单
        order.setStatus("PENDING");
        orderRepository.save(order);
    }

    public void confirmCreate(Order order) {
        // 确认订单创建
        order.setStatus("CONFIRMED");
        orderRepository.save(order);
    }

    public void cancelCreate(Order order) {
        // 取消订单创建
        orderRepository.delete(order);
    }
}
```


### 使用

```java
@Service
public class BusinessService {

    @Autowired
    private OrderService orderService;
    @Autowired
    private PaymentService paymentService;

    @Transactional
    public void createOrderWithPayment(Order order, Payment payment) {
        orderService.tryCreate(order);
        paymentService.tryCreate(payment);
        // 如果任何一个操作失败，会触发cancel方法
    }
}
```



## 4. 本地消息表 + 消息队列

这种方式结合了本地事务和消息队列，实现最终一致性。

### 实现

```java
@Service
public class OrderService {

    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Transactional
    public void createOrder(Order order) {
        // 创建订单
        jdbcTemplate.update("INSERT INTO orders (id, user_id, amount) VALUES (?, ?, ?)",
                order.getId(), order.getUserId(), order.getAmount());
        
        // 插入本地消息表
        jdbcTemplate.update("INSERT INTO message_table (message_id, message_body, status) VALUES (?, ?, ?)",
                UUID.randomUUID().toString(), JSON.toJSONString(order), "NEW");
    }

    @Scheduled(fixedDelay = 10000)
    public void sendMessage() {
        List<Message> messages = jdbcTemplate.query(
                "SELECT * FROM message_table WHERE status = 'NEW'",
                (rs, rowNum) -> new Message(rs.getString("message_id"), rs.getString("message_body"))
        );

        for (Message message : messages) {
            try {
                rabbitTemplate.convertAndSend("orderExchange", "orderCreated", message.getBody());
                jdbcTemplate.update("UPDATE message_table SET status = 'SENT' WHERE message_id = ?",
                        message.getId());
            } catch (Exception e) {
                // 处理发送失败的情况
            }
        }
    }
}
```


### 消费者端

```java
@Service
public class InventoryService {

    @RabbitListener(queues = "orderCreatedQueue")
    public void handleOrderCreated(String orderJson) {
        Order order = JSON.parseObject(orderJson, Order.class);
        // 更新库存
        // 实现幂等性处理，防止重复消费
    }
}
```



## 总结

1. **JTA/XA**: 适用于强一致性要求的场景，但性能开销较大。
2. **Seata**: 提供了多种模式，适应性强，但需要额外部署Seata服务器。
3. **TCC**: 灵活性高，但实现复杂，需要仔细处理各种异常情况。
4. **本地消息表 + MQ**: 实现简单，适合最终一致性场景，但一致性保证相对较弱。

选择合适的方案需要根据具体的业务需求、性能要求和系统架构来决定。在实际应用中，可能需要组合使用多种方案来满足复杂的业务需求。
