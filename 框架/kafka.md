分布式流媒体平台

应用：消息系统+日志手机+用户行为追踪+流式处理

特点：高吞吐量+消息持久化+高可靠性（分布式）+高扩展性

Broker是Kafka服务器

zookeeper独立的应用，能够用来管理其他的集群（用zookeeper管理）

topic是存放消息的位置，代表了消息的分类

消息队列实现的方法有两种：点对点（阻塞队列），发布订阅模式，生产者把消息放在某个位置，可以有很多消费者关注这个位置订阅这个位置，多个消费者读到先后读到

Partition对topic的一个分区

Offset是消息在分区存放的索引序列

Leader Replica，follower Replica，主副本能力比较强，当你从这个分区获取数据的时候，主副本可以给你这个数据，从副本只是备份，不负责响应，

复本，对数据做备份，Kafka是分布式消息引擎，为了让数据更可靠，不是把数据存一份，是通过副本的方式存多份，每个分区有多个副本，万一某一个副本坏了，还有其他副本



读取硬盘效率的高低取决于你对硬盘的使用，对硬盘顺序读取性能是很高的，甚至高于对内存的随机读取

生产者

​	kafkaTemplate.send(top,data)

消费者

​	@kafkaListener(topics={"test"})

public void handleMessage(ConsumerRecord record){}

