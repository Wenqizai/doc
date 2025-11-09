# 文档


# 架构

## 架构图

### 消费者

#### 位移

组内消费者保存消费的位移

![[消费者位移.png|300]]

## 生产者

## 消息体

### 压缩

消息的压缩，通常有两个地方：Producer 和 Broker。

消息体经历的流程：Producer 压缩 -> Broker 解压、压缩 -> Consumer 解压

- Producer 消息压缩

Producer 开启压缩，通常在 Producer 构建时，配置指定。

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("acks", "all");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("compression.type", "gzip"); // 开启GZIP压缩

Producer<String, String> producer = new KafkaProducer<>(props);
```

- Broker 消息压缩

在配置文件 `config/server.properties` 中指定 `compression.type=zstd`。 

**注意：** 

- 压缩算法不同

如果 Broker 和 Producer 的压缩算法不一致，Broker 会先将 Record 解压，然后再用 Broker 指定的压缩算法进行压缩。<font color="#f79646">这也是一个很大的性能损耗</font>，Broker 端的 CPU 会急剧飙升。

- 压缩算法相同

Kafka 2.4 之后，只会提取 Record 的必要信息进行校验，不会对这个消息体 key，value 进行解压缩，避免消耗大量的 CPU。校验通过后，会直接存储该消息体。


## Broker
### 分区

分区是 Topic 下的，存储消息的地方，一个 Topic 可以设置多个分区。

#### 分区策略

分区策略主要是指 Producer 将消息发送到哪个分区。默认的分区策略是 `org.apache.kafka.clients.producer.internals.DefaultPartitioner`，<font color="#f79646">指定 key hash 或者轮询</font>。

自定义的分区策略 CustomerPartitioner 可以实现以下接口，并在构建 Producer 时注入 CustomerPartitioner 即可。

```java
public interface Partitioner extends Configurable, Closeable {  
  
    /**  
     * Compute the partition for the given record.
     * @param topic The topic name  
     * @param key The key to partition on (or null if no key)  
     * @param keyBytes The serialized key to partition on( or null if no key)  
     * @param value The value to partition on or null  
     * @param valueBytes The serialized value to partition on or null  
     * @param cluster The current cluster metadata  
     */    
     public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster);  
  
    /**  
     * This is called when partitioner is closed.     
     */    
     public void close();  
}
```

-  自定义轮询

```java
List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
return ThreadLocalRandom.current().nextInt(partitions.size());
```

- 自定义随机

```java
List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
return Math.abs(key.hashCode()) % partitions.size();
```

## 消费者


# 实践

## 消息可靠性

**Producer**

Kafka 的消息发送都是异步发送。要保证发送消息的可靠性，需要使用带有 callback 的 API，处理异步消息发送遇到的异常，确保消息能够发送到 Broker。

```java
public interface Producer<K, V> extends Closeable {
	Future<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback);
}

public interface Callback {
	void onCompletion(RecordMetadata metadata, Exception exception);
}
```

