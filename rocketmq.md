## use RocketMQTemplate to send messages

```java
@Service
public class MessageServiceImpl {
    @Autowired
    RocketMQTemplate template;

    public void send(String msg) {
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
