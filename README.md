_README FILE_

---
# Kafka Consumer Utilities

Spring boot based kafka Consumer utility library. It handles kafka consumer configuration.

You only need to provide Kafka connection configuration, then the Kafka consumers will start automatically.

# Run in local

```
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Dweb.test.enabled=true"
```

Now Kafka consumer is waiting for the messages.

If you run maven in command line, follow the Maven installation instructions here:

```
https://maven.apache.org/install.html
```

# Usage Samples:

## Non-batch mode:

1. Configure a consumer:

```
app.kafka.consumers[0].consumerId=consumer0
app.kafka.consumers[0].bootstrapServers=${app.kafka.default.host}:${app.kafka.default.port}
app.kafka.consumers[0].topic=${app.kafka.default.topic}
app.kafka.consumers[0].groupId=group0
app.kafka.consumers[0].keyDeserializer=org.apache.kafka.common.serialization.StringDeserializer
app.kafka.consumers[0].valueDeserializer=org.apache.kafka.common.serialization.StringDeserializer
app.kafka.consumers[0].enableAutoCommit=true
app.kafka.consumers[0].autoCommitIntervalMs=1000
app.kafka.consumers[0].sessionTimeoutMs=30000
app.kafka.consumers[0].concurrency=2
app.kafka.consumers[0].batchListener=false
```

2. Integrate the consumer in application:

```
@DependsOn("kafkaListenerContainerFactoryList")
@Service
public class KafkaConsumerService0 {
  private static final Logger LOGGER = LoggerFactory.getLogger(KafkaConsumerService0.class);

  @KafkaListener(
    id = "testListener0",
    topics = "${app.kafka.default.topic}",
    autoStartup = "true",
    containerFactory = "${app.kafka.kafkaMessageListenerContainerFactoryBeanPrefix}.consumer0"
  )
  public void listen(ConsumerRecord<?, ?> record) {
    LOGGER.info("*********************record:{}", record);
  }
}
```

## if batch mode enabled:

1. Configure a consumer:

```
app.kafka.consumers[0].consumerId=consumer0
app.kafka.consumers[0].bootstrapServers=${app.kafka.default.host}:${app.kafka.default.port}
app.kafka.consumers[0].topic=${app.kafka.default.topic}
app.kafka.consumers[0].groupId=group0
app.kafka.consumers[0].keyDeserializer=org.apache.kafka.common.serialization.StringDeserializer
app.kafka.consumers[0].valueDeserializer=org.apache.kafka.common.serialization.StringDeserializer
app.kafka.consumers[0].enableAutoCommit=false
app.kafka.consumers[0].autoCommitIntervalMs=1000
app.kafka.consumers[0].sessionTimeoutMs=30000
app.kafka.consumers[0].concurrency=2
app.kafka.consumers[0].batchListener=true
```

2. Integrate the consumer in application:

```
@DependsOn("kafkaListenerContainerFactoryList")
@Service
public class KafkaBatchConsumerService0 {
  private static final Logger LOGGER = LoggerFactory.getLogger(KafkaBatchConsumerService0.class);

  @KafkaListener(
    id = "testListener0",
    topics = "${app.kafka.default.topic}",
    autoStartup = "true",
    containerFactory = "${app.kafka.kafkaMessageListenerContainerFactoryBeanPrefix}.consumer0"
  )
  public void pollResults(List<ConsumerRecord<?, ?>> record, Acknowledgment ack) {
    LOGGER.info("*********************record:{}", record);
    ack.acknowledge();
  }
}
```

*** Error Handling

SeekToCurrentErrorHandler provided by Spring is used to handle listener errors.

An error handler that seeks to the current offset for each topic in the remaining records. Used to rewind partitions
after a message failure so that it can be replayed.

After maximum number of retries exhausted, the messages failed will be sent to a dead letter topic: "topic.DLT". Then
listener will move on forward to process other messages.

