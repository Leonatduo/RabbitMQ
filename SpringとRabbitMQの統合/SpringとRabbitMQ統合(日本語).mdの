# SpringAMQP による RabbitMQ 統合

## 1. 基本概念

業務機能開発ではコンソールでのメッセージ送受信ではなく、プログラムベースでの実装が必要。RabbitMQはAMQPプロトコルを採用しているため、言語横断的な特性を持つ。

## 2. SpringAMQP の利点

- **公式Javaクライアントの問題点**: コーディングが複雑
- **Spring統合のメリット**:
  - 生産環境での使いやすさ
  - SpringBootによる自動設定
  - 豊富なヘルパー機能

## 3. 主な機能

### 3.1 コア機能
1. キュー/エクスチェンジ/バインディングの自動宣言
2. アノテーションベースのリスナーパターン（非同期受信）
3. `RabbitTemplate` によるメッセージ送信機能

### 3.2 プロジェクト構成
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
## 4. 実装手順
### 4.1 Publisher実装
設定ファイル
yaml

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

# テストクラス
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
### 4.2 Consumer実装
設定ファイル
yaml
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
### リスナー実装

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

### 5. テスト手順
Consumerサービスを起動

Publisherのテストケースを実行

Consumerコンソールで以下の出力を確認
