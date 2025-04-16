
# SpringAMQP集成RabbitMQ完整指南

## 1. 基本概念
RabbitMQ是基于AMQP协议的消息中间件，SpringAMQP是Spring对RabbitMQ的集成封装。

## 2. 核心优势
| 原生客户端 | SpringAMQP |
|------------|-----------|
| 编码复杂    | 简化开发   |
| 手动配置    | 自动配置   |
| 功能基础    | 功能增强   |

## 3. 项目配置
### 3.1 Maven依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>2.7.12</version>
</dependency>

### 3.2 项目构成
```xml
<!-- mq-demo/pom.xml -->
<project>
  <modules>
    <module>publisher</module>
    <module>consumer</module>
  </modules>
  
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
  </dependencies>
</project>
```
## 4. 安装顺序
### 4.1 生产者实现

# publisher/application.yml
```
spring:
  rabbitmq:
    host: 192.168.150.101
    port: 5672
    virtual-host: /hmall
    username: hmall
    password: 123
```

# 测试类
```
package com.itheima.publisher.amqp;

import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class SpringAmqpTest {
  
  @Autowired
  private RabbitTemplate rabbitTemplate;

  @Test
  public void testSimpleQueue() {
    // キュー名
    String queueName = "simple.queue";
    // メッセージ内容
    String message = "hello, spring amqp!";
    // メッセージ送信
    rabbitTemplate.convertAndSend(queueName, message);
  }
}
```
### 4.2 消费者实现
# consumer/application.yml
```
spring:
  rabbitmq:
    host: 192.168.150.101
    port: 5672
    virtual-host: /hmall
    username: hmall
    password: 123
```
### 监听器实现

```
package com.itheima.consumer.listener;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class SpringRabbitListener {
  
  @RabbitListener(queues = "simple.queue")
  public void listenSimpleQueueMessage(String msg) throws InterruptedException {
    System.out.println("spring コンシューマーがメッセージを受信: 【" + msg + "】");
  }
}
```

### 5. 测试步骤
Consumer服务启动

Publisher的测试用例实现

在消费者控制台上检查以下输出
