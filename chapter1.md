#MyMQ文档与使用指南
###MyMQ简介
&emsp;&emsp;MyMQ是一个简单版的消息队列，它的架构主要分为三部分：Producer，Broker和Consumer。
&emsp;&emsp;生产者支持同步发送消息和发送单向消息，生产者发送消息时需先通过一个消息主题向Broker申请队列，Broker根据自己的负载情况返回给生产者可用队列号，生产者将队列号添加到topic中，并用该消息主题发送消息；
&emsp;&emsp;Broker中有许多队列，每个队列中消息顺序一定，队列对消息主题Topic可以是多对多，一对多，多对一的关系，具体如何使用由使用者决定。Broker支持负载均衡和消息过滤功能，对消费者提供Push和Pull两种模式。Broker还实现了主从同步（Slave节点）和队列持久化存储与恢复来保证消息的可靠性。若消息由于网络原因发送失败时会重试，默认为16次，发送成功（返回ACK）或返回失败消息后才会发送下一条消息，以此来保证消息的有序性；
&emsp;&emsp;消费者可以同步获取消息，延时获取消息，支持Push和Pull两种模式。
&emsp;&emsp;Producer，Broker和Consumer三者支持单机和分布式环境，通过NIO的Socket通信。
&emsp;&emsp;Producer，Broker和Consumer三者均支持横向扩展，增加新的机器对旧的服务没有任何影响，保证了高可用性。
