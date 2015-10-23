---
layout: post
category : Android
tagline : "RxJava Android的使用技巧"
tags : [Android]
shortContent : "对于RxJava Android，看到一篇文章，十分棒！这里给大家分享一下。希望对大家认识RxJava-Android有所帮助，本文的受众是对RxJava或者RxJava-Android有一定了解的同学。"
---
{% include JB/setup %}

对于RxJava Android，看到一篇文章，十分棒！这里给大家分享一下。希望对大家认识RxJava-Android有所帮助，本文的受众是对RxJava或者RxJava-Android有一定了解的同学。[原文地址](http://futurice.com/blog/top-7-tips-for-rxjava-on-android)

文章中共分为7点，下面来一一说明。

##### 一，RxJava默认是同步的。

我们在使用RxJavaAndroid的时候，一般都是用来处理数据库读取、图片加载、网络请求、响应点击事件等一些不明确什么时候有返回的操作。也就是说，一般我们将它用作异步处理，来保证UI的顺畅不阻塞。然而，RxJava本身是同步的。
下面举个小例子：

这是我们定义的Observable
	
	Observable<String> observable = Observable.create(new 		Observable.OnSubscribeFunc<String>() {
        @Override
        public Subscription onSubscribe(Observer<? super String> observer) {
            try {
                Log.d("thread1", Thread.currentThread().getId() + "");
                observer.onNext("hello");
                observer.onCompleted();
            } catch (Exception e) {
                observer.onError(e);
            }
            return Subscriptions.empty();
        }
    });
    //            .subscribeOn(Schedulers.threadPoolForIO())
	//            .observeOn(AndroidSchedulers.mainThread());

这是我们的点击事件：

	mClickButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                observable.subscribe(new Action1<String>() {
                    @Override
                    public void call(String s) {
                        Log.d("thread2", Thread.currentThread().getId() + "");
                        resultTextView.setText(s);
                    }
                });
            }
        });

如果只是这样，当点击事件触发时，Log中记录的是：

	thread1:1
	thread2:1

可见，在不加任何控制的情况下，RxJava是同步的，在一个线程中运行。当我们将Observable上的设置程序运行的线程后（将上面的注释取消），运行之后，Log中记录的是：

	thread1:5587
	thread2:1

可见，两个线程异步执行，且onSubscribe在子线程，call在主线程。

##### 二、冷订阅和热订阅

冷订阅是指只有当有Subscriber时，Observable才开始运作，不然Observable不会工作。这个热订阅不一样，热订阅是指事件源一直在运行，即使他没有Subscriber，他也会将运行的结果散布给他的Subscriber。

RxJava是冷订阅，可以从Observable的create方法中可以看出，只有当有Subscriber的时候，才会触发onSubscribe来生成一个Subscription。并且，默认每次有Subscriber的时候，都会产生一个新的Observable来响应生成Subscription。我们来看一个小例子来证明：

	Observable<String> observable = Observable.create(new 	Observable.OnSubscribeFunc<String>() {
        @Override
        public Subscription onSubscribe(Observer<? super String> observer) {
            try {
                observer.onNext(new Random().nextInt(100)+"");
                observer.onCompleted();

            } catch (Exception e) {
                observer.onError(e);
            }
            return Subscriptions.empty();
        }
    })
            .subscribeOn(Schedulers.threadPoolForIO())
            .observeOn(AndroidSchedulers.mainThread())
      //      .cache()
            ;
      
     mClickButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                observable.subscribe(new Action1<String>() {
                    @Override
                    public void call(String s) {
                        Log.d("click1", s);
                    }
                });
                observable.subscribe(new Action1<String>() {
                    @Override
                    public void call(String s) {
                        Log.d("click2", s);
                    }
                });
            }
        }); 
 
上面的例子中，为Observable添加了两个Subscriber，点击之后，输出的Log中，click1和click2的数字的值是不相同的，也就是说，Observable分别对两个Subscriber进行了onSubscribe运算。但是，如果我们在Observable上设置.cache()方法(即将注释取消)，则click1和click2的数字输出是一样的，且不会有变化，即对运算结果进行了缓存。

##### 三、困境中使用subjects

Subject在RxJava中是作为Observer和Observerable的一个bridge或者proxy。因为它是一个观察者，所以它可以订阅一个或多个可观察对象，同时因为他是一个可观测对象，所以它可以传递和释放它观测到的数据对象，并且能释放新的对象。使用Subject能给我们提供很多帮助，包括BehaviorSubject，AsyncSubject，PublishSubject和ReplaySubject。

当Observer订阅了一个BehaviorSubject，它一开始就会释放Observable最近释放的一个数据对象，当还没有任何数据释放时，它则是一个默认值。接下来就会释放Observable释放的所有数据。如果Observable因异常终止，BehaviorSubject将不会向后续的Observer释放数据，但是会向Observer传递一个异常通知。

AsyncSubject仅释放Observable释放的最后一个数据，并且仅在Observable完成之后。然而如果当Observable因为异常而终止，AsyncSubject将不会释放任何数据，但是会向Observer传递一个异常通知。

PublishSubject仅会向Observer释放在订阅之后Observable释放的数据。

不管Observer何时订阅ReplaySubject，ReplaySubject会向所有Observer释放Observable释放过的数据。有不同类型的ReplaySubject，它们是用来限定Replay的范围，例如设定Buffer的具体大小，或者设定具体的时间范围。如果使用ReplaySubject作为Observer，注意不要在多个线程中调用onNext、onComplete和onError方法，因为这会导致顺序错乱，这个是违反了Observer规则的。


##### 四、留意线程

有些情况，我们没必要跳出主线程，可以在主线程中处理一些逻辑并改变UI。但有些情况，因为耗时的操作，不得不将这耗时的操作异步处理，我们得保证，在我们改变我们的UI之前，异步的操作已经将结果返回了。

在RxJava-Android中，我们使用subscribeOn()来将Observable内的方法异步处理，使用observeOn(AndroidSchedulers.mainThread())来将结果返回到主线程中。

另外，自定义的Observable只有在onSubscribe方法完成之后才能取消订阅，也就是说，如果正在执行一个长时间的网络请求，一般情况下没什么问题，但有时会有这种情况，你的请求还没返回，你想要改变的View已经被回收了，这个时候，你还没办法取消之前的请求。


##### 五、阅读RxJava的Wiki和图表

GitHub 地址为 [RxJava](https://github.com/ReactiveX/RxJava/wiki)

在使用RxJava过程中，有一些方法和技巧是特别常用和有用的。

1. 如果你用的是Java8，那么可以使用lambda表达式来简化你的代码；
2. from方法。它既可以用作集合，也可以用作遍历；
3. just方法。传递一个Object给onNext并调用onComplete；
4. map方法。可以添加任意多的方法，来改变最终的输出；
5. filter方法。对最终的输出进行过滤，返回值为true时为输出，false为不输出；
6. create方法。可以通过Observable创建一个自定义的Observable；
7. merge方法。可以将两个或者两个以上的Observable合在一起，每个Observable都会响应；
8. error方法。创建一个专门用于输出错误的Observable；
9. combineLatest方法。该方法用于比较多个Observable的结果，因此他的参数包括多个Observable和一个比较函数，比较函数的名称和参数都和Observable的数量相关；
10. cache方法。上面说过，会缓存之前的输出值；
11. mapMany/flatMap方法。能够将Observable的输出传递给另一个Observable。
12. zip方法。将多个Observable一一对应合并处理。例：Observable1产生A，Observable2产生B，则最终结果是A和B通过Fun1产生的值。如果Observable2还没有。

##### 六、对比订阅Observable和Action

订阅Observable的时候，我们可以实现完整的Observer，即包含onNext,onComplete和onError的方法，也可以使用简单的Action1。在Observable上订阅Action1时，他只会接受onNext的返回，如果有错误，即返回onError，那就会抛出IllegalArgumentException的异常。我们最好在Action1之前，定义一个专门用来接收Error和Log的Observer，只把onNext和onComplete传递下去。或者，在Observable上订阅Action时，使用subscribe(Action1 onNext, Action1 onError)。或者，实现一个完整的Observer。

##### 七、Subscriptions的内存泄漏问题

内存泄漏这问题可大可小，我们要做的是知道在哪儿泄漏和如何尽可能地防止内存泄漏。
我们在Observable上添加了一个订阅，会将一个强引用交给Observer，如果Observer是个匿名类，那么只要Obseverable没被销毁，那么Observer就不会被回收。另外，调用subscribe方法会返回一个subscription，在程序的任何地方都可以取消这个订阅，因此，我们必须在合适的地方来取消订阅，但有时候，我们没有把握能够找到这个合适的点。

一个比较合适的方式来解决内存泄漏的问题是将何时取消订阅等操作，交给Activity或者Fragment来处理。
##### 小结

对这篇文章还是有些不太清楚的地方，大家可以一起探讨。RxJava 的设计理念尤其适合Android这种请求与响应的软件架构，能够轻松地实现异步处理。我们使用过程中要注意上述的一些情况，并且在使用中，可以对一些方法进行二次封装，比如网络请求，我们可以将网络请求和响应，以及返回的数据解析，异常的处理等进行统一处理。好东西要灵活使用。Good Luck~