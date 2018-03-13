---
title: RXJava 2.X
---
1. 基本使用
 
1.1 方式1：分步骤实现
1.1.1 使用步骤
步骤1：创建被观察者 （Observable ）& 生产事件
步骤2：创建观察者 （Observer ）并 定义响应事件的行为
•	发生的事件类型包括：Next事件、Complete事件 & Error事件。具体如下：
事件类型	定义	作用	使用规则	使用方法
Next	普通事件	向观察者发送需要响应事件信号	被观察者可以发送无限个Next事件；观察者可以接受无限个Next事件	onNext()
Complete	事件队列完成事件	标志被观察者不再发送普通事件(Next)	1.当被观察者发送了一个Complete事件后：被观察者在Complete事件后的事件将会继续发送，但观察者收到Complete事件后将不再继续接受任何事件
2.被观察者可以不发送Complete事件	在一个正确运行的事件序列中：
	onCompleted()&onError()二者互斥(即只能有其一)
	onCompleted()&onError()是唯一的(只能有一个)
	onCompleted()
Error	事件队列异常事件	标志事件处理过程中出现异常(此时队列自动终止,不允许再有事件发出)	1.当被观察者发送了一个Error事件后：被观察者在Error事件后的事件将会继续发送，但观察者收到Error事件后将不再继续接受任何事件
2.被观察者可以不发送Error事件		onError()
•	具体实现
方式1：采用Observer 接口
1.	创建观察者(Observer)对象
Observer<String> observer = new Observer<String>() {}
2. 创建对象时通过对应复写对应事件方法从而响应对应事件
2.1观察者接收事件前，默认最先调用复写 onSubscribe()
2.2当被观察者生产Next事件 & 观察者接收到时，会调用onNext()方法进行响应
2.3当被观察者生产Error事件& 观察者接收到时，会调用onError()方法进行响应
2.4当被观察者生产Complete事件& 观察者接收到时，会调用onComplete()方法进行响应
方式2：采用Subscriber 抽象类
说明：Subscriber类 = RxJava 内置的一个实现了 Observer 的抽象类，对 Observer 接口进行了扩展
1.	创建观察者(Observer)对象
Subscriber<String> subscriber = new Subscriber<String>() {}
2. 创建对象时通过对应复写对应事件方法从而响应对应事件
2.1观察者接收事件前，默认最先调用复写 onSubscribe()
2.2当被观察者生产Next事件 & 观察者接收到时，会调用onNext()方法进行响应
2.3当被观察者生产Error事件& 观察者接收到时，会调用onError()方法进行响应
2.4当被观察者生产Complete事件& 观察者接收到时，会调用onComplete()方法进行响应
特别注意：2种方法的区别，即Subscriber 抽象类与Observer 接口的区别
相同点：二者基本使用方式完全一致（实质上，在RxJava的 subscribe 过程中，Observer总是会先被转换成Subscriber再使用）
不同点：Subscriber抽象类对 Observer 接口进行了扩展，新增了两个方法：
1.	onStart(): 这是 Subscriber 增加的方法。它会在subscribe刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。这是一个可选方法，默认情况下它的实现为空。需要注意的是，如果对准备工作的线程有要求（例如弹出一个显示进度的对话框，这必须在主线程执行）， onStart() 就不适用了，因为它总是在 subscribe 所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用 doOnSubscribe() 方法。
2.	unsubscribe(): 这是 Subscriber 所实现的另一个接口 Subscription 的方法，用于取消订阅。在这个方法被调用后，Subscriber 将不再接收事件。一般在这个方法调用前，可以使用 isUnsubscribed() 先判断一下状态。 unsubscribe() 这个方法很重要，因为在 subscribe() 之后， Observable 会持有 Subscriber 的引用，这个引用如果不能及时被释放，将有内存泄露的风险。所以最好保持一个原则：在不再使用的时候尽快在合适的地方（例如 onPause() onStop() 等方法中）调用 unsubscribe() 来解除引用关系，以避免内存泄露的发生。
步骤3：通过订阅（Subscribe）连接观察者和被观察者
•	具体实现
observable.subscribe(observer);
 // 或者 observable.subscribe(subscriber)；
•	扩展说明
<-- Observable.subscribe(Subscriber) 的内部实现 -->
public Subscription subscribe(Subscriber subscriber) {
    subscriber.onStart();
    // 步骤1中 观察者  subscriber抽象类复写的方法，用于初始化工作
    onSubscribe.call(subscriber);
    // 通过该调用，从而回调观察者中的对应方法从而响应被观察者生产的事件
    // 从而实现被观察者调用了观察者的回调方法 & 由被观察者向观察者的事件传递，即观察者模式
    // 同时也看出：Observable只是生产事件，真正的发送事件是在它被订阅的时候，即当 subscribe() 方法执行时
}
1.2 方式2：优雅的实现方法 - 基于事件流的链式调用
•	上述的实现方式是为了说明Rxjava的原理 & 使用
•	在实际应用中，会将上述步骤&代码连在一起，从而更加简洁、更加优雅，即所谓的 RxJava基于事件流的链式调用(整体方法调用顺序：观察者.onSubscribe（）> 被观察者.subscribe（）> 观察者.onNext（）>观察者.onComplete())
2. 额外说明
2.1 观察者 Observer的subscribe()具备多个重载的方法
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
2.2 可采用 Disposable.dispose() 切断观察者 与 被观察者 之间的连接
•	即观察者 无法继续 接收 被观察者的事件，但被观察者还是可以继续发送事件
•	具体使用
// 1. 定义Disposable类变量
private Disposable mDisposable;
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
                // 2. 对Disposable类变量赋值
                mDisposable = d;
            }
            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "对Next事件"+ value +"作出响应"  );
                if (value == 2) {
                    // 设置在接收到第二个事件后切断观察者和被观察者的连接
                    mDisposable.dispose();
                    Log.d(TAG, "已经切断了连接：" + mDisposable.isDisposed());
                }
            }
3. RxJava中的操作符
3.1 map()操作符
map是RxJava中最简单的一个变换操作符了, 就是把原来的Observable对象转换成另一个Observable对象，它的作用就是对上游发送的每一个事件应用一个函数, 使得每一个事件都按照指定的函数去变化. 通过Map, 可以将上游发来的事件转换为任意的类型, 可以是一个Object, 也可以是一个集合。
3.2 FlatMap ()操作符
FlatMap对于数据的转换比map()更加彻底，如果发送的数据是集合，flatmap()重新生成一个Observable对象，并把数据转换成Observer想要的数据形式。FlatMap将一个发送事件的上游Observable变换为多个发送事件的Observables，然后将它们发射的事件合并后放进一个单独的Observable里. 这里需要注意的是, flatMap并不保证事件的顺序,如果需要保证顺序则需要使用concatMap.
