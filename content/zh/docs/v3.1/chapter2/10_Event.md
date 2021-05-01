---
type: docs
title: "事件机制"
linkTitle: "事件机制"
weight: 11
description: "本文事件的相关机制"
---

框架基于ApplicationEvent定义了MaculaEvent事件

```java
public abstract class MaculaEvent extends ApplicationEvent {

	private static final long serialVersionUID = 1L;
	
	protected Serializable source;
	
	/**
	 * @param source
	 */
	public MaculaEvent(Serializable source) {
		super(source);
		this.source = source;
	}
	
	@Override
	public Serializable getSource() {
		return source;
	}
}
```
基于MaculaEvent，定义了异步事件 AsyncMaculaEvent
```java
public class AsyncMaculaEvent extends MaculaEvent {

	private static final long serialVersionUID = 1L;
	
	private ApplicationId applicationId;

	/**
	 * @param source
	 */
	public AsyncMaculaEvent(Serializable source) {
		super(source);
		this.applicationId = ApplicationId.current();
	}
	
	public String getInstanceIdentityKey() {
		return applicationId != null ? applicationId.toString() : "NONE";
	}
	
	/**
	 * @return the applicationId
	 */
	public ApplicationId getApplicationId() {
		return applicationId;
	}

	/**
	 * @param applicationId the applicationId to set
	 */
	public void setApplicationId(ApplicationId applicationId) {
		this.applicationId = applicationId;
	}
	
	public boolean isSourceInstance() {
		return isSourceInstance(ApplicationId.current());
	}

	public boolean isSourceInstance(ApplicationId applicationId) {
		return this.getApplicationId() != null && this.getApplicationId().equals(applicationId);
	}

	public boolean isSourceApplication() {
		return isSourceApplication(ApplicationId.current());
	}

	public boolean isSourceApplication(ApplicationId applicationId) {
		return this.getApplicationId() != null && this.getApplicationId().sameGroup(applicationId)
				&& this.getApplicationId().sameApplication(applicationId);
	}
}
```

MaculaEvent监听到后系统会同步调用其对应的Listener，而AsyncMaculaEvent会另外开启一个线程调用其对应的Listener实现。

基于AsyncMaculaEvent，框架定义了BroadcastEvent和ClusteredEvent, 前者会被同一个应用组中的所有实例接收，而后者只有应用中的一个实例接收，类似消息队列的Topic和Queue。BroadcastEVent框架默认处理，ClusteredEvent的处理见阿里MQ插件的介绍。

Macula框架支持多实例集群模式，一个应用有多个实例组成，多个应用属于一个分组，为了解决同一个应用组的应用之间、应用实例之间的同步和通讯问题，框架通过BroadcastEvent事件广播的方式进行通信，如下配置：

```
#是否关闭事件广播
#macula.disableBroadcast = true

#事件广播方式，默认是redis，可以配置http、redis
macula.events.transport = redis
```

* HTTP通信方式
框架通过您在应用配置中定义的应用及其实例以及他们的内部地址，利用HttpInvoker的方式把事件广播给相应的实例。由于HTTP网络的不可靠，这种方式不能保证100%成功广播所有事件。

* REDIS订阅方式
利用REDIS的Subpub功能，在实例启动的时候注册到redis的topic上，当需要广播事件是只要向topic发送事件，这种模式需要配置相应的redisTemplate，如下：
```xml
<alias name="redisTemplate" alias="transportRedisTemplate" />
```
这里是共用了Redis，可以单给定义transportRedisTemplate到不同的Redis集群上。