# Using Dubbo via Java

## Provider

Interface to be exposed:

```java
package com.example.provider.api;

public interface DubboProvider {
    public String echo(String msg);
}
```

Dependency:

```xml
<dependency>
    <groupId>org.apache.dubbo</group>
    <artifactId>dubbo</artifactId>
    <version>...</version>
</dependency>
```

Implementation of the interface:

```java
package com.example.provider.implementation;

import com.example.provider.api.DubboProvider;

public class DubboProviderImpl implements DubboProvider {
    @Override
    public String echo(String msg) {
        return msg;
    }
}
```

Configure the dubbo provider process:

```java
package com.example.provider.test;

import com.example.provider.api.DubboProvider;
import com.example.provider.implementation.DubboProviderImpl;
import org.apache.dubbo.config.ProtocolConfig;
import org.apache.dubbo.config.RegistryConfig;
import org.apache.dubbo.config.ServiceConfig;
import org.apache.dubbo.config.bootstrap.DubboBootstrap;

public class DubboProvider {
    public static void main(String[] args) {
        // interface configuration
        ServiceConfig<DubboProvider> serviceConfig = new ServiceConfig<>();
        // set the interface to be exposed
        serviceConfig.setInterface(DubboProvider.class);
        // set the implementation of the interface
        serviceConfig.setRef(new DubboProviderImpl());

        // Dubbo provider process
        DubboBootstrap provider = DubboBootstrap.getInstance();
        // application name of dubbo provider process
        provider.application("<provider-name>");
        // which registry center to connect to
        provider.registry(new RegistryConfig("nacos://localhost:8848"));
        // which network protocol to use
        provider.protocol(new ProtocolConfig("dubbo", 20990));

        // we can add multiple service configurations: provider.service(serviceConfig1), provider.service(serviceConfig2)...
        provider.service(serviceConfig);

        // start the provider process
        provider.start().await();
    }
}
```

## Consumer

First add the package where DubboProvider is defined as a dependency in pom.xml; the dubbo dependency should also be added:
```xml
<dependency>
    <groupId>org.apache.dubbo</group>
    <artifactId>dubbo</artifactId>
    <version>...</version>
</dependency>
<dependency>
    <groupId>com.example</groupId>
    <artifactId>provider</artifactId>
    <version>...</version>
</dependency>
```

```java
package com.example.consumer;

import com.example.provider.api.DubboProvider;
import org.apache.dubbo.config.ReferenceConfig;
import org.apache.dubbo.config.RegistryConfig;
import org.apache.dubbo.config.bootstrap.DubboBootstrap;

public class DubboConsumer {
    public static void main(String[] args) {
        // interface configuration
        ReferenceConfig<DubboProvider> consumerConfig=new ReferenceConfig<>();
        // set the interface that has been exposed
        consumerConfig.setInterface(DubboProvider.class);

        // Dubbo consumer process
        DubboBootstrap consumer = DubboBootstrap.getInstance();
        // application name of dubbo consumer process
        consumer.application("<consumer-name>");
        // which registry center to connect to
        consumer.registry(new RegistryConfig("nacos://localhost:8848"));

        // add consumer configuration
        consumer.reference(consumerConfig);

        // get the instance of the implementation class of the interface and call the method
        DubboProvider dubboProvider = consumerConfig.get();
        System.out.println(dubboProvider.echo("hello"));
    }
}
```

# Using Dubbo via xml and annotation

## Provider

Interface to be exposed:

```java
package com.example.provider.api;

public interface DubboProvider {
    public String echo(String msg);
}
```

Dependency:

```xml
<dependency>
    <groupId>org.apache.dubbo</group>
    <artifactId>dubbo</artifactId>
    <version>...</version>
</dependency>
```

Provider configuration file, `dubbo.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://dubbo.apache.org/schema/dubbo
    http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="${spring.application.name}">
        <dubbo:parameter key="qos.enable" value="false"/>
    </dubbo:application>
    <dubbo:registry id="nacos" protocol="nacos" address="localhost:8848"
        use-as-config-center="false" use-as-metadata-center="false"/>
    <dubbo:protocol name="dubbo" port="-1"/>
    <dubbo:service interface="com.example.provider.DubboProvider"
                   ref="dubboProvider"
                   registry="nacos"
    />
</beans>
```
Then add the @ImportResource("dubbo.xml") annotation above a configuration class.

Implementation of the interface:

```java
package com.example.provider.implementation;

import com.example.provider.api.DubboProvider;

@Component("dubboProvider")
public class DubboProviderImpl implements DubboProvider {
    @Override
    public String echo(String msg) {
        return msg;
    }
}
```

## Consumer

Dependency:

```xml
<dependency>
    <groupId>org.apache.dubbo</group>
    <artifactId>dubbo</artifactId>
    <version>...</version>
</dependency>
```

Consumer configuration file, `dubbo.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://dubbo.apache.org/schema/dubbo
    http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="${spring.application.name}">
        <dubbo:parameter key="qos.enable" value="falsej"/>
    </dubbo:application>
    <dubbo:registry id="nacos" protocol="nacos" address="localhost:8848"
        use-as-config-center="false" use-as-metadata-center="false"/>
    <dubbo:protocol name="dubbo" port="-1"/>
    <!-- check="true": check if provider exists, if not, the application won't start? -->
    <dubbo:reference id="dubboProvider" 
                     interface="com.example.provider.DubboProvider"
                     registry="nacos"
                     check="false"
    />
</beans>
```
Then add the @ImportResource("dubbo.xml") annotation above a configuration class.

Then inject `DubboProvider dubboProvider` as a bean to use it.
