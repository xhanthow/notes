* zk不适合存储大量数据，至少3个节点部署（奇数个节点），在分布式环境中保证数据的一致性
* 一半以上的节点挂了以后，就不对外提供服务。
* ZAB原子消息广播（Paxos）

### 特点：

* 顺序一致性：从一个客户端发起的事务请求，最终将会严格地按照其发起的顺序被应用到zk中去
* 原子性：所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，也就是说，要么整个集群所有的机器都成功应用了某一事务，要么都没有应用。
* 单一视图：无论客户端连接的是哪一个zk服务器，其要看的服务器数据模型都是一致的。
* 可靠性：一旦服务器成功应用了一个事务，并完成客户端的响应，那么该事务所引起的服务器端状态将会被一致保留下来。除非有另外一个事务对其更改。
* 实时性：通常所说的实时性是指一旦事务被成功应用，那么客户端就能立刻从服务器上获取变更后的新数据，zk仅仅能保证在一段时间内，客户端最终一定能从服务器端读取到最新的数据状态。




配置环境变量

![zk环境变量](C:\Users\Anthow\Desktop\IMG\zk环境变量.png)




### 目标：

* 简单的数据结构。树形命名 空间
* 可以构建集群
* 顺序访问。对于来自每一个客户端的每一个请求，zk都会分配一个全局唯一的递增编号，这个编号反应了所有事务操作的先后顺序
* 高性能。由于zk将全局数据存储在内存中，并直接服务于所有的非事务请求，因此尤其在读操作为主的场景下性能非常突出。



* delete 删除单个节点，如果有子节点不能删。rmr递归删除



分布式锁：同一时间只有一个客户端对zk进行操作，因而可以用这个机制来创建分布式锁。用户1首先get一个节点，没有则创建，然后进行操作，操作完了就删除了这个节点，临时节点，如果有的话就表示其他客户端在进行操作，自己只能撤了。创建一个节点相当于一个锁

先get在create：由于zk数据存在内存中，读的性能非常高。

![分布式锁](C:\Users\Anthow\Desktop\IMG\分布式锁.png)



watch只会生效一次，如果要每次都生效，需要手动再次添加watch为true。

连接状态：event.getState()

事件状态：event.getType()



temp节点只存在一个会话周期，zk关闭之后就消失。

zkClient可以递归创建节点。不像原生节点每个api都有watch这个参数，取而代之的是subscribeChildChanges，不需要每次注册watch，只监听自己或者子节点的新增或者删除。subscribeDataChanges，监听节点的修改。二者配合使用

 ![zkwatch](C:\Users\Anthow\Desktop\IMG\zkwatch.png)



apache的curator框架，链式编程风格

![curator](C:\Users\Anthow\Desktop\IMG\curator.png)





![curator监听节点1](C:\Users\Anthow\Desktop\IMG\curator监听节点1.png)





![curator监听节点2](C:\Users\Anthow\Desktop\IMG\curator监听节点2.png)





分布式计数器:

```java
	/** zookeeper地址 */
	static final String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";
	/** session超时时间 */
	static final int SESSION_OUTTIME = 5000;//ms 
	
	public static void main(String[] args) throws Exception {
		
		//1 重试策略：初试时间为1s 重试10次
		RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 10);
		//2 通过工厂创建连接
		CuratorFramework cf = CuratorFrameworkFactory.builder()
					.connectString(CONNECT_ADDR)
					.sessionTimeoutMs(SESSION_OUTTIME)
					.retryPolicy(retryPolicy)
					.build();
		//3 开启连接
		cf.start();
		//cf.delete().forPath("/super");
		

		//4 使用DistributedAtomicInteger
		DistributedAtomicInteger atomicIntger = 
				new DistributedAtomicInteger(cf, "/super", new RetryNTimes(3, 1000));
		
		AtomicValue<Integer> value = atomicIntger.add(1);
		System.out.println(value.succeeded());
		System.out.println(value.postValue());	//最新值
		System.out.println(value.preValue());	//原始值
```



curator:程序启动时，自动注册，很强这个功能。