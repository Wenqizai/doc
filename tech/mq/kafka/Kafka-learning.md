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


## IO 模型

### 网络请求模型

Kafka 采用 Reactor IO 模型来处理客户端的网络请求。

网络请求模型的基本功能包括：Read request、Decode request、Process service、Encode reply、Send reply。

各种 IO 模型的对比参考 Doug Lea 的《Scalable IO in Java》：[gee.cs.oswego.edu/dl/cpjslides/nio.pdf](https://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)<https://gee.cs.oswego.edu/dl/cpjslides/nio.pdf>

#### **传统 IO 模型**

传统的 IO 模型，由 Sever 层一个线程负责 accept Client 请求，然后分发给线程来处理。

传统的 IO 模型 accept、handle 都是会阻塞线程，意味着单个较慢的网络请求可能阻塞其他网络请求。同时使用线程来提供处理请求的吞吐量，也会极大消耗资源性能。线程都会独占 1m 的栈空间，同时线程越多，线程切换越频繁，CPU 上下文消耗越大。

- 缺点
1. 实现简单、直观、线程独立；
2. 适用于小量连接的应用。

- 优点
1. 无法支持大量并发连接，资源消耗大；
2. CPU 利用率低。

```java
while (true) {
	Request request = accept(connection);
	// 这里可以使用多线程来提高处理请求的吞吐量
	handle(request);
}
```

![[传统的网络请求模型.png|425]]

#### **Reactor 模型**

解决传统 IO 的缺点：资源消耗大、CPU 切换频繁、无法支持大量连接。

基于事件驱动的设计，将处理网络请求的功能划分成为一个个的事件，Read request、Decode request、Process service、Encode reply、Send reply。由分发器 dispatch 负责分发不同的事件到不同的事件处理器。

我们可以针对这个 SelectionKey 通知的事件来注册不同的处理器：java.nio.channels.SelectionKey

单线程的 Reactor IO 模型，实现了高效的事件分发，可以使用单个线程来处理大量的连接。因为创建的线程少，占用的内存也少，适合 I/O 密集型应用。

通过引入线程池，可以处理大量的请求连接。

``` java
while (true) {
	selector.select();
	Set selected = selector.selectedKeys();  // 获取到各种到达的事件
	Iterator it = selected.iterator();
	while (it.hasNext())
		// 这里可以使用线程池来提高处理请求的吞吐量
		dispatch((SelectionKey)(it.next()); // 分发事件给事件处理器
		selected.clear();
	}
}

dispatch(SelectionKey key) {
	// 获取与 SelectionKey 关联的Handler
	// handler 在初始化时，向 selector 注册 Acceptor
	// Acceptor 完成 accept 请求后组成 ReadHandler、WriteHandler、ResponseHandler 等等
	if (key.attachment() instanceof Handler) {
		Handler handler = (Handler) attachment;

		// 检查Handler是否仍然有效
		// Check if Handler is still valid
		if (handler.isValid()) {
			handler.handle(key);
		}
	}
}

public class Acceptor implements Handler {
	@Override  
	public void handle(SelectionKey key) throws Exception {  
	    if (key.isAcceptable()) {  
			// 处理 accept 请求
			
			// 并注册对应的处理事件
			clientChannel.register(selector, SelectionKey.OP_READ, new EchoHandler);
		}
    }
}


public class Acceptor implements Handler {
	@Override  
	public void handle(SelectionKey key) throws Exception {  
	    if (key.isAcceptable()) {  
			// 处理 accept 请求
		}
    }
}


public class EchoHandler implements Handler {
    @Override
    public void handle(SelectionKey key) throws Exception {
        if (key.isReadable()) {
                handleRead(key);
		}
		if (key.isWritable()) {
			handleWrite(key);
		}
    }
}

```

![[Reactor网络请求模型.png|375]]

#### Reactor 多线程模型

我们在单线程的 Reactor 模型中，可以引入线程池来提高整个处理请求的吞吐量。但是该模型会有一个问题，acceptor 请求快速的，handler 处理任务是慢速的，如果将 accpector 和 handler 放在同一个线程处理，有可能 acceptor 被慢速的 handler 阻塞了，影响到 acceptor 的处理速度。

```java
	while (it.hasNext())
		workerPool.submit(() -> {
			dispatch((SelectionKey)(it.next());
		});
	}
```

重新设计多线程版本的 Reactor 模型，将 dispatch 和 handler 线程隔离，处理 handler 的线程统一为 worker 线程。这样设计可以提高整体网络 IO 的处理速度。同时仅仅针对 worker 来设置线程数，会让资源分配、利用更加合理。

```java
while (it.hasNext())
	dispatch((SelectionKey)(it.next());
}

handeRead {
	workerPool.submit(() -> {
		// 处理读任务
	});
}

handleWrite {
	workerPool.submit(() -> {
		// 处理写任务
	});
}
```


![[Reactor多线程网络模型.png]]

#### 多 Reactor 模型

多 Reactor 模型基本是多线程版本的改进，将单线程处理 acceptor 设计为独立线程处理 accept 请求。

避免较慢的 accept 请求，阻塞了请求的 accept。对 CPU 更加友好和处理更多的网络请求。

![[多Reactor网络模型.png]]

#### Kafka 网络请求模型

Kafka 的 Broker 端的 SocketServer 组件，类似于 Reactor 模型中的 Dispatcher。负责处理网络的请求，然后提交给网络线程池来处理。提交给网络线程的策略是公平轮询。

`num.network.threads`：默认值 3，网络线程池的线程数。

![[kafka处理客户端网络请求模型.png]]

**网络线程池**拿到请求后，放入共享的请求队列，等待 IO 线程池来处理请求。

**IO 线程池**负责处理的请求类型：

1. <font color="#f79646">PRODUCE</font> 请求：将消息写入到底层的磁盘日志中；
2. <font color="#f79646">FETCH</font>：从磁盘或页缓存中读取消息。

`num.io.threads`：默认值 8，IO 线程池线程数。

**请求队列和响应队列**，请求队列是共享的，响应队列是独立的，每个请求线程都有一个独立的响应队列。

**Purgatory**，用来缓存延时请求，区别于 RocketMQ 的延时 message，这里的延时主要是，应对 acks=all的PRODUCE请求，等待 ISR 副本都写入日志后才能返回。IO 线程不能立即处理该请求，因此暂存到 purgatory。

![[kafka网络请求模型.png]]

>  **为什么设计成请求队列是共享的？而响应队列却是独立的？**

请求队列是共享的，因为请求是交给 IO 线程池来执行处理，IO 线程是固定数量的，因此需要一个队列来暂缓写入的请求。这是一个典型的生产者、消费者的模型设计。

响应队列是独立的，意味着一个队列响应一个连接请求，是一个简单高效的 request - response 的连接模型。

如果设计成共享队列，那么每个响应请求的线程都会从共享队内中获取响应连接，将会面临以下线程和并发问题：

1. 大量的锁控制，上下文切换；
2. 处理跨线程写入 socket 的复杂性；
3. 事件到达，设计线程 wakeup 的复杂性；
4. 同一个连接请求涉及顺序问题，也是很难保证

因此设计一个请求，一个独立响应队列是简单高效的。

>  **Kafka 网络模型存在的问题**

IO 线程池的线程都是平等的，当共享请求队列满了时候，意味着高优先的线程没办法得到及时处理。

比如：

1. 选举新的 Leader 同步新的日志进度时，请求队列中某些积压的请求是可以丢弃的，直接返回，无需等待 IO 线程池处理完。
2. Topic 被删除，无需处理该 Topic 的请求消息等。

**Kafka 2.3 解决方案**

1. 将数据按类型进行分离；
	1. 控制类请求，执行 LeaderAndIsr、StopReplica 等请求
	2. 数据类请求，执行 PRODUCE，FETCH 等请求。
2. 线程池分离，创建两套网络线程池、IO 线程池，分别处理控制类请求和数据累请求；
3. 控制类请求优先处理。

> **Kafka 在异步的网络模型中如何保证分区消息有序的？**

思考问题：kafka 在日志 apend 的通过加锁顺序保证来保证写入日志的单调性。当 producer 发送的顺序是 A -> B，IO线程池处理顺序是 B -> A，那么 append 时，是如何保证分区的顺序是发送的顺序 A -> B？

原理是 Producer 发送消息后，每个消息都会拿到一个 offset，当发送消息的顺序是 A -> B 时，那么 A.offset < B.offset。当 IO 线程池将消息 B 先提交给 append 时，会判断当前的运行 append 的 offset 是不是该提交的 offset，如果不是，该消息会暂缓 append。

```java
if (records.baseOffset != logEndOffset)
    throw OffsetOutOfOrderException
```

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

**Broker**

1. 防止落后的副本成为 Leader，`unclean.leader.election.enable = false` ；
2. 设置高可用，分区容错数量的副本 `replication.factor >= 3`，如 3, 5, 7;
3. 保证写入大多数的副本，`min.insync.replicas`；

**Consumer**

保证消费完成后手动提交位移。设置禁止自动提交位移 `enable.auto.commit = false`。