&emsp;&emsp;RocketMQ的消息存储是由consume queue和commit log配合完成的。  
1、Consume Queue  
&emsp;&emsp;consume queue是消息的逻辑队列，相当于字典的目录，用来指定消息在物理文件commit log上的位置。
&emsp;&emsp;我们可以在配置中指定consumequeue与commitlog存储的目录，每个topic下的每个queue都有一个对应的consumequeue文件：
Consume Queue文件组织，如图所示：
![](/assets/20180730092330951.png)
&emsp;&emsp;根据topic和queueId来组织文件，图中TopicA有两个队列0,1，那么TopicA和QueueId=0组成一个ConsumeQueue，TopicA和QueueId=1组成另一个ConsumeQueue。
&emsp;&emsp;按照消费端的GroupName来分组重试队列，如果消费端消费失败，消息将被发往重试队列中，比如图中的%RETRY%ConsumerGroupA。
&emsp;&emsp;按照消费端的GroupName来分组死信队列，如果消费端消费失败，并重试指定次数后，仍然失败，则发往死信队列，比如图中的%DLQ%ConsumerGroupA。
&emsp;&emsp;Consume Queue中存储单元是一个20字节定长的二进制数据，顺序写顺序读，如下图所示：
![](/assets/20180730092424448.png)
&emsp;&emsp;CommitLog Offset是指这条消息在Commit Log文件中的实际偏移量；
&emsp;&emsp;Size存储中消息的大小；
&emsp;&emsp;Message Tag HashCode存储消息的Tag的哈希值：主要用于订阅时消息过滤（订阅时如果指定了Tag，会根据HashCode来快速查找到订阅的消息）。
2、Commit Log
CommitLog：消息存放的物理文件，每台broker上的commitlog被本机所有的queue共享，不做任何区分。
CommitLog的消息存储单元长度不固定，文件顺序写，随机读。消息的存储结构如下表所示，按照编号顺序以及编号对应的内容依次存储。
![](/assets/20180730092535621.png)
3、消息的索引文件
如果一个消息包含key值的话，会使用IndexFile存储消息索引，文件的内容结构如图：
![](/assets/20180730092604373.png)