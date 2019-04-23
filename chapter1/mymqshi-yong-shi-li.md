###使用示例

####Producer
```java
SequenceUtil Sequence = new SequenceUtil();//新建一个序列号工具类实例
//创建一个消息主题Topic（包含Topic名称和请求队列个数）向Broker请求分配队列，

//同步消息示例
//返回值为一个新的Topic，里面包含了分配的队列编号
Topic topic = SyscProducerFactory.RequestQueue(new Topic("topic",1)/*请求队列的Topic*/, "127.0.0.1", 81);
//为消息主题添加消费者地址
topic.addConsumer(new IpNode("127.0.0.1", 8888));
int num = Sequence.getSequence();//获得全局唯一的序号
Message msg = new Message("message"+num,topic, num);//定义消息，指定消息内容，主题和序号
SyscProducerFactory.setReTry_Time(16);//设置发送失败重试次数
String string = SyscProducerFactory.Send(msg, "127.0.0.1", 81);//同步发送

//延时消息示例
Topic topic2 = DelaySyscProducerFactory.RequestQueue(new Topic("topic",1), "127.0.0.1", 81);
topic.addConsumer(new IpNode("127.0.0.1", 8888));
int num2 = Sequence.getSequence();//获得全局唯一的序号
Message msg2 = new Message("message"+num2,topic2, num2);//定义消息，指定消息内容，主题和序号
DelaySyscProducerFactory.setDelay_Time(1000);//设置延时发送时间
String string2 = DelaySyscProducerFactory.Send(msg2, "127.0.0.1", 81);//延时发送消息
System.out.println(string2);

//单向消息示例
Topic topic3 = UnidirectionalProducerFactory.RequestQueue(new Topic("topic",1), "127.0.0.1", 81);
topic.addConsumer(new IpNode("127.0.0.1", 8888));
int num3 = Sequence.getSequence();//获得全局唯一的序号
Message msg3 = new Message("message"+num3,topic3, num3);//定义消息，指定消息内容，主题和序号
UnidirectionalProducerFactory.Send(msg3, "127.0.0.1", 81);
```
####Broker
```java
//创建Broker(主从复制，push模式)
try {
		IpNode slaveIpNode = new IpNode("127.0.0.1", 83);//备份服务器地址
		List<IpNode> list = new ArrayList<IpNode>();
		list.add(slaveIpNode);
		Broker broker = new Broker(81,list);创建Broker节点，在81端口监听
		//push模式
		broker.setPush_Time(1000);//设置Broker推送时间
		broker.push();//创建推送服务
		} catch (IOException e) {
			e.printStackTrace();
	}
```
```
//创建Broker(非主从复制，push模式)
		//Broker(非主从复制)
				try {
					Broker broker = new Broker(81);
					broker.setPush_Time(1000);
					broker.setReTry_Time(16);
					broker.setSync_Time(1000);
					broker.setStore_Time(1000);
					broker.setStartPersistence(true);
					broker.push();
				} catch (IOException e) {
					e.printStackTrace();
				}
```
```
//创建Broker(非主从复制,pull模式)
	try {
		Broker broker = new Broker(81);
		} catch (IOException e) {
			e.printStackTrace();
	}
```
Consumer
```
//创建Consumer（Push模式）
		IpNode ipNode1 = new IpNode("127.0.0.1", 81);
		IpNode ipNode2 = new IpNode("127.0.0.1", 8888);//消费者地址
		try {
			ConsumerFactory.createConsumer(ipNode1, ipNode2);
		} catch (IOException e1) {
			System.out.println("Broker未上线！");
		}
		while(true) {
			try {
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
    		Message m1 = ConsumerFactory.getMessage(8888);
    		if(m1!=null) 
				System.out.println("消费者"+ipNode2.getIp()+ipNode2.getPort()+"收到消息："+m1.getMessage());	
		}
```
```
//创建Consumer（Pull模式）
		IpNode ipNode3 = new IpNode("127.0.0.1", 81);
		IpNode ipNode4 = new IpNode("127.0.0.1", 8888);
    	try {
			ConsumerFactory.createConsumer(ipNode3, ipNode4);
		} catch (IOException e) {
			System.out.println("Broker未上线！");
		}
		while(true) {
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
    		ConsumerFactory.Pull(ipNode3, ipNode4);
	}
```

