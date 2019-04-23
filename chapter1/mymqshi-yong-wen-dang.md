###MyMQ使用指南
####Broker.Broker
&emsp;&emsp;Broker为消息队列服务器节点，提供的服务有：消息存储，消息分发（Push模式与Pull模式），失败重试机制，消息过滤，负载均衡，死信队列，主从备份，持久化存储（同步或异步刷盘）与冗机恢复，横向扩展等。

Method|Description
---|:--:
public Broker(int port)|构造方法，让Broker在某个port监听
public Broker(int port,int queueNum)|构造方法，显示指定初始队列数量
public Broker(int port,List<IpNode> slave)|构造方法，同时创建备份broker
public Broker(int port,int queueNum,List<IpNode> slave)|构造方法
private void init(int port)|初始化Broker，包括初始化成员变量，默认创建十个生产者队列，创建Server对象在port监听，创建一个线程与slave通信
public void setStartPersistence(boolean startPersistence)|开启或关闭持久化功能
public void setQueueList(ConcurrentHashMap<String, MyQueue> queueList)|设置队列内容，用于slave同步
public void setStore_Time(int store_Time)|设置刷盘时间
public static void setSync_Time(int sync_Time)|设置同步时间，默认1s
public void setPush_Time(int time)|设置Push时间间隔默认1s
public void setReTry_Time(int reTry_Time) |设置重试次数，默认为16
public void getAll()|打印队列内容
public void recover()|恢复Broker
public void addConsumer(IpNode ipNode)|添加消费者
private void pushMessage()|为消费者推送消息，push方法调用
public void pullMessage(IpNode ipNode)|pull模式
private synchronized void createQueue(int queueNum)|创建队列
public List<Integer> choiceQueue(int queueNum)|当生产者请求队列时，根据负载均衡选择压力最小的队列
public synchronized void add(int queueNumber,Message value)|将消息添加到某个队列中
public synchronized List<Message> poll(int num)|每个队列出队num个元素
public HashMap<IpNode, List<Message>> filter(List<IpNode> index,List<Message> list)|根据消费者信息过滤消息
####Broker.BrokerResponseProcessor
&emsp;&emsp;该类实现了ResponseProcessor接口，为Broker制定了特殊的消息响应机制。

Method|Description
---|:--:
public void processorRespone(final SelectionKey key,Broker broker)|根据不同的消息类型做出不同的反应
private void addToBroker(Message msg,Broker broker)|将消息添加到Broker
####Broker.Filter
&emsp;&emsp;消息过滤器，将消息按照消费者地址分类。

Method|Description
---|:--:
public Filter(List<IpNode> index)|构造方法，输入为全部消费者地址列表
public HashMap<IpNode, List<Message>> filter(List<Message> list)|将Message按照地址分类
####Broker.LoadBalancer
&emsp;&emsp;负载均衡器，用于为生产者选择一个合适的消息队列。

Method|Description
---|:--:
public static synchronized List<Integer> balance(ConcurrentHashMap<String,MyQueue> queueList,int queueNum)|找到前queueNum小的队列号
####Broker.MyQueue
&emsp;&emsp;消息队列类，保证了消息的顺序性。

Method|Description
---|:--:
public MyQueue()|构造方法，初始化队列
public void putAtHeader(Message value)|在队列头插入消息
public Message getAndRemoveTail()|返回并移除队列尾元素
public Message getTail()|返回队尾元素
public int size()|返回队列大小
public void getAll()|打印队列元素
public List<Message> getReverseAll()|逆序列
####Broker.Slave
&emsp;&emsp;备份节点类，用于Slave的同步或异步备份。

Method|Description
---|:--:
public Slave(int port1,int port2)|构造方法，port1为slave监听端口，port2为slaveBroker监听端口
public void Sync(Synchronizer synchronizer)|同步函数，输入为同步器
####Broker.SlaveResponseProcessor
&emsp;&emsp;用于指定备份节点的特殊消息响应机制。

Method|Description
---|:--:
public void processorRespone(final SelectionKey key,Slave slave)|根据Slave服务器的消息类型做出不同反应
####Broker.Synchronizer
&emsp;&emsp;同步器，用于Broker主从节点的同步。

Method|Description
---|:--:
public Synchronizer(ConcurrentHashMap<String, MyQueue> queueList, List<IpNode> index)|构造方法，输入为队列列表和消费者地址集合
public ConcurrentHashMap<String,MyQueue> getQueueList()|返回队列集合
public List<IpNode> getIndex()|返回消费者地址
####Common.IpNode
&emsp;&emsp;定义一个网络地址。

Method|Description
---|:--:
public IpNode(String ip, int port)|构造方法，定义一个网络地址
public String getIp()|返回ip
public int getPort()|返回端口
public void setIp(String ip)|设置ip
public void setPort(int port)|设置端口
####Common.Message
&emsp;&emsp;定义了传输的消息结构。

Method|Description
---|:--:
public Message(String s,Topic topic,int num)|构造方法，输入为消息内容，消息主题，消息序号
public Message(String s,int type,int num)|构造方法，输入为消息内容，消息类型，消息序号
public Message(String s,int type,Topic topic,int num)|构造方法，输入为消息内容，消息类型，消息主题，消息序号
public String getMessage()|返回消息内容
public int getType()|返回消息类型
public void setType(int type)|设置消息类型，若类型不存在，设置为默认值1
public Topic getTopic()|返回消息主题
public void setTopic(Topic topic)|设置消息主题
public int getNum()|返回消息序号
public void setNum(int num)|设置消息序号
####Common.MessageType
&emsp;&emsp;定义了消息类型。

Method|Description
---|:--:
private static Set<Integer> getSet()|返回消息类型集合
public static boolean contains(Integer i)|判断类型是否合法
####Common.PullMessage
&emsp;&emsp;一种特殊的消息，用于消费者向Broker拉取消息。

Method|Description
---|:--:
public PullMessage(IpNode ipNode,String message,int num)|构造方法，构造一个请求拉取消息的消息
public IpNode getIpNode()|获得地址信息
public int getNum()|获得消息序号
public int getType()|获得消息类型
public String getMessage()|获得消息内容
####Common.RegisterMessage
&emsp;&emsp;一种特殊的消息，用与消费者向Broker注册。

Method|Description
---|:--:
public RegisterMessage(IpNode ipNode,String message,int num)|构造方法，构造一个Consumer注册消息
public IpNode getIpNode()|返回地址信息
public int getNum()|返回消息序号
public int getType()|返回消息类型
public String getMessage()|返回消息内容
####Common.Topic
&emsp;&emsp;消息主题。

Method|Description
---|:--:
public Topic(String s,int queueNum)|构造方法，输入为主题内容，请求队列数
public Topic(String s,HashSet<Integer> queueId,HashSet<IpNode> consumer_address)|构造方法，输入为主题内容，请求队列集合，消费者集合
private List<IpNode> transform(HashSet<IpNode> set)|HashSet元素转换为线性表
private List<Integer> transformforInteger(HashSet<Integer> set)|同上
public String getTopicName()|获得主题名字
public List<Integer> getQueue()|获得队列编号
public List<IpNode> getConsumer()|获得消费者列表
public void addConsumer(IpNode ipnode)|添加消费者
public void deleteConsumer(IpNode ipnode)|删除消费者
public void addQueueId(int i)|添加队列
public int getQueueNum()|获得请求队列数
####Consumer.ConsumerFactory
&emsp;&emsp;消费者工厂类，用于创建消费者。

Method|Description
---|:--:
private static void register(IpNode ipNode1,IpNode ipNode2)|消费者向Broker注册，输入为目的地址，本地地址
private static void waiting(int port)|消费者在某个端口监听消息
public static void createConsumer(IpNode ipNode1,IpNode ipNode2)|向Broker申请创建消费者
public static ConcurrentLinkedQueue<Message> getList(int port)|返回某个在某个端口监听的消息队列
public static Message getMessage(int port)|返回在某个端口的消息
public static void Pull(IpNode ipNode1,IpNode ipNode2)|请求拉取消息
####Consumer.ConsumerResponeProcessor
&emsp;&emsp;为消费者指定特殊的消息响应机制。

Method|Description
---|:--:
public void processorRespone(final SelectionKey key,int port)|消费者对消息的监听处理方法
####Producer.SyscProducerFactory
&emsp;&emsp;同步生产者工厂。

Method|Description
---|:--:
public static void setReTry_Time(int reTry_Time)|设置重试次数
private static String SendQueueRegister(Message msg,String ip,int port)|发送队列注册消息，失败返回null，成功返回 RequestQueue ACK
public static Topic RequestQueue(Topic topic,String ip,int port)|请求申请队列，输入为一个topic和目的地址，里面包含请求的队列个数
public static String Send(Message msg,String ip,int port)|发送消息
####Producer.DelaySyscProducerFactory
&emsp;&emsp;延时生产者工厂。

Method|Description
---|:--:
public static void setDelay_Time(int delay_Time)|设置延时发送时间，其余方法同上
####Producer.UndirectionalProducerFactory
&emsp;&emsp;单向消息生产者工厂。
&emsp;&emsp;API同SyscProducerFactory。
####Utils.Client
&emsp;&emsp;NIO通信模型客户端类，用于发送消息和接受回复。

Method|Description
---|:--:
public Client(String ip,int port)|构造方法，输入为目标地址
private void init(String ip,int port)|Client初始化
public String SyscSend(String msg)|同步发送字符串消息
public void Send(String msg)|单向发送字符串
public String SyscSend(Message msg)|同步发送消息对象
public void Send(Message msg)|单向发送消息对象
public String receive()|接受消息
####Utils.DefaultRequestProcessor
&emsp;&emsp;默认的请求接收响应类。

Method|Description
---|:--:
public void processorRequest(final SelectionKey key,Server server)|默认的请求处理方法
####Utils.DefaultResponeProcessor
&emsp;&emsp;默认的请求回复响应类。

Method|Description
---|:--:
public void processorRespone(final SelectionKey key)|默认的请求响应方法
####Utils.RequestProcessor接口
&emsp;&emsp;请求接收响应接口。

Method|Description
---|:--:
public void processorRequest(final SelectionKey key,Server server)|消息处理方法
####Utils.ResponseProcessor接口
&emsp;&emsp;请求回复响应接口。

Method|Description
---|:--:
default void processorRespone(final SelectionKey key)|默认空实现，为实现接口的类服务
default void processorRespone(final SelectionKey key,Broker broker)|默认空实现，为实现接口的类服务
default void processorRespone(final SelectionKey key,int port)|默认空实现，为实现接口的类服务
default void processorRespone(final SelectionKey key,Slave slave)|默认空实现，为实现接口的类服务
####Utils.SequenceUtil
&emsp;&emsp;生成唯一序列号的工具类（单机唯一）。

Method|Description
---|:--:
public synchronized int getSequence()|返回一个唯一的序列化（单机环境下唯一）
####Utils.SerializeUtil
&emsp;&emsp;序列化工具类。

Method|Description
---|:--:
public static String serialize(Object obj)|对象序列化为字符串
public static Object serializeToObject(String str)|字符串反序列化为对象
####Utils.Server
&emsp;&emsp;NIO通信模型服务器类，在某个端口上监听消息。

Method|Description
---|:--:
public Server(int port,RequestProcessor requestProcessor,ResponseProcessor responeProcessor)|构造方法，创建一个服务端对象
public Server(int port,RequestProcessor requestProcessor,ResponseProcessor responeProcessor,Broker broker)|构造方法，创建一个服务端对象，并为某个Broker服务
public Server(int port,RequestProcessor requestProcessor,ResponseProcessor responeProcessor,Slave slave)|构造方法，创建一个服务端对象，并为某个Slave服务
public void addWriteQueen(SelectionKey key)|添加SelectionKey到队列
void init(int port)|在某个端口上创建Server服务，初始化Server
void start(int port)|在某个端口上开始监听

