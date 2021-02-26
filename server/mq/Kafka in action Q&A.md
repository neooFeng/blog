# message queue

- producer/consumer vs. publisher/subscriber  
这两个其实没什么好比的，本质上都是一个生产消息，一个消费消息。在message queue领域，有两种典型的消息处理模式：
queue和sub-pub，queue比较原始，一条消息被消费者拿到之后就从queue中删除，也就是说message不能递送给多个消费者或重复消费；
sub-pub为了解决这个问题，引入topic的概念，其实还是一个queue，只不过消息被消费的时候不从queue中删除，这就允许多个消费者
消费同一条消息。  
常说的producer/consumer就message queue模式，PUB-SUB是topic模式，现在流行的MQ中间件都支持SUB/PUB模式，但rabbitMQ是个例外。

- PUB/SUB模式中，如何保证消息可被多个消费者消费的同时又能兼顾消费者的水平扩展能力  
对消费者的管理引入group的概念，每个group相当于一个subscribe，同一条消息会被每个group拿到，
但是同一个group内的不同消费者只能拿到不同的消息。

- 如何保证消息不丢失  
应用层ACK, 需要client配合。

- 消息递送的三个可靠性等级，exactly ONCE如何实现  
分别是at most once, at least once, exactly once. 

- kafka如何实现高性能吞吐的  
    * sequence write
    由于OS page cache和I/O schedule的优化，磁盘（包括SSD）的顺序读写性能都可以达到几十～几百M每秒
    * 充分利用page cache
    Kafka设计了一套producer/broker/consumer共用的二进制消息格式，这使得producer的数据可以不做逻辑处理直接写入磁盘，这样可以充分利用page cache，避免application cache，充分利用内存、减少JVM GC；
    同时，为了追求更好的性能，Kafka选择每次只是写入page cache之后便告诉producer已经确认，并不主动flush到磁盘，这一点可以提升几个数量级的写入性能。
    Kafka认为，replication能更好的解决数据可靠性问题，写入本地磁盘并不能保证数据可靠（因为大多数存储系统问题都是因为磁盘损坏，而磁盘损坏往往会丢失数据）
    * batch
    批量的好处体现在整个读写流程的多个环节，比如发送端的message压缩比例可以更大、TCP传输的package更少、server处理TCP读取的次数更少、写磁盘的次数更少
    * zero copy
    对于broker，需要频繁的把网络中的message写入磁盘，已经从磁盘读出message写入网络，使用zero copy可以减少CPU的参与，节约CPU cycle、CPU cache以及内存带宽。 
    * end-to-end data compression
    数据压缩，节省带宽。（批量消息一起压缩效果最好）
    
    
- Kafka消息在磁盘上的存储结构是怎样的  
每个topic的每个partition一个目录，目录名格式为：my-topic-0, my-topic-1；
每个partition目录内存放该partition的message，由于message可能很多，Kafka将这些message按segment存储，每个segment一个文件；
segment文件的大小由log.retention相关配置决定，默认最大1G或一周，保留一周，更久的将被删除，删除的最小单元就是segment文件；
segment命名是一个64位整数，值位其包含的第一条message在该partition中的offset，比如0000000003114.log;
segment文件内是一条条message，message的格式根据version的不同有所不同，是producer/broker/consumer之间的协议，整体格式大概是：4byte integer N表示message长度 + N bytes message body.


- Kafka中partition
topic包含多个partition，读取和写入的基本对象都是partition，相同topic的不同partition可以分布在不同更多broker上面，这使得server的水平扩展能力比较好；
每个partition有自己的log文件，决定了单broker上面不能有太多partition，不然会损害顺序读写的性能特性；
consumer的offset是直接跟partition关联的，而不是topic；
replica的也是partition，而不是broker或topic；


- kafka 如何决定每条消息写入哪一个partition  
根据message key做HASH

- 如果producer要写消息到不同partition，需要重新建立TCP连接吗  
应该不需要，producer在初始建立连接的时候，应该要获取到所有broker列表，每个broker包含的topic及partition情况。
然后和所有broker建立连接，并在本地为每个partition维护一个发送队列，publish方法先根据key判断目标partition及所在broker，然后写message到对应发送队列即可。

- Kafka producer写一条消息后，server在什么时候返回ACK(具体步骤)  
取决于producer的ACKS参数设置
    * acks=1，只有leader 写入page cache 就返回
    * acks=all, 所有in-sync的replicas都写入page cache再返回
注意不是写入磁盘

- kafka consumer poll到一条消息后，没有发送ACK, 然后再次poll，会再次拿到这条消息吗？  
不会，会正常的拿到下一条消息（如果有）。因为committed offset只会在发生rebalance的时候才会用到，正常的poll情况下，client会递增poll的offset。
在rebalance的时候，如果某consumer所属group没有committed offset，则根据auto.offset.reset的值决定，默认是返回latest的消息。
（需要注意rebalance之后可能拿到消费过但尚未commit的消息）

- 如何决定consumer group中的member数量  
Kafka中最大可同时POLL的group member数量等于topic partition数量。

- Kafka如何判断一个consumer已经死掉了  
consumer调用`poll()`方法的间隔超过`max.poll.interval.ms` || hearbeats 间隔超过 `session.timeout.ms`.   
The `max.poll.interval.ms` configuration is used to prevent a `livelock`, where the application did not crash but fails to make progress for some reason.
心跳是发给consumer coordinator的，coordinator负责rebalance，返回offset等。

- Kafka rebalance是什么  
Kafka限制任意时间每个consumer group中有且只有一个member可以poll得到message，当这个member离开group，Kafka会进行rebalance操作，
把离开member关联的partition分给其他subscribed members，并且是从这些partition的committed offset之后的消息开始发送给新的member。  
rebalance主要是为了提高Kafka consumer的可用性。
consumer可以发送FindCoordinatorRequest到任意broker获取group coordinator，然后OffsetCommitRequest获取最新的committed offset。

- 如果想要容忍3个节点的失效，Kafka集群应该至少部署几个节点，为什么
Kafka的读写都发生在partition leader，当leader意外终止后，需要选举出新的leader，选举思路类似quorum算法，其使用一个ISR(in-sync replication)数组维护可以被选举为leader（条件是alive并且数据是最新的）的partition，
任意时刻，只要保证ISR中有节点，就能够被选举为新leader。所以要容忍3个节点失效，至少需要部署4个节点并且都加到ISR中。

- Kafka如何应对所有IN-SYNC节点都失效了的情况
取决于配置，可以等待某个in-sync的节点复活，不过可能永远也复活不了；或者随便选一个活着的节点作为leader，但数据可能不是最新的。需要根据业务在可用性和数据一致性方面做取舍。

- controller是什么  
当Kafka节点失效时，其上host的leader partition需要重新在replica中选举出新的leader。为了缩短选举时间，Kafka cluster选了一个broker作为controller，批量处理这个节点上需要重新选举的partition。
controller需要复制zookeeper上的所有metadata。

- zookeeper在Kafka中扮演什么角色，为什么client连接server时不是连接zookeeper  
Kafka 主要使用 ZooKeeper 来保存它的元数据、监控 Broker 和分区的存活状态，并利用 ZooKeeper 来进行选举controller。
Kafka 在 ZooKeeper 中保存的元数据，主要就是 Broker 的列表和主题分区信息两棵树。具体的就是/brokers/[id], /brokers/topics/[topic_name]/partitions/[index]/state
这份元数据同时也被缓存到每个broker上，由于 ZooKeeper 提供了 Watcher 这种监控机制，Kafka 可以感知到 ZooKeeper 中的元数据变化，从而及时更新 Broker 中的元数据缓存。
客户端并不直接和 ZooKeeper 来通信，而是在需要的时候，通过 RPC 请求去 Broker 上拉取它关心的主题的元数据，然后保存到客户端的元数据缓存中，以便支撑客户端生产和消费。
https://www.confluent.io/blog/removing-zookeeper-dependency-in-kafka/

