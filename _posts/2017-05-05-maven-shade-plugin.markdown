### 背景
为了接收其他项目发过来的数据，我们提供了一个connector.jar，让各个项目的代码中引用。

connector.jar会把数据发送到Kafka集群中，所以connector.jar里面引用了kafka-clients，对应版本是0.8.2.1（我们投用Kafka比较早，因此版本比较旧）

但是有些外部项目因为流量比较大，所以他们自己也会用到kafka，但是他们引用的kafka-clients版本是0.10，所以与connector引用的kafka-clients有冲突。

### 问题
kafka-clients的0.8.2.1和0.10不兼容，所以就算exclude掉旧版本，也不能完全解决问题。

为了让外部项目中使用connector.jar不受kafka-clients版本约束，必须解决兼容性问题。

### 思路
修改kafka-clients 0.8.2.1的package结构，与0.10区分开。

可以把kafka-clients 0.8.2.1的所有源代码扔到connector里重新定义。

这里用到一个maven的plugin，在build过程自动把一些引用包结构给重新定义。[maven-shade-plugin](https://maven.apache.org/plugins/maven-shade-plugin/)

### Usage
在connector的buil中加maven-shade-plugin插件，把内部引用到的org.apache.kafka包重命名为org.apache.shaded.kafka
#### pom.xml
```xml
:
:
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>2.4.3</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <relocations>
                            <relocation>
                                <!-- 修改包结构 -->
                                <pattern>org.apache.kafka</pattern>
                                <shadedPattern>org.apache.shaded.kafka</shadedPattern>
                            </relocation>
                        </relocations>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
:
:
<dependencies>
:
:
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>0.8.2.1</version>
    </dependency>
:
:
</dependencies>
:
:
```

### 效果
用上面pom发布后，可以看到引用此connector的客户端，实际上里面用到的kafka-clients包结构已经变化了，

这样客户端可以随便引用其他版本的kafka-clients官方jar包。
#### ConnectorService.class
```java
package io.github.piaozaiguang.connect.service;
 
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.Map;
import java.util.Properties;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.Future;
import org.apache.shaded.kafka.clients.producer.KafkaProducer;  // 包结构已经变了
import org.apache.shaded.kafka.clients.producer.ProducerRecord;  // 包结构已经变了
import org.apache.shaded.kafka.clients.producer.RecordMetadata;  // 包结构已经变了
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
:
:
```
> 参考

* https://maven.apache.org/plugins/maven-shade-plugin/
