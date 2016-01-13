# Reactive Progrmming导论

This document describes guidelines that aid in developing applications and libraries that use the Reactive Extensions for RxJS library.

The guidelines listed in this document have evolved over time by the Rx team during the development of the RxJS library.

As RxJS continues to evolve, these guidelines will continue to evolve with it. Make sure you have the latest version of this document.

All information described in this document is merely a set of guidelines to aid development. These guidelines do not constitute an absolute truth. They are patterns that the team found helpful; not rules that should be followed blindly. There are situations where certain guidelines do not apply. The team has tried to list known situations where this is the case. It is up to each individual developer to decide if a certain guideline makes sense in each specific situation.

The guidelines in this document are listed in no particular order. There is neither total nor partial ordering in these guidelines.

Please contact us through the [RxJS Issues](https://github.com/Reactive-Extensions/RxJS) for feedback on the guidelines, as well as questions on whether certain guidelines are applicable in specific situations.

## The introduction to Reactive Programming you've been missing
(by [@andrestaltz](https://twitter.com/andrestaltz))

So you're curious in learning this new thing called Reactive Programming, particularly its variant comprising of Rx, Bacon.js, RAC, and others.

Learning it is hard, even harder by the lack of good material. When I started, I tried looking for tutorials. I found only a handful of practical guides, but they just scratched the surface and never tackled the challenge of building the whole architecture around it. Library documentations often don't help when you're trying to understand some function. I mean, honestly, look at this:

> **Rx.Observable.prototype.flatMapLatest(selector, [thisArg])**

> Projects each element of an observable sequence into a new sequence of observable sequences by incorporating the element's index and then transforms an observable sequence of observable sequences into an observable sequence producing values only from the most recent observable sequence.

Holy cow.

I've read two books, one just painted the big picture, while the other dived into how to use the Reactive library. I ended up learning Reactive Programming the hard way: figuring it out while building with it. At my work in [Futurice](https://www.futurice.com) I got to use it in a real project, and had the [support of some colleagues](http://blog.futurice.com/top-7-tips-for-rxjava-on-android) when I ran into troubles.

The hardest part of the learning journey is **thinking in Reactive**. It's a lot about letting go of old imperative and stateful habits of typical programming, and forcing your brain to work in a different paradigm. I haven't found any guide on the internet in this aspect, and I think the world deserves a practical tutorial on how to think in Reactive, so that you can get started. Library documentation can light your way after that. I hope this helps you.

## "Reactive Programming是神马？"

互联网上有很多不是很友好的解释。[维基百科](https://en.wikipedia.org/wiki/Reactive_programming) 宽泛而玄乎。 [Stackoverflow](http://stackoverflow.com/questions/1028250/what-is-functional-reactive-programming)教科书式的解释非常不适合信任[Reactive Manifesto](http://www.reactivemanifesto.org/) 听起来像是给给项目经理或者是销售的汇报。 微软的 [Rx 定义](https://rx.codeplex.com/) "Rx = Observables + LINQ + Schedulers" 太重并且太微软化了，让人看起来不知所云。“响应”、“变化发生”这些术语无法很好地阐释Reactive Programming的显著特点，听起来和你熟悉的MV*、编程语言差别不大。 当然，我的视角也是基于模型和变换的，要是脱离了这些概念，一切都是无稽之谈了。

那么我要开始吧啦吧啦了，(后文中，将使用RP代替Reactive Programming，私底下译者将Reactive Programming，翻译为响应式编程)。

#### RP 是针对异步数据流的编程。

一定程度而言，RP并不算新的概念。Event Bus、点击事件都是异步流。开发者可以观测这些异步流，并调用特定的逻辑对它们进行处理。使用Reactive如同开挂：你可以创建点击、悬停之类的任意流。通常流廉价(点击一下就出来一个)而无处不在，种类丰富多样：变量，用户输入，属性，缓存，数据结构等等都可以产生流。举例来说：微博回文(译者注：比如你关注的微博更新了)和点击事件都是流：你可以监听流并调用特定的逻辑对它们进行处理。

**基于流的概念，RP赋予了你一系列神奇的函数工具集，使用他们可以合并、创建、过滤这些流。** 一个流或者一系列流可以作为另一个流的输入。你可以 _合并_
两个流，从一堆流中 _过滤_ 你真正感兴趣的那一些，将值从一个流 _映射_ 到另一个流。

如果流是RP的核心，我们不妨从“点击页面中的按钮”这个熟悉的场景详细地了解它。

![Click event stream](http://i.imgur.com/cL4MOsS.png)

流是包含了**有时序，正在进行事件**的序列，可以发射(emmit)值(某种类型)、错误、完成信号。流在包含按钮的浏览器窗口被关闭时发出完成信号。

我们**异步地**捕获发射的事件，定义一系列函数在值被发射后，在错误被发射后，在完成信号被发射后执行。有时，我们忽略对错误，完成信号地处理，仅仅关注对值的处理。对流进行监听，通常称为**订阅**，处理流的函数是观测者，流是被观测的主体。这就是[观测者设计模式](https://en.wikipedia.org/wiki/Observer_pattern)。

教程中，我们有时会使用ASCII字符来绘制图表：

```
--a---b-c---d---X---|->

a, b, c, d 是数据流发射的值
X 是数据流发射的错误
| 是完成信号
---> 是时序轴
```

哔哔完了，我们来点新的，不然很快你就感觉到寂寞了。我们将把原来的点击事件流转换为新的点击事件流。

首先我们创建一个计数流来表明按钮被点击的次数。在RP中，每一个流都拥有一系列方法，例如`map`，`filter`，`scan` 等等。当你在流上调用这些方法，例如`clickStream.map(f)`，会返回基于点击事件流的**新的流**，同时原来的点击事件流并不会被改变，这个特性被称为**不可变性(immutability)**。不可变性与RP配合相得益彰，如同美酒加咖啡。我们可以链式地调用他们：`clickStream.map(f).scan(g)`

```
  clickStream: ---c----c--c----c------c--->
               vvvvv map(c becomes 1) vvvvv
               ---1----1--1----1------1--->
               vvvvvvvvv  scan(+) vvvvvvvvv
counterStream: ---1----2--3----4------5--->
```

`map(f)` 函数对原来的流使用我们出入的`f`函数进行转换，并生成新的流。在上面的例子中，我们将每一次点击映射为数字1。`scan(g)`函数将所有流产生的值进行汇总，通过传入`x = g(accumulated, current)`函数产生新的值，`g` 是简单的求和函数。最后 `counterStream`在点击发生后发射点击事件发生的总数。

为了展示Reactive的真正力量，我们举个例子：你想要“两次点击”事件的流，或者是“三次点击”，或者是n次点击的流。深呼吸一下，试着想想怎么用传统的命令、状态式方法来解决。我打赌这个这会相当操蛋，你会搞些变量来记录状态，还要搞些处理时延的机制。

如果用RP来解决，太他妈简单了。实际上[4行代码就可以搞定](http://jsfiddle.net/staltz/4gGgs/27/)。先不要看代码，不管你是菜鸟还是牛逼，使用图表来思考可以使你更好地理解构建这些流的方法。

![Multiple clicks stream](http://i.imgur.com/HMGWNO5.png)

灰色框里面的函数会把一个流转换成另外一个流。首先我们把点击打包到list中，如果点击后消停了250毫秒，我们就重新打包一个新的list(显然`buffer(stream.throttle(250ms))`就是用来干这个的，不明白细节没有关系，反正是demo嘛)。我们在列表上调用`map()`，将列表的长度映射为一个整数的流。最后，我们通过`filter(x >= 2)`过滤掉整数`1`。哈哈：3个操作就生成了我们需要的流，现在我们可以订阅(监听)这个流，然后来完成我们需要的逻辑了。

通过这个例子，我希望你能感受到使用RP的牛逼之处了。这仅仅是冰山一角。你可以在不同地流上(比如API响应的流)进行同样的操作。同时，Reactive还提供了许多其他实用的函数。

## "我要在今后采用RP范式进行编程吗？"

RP 提高了编码的抽象程度，你可以更好地关注在商业逻辑中各种事件的联系避免大量细节而琐碎的实现，使得编码更加简洁。

使用RP，将使得数据、交互错综复杂的web、移动app开发收益更多。10年以前，与网页的交互仅仅是提交表单、然后根据服务器简单地渲染返回结果这些事情。App进化得越来越有实时性：修改表单中一个域可以同步地更新到后端服务器。“点赞”信息实时地在不同用户设备上同步。

现代App中大量的实时事件创造了更好的交互和用户体验，披荆斩棘需要利剑在手，RP就是你手中的利剑。

## 通过实例RP编程思想

我们将从实例可以深入RP的编程思想，文章末尾，一个完整地实例应用会被构建，你也会理解整个过程。

我选择 **JavaScript** 和 **[RxJS](https://github.com/Reactive-Extensions/RxJS)** 作为实例的构建工具。因为大多开发者都熟悉JavaScript语言。[Rx* library family](http://www.reactivex.io) 在各种语言和平台都是实现 ([.NET](https://rx.codeplex.com/), [Java](https://github.com/Netflix/RxJava), [Scala](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-scala), [Clojure](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-clojure),  [JavaScript](https://github.com/Reactive-Extensions/RxJS), [Ruby](https://github.com/Reactive-Extensions/Rx.rb), [Python](https://github.com/Reactive-Extensions/RxPy), [C++](https://github.com/Reactive-Extensions/RxCpp), [Objective-C/Cocoa](https://github.com/ReactiveCocoa/ReactiveCocoa), [Groovy](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-groovy), 等等)。无论你选择在哪个平台或者那种语言实践RP，你都将从本教程中受益。(译者注：Rx，即ReactiveX，其中X代表不同的语言和技术栈，比如.NET，Java，Scala，Ruby，Javascript。RxJS表示RP基于Javascript语言的实现。后文中Rx代表所有实现了RP的特定技术栈)

## 微博(Twitter)简易版“你可能感兴趣的人”

微博主页，有一个组件会推荐给你那些你可能感兴趣的人。

![Twitter Who to follow suggestions box](http://i.imgur.com/eAlNb0j.png)

我们的Demo将使用这个场景，关注下面这些主要特性：

* 页面打开后，通过API加载数据展示3个你可能感兴趣的用户账号
* 点击“刷新”按钮，重新加载三个新的用户账号
* 在一个用户账号上点击'x' 按钮，清除当前这个账户，重新加载一个新的账户
* 每行展示账户的信息和这个账户主页的链接

其他特性和按钮我们暂且忽略，由于Twitter在最近关闭了公共API授权接口，我们选择Github作为代替，展示GitHub用户的账户。实例中我们使用该接口[获取GitHub用户](https://developer.github.com/v3/users/#get-all-users).

如果你希望先睹为快，完成后的代码已经发布在了[Jsfiddle](http://jsfiddle.net/staltz/8jFJH/48/)。

## "你可能感兴趣的用户"请求&响应

**这个问题使用Rx怎么解?**，呵呵，我们从Rx的箴言开始： _神马都是流_ 。首先我们做最简单的部分——页面打开后通过API加载3个账户的信息。分三步走：(1)发一个请求(2)获得响应(3)依据响应渲染页面。那么，我们先使用流来表示请求。我靠，表示个请求用得着吗？不过千里之行始于足下。

页面加载时，仅需要一个请求。所以这个数据流只包含一个简单的反射值。稍后，我们再研究如何多个请求出现的情况，现在先从一个请求开始。

```
--a------|->

a是字符串 'https://api.github.com/users'
```

这个流中包含了我们希望请求的URL地址。一旦这个请求事件发生，我们可以获知两件事情：请求流发射值(字符串URL)的时间就是请求需要被执行的时间，请求需要请求的地址就是请求流发射的值。

在Rx*中构建一个单值的流很容易。官方术语中把流称为“观察的对象”("Observable")，因为流可以被观察、订阅，这么称呼显得很蠢，我自己把他们称为 _stream_ 。


```javascript
var requestStream = Rx.Observable.just('https://api.github.com/users');
```

目前这个携带字符串的流没有其他操作,我们需要在这个流发射值之后，做点什么：通过[订阅](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypesubscribeobserver--onnext-onerror-oncompleted) 这个流来实现。

```javascript
requestStream.subscribe(function(requestUrl) {
  // 执行异步请求
  jQuery.getJSON(requestUrl, function(responseData) {
    // ...
  });
}
```

我们采用了jQuery的Ajax回调 (假设读着已经了解jQuery [ajax回调](http://devdocs.io/jquery/jquery.getjson)) 来处理异步请求操作。 且慢，Rx天生就是处理**异步** 数据流的，
为何不把请求的响应作为一个携带数据的流呢？ 么么哒，概念上没有问题，我们就来操作一下。

```javascript
requestStream.subscribe(function(requestUrl) {
  // 执行异步请求
  var responseStream = Rx.Observable.create(function (observer) {
    jQuery.getJSON(requestUrl)
    .done(function(response) { observer.onNext(response); })
    .fail(function(jqXHR, status, error) { observer.onError(error); })
    .always(function() { observer.onCompleted(); });
  });
  
  responseStream.subscribe(function(response) {
    // 业务逻辑
  });
}
```
 
使用[`Rx.Observable.create()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservablecreatesubscribe)方法可以自定义你需要的流。你需要明确通知观察者(或者订阅者)数据流的到达(`onNext()`) 或者错误的发生(`onError()`)。这个实现中，我们封装了jQuery 的异步 Promise。**那么Promise也是可观察对象吗？**

![funny face](http://7xq0ve.com1.z0.glb.clouddn.com/9e770b46f21fbe0926e5e61168600c338644adbc.jpg)


冰狗，你猜对啦！

可观察对象(Observable)是超级Promise(原文Promise++，可以对比C，C++，C++在兼容C的同时引入了面向对象等特性)。 在Rx环境中，你可以简单的通过`var stream = Rx.Observable.fromPromise(promise)`将Promise转换为可观察对象， 我们后面将这样使用， 唯一的区别是，可观察对象与[Promises/A+](http://promises-aplus.github.io/promises-spec/) 并不兼容, 但是理论上不会产生冲突。 Promise 可以看做只能发射单值的可观察对象，Rx流则允许返回多个值。

不过，可观察对象至少和Promise一样强大。如果你相信针对Promise的那些吹捧，不妨也留意一下Rx环境中的可观察对象。

回到我们的例子，细心的你肯定看到了`subscribe()`的嵌套使用，这和回调函数嵌套一样令人恼火。`responseStream` 的确和 `requestStream` 存在依赖关系。前面我们不是提到过Rx有一些牛逼的工具集吗？在Rx中我们拥有简单的机制把一个流转化为一个新的流，我们不妨试试。

我们先介绍 [`map(f)`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemapselector-thisarg)函数。该函数在流A的每个之上调用函数`f()` ， 然后在流B上生成对应的新值。如果在请求、响应流上调用`map(f)`，我们可以将请求的URL隐射为响应流中的Promise(此时响应流中包含了Promise的序列)。

```javascript
var responseMetastream = requestStream
  .map(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });
```

我们把上面代码执行后的返回结果称为 _metastream_ (译者注：按字面可以翻译为“元流”，即包含流的流。类似概念例如：元编程——用于生成程序的编程方法；元知识——获取知识的知识)：包含其他流的流。没什么吓人的， 一个metastream会在执行后发射一个流。 你可以把它看做一个指针 [指针](https://en.wikipedia.org/wiki/Pointer_(computer_programming))： 每一个发射的值是指向另外一个流的 _指针_ 。在我们的例子中，每一个URL被映射为一个指向Promise流的指针，每一个Promise流中包含了相应的响应信息。

![Response metastream](http://i.imgur.com/HHnmlac.png)

(译者注：以下给出 _metastream_ 的方法的解析方法，方便与下面的方法进行对比)：

```javascript
responseMetastream.subscribe(function(streamedPromise) {
	// 首先展开metastream，获取内部的流
	streamedPromise.subscribe(function(responseJsonObject) {
		// 返回内部流发射的值
		return responseJsonObject;
	});
});
```

当前版本响应产生的metastream看起来有些让人疑惑，似乎用处不大。当前场景中，我们仅仅需要获得简单的响应流，流中发射的值为简单的JSON对象。使用[flatMap](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeflatmapselector-resultselector):这个函数可以将枝干的流的值发射到主干流之上。当然metastream的产生并不是bug，只是这个场景不适合而已，`map()`，`flatMap()`都是Rx处理异步请求工具中的一部分。(译者注：如果流A中包含了若干其他流，在流A上调用`flatMap()`函数，将会发射其他流的值，并将发射的所有值组合生成新的流。)

```javascript
var responseStream = requestStream
  .flatMap(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });
```

![Response stream](http://i.imgur.com/Hi3zNzJ.png)

赞！响应流是依照请求流定义的，**如果** 场景中生成了更多的请求流，我们也会生成同样多的响应流：

```
请求流:  --a-----b--c------------|->
响应流:  -----A--------B-----C---|->

(小写字母表示请求, 大写字母代表响应)
```

获得响应流之后，我们就可以再订阅后渲染页面了:

```javascript
responseStream.subscribe(function(response) {
  // 在浏览器中渲染响应数据的逻辑
});
```

马克一下目前的代码:

```javascript
var requestStream = Rx.Observable.just('https://api.github.com/users');

var responseStream = requestStream
  .flatMap(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });

responseStream.subscribe(function(response) {
  // 在浏览器中渲染响应数据的逻辑
});
```

## 刷新“你可能感兴趣的用户”

忘了说了，我们每一次请求都会返回100个GitHub用户的数据。GitHub的API只允许我们设置页面的偏移量但是不能设置每次获得数据的数量。嗯，我们需要3个推荐用户的数据，其他97个就这样浪费了。暂时忽略这个问题，后面我们看看怎么缓存数据来减少数据的浪费。

每一次点击刷新按钮(高能注意：是一个按钮，点击后刷新“我可能感兴趣的人”的数据，而不是浏览器的刷新按钮)，请求流都会发射新的URL值，我们以此获得新的响应。刷新分为两步：产生一个刷新按钮被点击的事件流(RP箴言：神马都是流)；订阅刷新事件流后改变请求流的URL地址。RxJS提供了工具方便我们将时间监听器转换为可观察对象。

```javascript
var refreshButton = document.querySelector('.refresh');
var refreshClickStream = Rx.Observable.fromEvent(refreshButton, 'click');
```

因为点击刷新事件并不会携带需要请求的API的URL，我们需要把每一次点击映射到真正的URL之上。具体实现方式是，在刷新点击流发生后，我们通过产生随机的页面拼凑出URL，并向GitHub发起请求。

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

由于是简单的教程，我并没有写相关的测试，但是我仍然知道原先的功能被我搞砸啦。呃。。。页面打开后居然没有请求流了，除非我点击刷新按钮，否则数据怎么都出不来。擦。。。我希望 _不管_ 是点击刷新按钮"_还是_"第一次打开页面，都可以产生获得“我可能感兴趣的人”的数据的GitHub的请求流。

把两个流分开写特别简单，我们已经知道怎么做了：

```javascript
var requestOnRefreshStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
  
var startupRequestStream = Rx.Observable.just('https://api.github.com/users');
```

但是我们怎么把两个流“合并”在一块呢？使用 [`merge()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemergemaxconcurrent--other)函数吧。我们用ASCII图表来解释这个函数的作用：


```
流 A: ---a--------e-----o----->
流 B: -----B---C-----D-------->
          vvvvvvvvv merge vvvvvvvvv
          ---a-B---C--e--D--o----->
```

使用`merge()`后简单多了:

```javascript
var requestOnRefreshStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
  
var startupRequestStream = Rx.Observable.just('https://api.github.com/users');

var requestStream = Rx.Observable.merge(
  requestOnRefreshStream, startupRequestStream
);
```

如果不需要requestOnRefreshStream、startupRequestStream这两个中间流，写法更干净、简洁。

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  })
  .merge(Rx.Observable.just('https://api.github.com/users'));
```

还能更简单，更有可读性:

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  })
  .startWith('https://api.github.com/users');
```

[`startWith()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypestartwithscheduler-args) 函数的作用和它的命名一样。 无论是什么样的流，`startWith(x)` 都会把x作为这个流的启示输入并发射出来。 上面的实现，还不够[DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself)(Don't repeat yourself，不要重复!)，API请求的URL地址重复了两遍。我们将 `startWith()` 紧接在`refreshClickStream`之后，在页面打开后就模拟一次点击。  

```javascript
var requestStream = refreshClickStream.startWith('startup click')
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

Nice！事情不会被搞砸了，`startWith()`完美解决了问题。

## 3位“你可能感兴趣的用户”的流的构建

目前为止，仅仅在订阅(`subscribe()`)时，你会触及到“感兴趣的用户”区块的渲染。但是通过刷新按钮，问题接踵而至：你点击了刷新按钮，在新的响应到达之前，原来的“你可能感兴趣的”3个用户并不会马上消失。为了增强用户体验，我们希望在用户点击了刷新按钮后就清楚老数据。

```javascript
refreshClickStream.subscribe(function() {
  // 清楚旧数据： 3个你可能感兴趣的用户的DOM元素
});
```

停！不要用力过猛。**两个** 订阅行为都会影响到这个区块的渲染。(`responseStream.subscribe()`、`refreshClickStream.subscribe()`)，并且上面的设计也不符合[关注分离](https://en.wikipedia.org/wiki/Separation_of_concerns)的理念。还记得RP _神马都是流_ 的箴言吗？

![Mantra](http://i.imgur.com/AIimQ8C.jpg)

那么开始构建这个专门的推荐流：流会发射“你可能感兴趣的用户”的JSON对象。我们会分别构建三种这样的流，第一种长这个样：

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // 随机从列表中取出一个用户
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  });
```

另外两个流`suggestion2Stream` 和 `suggestion3Stream`复制粘贴就好啦。呃。。。DRY不要重复，我把这个问题作为这个教程的联系，自己做一遍你会去思考这类场景中如何避免代码的重复。

译者注：如果使用UnderScore，一种方法是，新的方法总是会返回JSON Object数组:

```javascript
var suggestionStream = responseStream
  .map(suggestionN(listUsers, n));

function suggestionN(listUsers, n) {
	_.times(n, function() {
		return listUsers[Math.floor(Math.random()*listUsers.length)];
	})
}
```

我们不再订阅响应流，而是变更为：

```javascript
suggestion1Stream.subscribe(function(suggestion) {
  // 在区块中渲染1位用户的DOM元素
});
```

回到原始需求：“每一次刷新后，清除原来的用户”，我们可以在刷新后，返回null作为推荐流：

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // 随机从列表中取出一个用户
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  })
  .merge(
    refreshClickStream.map(function(){ return null; })
  );
```

在渲染环节，`null`代表无数据，我们就隐藏之前的DOM元素。

```javascript
suggestion1Stream.subscribe(function(suggestion) {
  if (suggestion === null) {
    // 在区块中隐藏一个推荐用户的DOM元素
  }
  else {
    // 在区块中渲染一个推荐用户的DOM元素
  }
});
```

整个事件流如图所示:

```
  刷新按钮流: ----------o--------o---->
     请求流: -r--------r--------r---->
     响应流: ----R---------R------R-->   
 推荐1个用户: ----s-----N---s----N-s-->
```

 `N` 表示 `null`.

页面打开后，我们渲染“空”推荐区块，可以通过在推荐流中附加`startWith(null)`实现：

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // 随机从列表中取出一个用户
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  })
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

Which results in:

```
   刷新按钮流: ----------o---------o---->
      请求流: -r--------r---------r---->
      响应流: ----R----------R------R-->   
 推荐1个用户:  -N--s-----N----s----N-s-->
```

## 关闭一个推荐元素，从缓存获得新的推荐元素

最后一个需要实现的功能是：点击'x'按钮后关闭当前的推荐元素，载入一个新的数据并渲染。拍脑袋意向，无论点击了啥按钮，我们重新请求一次新数据，生成一个新的响应流就好了：

```javascript
var close1Button = document.querySelector('.close1');
var close1ClickStream = Rx.Observable.fromEvent(close1Button, 'click');
// close2Button 和 close3Button 作为练习

var requestStream = refreshClickStream.startWith('startup click')
  .merge(close1ClickStream) // 加上这个
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

擦，点击了关闭按钮整个推荐区块都被刷新了！看来我们只有使用原来的相应流才能解决这个bug，况且每次慷慨大方的GitHub给我们100个用户的数据，我们只使用3个，还有1大堆留着等我们用呢，没有必要再请求更多的数据了。

让我们从流的角度思考，当点击'x'事件发生后，我们使用 _最近一次的相应流_ 并从中随机取出用户就好了：

```
      请求流: --r--------------->
      响应流: ------R----------->
   点击关闭流: ------------c----->
推荐1个用户流: ------s-----s----->
```

在Rx*框架中，一个使用函数叫 [`combineLatest`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypecombinelatestargs-resultselector) 。 函数将两个流作为输入，并且当其中任意一个流发射之后， `combineLatest` 都会组合两个流中最新的值 `a` 和 `b`然后输出一个新的流，流的值为 `c = f(x,y)` 其中 `f(x, y)`	是传入的自定义函数，配合上时序图更好理解:

```
流 A:     --a-----------e--------i-------->
流 B:     -----b----c--------d-------q---->
          vvvvvvvv combineLatest(f) vvvvvvv
          ----AB---AC--EC---ED--ID--IQ---->

这里的函数f，将输入的字符串变为大写
```

现在我们在 `close1ClickStream` 和 `responseStream`使用combineLatest() ， 只要用户点击关闭按钮，我们就结合最新的响应流来产生`suggestion1Stream`。 另一个方面，combineLatest() 是一个同步操作：每当新的响应流发射了值， 同样会结合 `close1ClickStream`产生新的推荐数据。这样我们大大简化了`suggestion1Stream`：

```javascript
var suggestion1Stream = close1ClickStream
  .combineLatest(responseStream,             
    function(click, listUsers) {
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

最后还有一点点问题：combineLatest()需要结合传入的两个流，如果其中一个流从未发射过任何值，combineLatest()将不会输入任何新的流。回顾一下上面的ASCII图表，当第一个流发射值`a`时，不会有任何输出，仅当第二个流也发射了值`b`后，combineLatest()才会开始向外输出。

解决方法很多，我们采取最简单的方式(上面例子也用到过)，我们在页面打开时限模拟一次关闭按钮的点击：

```javascript
var suggestion1Stream = close1ClickStream.startWith('startup click') // we added this
  .combineLatest(responseStream,             
    function(click, listUsers) {
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

## 总结

再Mark一下当前的代码，是不是很有成就感：

```javascript
var refreshButton = document.querySelector('.refresh');
var refreshClickStream = Rx.Observable.fromEvent(refreshButton, 'click');

var closeButton1 = document.querySelector('.close1');
var close1ClickStream = Rx.Observable.fromEvent(closeButton1, 'click');
// close2 和 close3 作为练习

var requestStream = refreshClickStream.startWith('startup click')
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });

var responseStream = requestStream
  .flatMap(function (requestUrl) {
    return Rx.Observable.fromPromise($.ajax({url: requestUrl}));
  });

var suggestion1Stream = close1ClickStream.startWith('startup click')
  .combineLatest(responseStream,             
    function(click, listUsers) {
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
// suggestion2Stream 和 suggestion3Stream 作为练习

suggestion1Stream.subscribe(function(suggestion) {
  if (suggestion === null) {
    // 隐藏一个用户的DOM元素
  }
  else {
    // 渲染一个新的推荐用户的DOM元素
  }
});
```

**You can see this working example at http://jsfiddle.net/staltz/8jFJH/48/**

That piece of code is small but dense: it features management of multiple events with proper separation of concerns, and even caching of responses. The functional style made the code look more declarative than imperative: we are not giving a sequence of instructions to execute, we are just **telling what something is** by defining relationships between streams. For instance, with Rx we told the computer that _`suggestion1Stream` **is** the 'close 1' stream combined with one user from the latest response, besides being `null` when a refresh happens or program startup happened_.

Notice also the impressive absence of control flow elements such as `if`, `for`, `while`, and the typical callback-based control flow that you expect from a JavaScript application. You can even get rid of the `if` and `else` in the `subscribe()` above by using `filter()` if you want (I'll leave the implementation details to you as an exercise). In Rx, we have stream functions such as `map`, `filter`, `scan`, `merge`, `combineLatest`, `startWith`, and many more to control the flow of an event-driven program. This toolset of functions gives you more power in less code.

## What comes next

If you think Rx* will be your preferred library for Reactive Programming, take a while to get acquainted with the [big list of functions](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md) for transforming, combining, and creating Observables. If you want to understand those functions in diagrams of streams, take a look at [RxJava's very useful documentation with marble diagrams](https://github.com/Netflix/RxJava/wiki/Creating-Observables). Whenever you get stuck trying to do something, draw those diagrams, think on them, look at the long list of functions, and think more. This workflow has been effective in my experience.

Once you start getting the hang of programming with Rx*, it is absolutely required to understand the concept of [Cold vs Hot Observables](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/gettingstarted/creating.md#cold-vs-hot-observables). If you ignore this, it will come back and bite you brutally. You have been warned. Sharpen your skills further by learning real functional programming, and getting acquainted with issues such as side effects that affect Rx*.

But Reactive Programming is not just Rx*. There is [Bacon.js](http://baconjs.github.io/) which is intuitive to work with, without the quirks you sometimes encounter in Rx*. The [Elm Language](http://elm-lang.org/) lives in its own category: it's a Functional Reactive Programming **language** that compiles to JavaScript + HTML + CSS, and features a [time travelling debugger](http://debug.elm-lang.org/). Pretty awesome.

Rx works great for event-heavy frontends and apps. But it is not just a client-side thing, it works great also in the backend and close to databases. In fact, [RxJava is a key component for enabling server-side concurrency in Netflix's API](http://techblog.netflix.com/2013/02/rxjava-netflix-api.html). Rx is not a framework restricted to one specific type of application or language. It really is a paradigm that you can use when programming any event-driven software.

If this tutorial helped you, [tweet it forward](https://twitter.com/intent/tweet?original_referer=https%3A%2F%2Fgist.github.com%2Fstaltz%2F868e7e9bc2a7b8c1f754%2F&amp;text=The%20introduction%20to%20Reactive%20Programming%20you%27ve%20been%20missing&amp;tw_p=tweetbutton&amp;url=https%3A%2F%2Fgist.github.com%2Fstaltz%2F868e7e9bc2a7b8c1f754&amp;via=andrestaltz).

