# 3.3. WorkQueues 模型

## 模型概述
Work queues（任务队列）是一种让多个消费者绑定到同一个队列，共同消费队列中消息的机制。

### 使用场景
- 当消息处理耗时较长时
- 消息生产速度 > 消费速度导致消息堆积
- 需要提高消息处理效率的场景

## 3.3.1. 消息发送

模拟消息堆积的测试方法：

```java
/**
 * workQueue
 * 向队列中不停发送消息，模拟消息堆积
 */
@Test
public void testWorkQueue() throws InterruptedException {
    // 队列名称
    String queueName = "simple.queue";
    // 消息内容
    String message = "hello, message_";
    
    for (int i = 0; i < 50; i++) {
        // 发送消息，每20毫秒发送一次
        rabbitTemplate.convertAndSend(queueName, message + i);
        Thread.sleep(20); // 相当于每秒发送50条消息
    }
}
# 3.3. WorkQueues 模型

## 模型概述
Work queues（任务队列）是一种让多个消费者绑定到同一个队列，共同消费队列中消息的机制。

### 使用场景
- 当消息处理耗时较长时
- 消息生产速度 > 消费速度导致消息堆积
- 需要提高消息处理效率的场景

## 3.3.1. 消息发送

模拟消息堆积的测试方法：

```java
/**
 * workQueue
 * 向队列中不停发送消息，模拟消息堆积
 */
@Test
public void testWorkQueue() throws InterruptedException {
    // 队列名称
    String queueName = "simple.queue";
    // 消息内容
    String message = "hello, message_";
    
    for (int i = 0; i < 50; i++) {
        // 发送消息，每20毫秒发送一次
        rabbitTemplate.convertAndSend(queueName, message + i);
        Thread.sleep(20); // 相当于每秒发送50条消息
    }
}
```
# 3.3.2. 消息接收

## 两个消费者的实现：

```java
// 消费者1（快速消费者）
@RabbitListener(queues = "work.queue")
public void listenWorkQueue1(String msg) throws InterruptedException {
    System.out.println("消费者1接收到消息：【" + msg + "】" + LocalTime.now());
    Thread.sleep(20); // 模拟处理耗时20ms
}

// 消费者2（慢速消费者）
@RabbitListener(queues = "work.queue")
public void listenWorkQueue2(String msg) throws InterruptedException {
    System.err.println("消费者2........接收到消息：【" + msg + "】" + LocalTime.now());
    Thread.sleep(200); // 模拟处理耗时200ms
}
```

## 消费者处理能力对比：

| 消费者   | 处理速度    | 每秒处理量 |
|----------|-------------|------------|
| 消费者1  | 20ms/条     | 50条       |
| 消费者2  | 200ms/条    | 5条        |

# 3.3.3. 初始测试结果

## 问题现象

**消息被平均分配：**

- 消费者1：25条（快速处理完成）
- 消费者2：25条（处理缓慢）

**导致问题：**

- 消费者1空闲
- 消费者2过载
- 总体处理时间延长

## 原始输出样例

```
消费者1接收到消息：【hello, message_0】21:06:00.869555300
消费者2........接收到消息：【hello, message_1】21:06:00.884518
消费者1接收到消息：【hello, message_2】21:06:00.907454400
...
消费者2........接收到消息：【hello, message_49】21:06:05.723106700
```

# 3.3.4. 能者多劳

在 Spring 中有一个简单的配置，可以解决消费者处理能力不均的问题。我们修改 `consumer` 服务的 `application.yml` 文件，添加如下配置：

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1 # 每次只能获取一条消息，处理完成才能获取下一个消息

```

## 再次测试，结果如下：

```text
消费者1接收到消息：【hello, message_0】21:12:51.659664200  
消费者2........接收到消息：【hello, message_1】21:12:51.680610  
消费者1接收到消息：【hello, message_2】21:12:51.703625  
消费者1接收到消息：【hello, message_3】21:12:51.724330100  
消费者1接收到消息：【hello, message_4】21:12:51.746651100  
消费者1接收到消息：【hello, message_5】21:12:51.768401400  
消费者1接收到消息：【hello, message_6】21:12:51.790511400  
消费者1接收到消息：【hello, message_7】21:12:51.812559800  
消费者1接收到消息：【hello, message_8】21:12:51.834500600  
消费者1接收到消息：【hello, message_9】21:12:51.857438800  
消费者1接收到消息：【hello, message_10】21:12:51.880379600  
消费者2........接收到消息：【hello, message_11】21:12:51.899327100  
消费者1接收到消息：【hello, message_12】21:12:51.922828400  
...  
消费者1接收到消息：【hello, message_47】21:12:52.709266500  
消费者2........接收到消息：【hello, message_48】21:12:52.725884900  
消费者1接收到消息：【hello, message_49】21:12:52.746299900
```

可以发现，由于 **消费者1** 处理速度较快，所以处理了更多的消息；而 **消费者2** 处理速度较慢，只处理了 6 条消息。最终总的执行耗时也在约 **1 秒左右**，效率大大提升。

> 正所谓“能者多劳”，这样充分利用了每一个消费者的处理能力，可以有效避免消息积压问题。

---

## 3.3.5. 总结

### Work 模型的使用：

- ✅ 多个消费者绑定到一个队列，同一条消息只会被一个消费者处理  
- ✅ 通过设置 `prefetch` 来控制消费者预取的消息数量，实现动态负载均衡
