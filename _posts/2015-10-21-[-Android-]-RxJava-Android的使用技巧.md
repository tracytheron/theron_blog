---
layout: post
category : Android
tagline : "RxJava Android要点"
tags : [Android]
shortContent : "对于RxJava Android，看到一篇文章，十分棒！这里给大家分享一下。希望对大家认识RxJava-Android有所帮助，本文的受众是对RxJava或者RxJava-Android有一定了解的同学。"
---
{% include JB/setup %}

对于RxJava Android，看到一篇文章，十分棒！这里给大家分享一下。希望对大家认识RxJava-Android有所帮助，本文的受众是对RxJava或者RxJava-Android有一定了解的同学。[原文地址](http://futurice.com/blog/top-7-tips-for-rxjava-on-android)

文章中共分为7点，下面我会伴随着小例子来一一说明。

#### 一，RxJava默认是同步的。

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

#### 二、冷订阅和热订阅

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

#### 三、困境中使用subjects

#### 四、留意线程

#### 五、阅读RxJava的Wiki和图表

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

#### 六、对比订阅Observable和Action

#### 七、Subscriptions的内存泄露问题

