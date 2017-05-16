- sh mqadmin clusterList -n '192.168.1.110:9876;192.168.1.110:9876'

  sh mqshutdown broker
  sh mqshutdown namesrv

  nohup sh mqnamesrv & 

  nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-noslave/broker-a.properties >/dev/null 2>&1 & 

  tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/broker.log 

  tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log 

- 注意rocketmq的producer和consumer启动顺序。会影响batch（批量拉数据的条数）  先启动consumer再启动producer

- producer重试;consumer重试（time out,exception）;mq重试(没有收到状态码)

- 同步双写(sync 效率低一些 主、从都写才返回)  异步复制(async 主写就返回) <主从节点的策略>

- Topic主题，每个主题表示一个逻辑上的概念，而在MQ上会有与之对应的多个queue队列，这个是物理存储的概念。

- NormalProducer(普通) OrderProducer(顺序),TransactionProducer(事物)

- MessageListenerOrderly  启动consumer之前可以指定线程池大小  

- 顺序消费（从两端考虑）：P->同样的"订单"(业务)指定到一个queue中，保证进入同一个队列

  C-> 指定MessageListenerOrderly  接口，保证一个线程从一个queue中拿数据，其他线程只能干瞪眼

- TranscationMQProducer LocalTransactionExcuter(执行本地事物，由客户端回调)

  检查本地事物分支是成功还是失败   producer.setTransacaitonState

- 消息重试、顺序消费、事物机制

![producer回掉](C:\Users\Anthow\Desktop\IMG\producer回掉.png)

高版本开原版已经去掉了，需要自己手动处理！

rocketmq的事物消息：producer：把消息发给mq然后返回消息（即表示已经发送成功一条消息），该消息给本地事物回调，接着执行一个本地事物，根据是否成功返回不同的标识，并把标识发送给MQ



![rmq事物](C:\Users\Anthow\Desktop\IMG\rmq事物.png)



过程：（1）首先producer会发一条消息到mq

​	    （2）然后mq发送一个确认消息给producer

​	    （3）执行本地事物，然后根据执行的结果发送确认消息给mq。即这个时候mq会产生2条消息

​	      

* 过滤器：在mq端进行过滤，可以进行配置过滤器的个数，customer上传java文件（文件中不可以含有中文），然后在过滤器上取消息。

  ​