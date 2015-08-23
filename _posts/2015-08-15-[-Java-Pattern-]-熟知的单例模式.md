---
layout: post
category : Java
tagline : "熟知的单例模式"
tags : [Java Pattern]
shortContent : "搬宿舍，为啥我的东西这么多呢？虽然只是从410搬到411，还是花了整整一天时间。周末去篮球场和陌生人打球，果然不太适合我，因为我就是去搞笑的，熟人才好玩。喵~"
---
{% include JB/setup %}

搬宿舍，为啥我的东西这么多呢？虽然只是从410搬到411，还是花了整整一天时间。周末去篮球场和陌生人打球，果然不太适合我，因为我就是去搞笑的，熟人才好玩。喵~

言归正传，差不多每个程序员都知道单例模式，而在一些初级笔试面试中也经常出现什么是单例模式，或者让你写一个单例模式的例子的笔试面试题。大部分的程序员也都会写，也都会说，但往往会忽略一些细节，或者遗忘一些东西。这种遗忘，大部分情况看不出来问题，但当问题现象一旦暴露时，很难发现问题所在，并且是致命的。

##### 什么是单例模式？
简单来说，就是程序无论如何运行，无论运行多久，某个对象实例最多只存在一个(可以不存在)。这也就在某种意义下定义了这个实例是全局的。

##### 单例模式和静态类有啥区别？
有人说，单例模式是否可以被静态类代替？在某种意义上讲，可以。但仍然存在一些区别，首先，静态类一开始就会被创建，而单例模式并不会。其次，单例模式可以有继承、多态。第三，单例类更加容易扩展和被测试。还有其他一波不同点。

##### 为什么要用单例模式？
单例模式可以让程序在任何时候，都能从该实例中获取你需要的数据，同时可以保证在同一时间不同的地点取得的都是相同的值，在Alice处设置了实例的某个数据，在Bob处可以取出刚才设置的数据。全局维护着唯一的实例。

##### 怎么写好一个单例模式？
一般知道单例模式的人都能写个大概。没有共有的构造函数，提供一个静态的实例和创建实例的方法。如下：

	public class Singleton { 
		private static final Singleton instance = new Singleton(); 
		public static Singleton getInstance() { 
    		return instance; 
		} 
		private Singleton() { 
		}   
	}

上面这种创建的单例模式，无论这个类会不会被使用，instance实例都会被创建。下面我们进行改进成懒汉模式，即在真正使用的时候才创建实例。

	public class Singleton { 
		private static Singleton instance = null; 
		public static Singleton getInstance() { 
			if(instance == null){
				instance = new Singleton();
			}
    		return instance; 
		} 
		private Singleton() { 
		}   
	}

看上去很好，在单线程的程序中，使用这种懒汉模式的单例已经可以应付了。那么问题来了，在多线程下，这种懒汉模式是否存在问题呢？是的，如果不巧，你将陷入困境。场景，线程Alice想要创建Singleton实例，首先检查instance是否为null，发现是null，那么线程Alice就开始创建instance实例。不巧的是，在Alice检查之后，创建之前，线程Bob开始被执行，同样的，Bob检查instance，因为Alice没有创建，所以Bob看到instance为null，他也开始创建instance实例。创建完之后，Alice被唤起，执行她之前没执行完的创建任务。这样一来，Alice和Bob都创建了instance实例，并使用着各自创建的instance实例。

相应的，解决办法也就出来了，在getInstance方法上加上同步锁，即在同一时间只能有一个线程能够执行getInstance操作，也就保证了instance实例的唯一性。

	public class Singleton { 
		private static Singleton instance = null; 
		public synchronized static Singleton getInstance() { 
			if(instance == null){
				instance = new Singleton();
			}
    		return instance; 
		} 
		private Singleton() { 
		}   
	}

进一步，我们考虑的就是性能问题，同步即意味着性能的降低。我们看一下是否需要在getInstance方法上同步，我们得找一下同步的真正原因是什么，需要同步的真正场景是什么。

首先，在getInstance方法过程中，其实存在两个步骤，一个是检查instance实例的存在性，第二是创建instance实例。正是因为这两个步骤，导致我们必须给他加上同步锁。也就是说，真正应该同步的并不是getInstance方法，而是这两个步骤。所以，改进如下：

	public class Singleton { 
		private static Singleton instance = null; 
		public static Singleton getInstance() { 
			synchronized (Singleton.class) { 
				if(instance == null){
					instance = new Singleton();
				}
			}
    		return instance; 
		} 
		private Singleton() { 
		}   
	}

第二，如果instance本身就不为null，那就没有必要进入同步模块，直接返回就好了。继续改进：

	public class Singleton { 
		private static Singleton instance = null; 
		public static Singleton getInstance() { 
			if(instance == null){
				synchronized (Singleton.class) { 
					if(instance == null){
						instance = new Singleton();
					}
				}
			}
    		return instance; 
		} 
		private Singleton() { 
		}   
	}

接着，这一切都看似完美，但一道晴天霹雳再次击碎了你。JVM在创建对象是需要过程的，而这个过程并没有一个标准流程。它可以先为实例分配一块内存空间，然后调用构造方法，然后分配指针指向内存空间；也可以先分配指针，最后再调用构造方法来进行初始化。那么问题来了，如果是后者，上面写的单例模式就有问题了。

线程Alice开始创建Singleton的实例，此时线程Bob调用了getInstance()方法，首先判断instance是否为null。Alice已经把instance指向了那块内存，只是还没有调用构造方法，因此Bob检测到instance不为null，所以直接把instance返回了。尽管instance不为null，但它并没有构造完成，如果Bob在Alice将instance构造完成之前就是使用这个实例，呵呵，完蛋。继续改进。

##### 改进的单例模式

第一种，是利用Java为我们提供的volatile关键字。volatile的作用在于对单个读/写做了同步。所以我们的单例可以这么写：

	public class Singleton { 
		private volatile static Singleton instance = null; 
		public static Singleton getInstance() { 
			if(instance == null){
				synchronized (Singleton.class) { 
					if(instance == null){
						instance = new Singleton();
					}
				}
			}
    		return instance; 
		} 
		private Singleton() { 
		}   
	}

第二种，利用Java的静态内部类来实现，代码如下，使用这种方式，将创建instance实例的工作交给了Singleton的静态内部类来完成。由于类的构造是原子的，所以不需要同步。而且由于Singleton并不是静态类，所以内部的SingletonInstance类也不会被一开始就加载。同时，instance实例是static的，所以也不会有多个实例的出现。

	public class Singleton { 
		private static class SingletonInstance { 
    		private static final Singleton instance = new Singleton(); 
		} 

		public static Singleton getInstance() { 
			return SingletonInstance.instance; 
		} 

		private Singleton() { 
		}  
	}

第三种，比较少见，在Java1.5以后，可以使用枚举类来实现单例。

	public enum Singleton { 
		INSTANCE;
	}


##### 小结
单例模式是比较简单易懂的一种设计模式，只是针对不同的使用场景，需要一些变动。对于单线程程序，使用最为简单的单例即可，没有必要考虑多线程情况下的种种场景。对于多线程，volatile关键字能个我们带来些方便，但需要注意的是，该关键字是在Java1.5版本之后明确其语义，所以，使用静态内部类来绕开这个变量，是个比较不错的选择。而使用枚举类来实现单例虽然看起来有点怪，但是却是一个最简洁最有效的方式。
