## 核心##

* 远程通讯：提供对多种基于长连接的NIO框架抽象封装，包括多种线程模型 ，序列化，以及“请求-响应”模式的信息交换方式
* 集群容错：提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持
* 自动发现：基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。



基本配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        ">

	<!-- 具体的实现bean -->
	<bean id="sampleService" class="bhz.dubbo.sample.provider.impl.SampleServiceImpl" />

	<!-- 提供方应用信息，用于计算依赖关系 -->
	<dubbo:application name="sample-provider" />

	<!-- 使用zookeeper注册中心暴露服务地址 -->
	<dubbo:registry address="zookeeper://192.168.1.111:2181?backup=192.168.1.112:2181,192.168.1.113:2181" />

	<!-- 用dubbo协议在20880端口暴露服务 -->
	<dubbo:protocol name="dubbo" port="20880" />

	<!-- 声明需要暴露的服务接口  写操作可以设置retries=0 避免重复调用SOA服务 -->
	<dubbo:service retries="0" interface="bhz.dubbo.sample.provider.SampleService" ref="sampleService" />

</beans>
```



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        ">

	<!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
	<dubbo:application name="sample-consumer" />

	<dubbo:registry address="zookeeper://192.168.1.111:2181?backup=192.168.1.112:2181,192.168.1.113:2181" />

	<!-- 生成远程服务代理，可以像使用本地bean一样使用demoService 检查级联依赖关系 默认为true 当有依赖服务的时候，需要根据需求进行设置 -->
	<dubbo:reference id="sampleService" check="false"
		interface="bhz.dubbo.sample.provider.SampleService" />

</beans>
```





dubbo-admin里面的dubbo-registry-address