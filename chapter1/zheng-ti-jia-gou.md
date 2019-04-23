###MyMQ架构

+ Broker
	+ Broker.java
	+ BrokerResponseProcessor.java
	+ Filter.java
	+ LoadBalancer.java
	+ MyQueue.java
	+ Slave.java
	+ SlaveResponseProcessor.java
	+ Synchronizer.java

&emsp;&emsp;Broker包的作用主要是创建Broker实例对象，以及提供主从同步，负载均衡，消息过滤服务。

+ Common
	+ IpNode.java
	+ Message.java
	+ MessageType.java
	+ PullMessage.java
	+ RegisterMessage.java
	+ Topic.java

&emsp;&emsp;Common包定义了一些通用的类，如消息类，地址类等。

+ Consumer
	+ ConsumerFactory.java
	+ ConsumerResponseProcessor.java

&emsp;&emsp;消费者包定义了消费者工厂，可通过工厂方法添加消费者。

+ Producer
	+ DelaySyscProducerFactory.java
	+ SyscProducerFactory.java
	+ UnidirectionalProducerFactory.java

&emsp;&emsp;生产者包定义了生产者工厂，支持同步生产者工厂，延时生产者工厂和单向生产者工厂。

+ Test
	+ ConsumerTest.java
	+ DaoTest.java
	+ BrokerTest.java
	+ ProducerTest.java
	+ PressTest.java

&emsp;&emsp;测试包，里面包含了MyMQ的基本使用方法。

+ Utils
	+ Client.java
	+ DefaultRequestProcessor.java
	+ DefaultResponseProcessor.java
	+ JsonFormatUtil.java
	+ PersistenceUtil.java
	+ MessageUtil.java
	+ RequestProcessor.java
	+ ResponseProcessor.java
	+ SequenceUtil.java
	+ SerializeUtil.java
	+ Server.java

&emsp;&emsp;工具包，定义了一些通用的工具类。

