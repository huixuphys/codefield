# RocketMQ basic usage

## Use MQProducer to send messages

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.MQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import java.nio.charset.StandardCharsets;

public void send() throws Exception {
    //create an MQProducer object
    DefaultMQProducer producer = new DefaultMQProducer();
    producer.setNamesrvAddr("localhost:9876");
    producer.setProducerGroup("test-producer-group");
    producer.start();
    
    //create a Message object
    Message message = new Message();
    String msg = "hello world";
    message.setBody(msg.getBytes(StandardCharsets.UTF_8));
    message.setTopic("topic-01");
    
    //send the message
    SendResult sendResult = producer.send(message);
    
    //get send status
    System.out.println(sendResult.getSendStatus());
    while(true);
}
```

## Use MQPushConsumer to receive messages

```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListener;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;
import java.nio.charset.StandardCharsets;
import java.util.List;

public void consume() throws Exception {
    //prepare a MessageListener object that defines how a message should be handled when received
    MessageListenerConcurrently messageListener = (msgs, consumeConcurrentlyContext) -> {
        //obtain the message
        MessageExt messageExt = msgs.get(0);
        byte[] body = messageExt.getBody();
        String message = new String(body, StandardCharsets.UTF_8);
        //handle the message
        System.out.println("Got a message: " + message);

        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    };

    //create an MQPushConsumer object that subscribes to a particular topic
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer();
    consumer.setNamesrvAddr("localhost:9876");
    consumer.setConsumerGroup("test-consumer-group");
    consumer.subscribe("topic-01", "*");
    consumer.setMessageListener(messageListener);
    consumer.start();
    while(true);
}
```

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

## Example of using RocketMQ to queue a request

In the controller, send the request parameter data as a message:

```java
import com.tarena.demo.luban.protocol.order.param.OrderAddParam;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.spring.core.RocketMQTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.Message;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/base/order")
public class OrderController {
    @Autowired
    private RocketMQTemplate template;

    @PostMapping("/add")
    public String addOrder(OrderAddParam orderAddParam){
        Message<OrderAddParam> message = MessageBuilder.withPayload(orderAddParam).build();
        template.asyncSend("order-add-topic", message, new SendCallback() {
            @Override
            public void onSuccess(SendResult sendResult) {

            }

            @Override
            public void onException(Throwable throwable) {

            }
        });

        return "success";
    }
}

```

Consumer to handle the request:

```java
import com.tarena.demo.luban.all.main.service.OrderService;
import com.tarena.demo.luban.protocol.order.param.OrderAddParam;
import org.apache.rocketmq.spring.annotation.RocketMQMessageListener;
import org.apache.rocketmq.spring.core.RocketMQListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
@RocketMQMessageListener(
        topic = "order-add-topic",
        consumerGroup = "${rocketmq.consumer.group}"
)
public class OrderAddConsumer implements RocketMQListener<OrderAddParam> {
    @Autowired
    private OrderService orderService;

    @Override
    public void onMessage(OrderAddParam message) {
        orderService.addOrder(message);
    }
}

```
