---
title: RXJava 2.X基本用法(一)
---
# 前言
开始使用 RxJava 有一段时间了，刚刚入手学会RxJava1的走，看到Rxjava2出来了，本以为只是普通版本的升级，没当回事，过了段时间看到网上在聊RxJava2.X与1.X的异同，了解完后瞬间心里有一万只神兽奔腾。忍痛放弃了1.X的学习直奔2.X,不是我喜新厌旧。只是都是入门学习还是学点新的。况且2.X的学习不依赖1.X.
目前随便一百度关于RxJava都出来一大推文章，有人会问你现在才开始写会不会太迟了。前一段时间一直忙于项目，最近回头整理知识，发现有些知识点有点模糊，还容易混淆两个版本，于是决定系统性的整理一下笔记同时查漏补缺一下。所以这系列文章的目的有两个： 
- 给对`RxJava2.X`感兴趣的小伙伴一些入门的探讨 
- 给刚入门使用`RxJava2.X`但仍然心存疑惑的人一些我学习中遇到的疑问的解答

# 1.基本概念
要想理解好RxJava，首先有几个关键概念需要我们理解清楚。由于RxJava是利用观察者模式来实现一些列的操作，所以对于观察者模式中的观察者，被观察者，以及订阅、事件需要有一个了解。如果第一次看完是一脸懵逼，那很正常，不要慌，记得这几个概念就好~后面我会慢慢的带你们走进去跟走出来。
- **被观察者**： <font color=#ff0000>Observable</font>在观察者模式中称为“被观察者”,用于创造事件；
- **观察者**： <font color=#ff0000>Observer</font>观察者模式中的“观察者”，可接收Observable发送的事件,定义响应事件的行为；
**订阅**：<font color=#ff0000>subscribe</font>，用于连接观察者与被观察者；
**Subscriber**：也可以是一种观察者，在2.X中 它与Observer没什么实质的区别，不同的是 Subscriber要与Flowable(也是一种被观察者)联合使用，该部分内容是2.X新增的，后续文章再介绍。Obsesrver用于订阅Observable，而Subscriber用于订阅Flowable

# 2. 基本使用
![](https://i.imgur.com/4L97dKg.png)
## 2.1 实现方式1：分步实现
### 2.1.1步骤1：创建被观察者(**<font color=#ff0000>Observable</font>**)并生产事件
- 具体实现
	*Observable的创建方式很多,我们这边介绍最基本的create创建方式,其他的创建方式后续介绍*
```Java
		Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
			public void subscribe(@NonNull ObservableEmitter<String> e)
					throws Exception {
				e.onNext("产生事件1");
				System.out.println( "Observable emit 1" + "\n");
				e.onNext("产生事件2");
                System.out.println( "Observable emit 2" + "\n");
                e.onNext("产生事件3");
                System.out.println( "Observable emit 3" + "\n");
                e.onComplete();
                e.onNext("产生事件4");
                System.out.println( "Observable emit 4" + "\n" );
			}
		});
```
	
### 2.1.2步骤2：创建观察者 （**<font color=#ff0000>Observer</font>**）并定义响应事件的行为
- 发生的事件类型包括：Next事件、Complete事件 & Error事件。具体如下：
*本想通过表格实现的，发现md表格不能合并单元格，后来通过html实现，发现发布后页面格式变了(⊙o⊙)…要是哪个小伙伴知道这么直接在md文件里绘制这样的表格，希望不吝赐教o(*￣︶￣*)o*
![](https://i.imgur.com/CqhT5Ua.jpg)

- 具体实现
	- 方式1：采用Observer抽象接口
		- 创建观察者(Observer)对象
		```Java
		Observer<String> observer = new Observer<String>() {
			// 观察者接收事件前，默认最先调用复写 onSubscribe()
			public void onSubscribe(@NonNull Disposable d) {
				// onSubscribe 是2.x新添加的方法，在发射数据前被调用，相当于1.x的onStart方法
			}
			public void onComplete() {
				// 对Complete事件作出响应，如上面事件类型详细介绍里的，被观察者可以不发送该事件
			}
			public void onError(@NonNull Throwable arg0) {
				// onError事件作出响应，如上面事件类型详细介绍里的，被观察者也可以不发送该事件
			}

			public void onNext(@NonNull String arg0) {
				// 对onNext事件作出响应，被观察者可以发送多个该事件
			}
		};
```
	- 方式2：采用Subscriber接口
		- 创建观察者(Observer)对象
	```Java
	Subscriber<String> subscriber = new Subscriber<String>() {
			// 观察者接收事件前，默认最先调用复写 onSubscribe()
			public void onSubscribe(@NonNull Disposable d) {
				// onSubscribe 是2.x新添加的方法，在发射数据前被调用，相当于1.x的onStart方法
			}
			public void onComplete() {
				// 对Complete事件作出响应，如上面事件类型详细介绍里的，被观察者可以不发送该事件
			}
			public void onError(@NonNull Throwable arg0) {
				// onError事件作出响应，如上面事件类型详细介绍里的，被观察者也可以不发送该事件
			}
			public void onNext(@NonNull String arg0) {
				// 对onNext事件作出响应，被观察者可以发送多个该事件
			}
		};
```

### 2.1.3 步骤3：订阅事件(**<font color=#ff0000>Subscribe</font>**),连接观察者与被观察者
通过上面两步我们已经成功创建了观察者`Observer`和被观察者`Observable`，但他们目前两对是独立的，我们需要通过`Subscribe`来连接这两个对象，实现方式只需要一行代码就可以：
```Java
observable.subscribe(observer);//对于观察者采用Observer抽象接口实现
```
需要注意的是**<font color=#ff0000>~~observable.subscribe(subscriber)~~</font>**这个在RxJava2.X里使用是会报错的,小伙伴们或许会问那我要是想用Subscriber该怎么办呢？你要是这样问，那我只能说你没认真看哦~一开始讲基本概念时候特地强调了**<font color=#ff0000>Subscriber要与Flowable联合使用</font>**，所以具体的实现方式如下：
```Java
//我们需要修改被观察者的实现方式，采用Flowable
Flowable<String> flowable = Flowable.create(new FlowableOnSubscribe<String>() {
			public void subscribe(@NonNull FlowableEmitter<String> e)
					throws Exception {
				//实现方式跟observable里一样滴
			}
		}, BackpressureStrategy.ERROR);//需要注意的是这里增加了一个参数，Flowable具体讲解在后续会介绍
flowable.subscribe(subscriber);//对于观察者采用Subscriber接口实现时连接观察者跟被观察者的方式
```
## 2.2 实现方式2：基于事件流的链式实现
- 方式1的实现方式是为了从基本概念上来说明Rxjava的原理和使用观察者模式各大参与者的角度的方式实现
- 在实际应用中，会将上述步骤和代码连在一起，从而体现RxJava的简洁、优雅，即所谓的 RxJava基于事件流的链式调用,实现方式如下：
```Java
Observable.create(new ObservableOnSubscribe<String>() {
			public void subscribe(@NonNull ObservableEmitter<String> e)
					throws Exception {
				//生产事件，实现方式跟方式一一致
			}
		}).subscribe(new Observer<String>() {
			// 观察者接收事件前，默认最先调用复写 onSubscribe()
			public void onSubscribe(@NonNull Disposable d) {
				// onSubscribe 是2.x新添加的方法，在发射数据前被调用，相当于1.x的onStart方法
			}
			public void onComplete() {
				// 对Complete事件作出响应，如上面事件类型详细介绍里的，被观察者可以不发送该事件
			}
			public void onError(@NonNull Throwable arg0) {
				// onError事件作出响应，如上面事件类型详细介绍里的，被观察者也可以不发送该事件
			}
			public void onNext(@NonNull String arg0) {
				// 对onNext事件作出响应，被观察者可以发送多个该事件
			}
		});
```

# 3. 额外说明
*本篇文字最后对前面来点额外说明,就跟吃完正餐来点水果点心,但你可别认为点心就不重要哦~*
## 3.1 Observable的其他创建方式
- just()方式
```Java
Observable<String> observable = Observable.just("Hello world!");
```
使用just()，将创建一个Observable并自动为调用onNext()来产生事件。需要注意的是通过just()方式会直接触发onNext()，传递的参数将直接在Observer的onNext()方法中获取到。
- fromIterable()方式
```Java
List<String> list = new ArrayList<String>();
for (int i = 0; i < 10; i++) {
	list.add("list" + i);
}
Observable<String> observable = Observable
		.fromIterable((Iterable<String>) list);
```
使用fromIterable()方式创建采用遍历集合，发送每个item。相当于循环调用onNext()方法，同样传递的item将直接在Observer的onNext()方法中获取到。
*PS注意：fromIterable()方式需要实现Iterable接口的对象才能传递，而Collection接口是Iterable接口的子接口，所以所有Collection接口的实现类都可以作为Iterable对象直接传入fromIterable()方法。*
- interval()方式
创建一个按固定时间间隔发射整数序列的Observable。interval()方式创建，入参有四种方式，这边分别介绍一下，其他的大家根据API看了对应注释就知道什么意思怎么用了。
	- 两个参数
	```Java
	interval(long period, TimeUnit unit)
	//第一个参数：代表两个消息发送之间的间隔时间(轮训使用时也就是轮训间隔时间)
	//第二个参数：时间单位：(毫秒，秒，分钟)TimeUtil时间工具类
	```
	- 三个参数
	```Java
	interval(long initialDelay, long period, TimeUnit unit)
	//第一个参数：代表初始话延迟的时间，就是延时多久开始发送消息
    //第二个参数：代表两个消息发送之间的间隔时间(轮训使用时也就是轮训间隔时间)
    //第三参数：时间单位：(毫秒，秒，分钟) TimeUtil时间工具类
	interval(long period, TimeUnit unit, Scheduler scheduler)
	//第一个参数：代表两个消息发送之间的间隔时间(轮训使用时也就是轮训间隔时间)
    //第二个参数：时间单位：(毫秒，秒，分钟) TimeUtil时间工具类
    //第三参数：Scheduler指定等待与产生事件的线程,关于调度线程控制将在后续文章详细介绍
	```
	- 四个参数
	```Java
 	interval(long initialDelay, long period, TimeUnit unit, Scheduler scheduler)
	这个也就是上面情况的综合体,就不一一介绍了
	```
- time()方式
说完interval()方式就不得不说time()方式了，time()方式创建的话也有两种方式：
	- 两个参数
	```Java
	timer(long delay, TimeUnit unit)
	//第一个参数：代表初始话延迟的时间，就是延时多久开始发送消息
	//第二个参数：时间单位：(毫秒，秒，分钟)TimeUtil时间工具类
	```
	- 三个参数
	```Java
	timer(long delay, TimeUnit unit, Scheduler scheduler)
	//第一个参数：代表初始话延迟的时间，就是延时多久开始发送消息
	//第二个参数：时间单位：(毫秒，秒，分钟)TimeUtil时间工具类
	//第三参数：Scheduler指定等待与产生事件的线程
	```
*PS:需要注意的是interval()方式创建的是一个<font color=#ff0000>无限循环</font>产生事件的Observable，而time()方式是发射一次数据*
- range( )方式
```Java
Observable.range(1, 5)
```
创建一个发射特定整数序列的Observable，第一个参数为起始值，第二个为发送的个数，如果为0则不发送，负数则抛异常。上述表示发射1到5的数。即调用5次nNext()方法，依次传入1~5数字。
**关于Observable的其他创建方式，当然还有很多其他的创建方式，我这边就不一一介绍了，有兴趣的小伙伴可以通过API去进一步了解。除了Observable的创建之外，RxJava 2.x 还提供了多个函数式接口 ，用于实现简便式的观察者模式。**

## 3.2 观察者 Observer的subscribe()具备多个重载的方法
```Java
public final Disposable subscribe() {}
// 表示观察者不对被观察者发送的事件作出任何响应（但被观察者还是可以继续发送事件）
public final Disposable subscribe(Consumer<? super T> onNext) {}
// 表示观察者只对被观察者发送的Next事件作出响应
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError) {} 
// 表示观察者只对被观察者发送的Next事件 & Error事件作出响应
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete) {}
// 表示观察者只对被观察者发送的Next事件、Error事件 & Complete事件作出响应
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete, Consumer<? super Disposable> onSubscribe) {}
// 表示观察者只对被观察者发送的Next事件、Error事件 、Complete事件 & onSubscribe事件作出响应
public final void subscribe(Observer<? super T> observer) {}
// 表示观察者对被观察者发送的任何事件都作出响应
```
## 3.3可采用 Disposable.dispose()切断观察者与被观察者之间的连接
- 即观察者无法继续接收被观察者的事件，但被观察者还是可以继续发送事件
- 具体使用
	- 1.定义`Disposable`类变量
```Java
	private Disposable mDisposable;
```
	- 2.对`Disposable`类变量赋值
```Java
//创建观察者时，对Disposable类变量赋值
public void onSubscribe(Disposable d) {
		mDisposable = d;
	}
```
	- 3.设置在`符合条件`的事件发生后切断观察者和被观察者的连接
```Java
 public void onNext(@NonNull Integer integer) {
        i++;
        if (i == 2) {
            // 在RxJava 2.x 中，新增的Disposable可以做到切断的操作，让Observer观察者不再接收上游事件
            mDisposable.dispose();
        }
    }
```
*以上仅仅是介绍RxJava2.X的基本概念与使用方式。通过本篇文章，可以对RxJava有个简单的了解。后面我会继续介绍RxJava中操作符以及线程调度等内容。*


