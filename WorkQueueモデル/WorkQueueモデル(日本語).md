# 3.3. WorkQueues モデル

## モデル概要
Work queues（タスクキュー）は、複数の消費者(consumer)が同じキューにバインドして、キュー内のメッセージを共同で消費するメカニズムです。

### 使用シナリオ
- メッセージ処理に時間がかかる場合
- メッセージ生産速度 > 消費速度でメッセージが蓄積する場合
- メッセージ処理効率を向上させる必要がある場面

## 3.3.1. メッセージ送信

メッセージ蓄積をシミュレートするテスト方法：

```java
/**
 * workQueue
 * キューに継続的にメッセージを送信し、メッセージ蓄積をシミュレート
 */
@Test
public void testWorkQueue() throws InterruptedException {
    // キュー名
    String queueName = "simple.queue";
    // メッセージ内容
    String message = "hello, message_";
    
    for (int i = 0; i < 50; i++) {
        // メッセージ送信、20ミリ秒ごとに1回送信
        rabbitTemplate.convertAndSend(queueName, message + i);
        Thread.sleep(20); // 1秒あたり50メッセージ送信に相当
    }
}
```

# 3.3.2. メッセージ受信

## 2つの消費者(Consumer)の実装

```java
// 消費者1（高速処理）
@RabbitListener(queues = "work.queue")
public void listenWorkQueue1(String msg) throws InterruptedException {
    System.out.println("消費者1がメッセージを受信：【" + msg + "】" + LocalTime.now());
    Thread.sleep(20); // 20ミリ秒の処理時間を模擬
}

// 消費者2（低速処理）
@RabbitListener(queues = "work.queue")
public void listenWorkQueue2(String msg) throws InterruptedException {
    System.err.println("消費者2........がメッセージを受信：【" + msg + "】" + LocalTime.now());
    Thread.sleep(200); // 200ミリ秒の処理時間を模擬
}
```

## 消費者処理性能比較表
- 消費者	処理速度	秒間処理可能数
- 消費者1	20ms/メッセージ	約50件
- 消費者2	200ms/メッセージ	約5件
- 性能特性の解説：
- 消費者1は20ミリ秒/件で処理可能 → 理論上1秒間に50件処理可能

- 消費者2は200ミリ秒/件で処理 → 理論上1秒間に5件のみ処理可能
