```yaml
server:
  port: 10000
spring:
  application:
    name: test-app
  main:
    web-application-type: reactive
  cloud:
    gateway:
      routes:
        - id: id1
          uri: http://localhost:10000
          predicates:
            - Path=/patha/**
            - Host=127.0.0.1:10000
          filters:
            # strip a level of path from left
            - StripPrefix=1
        - id: id2
          uri: http://localhost:10001
          predicates:
            - Path=/pathb/**

```
