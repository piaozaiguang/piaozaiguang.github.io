_测试版本：Kafka 0.10_

### 查看当前offset信息
```sh
{kafka_home}/bin/kafka-consumer-groups.sh --bootstrap-server {your broker list} --group {your consumer group} --describe
```

### 手动设置offset
```sh
{kafka_home}/bin/kafka-run-class.sh kafka.tools.UpdateOffsetsInZK [earliest | latest] config/consumer.properties {your topic}
```

### kafka-streams重置
```sh
{kafka_home}/bin/kafka-streams-application-reset.sh --bootstrap-servers {your broker list} --zookeeper {your zookeeper} --application-id {app id}
```
