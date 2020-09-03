## RocketMQ源码学习  

Apache RocketMQ是一个分布式消息传递和流媒体平台，具有低延迟，高性能和可靠性，万亿级容量和灵活的可伸缩性。
它具有多种功能：

* 发布/订阅消息传递模型
* 财务级交易消息
* 各种跨语言客户端，例如Java，C / C ++，Python，Go
* 可插拔的传输协议，例如TCP，SSL，AIO
* 内置的消息跟踪功能，还支持开放式跟踪
* 多功能的大数据和流生态系统集成
* 按时间或偏移量追溯消息
* 可靠的FIFO和严格的有序消息传递在同一队列中
* 高效的推拉消费模型
* 单个队列中的百万级消息累积容量
* 多种消息传递协议，例如JMS和OpenMessaging
* 灵活的分布式横向扩展部署架构
* 快如闪电的批量消息交换系统
* 各种消息过滤器机制，例如SQL和Tag
* 用于隔离测试和云隔离群集的Docker映像
* 功能丰富的管理仪表板，用于配置，指标和监视
* 认证与授权

----------
RocketMQ源码主要分为以下几个package：
rocketmq-broker：mq的核心，它能接收producer和consumer的请求，并调用store层服务对消息进行处理。HA服务的基本单元，支持同步双写，异步双写等模式。    
rocketmq-client：mq客户端实现，目前官方仅仅开源了java版本的mq客户端，c++，go客户端有社区开源贡献。   
rocketmq-common：一些模块间通用的功能类，比如一些配置文件、常量。   
rocketmq-filter：消息过滤服务，相当于在broker和consumer中间加入了一个filter代理。   
rocketmq-namesrv：命名服务，更新和路由发现 broker服务。    
rocketmq-remoting：基于netty的底层通信实现，所有服务间的交互都基于此模块。  
rocketmq-store：存储层实现，同时包括了索引服务，高可用HA服务实现。   
rocketmq-tools：mq集群管理工具，提供了消息查询等功能。   

----

RocketMQ主要的功能集中在rocketmq-broker、rocketmq-remoting、rocketmq-store 三个模块中   

#### Client生产者发送消息
1.org.apache.rocketmq.client.producer.DefaultMQProducer.send(org.apache.rocketmq.common.message.Message)  
2.defaultMQProducerImpl.send(msg);   
3.DefaultMQProducerImpl.sendKernelImpl()  
4.DefaultMQProducerImpl.mQClientFactory.getMQClientAPIImpl().sendMessage()  
5.MQClientAPIImpl.sendMessageSync()
6.RemotingClient.remotingClient.invokeSync(addr, request, timeoutMillis)  
7.NettyRemotingAbstract->channel.writeAndFlush(request).addListener()  写入通道，等待Netty的Selector轮询出来



#### Broker(发送消息到达Broker)后续流程
1.消息到达Broker()
-NettyRemotingClient.NettyClientHandler.channelRead0 
-processMessageReceived() processRequestCommand(ctx, cmd)  
-NettyRequestProcessor【接口】.processRequest(ctx, cmd);
-SendMessageProcessor【具体实现类】.processRequest()    
3.SendMessageProcessor.brokerController.getMessageStore().putMessage(msgInner);【非事务消息存储】   
4.刷盘 


#### consumer消息获取流程  
DefaultMQPushConsumer:系统控制读取操作
DefaultMQPullConsumer:读取操作大部分功能自己控制 


#### nameserver中5个变量RouteInfoManager
HashMap<String/* topic */, List<QueueData>> topicQueueTable;   
HashMap<String/* brokerName */, BrokerData> brokerAddrTable;   
HashMap<String/* clusterName */, Set<String/* brokerName */>> clusterAddrTable;   
HashMap<String/* brokerAddr */, BrokerLiveInfo> brokerLiveTable;   
HashMap<String/* brokerAddr */, List<String>/* Filter Server */> filterServerTable;   




