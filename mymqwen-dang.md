#MyMQ文档与使用指南
###MyMQ架构简介
+ Broker
	+ Broker.java
	+ BrokerResponseProcessor.java
	+ Filter.java
	+ LoadBalancer.java
	+ MyQueue.java
	+ Slave.java
	+ SlaveResponseProcessor.java
	+ Synchronizer.java
+ Common
	+ IpNode.java
	+ Message.java
	+ MessageType.java
	+ PullMessage.java
	+ RegisterMessage.java
	+ Topic.java
+ Consumer
	+ ConsumerFactory.java
	+ ConsumerResponseProcessor.java
+ Producer
	+ DelaySyscProducerFactory.java
	+ SyscProducerFactory.java
	+ UnidirectionalProducerFactory.java
+ Test
	+ ConsumerTest.java
	+ DaoTest.java
	+ BrokerTest.java
	+ ProducerTest.java
+ Utils
	+ Client.java
	+ DefaultRequestProcessor.java
	+ DefaultResponseProcessor.java
	+ MessageUtil.java
	+ RequestProcessor.java
	+ ResponseProcessor.java
	+ SequenceUtil.java
	+ SerializeUtil.java
	+ Server.java
	
###MyMQ使用指南
####Broker.Broker
Method|Description
---|:--:
public Broker(int port)|构造方法，让Broker在某个port监听
public Broker(int port,int queueNum)|构造方法，显示指定初始队列数量
public Broker(int port,List<IpNode> slave)|构造方法，同时创建备份broker
public Broker(int port,int queueNum,List<IpNode> slave)|构造方法
private void init(int port)|初始化Broker，包括初始化成员变量，默认创建十个生产者队列，创建Server对象在port监听，创建一个线程与slave通信
public void setQueueList(ConcurrentHashMap<String, MyQueue> queueList)|设置队列内容，用于slave同步
public static void setSync_Time(int sync_Time)|设置同步时间，默认1s
public void setPushTime(int time)|设置Push时间间隔默认1s
public void setReTry_Time(int reTry_Time) |设置重试次数，默认为16
public void getAll()|打印队列内容
public void addConsumer(IpNode ipNode)|添加消费者
private void pushMessage()|为消费者推送消息，push方法调用
public void pullMessage(IpNode ipNode)|pull模式
private synchronized void createQueue(int queueNum)|创建队列
public List<Integer> choiceQueue(int queueNum)|当生产者请求队列时，根据负载均衡选择压力最小的队列
public synchronized void add(int queueNumber,Message value)|将消息添加到某个队列中
public synchronized List<Message> poll(int num)|每个队列出队num个元素
public HashMap<IpNode, List<Message>> filter(List<IpNode> index,List<Message> list)|根据消费者信息过滤消息
####Broker.BrokerResponseProcessor
Method|Description
---|:--:
public void processorRespone(final SelectionKey key,Broker broker)|根据不同的消息类型做出不同的反应
####Broker.Filter
Method|Description
---|:--:
public Filter(List<IpNode> index)|构造方法，输入为全部消费者地址列表
public HashMap<IpNode, List<Message>> filter(List<Message> list)|将Message按照地址分类
####Broker.LoadBalancer
Method|Description
---|:--:
public static synchronized List<Integer> balance(ConcurrentHashMap<String,MyQueue> queueList,int queueNum)|找到前queueNum小的队列号
####Broker.MyQueue
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
Method|Description
---|:--:
public Slave(int port1,int port2)|构造方法，port1为slave监听端口，port2为slaveBroker监听端口
public void Sync(Synchronizer synchronizer)|同步函数，输入为同步器
####Broker.SlaveResponseProcessor
Method|Description
---|:--:
public void processorRespone(final SelectionKey key,Slave slave)|根据Slave服务器的消息类型做出不同反应
####Broker.Synchronizer
Method|Description
---|:--:
public Synchronizer(ConcurrentHashMap<String, MyQueue> queueList, List<IpNode> index)|构造方法，输入为队列列表和消费者地址集合
public ConcurrentHashMap<String,MyQueue> getQueueList()|返回队列集合
public List<IpNode> getIndex()|返回消费者地址
####Common.IpNode
Method|Description
---|:--:
public IpNode(String ip, int port)|构造方法，定义一个网络地址
public String getIp()|返回ip
public int getPort()|返回端口
public void setIp(String ip)|设置ip
public void setPort(int port)|设置端口
####Common.Message
Method|Description
---|:--:
public Message(String s,Topic topic,int num)|构造方法，输入为消息内容，消息主题，消息序号
public Message(String s,int type,int num)|构造方法，输入为消息内容，消息类型，消息序号
public Message(String s,int type,Topic topic,int num)|构造方法，输入为消息内容，消息类型，消息主题，消息序号
public String getMessage()|返回消息内容
