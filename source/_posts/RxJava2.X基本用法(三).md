---
title: RXJava 2.X基本用法(三)
---
# 前言
通过前面两节教程讲解我们算是对Rxjava有了初步的了解，如何你没看过，可以移步[RxJava2.X基本用法(二)][1], 在本节中, 我们将学习RxJava线程调度。这一篇我们开始聊聊RxJava中的线程调度。RxJava的目的就是异步，所以线程调度是RxJava的最基础的常用点。本篇文章主要介绍线程调度器`Scheduler`，通过对线程调度器的了解，方便我们更好的处理日常的异步操作需求，从而在合适的场景选择合适的线程。
# 正文
首先我们需要清楚的是`RxJava`遵循的是线程不变的原则，何为线程不变原则？即：在哪个线程调用 subscribe()，那么就将在哪个线程生产事件，如果不切换线程那就将在哪个线程中消费事件。如果需要切换线程，这时候我们就需要用到`Scheduler`(调度器)。
## Scheduler种类
在RxJava中我们是通过Scheduler来指定每一段代码应该运行在哪个线程中。RxJava已经内置了几个Scheduler同时RxAndroid也内置了其个性化的Scheduler。下面我们具体介绍一下Scheduler 的API：
- **Schedulers.immediate()**
直接在当前线程运行，相当于不指定线程，使用默认的Scheduler。
- **Schedulers.computation()**
计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。
- **Schedulers.newThread()**
总是启用新线程，并在新线程执行操作。
- **Schedulers.io()**
I/O操作(读写文件、读写数据库、网络信息交互等)所使用的Scheduler，行为模式和newThread()差不多，区别在于io()的内部实现是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下，io()比newThread()更有效率。不要把计算工作放在io(),可以避免穿件不必要的线程。

**RxJava中常用的Schedulers即为以上几种，下面介绍几个不常用的**
- Schedulers.trampoline()
当其它排队的任务完成后，在当前线程排队开始执行
- Schedulers.from(executor)
使用指定的Executor作为调度器。

**最后介绍一个Android内置的个性化Scheduler**
- **AndroidSchedulers.mainThread()**
在RxAndroid中，他指定操作将在Android主线程中执行。

## Scheduler用法
有了这几个`Scheduler`，我们就可以使用 subscribeOn() 和 observeOn() 两个方法来进行线程切换和控制了(*浪滴个浪~*)。我们只需要在之前的代码中增加了以下的两行代码即可实现线程间的切换(*是不是so easy*)：
```Java
.subscribeOn(Schedulers.newThread())
.observeOn(AndroidSchedulers.mainThread())
```
大家需要记住几个要点, 就可以知道如何正确的去使用了：
- **subscribeOn()**: 指定Observable(被观察者)所在的线程，或者叫做事件产生的线程。 
- **observeOn()**: 指定 Observer(观察者)所运行在的线程，或者叫做事件消费的线程。

简单的来说, subscribeOn() 指定的是上游发送事件的线程, observeOn() 指定的是下游接收事件的线程.同时需要注意的是<font color="#ff0000">**多次指定上游的线程只有第一次指定的有效**</font>, 也就是说多次调用subscribeOn() 只有第一次的有效, 其余的会被忽略。
<font color="#ff0000">**多次指定下游的线程是可以的**</font>, 也就是说每调用一次observeOn() , 下游的线程就会切换一次。

# 实践
毛爷爷曾经说过“实践是检验真理的唯一标准”，来来来，我们检验一下^_^ 对于我们Android攻城狮来说, 经常需要将一些耗时的操作放在后台进程中去执行, 譬如网络请求，读写文件或者操作数据库等等,等到这些操作完成之后回到主线程去更新UI, 有了Scheduler上面的这些知识点, 们就可以轻松的去实现这样的一些操作。走咱们组队刷怪去咯~
在这里我们通过最常用的的网络请求(RxJava 2.x 网络请求使用)来实践检验一下（*我们这边采用的网络请求框架是Retrofit2.X有没用过或不知道的小伙伴自行百度哦*）
下面例中我们使用某品牌手机的天气API
接口url：http://******/app/weather/listWeather?cityIds=101190101其中城市 id。
具体请求的返回数据形式如下：
```Java
{
	"code": "200",
	"message": "",
	"redirect": "",
	"value": [{
		"alarms": [],
		"city": "****",
		"cityid": 101190101,
		"indexes": [{
			"abbreviation": "gm",
			"alias": "",
			"content": "感冒极易发生，避免去人群密集的场所，勤洗手勤通风有利于降低感冒几率。",
			"level": "极易发",
			"name": "感冒指数"
		}, {
			"abbreviation": "xc",
			"alias": "",
			"content": "洗车后，可保持2天车辆清洁，比较适宜洗车。",
			"level": "较适宜",
			"name": "洗车指数"
		}, {
			"abbreviation": "ct",
			"alias": "",
			"content": "春夏之交，天气渐暖，单衣单裤。",
			"level": "暖",
			"name": "穿衣指数"
		}, {
			"abbreviation": "uv",
			"alias": "",
			"content": "辐射较弱，涂擦SPF12-15、PA+护肤品。",
			"level": "弱",
			"name": "紫外线强度指数"
		}],
		....
	}]
}
```
网络请求具体实现的步骤如下：
- 定义Api接口:
```Java
public interface Api {
@GET("listWeather")
Observable<AllCity> getCityWeather(@Query("cityIds") String cityIds);
}
```
- 创建一个Retrofit客户端:
```Java
OkHttpClient.Builder builder = new OkHttpClient().newBuilder();
builder.readTimeout(8, TimeUnit.SECONDS);
builder.connectTimeout(5, TimeUnit.SECONDS);
Retrofit retrofit =  new Retrofit.Builder().baseUrl(baseUrl)
        .client(builder.build())
        //增加返回值为Oservable<T>的支持
        .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
        //增加返回值为String的支持
        .addConverterFactory(ScalarsConverterFactory.create())
        //增加返回值为Gson的支持(以实体类返回)
        .addConverterFactory(GsonConverterFactory.create())
        .build();
```
- 实现网络请求
```Java
Api api = retrofit.create(Api.class);
api.getCityWeather("101190101")
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Consumer<Weather>() {
            @Override
            public void accept(Weather weather) throws Exception {
                Log.d(TAG, "getContent = " + weather.getValue().get(0).getIndexes().get(0).getContent());
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(Throwable throwable) throws Exception {

            }
        });
```
以上整个实践过程完成了网络请求，同时进行异步操作，防止阻塞UI线程。不过这个仅仅是实例介绍RxJava的基础使用，RxJava的功能远不止于此。实际使用时我们不能仅仅这么简单操作，因为我们忽略了一点, 如果在请求的过程中Activity已经退出了, 这个时候如果回到主线程去更新UI, 那么APP肯定就崩溃了, 怎么办呢？下面我们讲述另一个重要的概念Disposable。它相当于是个开关。当Observer(观察者)与Observable(被观察者)通过subscribe()建立连接后，事件可以进行传递。当发生一些其他情况，不得不断开两者之间的连接时，该怎么操作?这个时候就该Disposable上场了，我们可以用它切断Observer(观察者)与Observable(被观察者)的连接。

# Disposable基本使用
Disposable用于连接管理，在RxJava中,用它来切断Observer(观察者)与Observable(被观察者)之间的连接，其dispose()方法可断开将Observer(观察者)与Observable(被观察者)之间的连接（取消订阅),从而导致Observer(观察者)收不到事件。Disposable的作用是切断连接，确切地讲是将Observer(观察者)切断，不再接收来自被观察者的事件，而<font color="#ff00000">**被观察者的事件却仍在继续执行**</font>。
Disposable的对象通过观察者获得，具体分为以下两种方式：
- Observer接口
```Java
private Disposable mDisposable;
Observer<Long> observer = new Observer<Long>() {
    // 观察者接收事件前，默认最先调用复写 onSubscribe()
    public void onSubscribe(@NonNull Disposable d) {
		mDisposable = d;
    }
    public void onComplete() {
    }
    public void onError(@NonNull Throwable arg0) {
    }
    public void onNext(@NonNull Long arg0) {
    }
};
mDisposable.dispose()  //在需要切断被观察者和观察者连接时通过该方法取消订阅
```
*这里我们通过创建Observer接口,当订阅后，建立与Observable的联系，我们在onSubscribe(Disposable d)中便可以获得Disposable对象。*
- Consumer等其他函数式接口
```Java
Disposable disposable = Observable.just("你好").subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
    }
});
```
*当subscribe()后直接返回一个`Disposable`对象,获得了`Disposable`对象后，我们便可以调用`dispose()`方法，在恰当的时机断开连接，停止接收Observable(被观察者)发送的事件。*

**PS：**到此RxJava的基本概念和用法也接近尾声了，小伙伴们掌握了上面三篇文章,也可以算是跨入RxJava的门槛了，可以进行RxJava的简单日常运用了。后续计划写一下RxJava的进阶篇将会介绍RxJava2.X中新增加的内容：Flowable及backpressure以及RxJava一些其他强大功能点，敬请期待。


[1]:https://wolf-tian.github.io/2018/03/15/RxJava2.X%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95(%E4%BA%8C)/ "RxJava2.X基本用法(二)"

