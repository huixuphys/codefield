# Create rules in the main class:

```java
import com.alibaba.csp.sentinel.slots.block.flow.FlowRule;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRuleManager;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.util.ArrayList;
import java.util.List;

@SpringBootApplication
public class SentinelDemo01 {
    public static void main(String[] args) {
        SpringApplication.run(SentinelDemo01.class,args);
        List<FlowRule> flowRules=new ArrayList<>();
        FlowRule flowRule=new FlowRule();
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
        List<FlowRule> flowRules=new ArrayList<>();

        FlowRule flowRule=new FlowRule();
        flowRule.setResource("sayHi");
        flowRule.setGrade(1);
        flowRule.setCount(1);
        flowRules.add(flowRule);

        FlowRuleManager.loadRules(flowRules);
    }
}
```

# Create rules by using configuration files

Create the configuration file `flowRule.json`:
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
        System.out.println("当前自定义数据源初始化init加载了");
        ClassLoader classLoader = MyDatasource.class.getClassLoader();
        URL resource = classLoader.getResource("flowRules.json");
        String fileName = resource.getFile();
        ReadableDataSource<String,List<FlowRule>> datasource=
                new FileRefreshableDataSource<List<FlowRule>>(fileName,
                        new Converter<String, List<FlowRule>>() {
                            @Override
                            public List<FlowRule> convert(String json) {
                                return JSON.parseArray(json,FlowRule.class);
                            }
                        });
        FlowRuleManager.register2Property(datasource.getProperty());
    }
}
```