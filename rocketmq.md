# Use RocketMQ with Spring Boot

## RocketMQ simple yaml configuration file

```yaml
rocketmq:
  name-server: localhost:9876
  producer:
    group: test-producer-group
  consumer:
    group: test-consumer-group
```

## Use RocketMQTemplate to send messages

```java
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.spring.core.RocketMQTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.Message;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Service;
import java.util.Date;

@Service
public class MessageServiceImpl {
    @Autowired
    RocketMQTemplate template;

    //use RocketMQTemplate to send messages synchronously
    public void syncSend(String msg) {
        Message<String> message = MessageBuilder.withPayload(msg).build();
        SendResult result = template.syncSend("test-producer-group", message);
        System.out.println(result.getSendStatus());
    }

    //use RocketMQTemplate to send delayed messages synchronously
    public void syncDelayedSend(String msg) {
        Message<String> message = MessageBuilder.withPayload(msg).build();
        System.out.println("Delayed message sent at " + new Date());
        SendResult result = template.syncSend("test-producer-group", message, 1000, 3);
        System.out.println(result.getSendStatus());
    }

    //use RocketMQTemplate to send messages asynchronously
    public void asyncSend(String msg) {
        Message<String> message = MessageBuilder.withPayload(msg).build();
        template.asyncSend("test-producer-group", message, new SendCallback() {
            @Override
            public void onSuccess(SendResult sendResult) {
                System.out.println("succeeded.");
            }

            @Override
            public void onException(Throwable throwable) {
                System.out.println("failed.");
            }
        });
    }

}
```

## Use RocketMQListener interface to receive messages

```java
import org.apache.rocketmq.spring.annotation.RocketMQMessageListener;
import org.apache.rocketmq.spring.core.RocketMQListener;
import org.springframework.stereotype.Component;
import java.util.Date;

@Component
@RocketMQMessageListener(
        topic = "test-producer-group",
        consumerGroup = "${rocketmq.consumer.group}",
        selectorExpression = "*"
)
public class MyConsumer implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        System.out.println("Got message: " + message + " at " + new Date());
    }
}
```
