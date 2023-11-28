# define resource using the Entry class

```java
import com.alibaba.csp.sentinel.Entry;
import com.alibaba.csp.sentinel.SphU;
import com.alibaba.csp.sentinel.slots.block.BlockException;

public void test() {
    Entry entry = null;
    try {
        entry = SphU.entry("resource-name");
        // code block as resource
    } catch (BlockException e){
        // handle exception
    } finally {
        if (entry != null){
            entry.exit();
        }
    }
}
```

# define resource using Annotations

```java
import com.alibaba.csp.sentinel.annotation.SentinelResource;

@SentinelResource(value = "resource-name")
public void test() {
    // method body
}
```

# Create rules in the main class

```java
import com.alibaba.csp.sentinel.slots.block.flow.FlowRule;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRuleManager;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.util.ArrayList;
import java.util.List;

@SpringBootApplication
public class SentinelDemo {
    public static void main(String[] args) {
        SpringApplication.run(SentinelDemo.class, args);
        List<FlowRule> flowRules = new ArrayList<>();
        FlowRule flowRule = new FlowRule();
        flowRule.setResource("sayHi");
        flowRule.setGrade(1);
        flowRule.setCount(1);
        flowRules.add(flowRule);
        FlowRuleManager.loadRules(flowRules);
    }
}
```

# Create rules by implementing InitFunc interface

```java
import com.alibaba.csp.sentinel.init.InitFunc;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRule;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRuleManager;
import java.util.ArrayList;
import java.util.List;

public class MyDatasource implements InitFunc {
    @Override
    public void init() throws Exception {
        List<FlowRule> flowRules = new ArrayList<>();

        FlowRule flowRule = new FlowRule();
        flowRule.setResource("sayHi");
        flowRule.setGrade(1);
        flowRule.setCount(1);
        flowRules.add(flowRule);

        FlowRuleManager.loadRules(flowRules);
    }
}
```

Then create a file under `resources/META-INF/services` named `com.alibaba.csp.sentinel.init.InitFunc` and add the full qualified name of the `MyDatasource` class in this file.

# Create rules by using configuration files

Create the configuration file `flowRule.json` in the resources directory:
```json
[
    {
        "resource":"sayHi",
        "grade":1,
        "count":100
    },
    {
        "resource":"sayHiRepo",
        "grade":1,
        "count":1
    }
]
```

Load the rules from the file:
```java
import com.alibaba.csp.sentinel.datasource.Converter;
import com.alibaba.csp.sentinel.datasource.FileRefreshableDataSource;
import com.alibaba.csp.sentinel.datasource.ReadableDataSource;
import com.alibaba.csp.sentinel.init.InitFunc;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRule;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRuleManager;
import com.alibaba.fastjson.JSON;
import java.net.URL;
import java.sql.SQLOutput;
import java.util.ArrayList;
import java.util.List;

public class MyDatasource implements InitFunc {
    @Override
    public void init() throws Exception {
        ClassLoader classLoader = MyDatasource.class.getClassLoader();
        URL resource = classLoader.getResource("flowRules.json");
        String fileName = resource.getFile();
        ReadableDataSource<String,List<FlowRule>> datasource = new FileRefreshableDataSource<List<FlowRule>>(fileName, new Converter<String, List<FlowRule>>() {
            @Override
            public List<FlowRule> convert(String json) {
                return JSON.parseArray(json,FlowRule.class);
            }
        });
        FlowRuleManager.register2Property(datasource.getProperty());
    }
}
```
