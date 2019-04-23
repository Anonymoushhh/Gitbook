###主要架构与功能实现详解

####消息结构
```
public class Message implements Serializable{

	private static final long serialVersionUID = 1L;
	private int num;//消息序号
	private String message;//消息
	private int type;//消息类型
	private Topic topic;//消息主题
	...
	}
```
&emsp;&emsp;Message类实现了序列化接口，每个Message有消息序号（该序号是否具有唯一性由使用者决定），消息内容，消息类型和消息主题。消息内容由使用者自己定义，可以是某个手机号（用于给该手机号发送短信）或订单信息（用于更新数据库）等等。消息类型定义了5种：
```
	public static final int ONE_WAY = 0;//单向消息
	public static final int REPLY_EXPECTED = 1;//需要得到回复的消息
	public static final int REQUEST_QUEUE = 2;//请求包,用户生产者向Broker申请队列
	public static final int REGISTER = 3;//用于消费者向Broker注册
	public static final int PULL = 4;//用于消费者向Broker注册
```
&emsp;&emsp;消息主题类Topic定义如下：
```
	private HashSet<Integer> queueId;//该Topic在Broker中对应的queueId
	private HashSet<IpNode> consumer_address;//该Topic对应的cunsumer
	String topic_name;//主题名称
	int queueNum;//请求队列数
```
&emsp;&emsp;该类同样实现了序列化接口，主要用于记录消息主题名称，请求队列数，请求队列号和消费者地址。当用户首次定义一个Topic时，需要向Broker申请分配可用的消息队列号，之后将可用的队列号存储进Topic中，以后使用该Topic时就无需申请队列。

####消息存储
```
public class MyQueue implements Serializable{
	private static final long serialVersionUID = 1L;
	private ConcurrentLinkedDeque<Message> queue;
	}
```
&emsp;&emsp;MyQueue定义了消息存储队列，它的实现是一个同步的双向队列，一个Broker中可以同时存在一个或多个队列。

####消息过滤
```
public HashMap<IpNode, List<Message>> filter(List<Message> list) {
		//将Message按照分发地址分类
		HashMap<IpNode, List<Message>> map = new HashMap<IpNode, List<Message>>();
		//初始化
		for(IpNode address:index) {
			if(map.get(address)==null) {
				map.put(address, new ArrayList<Message>());
			}
		}
		//遍历消息，将每条message分类
		Iterator<Message> iterator = list.iterator();
		while(iterator.hasNext()) {
			Message message = iterator.next();
			//每个message可能有很多消费者
			List<IpNode> consumer_address = message.getTopic().getConsumer();
			Iterator<IpNode> it = consumer_address.iterator();
			while(it.hasNext()) {
				IpNode address = it.next();
				List<Message> l = map.get(address);
				if(l!=null)
					l.add(message);
			}
		}
		return map;
	}
```
&emsp;&emsp;过滤器的主要作用就是将要发送的消息按照消费者地址分类，一个消息可能有一个或多个消费者。

####消息分发（Push模式与Pull模式）
```
//为消费者推送消息
	private void pushMessage() {
		HashMap<IpNode, List<Message>> map = filter(index,poll(1));
		for(IpNode ip:map.keySet())
				{
					List<Message> message = map.get(ip);
					for(Message m:message) {
						Client client = clients.get(ip);
						if(client!=null) {
							int i=0;
							for(i=0;i<reTry_Time;i++) {//失败重试三次
								String ack=null;
								try {
									ack = client.SyscSend(m);
								} catch (IOException e) {
									System.out.println("发送失败！正在重试...");
								}
								if(ack!=null)
									break;
							}
						if(i>=reTry_Time) {
							//todo 进入死信队列
						}
						}else {
							System.out.println("消费者不存在");
							//todo 进入死信队列
						}					
					}		
				}
	}
	//push模式
	public void push() {
		new Thread(){
	        public void run() {
	        	while(true) {
	        		try {
	    				Thread.sleep(push_Time);
	    			} catch (InterruptedException e) {
	    				e.printStackTrace();
	    			}
	        		pushMessage();
	        	}
	    		
	        };
	}.start();
	}
```
&emsp;&emsp;push模式启动一个线程，每次push过程是所有队列出队一个元素，使用过滤器将所有消息分类，发送给相应的消费者，如果发送失败则重试一定次数（默认16次），次数达到上限后依然失败的话会进入死信队列，并告知相应的生产者。

####负载均衡
```
public static List<Integer> balance(ConcurrentHashMap<String,MyQueue> queueList,int queueNum){
		//此时queueList的size一定大于queueNum
		List<Integer> list = new ArrayList<>();
		for(int i=0;i<queueNum;i++) {
			int index = 0;
			int min = Integer.MAX_VALUE;
			for(java.util.Map.Entry<String, MyQueue> entry:queueList.entrySet()) {
				if(entry.getValue().size()<min&&!list.contains(Integer.valueOf(entry.getKey()))) {
					min = entry.getValue().size();
					index = Integer.valueOf(entry.getKey());
				}
			}
			list.add(index);
		}
		return list;
	}
```

&emsp;&emsp;负载均衡器提供一个负载均衡的方法，遍历队列找到前queueNum小的队列号。

####主从备份
```
//slave同步
        new Thread(){
          public void run() {
              	while(true) {
              		if(hasSlave) {
              			try {
							Thread.sleep(sync_Time);
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
              			Synchronizer sync = new Synchronizer(queueList, index);
              			try {
							String s = SerializeUtil.serialize(sync);
							for(IpNode ip:slave) {
								Client client = new Client(ip.getIp(), ip.getPort());
								client.Send(s);
							}
						} catch (IOException e) {
							System.out.println("Slave未上线!");
						}
              		}
              	}
          };
      }.start();
```
&emsp;&emsp;Broker会在init方法中创建一个线程。如果创建带Slave节点备份的消息队列的话,该线程会不停的向Slave节点同步消息，同步不可保证强一致性。

####持久化存储（同步或异步刷盘）与冗机恢复

```
//持久化
        new Thread(){
            public void run() {
            	while(true) {
            		if(startPersistence) {
            			try {
							Thread.sleep(store_Time);
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
            			try {
            				String path = PersistenceUtil.class.getResource("").getPath().toString().substring(1);
            				File file = new File(path);
            				String newPath1 = file.getParentFile().getParent()+"\\QueueList.json";
            				String newPath2 = file.getParentFile().getParent()+"\\ConsumerAddress.json";
                        	PersistenceUtil.Export(PersistenceUtil.persistenceQueue(broker.queueList),newPath1);
                        	PersistenceUtil.Export(PersistenceUtil.persistenceConsumer(broker.index),newPath2);
        				} catch (IOException e) {
        					e.printStackTrace();
        				}
            		}
            	}
            };
        }.start();
```
&emsp;&emsp;Broker在init方法中创建一个线程。如果用户开启持久化功能，该线程会每隔一段时间将队列内容写入磁盘，存储格式为2个json，一个存队列内容，一个存消费者地址。
&emsp;&emsp;若不幸冗机，用户可根据recover方法来恢复Broker。

```
//恢复Broker
	public void recover() {
		String path = PersistenceUtil.class.getResource("").getPath().toString().substring(1);
		File file = new File(path);
		String newPath1 = file.getParentFile().getParent()+"\\QueueList.json";
		String newPath2 = file.getParentFile().getParent()+"\\ConsumerAddress.json";
		ConcurrentHashMap<String,MyQueue> queueList = PersistenceUtil.Extraction(PersistenceUtil.Import(newPath1));
		this.setQueueList(queueList);
		List<IpNode> address= PersistenceUtil.ExtractionConsumer(PersistenceUtil.Import(newPath2));
		for(IpNode ipNode:address)
			addConsumer(ipNode);
	}
```
####生产者工厂(这里以延时同步工厂为例)
```
private static ConcurrentHashMap<IpNode, Boolean> requestMap= new ConcurrentHashMap<IpNode, Boolean>();
	private static int reTry_Time = 16;
	private static int Delay_Time = 2000;//延时时间默认2s
```
&emsp;&emsp;requestMap用于记录该消费者地址是否已向Broker注册，reTry_Time定义发送失败重试的次数，Delay_Time定义了延时发送时间。
&emsp;&emsp;生产者需先向Broker申请队列：
```
public static Topic RequestQueue(Topic topic,String ip,int port){//输入为一个topic，里面包含请求的队列个数
		System.out.println("请求向Broker申请队列...");
		Topic t = topic;
		Message m = new Message("RequestQueue",MessageType.REQUEST_QUEUE,t, -1);
		String queue = DelaySyscProducerFactory.SendQueueRegister(m, ip, port);
		String[] l = queue.substring(7).split(" ");
		for(String i:l)
			topic.addQueueId(Integer.parseInt(i));
		IpNode ipNode = new IpNode(ip, port);
		requestMap.put(ipNode, true);
		return t;
	}
```
&emsp;&emsp;申请队列时向Broker发送一个MessageType.REQUEST_QUEUE类型的消息：
```
private static String SendQueueRegister(Message msg,String ip,int port) {//未申请队列返回null
		Client client;
	if(msg.getType()!=MessageType.REPLY_EXPECTED&&msg.getType()!=MessageType.REQUEST_QUEUE)
			msg.setType(MessageType.REPLY_EXPECTED);
		try {
			client = new Client(ip, port);
			//失败重复，reTry_Time次放弃
			for(int i=0;i<reTry_Time;i++) {
				String result = client.SyscSend(msg);
				if(result!=null) {
					System.out.println("队列申请成功！");
					return result;
				}	
				if("".equals(result))
					return null;
			}
		} catch (IOException e) {
			System.out.println("Broker未上线！");
		}
		return null;
	}
```
&emsp;&emsp;Broker收到该消息后会返回可用的消息队列序号，生产者工厂将这些消息序号添加到topic中，之后就可用该topic发送消息了：
```
//发送成功返回值为消息号+ACK
//发送失败返回值为null
	public static String Send(Message msg,String ip,int port) {//未申请队列返回null
		IpNode ipNode = new IpNode(ip, port);
		if(requestMap.get(ipNode)==null) {
			System.out.println("未向Broker申请队列！");
			return null;
		}
		//等待Delay_Time秒
		try {
			Thread.sleep(Delay_Time);
		} catch (InterruptedException e1) {
			e1.printStackTrace();
			return null;
		}
		Client client;
		if(msg.getType()!=MessageType.REPLY_EXPECTED&&msg.getType()!=MessageType.REQUEST_QUEUE)
			msg.setType(MessageType.REPLY_EXPECTED);
		//失败重复，reTry_Time次放弃
		for(int i=0;i<reTry_Time;i++) {
			try {
				client = new Client(ip, port);
				String result = client.SyscSend(msg);
				if(result!=null)
					return result;
				if("".equals(result))
					return null;
			} catch (IOException e) {
				System.out.println("生产者消息发送失败，正在重试第"+(i+1)+"次...");
			}
		}
		return null;
	}
```
&emsp;&emsp;若发送成功返回值为消息号+空格+ACK，发送失败返回值为null。
####消费者工厂
```
	private static ConcurrentHashMap<Integer, ConcurrentLinkedQueue<Message>> map = new ConcurrentHashMap<Integer,ConcurrentLinkedQueue<Message>>();
```
&emsp;&emsp;这个map用于缓存Broker发来的消息，键为本地端口号，值为该消费者的消息缓冲区。
&emsp;&emsp;消费者工厂调用createConsumer向Broker注册消费者：
```
public static void createConsumer(IpNode ipNode1/*Broker地址*/,IpNode ipNode2/*本地地址*/) throws IOException {
		if(map.containsKey(ipNode2.getPort())) {
			System.out.println("端口已被占用!");
			return;
		}
		ConsumerFactory.register(ipNode1,ipNode2);
		ConsumerFactory.waiting(ipNode2.getPort());
		map.put(ipNode2.getPort(), new ConcurrentLinkedQueue<Message>());
	}
```
&emsp;&emsp;register方法向Broker发送注册消息：
```
//向Broker注册
	private static void register(IpNode ipNode1/*目的地址*/,IpNode ipNode2/*本地地址*/){
		System.out.println("正在注册Consumer...");
		Client client;
		try {
			client = new Client(ipNode1.getIp(), ipNode1.getPort());
			RegisterMessage msg = new RegisterMessage(ipNode2, "register", 1);
			if(client.SyscSend(msg)!=null)
				System.out.println("注册成功!");
			else
				System.out.println("注册失败！");
		} catch (IOException e) {
			System.out.println("Connection Refuse.");
		}
		
	}
```
&emsp;&emsp;waiting方法的作用是在某个端口监听，接受消息队列发送来的消息。
```
//在某个端口监听
	private static void waiting(int port) throws IOException {
		DefaultRequestProcessor defaultRequestProcessor = new DefaultRequestProcessor();
		ConsumerResponeProcessor consumerResponeProcessor = new ConsumerResponeProcessor();
		new Thread(){
            public void run() {
            	System.out.println("Consumer在本地端口"+port+"监听...");
            	try {
					new Server(port,defaultRequestProcessor,consumerResponeProcessor);
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
                };
		}.start();
	}
```
