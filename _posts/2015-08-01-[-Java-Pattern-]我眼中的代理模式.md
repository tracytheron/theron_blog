---
layout: post
category : Java
tagline : "我眼中的代理模式"
tags : [Java Pattern]
shortContent : "我理解的代理模式的目的是为了控制对象访问。它可以大致有下面几类，分别是本地代理、远程代理、虚拟代理和动态代理，另外还有很多变种。"
---
{% include JB/setup %}

我理解的代理模式的目的是为了控制对象访问。它可以大致有下面几类，分别是本地代理、远程代理、虚拟代理和动态代理，另外还有很多变种。

##### 一. 本地代理
举个小例子，我是曼联的球迷，曼联新赛季加油啊！我想看英超2015赛季首轮比赛，我想知道曼联和热刺的比分。但是妈妈说：“你还有很多作业没完成，不能看，我会替你去现场看，如果你想知道比分，我就把比分告诉你。” 这样，比赛对我来说是不可见的，而是由我妈来进行了代理，当我想知道比分的时候，让我妈来告诉我比分。我妈也就控制了我对比赛的访问。

首先，得有一个比赛实体Match，内部有主客队比分。
	
	public class Match {
		private String score;
		public Match(String score){
			this.score = score;
		}
		//getter and setter of score
	}
	
然后，得有比赛的代理，或者说是比赛的监控者Mother，她负责接收比赛，从而告诉我比分。
	
	public class Mother {
		private Match match;
		public Mother(Match match){
			this.match = match;
		}
		public void tellMe(){
			System.out.println(match.getScore());
		}
	}

最后,当Mother接收到比赛的时候，她就能够把比赛的结果输出。

	public class Main {
		public static void main(String[] args) {
			Match match = new Match("3:1");
			Mother mother = new Mother(match);
			mother.tellMe();
		}
	}

##### 二. 远程代理
但是，事实是残酷的，我家穷，去不了现场。我妈说，你还是好好做作业，我在电视机上看，同样告诉你比分。这里，可以看到本地代理和远程代理的区别了，本地代理中，比赛Match和监控者Mother在本地，而远程代理中，比赛Match在遥远的英国，在比赛和我妈之间，又出现了一个新的远程对象的本地代表——电视机。电视机作为比赛的代理来被我妈来监控。但在我看来，我妈依旧是我妈，依旧是从我妈妈口中获得比分，好像我妈真的在现场一样。

整理一下思路，我们要写一个本地的调用方法，让后传到网络上去，然后调用远程对象的某些方法。这里涉及到Java的RMI（Remote Method Invocation），Java的远程方法调用。

普及一下实现远程方法调用，一共有四个角色，客户对象Client，客户辅助对象Stub，服务辅助对象Skeleton，服务对象Server。
远程方法调用的流程是这样的：Client向Stub发出doSomething的请求，Stub告诉Skeleton我想要doSomething，Skeleton真正的让Server去doSomething，Server把doSomething的结果返回给Skeleton，Skeleton再把结果通过网络返回给Stub，Stub把结果返回给Client，在Client看来，doSomething就是Stub做的一样。

创建远程方法调用有以下步骤：

**1.创建远程接口**

创建一个远程接口，这个接口内的方法，客户端和服务器端都得实现这个接口。

**2.实现远程接口**

也就是真真正正的doSomething。

**3.执行rmic，产生stub和skeleton**

使用rmic工具来实现客户辅助对象Stub和服务辅助对象Skeleton。而在Java 5以后，不再需要rmic。

**4.执行rmiregistry**

cd到classes目录，使用rmiregistry来注册这个远程方法调用。

**5.启动服务**

开启远程服务
	
	Naming.bind("//IP地址or主机名/注册的名字", 远程对象);

客户端调用远程方法的方式：

	Naming.lookup("//IP地址or主机名/注册的名字");


##### 让我们回到看比赛的例子Step by Step

**服务端实现步骤：**

第一步：
创建远程接口

	public interface MatchRemoteInterface extends Remote{
		public String tellMe() throws RemoteException;//返回的参数一定是Serializable,并必须抛出RemoteException
	}

第二步：
实现远程接口

	public class MatchRemote extends UnicastRemoteObject implements
		MatchRemoteInterface {
		private static final long serialVersionUID = -8990660014833749798L;
		private String score;

		protected MatchRemote(String score) throws RemoteException {
			this.score = score;
		}

		@Override
		public String tellMe() {
			return this.score;
		}
	}
	
第三步：
执行rmic MatchRemote 来生成服务端的skeleton，即服务辅助对象。

第四步：
注册远程服务
cd到classes，并执行rmiregistry完成注册。

第五步：
编写启动服务的代码并启动

	public class RemoteServer {
		public static void main(String[] args) {
			MatchRemote matchRemote = null;
			try {
				matchRemote = new MatchRemote("3:1");
				Naming.bind("//127.0.0.1/MatchRemote", matchRemote);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}

**客户端实现步骤：**
改造原有的Mother类，因为有网络，就必定有异常，看电视，就有可能信号不好。

	public class Mother {
		private MatchRemoteInterface match;
		public Mother(MatchRemoteInterface match){
			this.match = match;
		}
		public void tellMe(){
			try {
				System.out.println(match.tellMe());
			} catch (RemoteException e) {
				e.printStackTrace();
			}
		}
	}

改造原来的Main方法,原来的Mother的参数是本地取得的Match，而现在，Mother的参数是从网络获得的MatchRemote
	
	public class Main {
		public static void main(String[] args) {
			try {
				MatchRemote matchRemote = (MatchRemote) Naming.lookup("//127.0.0.1/MatchRemote");
				Mother mother = new Mother(matchRemote);
				mother.tellMe();
			}catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

##### 三. 虚拟代理
虚拟代理相对比较好理解，用上面看球的例子也可以解释。场景是这样的，我在比赛刚开场5分钟的时候，就问我妈妈，比赛的最终结果是什么？这种场景下，Mother就不能直接返回当前的比分，因为我想知道的是最终结果，也就是说，这场90分钟的比赛是一个开销较大的对象，你只有等他创建完（比赛完）才能访问真正的对象。所以，Mother这时可以返回一个“正在比赛中”的消息，来告知她所代理的对象还在创建中。当对象创建完成后，才把真正要做的事情交给真正的对象去做。可见，使用虚拟代理一般只有在需要访问某个对象的时候才开始创建这个开销比较大的对象，而虚拟代理一般都能返回多种状态。

我们稍微修改一下本地代理的代码：
为了让代理的对象和代理看上去一样，我们定义一个接口，这样，感觉就像自己在看比赛一样。

	public interface IMatch{
		public void tellMe();
	}

改造Match类

	public class Match implements IMatch{
		private String score;
		public Match(String score){
			this.score = score;
		}
		
		@Override
		public void tellMe(){
			System.out.printlen(score);
		}
	}

改造Mother类

	public class Mother implements IMatch {
		private Match match;
		
		public Mother(Match match){
			this.match = match;
		}
			
		@Override
		public void tellMe(){
			if(match != null){
				match.tellMe();
			}else{
				System.out.println("比赛进行中");
			}
		}
	}

当Mother获得match并没有构建完成时，Mother就会告诉我，比赛进行中，否则，就会把控制权交给match,让match来tellMe比赛的结果。

##### 四. 动态代理
动态代理的概念是指代理在运行时创建的。利用Java的Proxy类和InvocationHandler接口来动态创建代理。

举个例子，Alice是一个经纪人。

她可以做的事情，分别为查看自己的名字和别人对我的评价，修改自己的名字和对他人进行评价；
她不能做的事情，分别为修改别人的名字和对自己进行评价。

首先，需要有一个接口，里面包括所有的操作，如下：

	public interface IAgentEntity{
		public String getName();//查看名字
		public void setName(String name);//修改名字
		public int getScore();//查看评价
		public void setScore(int score);//修改评价
	}
	
第二，实现真正的实体类和代理类

	//这是真正的实体类
	public class AgentDetailEntity implements IAgentEntity{
		private String name;
		private int score;
		public AgentDetailEntity(String name,int score){
			this.name = name;
			this.score = score;
		}	
		
		@Override
		public String getName() {
			return name;
		}
		
		@Override
		public void setName(String name) {
			this.name = name;
		}
	
		@Override
		public int getScore() {
			return score;
		}
	
		@Override
		public void setScore(int score) {
			this.score = score;
		}
	}

关键在于代理类如何产生。我们知道，IAgentEntity有所为有所不为。也就是有Owner的代理类，也有非Owner的代理类。即当经纪人Alice自己使用自己这个实体的时候，她应该使用Owner代理类，Owner代理类中对她的行为进行了限制，比如不能修改比人的名字和修改自己的评价。而当别人使用这个实体时，就应该分配给他非Owner的代理类，同样做了些限制，比如不能修改Alice的名字。

Proxy用来创建不同风格的代理类，InvocationHandler来实现代理的行为，在InvocationHandler中定义了哪个方法“可为”，哪个方法“不可为”，在我看来，InvocationHandler算是真正的代理。

我们定义两种InvocationHandler，分别为OwnerInvocationHandler和NotOwnerInvocationHandler.

	public class OwnerInvocationHandler implements InvocationHandler {
		private IAgentEntity agentEntity;	
		public OwnerInvocationHandler(IAgentEntity agentEntity) {
			this.agentEntity = agentEntity;
		}
	
		@Override
		public Object invoke(Object proxy, Method method, Object[] args)
				throws IllegalAccessException {
			try {
				if (method.getName().equals("setScore")) {//当是本人时，不能修改评价
					throw new IllegalAccessException();
				} else {
					method.invoke(agentEntity, args);//合法，就调用agentEntity的方法
				}
			}catch (InvocationTargetException e) {
				e.printStackTrace();
			}
			return null;
		}
	}
	
	public class NotOwnerInvocationHandler implements InvocationHandler {
		private IAgentEntity agentEntity;
	
		public NotOwnerInvocationHandler(IAgentEntity agentEntity) {
			this.agentEntity = agentEntity;
		}
	
		@Override
		public Object invoke(Object proxy, Method method, Object[] args)
				throws IllegalAccessException {
			try {
				if (method.getName().equals("setName")) {//当是他人时，不能修改名字
					throw new IllegalAccessException();
				} else {
					method.invoke(agentEntity, args);//合法，就调用agentEntity的方法				}
			}catch (InvocationTargetException e) {
				e.printStackTrace();
			}
			return null;
		}
	}

上面是两类InvocationHandler，相应的，需要产生两类Proxy，由Proxy类的Proxy.newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)产生。

		private IAgentEntity getOwnerProxy(IAgentEntity agentEntity){
			return (IAgentEntity) Proxy.newProxyInstance(agentEntity.getClass().getClassLoader(),
			 agentEntity.getClass().getInterfaces(), 
			 new OwnerInvocationHandler(agentEntity));
		}
			
		private IAgentEntity getNotOwnerProxy(IAgentEntity agentEntity){
			return (IAgentEntity) Proxy.newProxyInstance(agentEntity.getClass().getClassLoader(),
			 agentEntity.getClass().getInterfaces(),
			 new NotOwnerInvocationHandler(agentEntity));
		}

测试一下：

		IAgentEntity Alice = new AgentDetailEntity("Alice",10);//创建一个Alice实体
		IAgentEntity ownerProxy = getOwnerProxy(Alice);//获得一个Owner代理
		IAgentEntity notOwnerProxy = getNotOwnerProxy(Alice);//获得一个非Owner代理
		
		try {
			ownerProxy.setScore(-1);//Owner改变评价是不允许的
		} catch (Exception e) {
			e.printStackTrace();//会抛出IllegalAccessException
		}
		ownerProxy.setName("Bob");//Owner可以修改自己的名字
		
		try {
			notOwnerProxy.setName("Bob");//非Owner不能修改名字
		} catch (Exception e) {
			e.printStackTrace();//会抛出IllegalAccessException
		}
		notOwnerProxy.setScore(-1);//非Owner可以对别人进行评价

可见，动态代理保护了Alice这个实体的安全性，所以，动态代理也被成为保护代理。

##### 小结
上面介绍了几种代理模式，当然其实还有很多其他的变种，例如缓存代理、同步代理等。代理模式用于对访问对象的控制，对真正的对象进行控制访问，有时加以限制保护，有时加以利用扩展。

**希望曼联这个赛季能顺利一些！Glory Glory ManUnited!**
