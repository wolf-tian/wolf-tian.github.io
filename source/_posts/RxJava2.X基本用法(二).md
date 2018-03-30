---
title: RxJava 2.X基本用法(二)
---
# 前言
上一节教程讲解了RxJava 2.X最基本的概念和用法，如何你没看过，可以移步[RxJava2.X基本用法(一)][1], 在本节中, 我们将学习RxJava强大的操作符。操作符就是用于在Observable和最终的Observer之间，通过操作符转换Observable为其他观察者对象修改或组合发出的事件，将转化后最终最简洁便捷的数据传递给Observer对象。这一篇我们开始聊聊RxJava中的操作符(Operators)，RxJava中的操作符主要分成了三类：
- 1.转换类操作符(`map` `flatMap` `concatMap` `flatMapIterable` `switchMap` `scan` `groupBy`...)
- 2.过滤类操作符(`fileter` `take` `takeLast` `takeUntil` `distinct` `distinctUntilChanged` `skip` `skipLast` ...)
- 3.组合类操作符(`mergeWith` `zip` `join` `switchMap` `startWith`...)

当然这一篇我也不可能介绍这么多的操作符，我介绍几个常用的，没有介绍的大家可以查看官方文档或者其他文章
# 正文
下面我们将按照分类介绍几个常用操作符，没上车的小伙伴赶紧上车![](https://i.imgur.com/P6D4itg.gif)
## 1.转换类操作符
对于转换类操作符，我这边主要介绍一下`map` 和 `flatMap` ,`concatMap` 和 `flatMapIterable`会稍微提一下，下面我们进入正题。
### 1.1 Map
map()是RxJava中最简单的一个变换操作符了,函数接受一个Function类型的参数,然后把这个Function应用到每一个由Observable发射的值上，将发射的值转换为我们期望的值。用图表示如下:
![](https://i.imgur.com/SrdR57b.jpg)
图中map函数作用就是将圆形事件转换为矩形事件, 从而导致下游接收到的事件就变为了矩形.假设我们需要将字符串转化为它的长度，我们可以通过map这样实现：
```Java
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("Hello");
        emitter.onNext("ObservableOnSubscribe");
    }
	}).map(new Function<String, Integer>() {
	    @Override
	    public Integer apply(@NonNull String s) throws Exception {
	        return s.length();
	    }
	}).subscribe(new Consumer<Integer>() {
	    @Override
	    public void accept(@NonNull Integer i) throws Exception {
	        System.out.println( "accept String length = " + i);
	    }
	});
```
在上游我们发送的是一个字符串, 而在下游我们接收的是Integerg类型, 中间起转换作用的就是map操作符, 运行结果为:
```Java
accept String length = 5
accept String length = 21
```
*在此说明一下后续为了减少页面篇幅，简单的运行结果我将不再贴出，有需要的可以拷贝代码自己运行，不过还是建议你自己手动敲一遍增加一下印象*
### 1.2 FlatMap
`FlatMap`函数同样也是做转换的，但是作用跟`Map`却不一样,相对于`Map` `FlatMap`是一个非常强大的操作符，`FlatMap`的转换是将一个发送事件的Observable变换为多个发送事件的Observables，然后将它们发射的事件合并后放进一个单独的Observable里发给事件的接受端也就是observer.同样的用图表示如下:
![](https://i.imgur.com/kgkF5C2.png)
图中flatMap函数作用就是将一个圆形事件转换为两个菱形事件, 从而导致下游接收到的事件就变为了多个菱形事件.假设我们取到了全国各省份的天气数据weathers,现在我们要输出每个市级的天气状况，你或许说我们可以这样实现:
```Java
Observable.fromIterable(weathers).subscribe(new Consumer<Weathers>() {
    public void accept(Weathers weathers) throws Exception {
        List<Weather> datas = weathers.getDatas();
        for (Weather weather : datas)
        {
            System.out.println("城市名：" + weather.getCityName());
            System.out.println("城市天气：" + weather.getCityWeather());
        }
    }
});
```
如果我不想在Subscriber中使用for循环，而是希望Subscriber中直接传入单个的天气对象呢（因为这对于代码复用很重要）？用map()显然是不行的，因为map()是一对一的转化，而我现在的要求是一对多的转化。那么我们可以使用flatMap()把一个weathers转化成多个weather。此时我们可以这样实现：
```Java
Observable.fromIterable(weathers).flatMap(new Function<Weathers, Observable<Weather>>() {
		@NonNull
		public Observable<Weather> apply(@NonNull Weathers arg0)
				throws Exception {
			return Observable.fromIterable(arg0.getDatas());
		}
	}).subscribe(new Consumer<Weather>() {
		public void accept(@NonNull Weather weather) throws Exception {
			System.out.println("城市名：" + weather.getCityName());
	        System.out.println("城市天气：" + weather.getCityWeather());
		}
	});
```
*看完上面的代码,你或许会火上心头，你会说对于上述的例子，第二种方式代码量多了哇~你要这样想，我只能说图样图森破，方式二虽然代码量多了些，但他剔除了嵌套结构，使得整体的逻辑性更强。譬如要是我们在各地域里面有一个面积这个量，这里我们需要过滤一下面积大于某一值的，你是不是还要再for循环里加上if？可是方式二却不需要，只需要加上过滤操作符就可以实现，运用flatMap代码看起来非常优雅而且逻辑清晰，可以充分体现RxJava链式编程的优点。*
flatMap的特点：经过flatMap操作变换后，最后输出的序列有可能是交错的，城市数据可能不能按照省份汇总在一起，因为flatMap最后合并结果采用的是merge操作符。如果要想经过变换后，最终输出的序列和原序列一致，那就会用到另外一个操作符，`concatMap`。
### 1.3 转换类操作符最后说明
对于concatMap这边不详细讲解，因为他的用法跟flatMap是一样的，只是有一点，`concatMap` 最终输出的数据序列和原数据序列是一致，它是按顺序链接Observables，而不是合并(flatMap用的是合并)。
`flatMapIterable()`和 `flatMap()`也几乎是一样的，不同的是flatMapIterable()它转化的多个Observable是使用Iterable作为源数据的。
```Java
Observable.fromIterable(weathers).flatMapIterable(new Function<Weathers, Iterable<Weather>>() {
	    public Iterable<Weather> apply(Weathers weathers) throws Exception {
	        return weathers.getDatas();
	    }
	}).subscribe(new Consumer<Weather>() {
	    public void accept(Weather weather) throws Exception {
	    	System.out.println("城市名：" + weather.getCityName());
	        System.out.println("城市天气：" + weather.getCityWeather());
	    }
	});
```
## 2.过滤类操作符
对于过滤类操作符，也采用上面的方式这边主要介绍一下`filter` 和 `take` ,`takeLast` 和 `takeUntil`会稍微提一下，下面我们进入正题。
### 2.1 filter
filter(predicate)用来过滤观测序列中我们不想要的值，只返回满足条件的值，类似于我们常用的if判断条件我们看下原理图：
![](https://i.imgur.com/KaPj6L4.png)
图中filter函数作用就是将一个发射多种图形事件过滤转化后下游只接受圆形事件的简单事件, 从而导致下游接收到的事件就变为了只有圆形事件事件.接着上面说的我们需要过滤一下面积大于某一值的地域，了解他的天气情况，这个时候你只需要在flatMap操作符之后加上过滤条件就可以，现在我们可以这样实现：
```Java
Observable.fromIterable(weathers).flatMap(new Function<Weathers, Observable<Weather>>() {
		@NonNull
		public Observable<Weather> apply(@NonNull Weathers arg0)
				throws Exception {
			return Observable.fromIterable(arg0.getDatas());
		}
	}).filter(new Predicate<Weather>() {
            @Override
            public boolean test(Weather weather) throws Exception {
                return weather.getAcreage() >= 8;
            }
        }).subscribe(new Consumer<Weather>() {
		public void accept(@NonNull Weather weather) throws Exception {
			System.out.println("城市名：" + weather.getCityName());
			System.out.println("城市面积：" + weather.getAcreage());
	        System.out.println("城市天气：" + weather.getCityWeather());
		}
	});
```
### 2.2 take
其实take操作符是一个系列性的操作符，它包含`take` `takeLast` `takeUntil` `takeWhile`...而且每一个操作符还有不同入参的方法，所以这边只是简单的聊一下take操作符,虽然简单也够日常开发的使用了，具体详细的有需要了解的可以去查看官方API或者自己去一个方法详细查看使用。
take操作符有三个如参方法：
我们先聊一下一个参数的：
```Java
take(long count) //这个唯一的如参表示需要发射的最大个数。即从原始的序列中发射前n个元素。
//与之对应的就是
takeLast(int count)//他与take相反它发射的是观测序列中后n个元素。
```
当我们去了解两个三个参数的时候我们发现后面的是采用对应如参的time方式创建Observable作为takeUntil的入参去过滤数据滴~，对应的我们可以看一下源码如下：
```Java
//发送给定时间内的数据
public final Observable<T> take(long time, TimeUnit unit) {
    return takeUntil(timer(time, unit));
}
//发送给定时间内的数据,并指定线程
public final Observable<T> take(long time, TimeUnit unit, Scheduler scheduler) {
    return takeUntil(timer(time, unit, scheduler));
}
```
**相对应的takeLast也有相应如参的方法,但takeLast有一个比take多的入参组合<font color=#ff0000> boolean delayError </font>，这边对这个入参进行解释一下后就不再对takeLast进行详细介绍了**
- boolean delayError为true时，表示当当前Observable发送异常时该异常会被被延迟，直到下游元素被处理消耗完

- boolean delayError为false时，表示当当前Observable的异常时该异常会被立即发送出去，下游未处理的的元素将被丢弃掉。

### 2.2 takeUntil
`takeUntil()`有两种类型个入参方法：
- `takeUntil(ObservableSource<U> other)` 订阅并开始发射原始Observable，同时监视我们提供的第二个Observable。如果第二个Observable发射了一项数据或者发射了一个终止通知，takeUntil()返回的Observable会停止发射原始Observable并终止。我们来看一下原理图：
![](https://i.imgur.com/o7TcfpL.png)
- `takeUntil(Predicate<? super T> stopPredicate)`通过stopPredicate方法来判断是否需要终止发射数据。我们来看一下原理图：
![](https://i.imgur.com/IQfc970.png)
通过上面两张原理图，我相信都不用多解释，小伙伴们都知道他们的用途了哇~
## 3.组合类操作符
对于组合类操作符，我们这边主要讲解一下用的最多的zip操作符，`zip`通过一个函数将多个Observable发送的事件结合到一起，然后发送这些组合到一起的事件. 首先我们来看一下原理图：
![](https://i.imgur.com/UHbxiol.png)
PS：需要说明的是zip它严格按照顺序应用这个函数。它只发射与发射数据项最少的那个Observable一样多的数据。应用举例：譬如我们一个页面需要从两个接口，而只有当两个都正确获取到了数据之后才能进行展示, 这个时候就可以用Zip了，代码示例如下：
```Java
//首先我们取到了两个用户信息分别为：
Observable<Info1> info1observable;
Observable<Info2> info2observable;
然后通过zip操作符我们采用合并方式处理数据
Observable.zip(observable1, observable2,
        new BiFunction<info1observable, info2observable, Info>() {
            @Override
            public Info apply(Info1 info1,
                                  Info2 info2) throws Exception {
				//数据合并方式
                return new Info(info1, info2);
            }
        }).observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Consumer<Info>() {
            @Override
            public void accept(Info info) throws Exception {
			//拿到合并后的数据进行操作;
            }
        });
```
# 放在最后
操作符这一章节终于要进行到最后了，我尽量采用通俗易懂的方式给大家讲解，不过也不知道大家接受的怎么样。不过这边依然需要说明的是我们真正讲解的操作符相对于Rxjava来说只是九牛一毛，更多的操作符大家可以查看官方文档，同时也可以一边使用一边详细了解。



[1]:https://wolf-tian.github.io/2018/03/13/RxJava2.X%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95(%E4%B8%80)/ "RxJava2.X基本用法(一)"