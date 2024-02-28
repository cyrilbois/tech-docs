---
title:  Spring Kafka 顺序消费问题
showOnHome: false
...


本篇记录下在用Spring Kafka的时候如何保证顺序消费。

https://docs.spring.io/spring-kafka/docs/2.2.14.RELEASE/reference/html/


## 背景
在集群情况下，多个实例同时消费消息的时候，需要保证顺序消费， 为什么？

比如： 处理景区门票的订单业务中， 如果一个订单有多张票（或者叫做子订单），多张票可能出现同时入园过闸机， 闸机这时候会发送消费消息，订单服务收到消息后需要更新票的状态为已使用，同时需要检查是否主订单所有票都消费了，从而判断是否是否更新主订单状态。如果一个订单的多个消费消息同时被不同实例处理，可能导致主订单状态没有更新的情况。 如果同一个订单的消息保证发送到同一个分区，由于kafka的特性， 同一个分区只能被同一个group的一个实例订阅， 则很容易避免此类事件。 


## Spring Kafka 使用

### 消息发送

首先，在消息发送方，发送消息的时候指定key即可。 

> The key is not the partition number but Kafka uses the key to specify the target partition. The default strategy is to choose a partition based on a hash of the key or use round-robin algorithm if the key is null. 

```

import lombok.extern.slf4j.Slf4j;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.stereotype.Service;
import org.springframework.util.concurrent.ListenableFuture;
import org.springframework.util.concurrent.ListenableFutureCallback;

import javax.annotation.Resource;

/**
 * @author : Joe
 * @date : 2021/1/2
 */
@Service
@Slf4j
public class KafkaMessageSender {
    @Resource
    private KafkaTemplate<String, String> kafkaTemplate;

    /**
     * 发送kafka消息
     * @param topic
     * @param key 这里应该是订单的ID
     * @param message
     */
    public void send(String topic, String key, String message){
        ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send(topic, key, message);
        future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {

            @Override
            public void onSuccess(SendResult<String, String> result) {
                log.info("[发送kafka消息] Topic={}, Message={} with offset={}", topic, message, result.getRecordMetadata().offset());
            }
            @Override
            public void onFailure(Throwable ex) {
                log.info("发送kafka消息失败 {} message={} due to: {}", topic, message, ex.getMessage());
                log.error(ex.getMessage(), ex);
            }
        });
    }
}
```


### 消息消费端


由于kafka的特性，一个分区最多只能被同一个group的一个分区订阅，就可以保证同一个订单的消息被同一个实例顺序消费。

> 这里就算设置了concurrency>1,同一个分区的消息也是顺序消费的。 即这个消息ack之后才会接受到下一个消息。

```
 @KafkaListener(topics = "test-topic", concurrency = "2")
    public void process(@Payload String notification, Acknowledgment ack) {
            //处理逻辑
            ack.acknowledge();
       
    }
```




![kafka-sequence.png](http://tech.icoding.tech/Spring-Boot-And-Spring-Cloud/kafka-sequence.png)


