## Nacos discovery client configuration

```yaml
spring:
  application:
    name: <app-name>
  cloud:
    nacos:
      discovery:
        server-addr: <host>:<port>
        # ip address of the current client
        ip: <ip>
        # ephemeral为true则设置为临时实例，false则设置为永久实例
        ephemeral: <true/false>
        heart-beat-interval: <interval>
        # 超时剔除时间。如果网络好，时间可以设置得短一点，反之则长一些
        ip-delete-timeout: <timeout>
        namespace: <namespace-id>
        group: <group>
        service: <service-name>
```

## Nacos configuration client configuration
```yaml
# 指定一个环境后缀，值无法被application.yaml覆盖
spring:
  profiles:
    active: <profile1>

# 分隔符 --- 用于区分不同的环境配置
---
#当前区域配置环境local
spring:
  config:
    activate:
      on-profile: <profile1>
  application:
    name: luban-demo-order
  cloud:
    nacos:
      config:
        # 这部分配置的文件与shared-configs不同，为默认的{spring.application.name}, {spring.application.name}.{fileExtension}, {spring.application.name}-{spring.profiles.active}.{fileExtension}
        server-addr: <host:port>
        namespace: <namespace-id>
        # 不会影响sharded-config
        group: 1.0.0
        # 不会影响 sharded-configs
        file-extension: yaml
        shared-configs:
          - dataId: <yaml-name>.yaml
          - dataId: <json-name>.json
---
spring:
  config:
    activate:
      on-profile: <profile2>
```
