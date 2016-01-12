# Introduction

This document describes guidelines that aid in developing applications and libraries that use the Reactive Extensions for RxJS library.

The guidelines listed in this document have evolved over time by the Rx team during the development of the RxJS library.

As RxJS continues to evolve, these guidelines will continue to evolve with it. Make sure you have the latest version of this document.

All information described in this document is merely a set of guidelines to aid development. These guidelines do not constitute an absolute truth. They are patterns that the team found helpful; not rules that should be followed blindly. There are situations where certain guidelines do not apply. The team has tried to list known situations where this is the case. It is up to each individual developer to decide if a certain guideline makes sense in each specific situation.

The guidelines in this document are listed in no particular order. There is neither total nor partial ordering in these guidelines.

Please contact us through the [RxJS Issues](https://github.com/Reactive-Extensions/RxJS) for feedback on the guidelines, as well as questions on whether certain guidelines are applicable in specific situations.

{% if book.isPdf %}



{% else %}

## The introduction to Reactive Programming you've been missing
(by [@andrestaltz](https://twitter.com/andrestaltz))

So you're curious in learning this new thing called Reactive Programming, particularly its variant comprising of Rx, Bacon.js, RAC, and others.

Learning it is hard, even harder by the lack of good material. When I started, I tried looking for tutorials. I found only a handful of practical guides, but they just scratched the surface and never tackled the challenge of building the whole architecture around it. Library documentations often don't help when you're trying to understand some function. I mean, honestly, look at this:

> **Rx.Observable.prototype.flatMapLatest(selector, [thisArg])**

> Projects each element of an observable sequence into a new sequence of observable sequences by incorporating the element's index and then transforms an observable sequence of observable sequences into an observable sequence producing values only from the most recent observable sequence.

Holy cow.

I've read two books, one just painted the big picture, while the other dived into how to use the Reactive library. I ended up learning Reactive Programming the hard way: figuring it out while building with it. At my work in [Futurice](https://www.futurice.com) I got to use it in a real project, and had the [support of some colleagues](http://blog.futurice.com/top-7-tips-for-rxjava-on-android) when I ran into troubles.

The hardest part of the learning journey is **thinking in Reactive**. It's a lot about letting go of old imperative and stateful habits of typical programming, and forcing your brain to work in a different paradigm. I haven't found any guide on the internet in this aspect, and I think the world deserves a practical tutorial on how to think in Reactive, so that you can get started. Library documentation can light your way after that. I hope this helps you.

## "啥是Reactive Programming?"

互联网上充斥着很多操蛋的解释。[维基百科](https://en.wikipedia.org/wiki/Reactive_programming) 又宽泛有玄乎。 [Stackoverflow](http://stackoverflow.com/questions/1028250/what-is-functional-reactive-programming)教科书式的解释非常不适合信任[Reactive Manifesto](http://www.reactivemanifesto.org/) 听起来像是给给项目经理或者是销售的汇报。 微软的 [Rx 定义](https://rx.codeplex.com/) "Rx = Observables + LINQ + Schedulers" 太重并且太微软化了，让人看起来不知所云。“响应”、“变化发生”这些术语无法很好地阐释Reactive Programming的显著特点，听起来和你熟悉的MV*、编程语言差别不大。 当然，我的视角也是基于模型和变换的，要是脱离了这些概念，一切都是无稽之谈了。

那么我要开始吧啦吧啦了。

#### Reactive programming 是针对异步数据流的编程。

一定程度而言，Reactive programming并不算新的概念。事件总线、点击事件都是异步流。开发者可以观测这些异步流，并调用特定的逻辑对它们进行处理。使用Reactive如同开挂：你可以创建点击、悬停之类的任意流。通常流廉价（点击一下就出来一个）而无处不在，种类丰富多样：变量，用户输入，属性，缓存，数据结构等等都可以产生流。举例来说：微博回文（译者注：比如你关注的微博更新了）和点击事件都是流：你可以监听流并调用特定的逻辑对它们进行处理。

**基于流的概念，Reactive赋予了你一系列神奇的函数工具集，使用他们可以合并、创建、过滤这些流。** 一个流或者一系列流可以作为另一个流的输入。你可以_合并_
两个流，从一堆流中_过滤_你真正感兴趣的那一些，将值从一个流_映射_到另一个流。

如果流是Reactive programming的核心，我们不妨从“点击页面中的按钮”这个熟悉的场景详细地了解它。

![Click event stream](http://i.imgur.com/cL4MOsS.png)

流是包含了**有时序，正在进行事件**的序列，可以反射值（某种类型）、错误、完成信号。流在包含按钮的浏览器窗口被关闭时发出完成信号。

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

首先我们创建一个计数流来表明按钮被点击的次数。在Reactive中，每一个流都拥有一些列方法，例如`map`，`filter`，`scan` 等等。当你在流上调用这些方法，例如`clickStream.map(f)`，会返回基于点击事件流的**新的流**，同时原来的点击事件流并不会被改变，这个特性被称为**不可变性（immutability）**。不可变性与Reactive配合相得益彰，如同美酒加咖啡。我们可以链式地调用他们：`clickStream.map(f).scan(g)`

```
  clickStream: ---c----c--c----c------c--->
               vvvvv map(c becomes 1) vvvvv
               ---1----1--1----1------1--->
               vvvvvvvvv  scan(+) vvvvvvvvv
counterStream: ---1----2--3----4------5--->
```

`map(f)` 函数对原来的流使用我们出入的`f`函数进行转换，并生成新的流。在上面的例子中，我们将每一次点击映射为数字1。`scan(g)`函数将所有流产生的值进行汇总，通过传入`x = g(accumulated, current)`函数产生新的值，`g` 是简单的求和函数。最后 `counterStream`在点击发生后发射点击事件发生的总数。

为了展示Reactive的真正力量，我们举个例子：你想要“两次点击”事件的流，或者是“三次点击”，或者是n次点击的流。深呼吸一下，试着想想怎么用传统的命令、状态式方法来解决。我打赌这个这会相当操蛋，你会搞些变量来记录状态，还要搞些处理时延的机制。

如果用Reactive来解决，太他妈简单了。实际上[4行代码就可以搞定](http://jsfiddle.net/staltz/4gGgs/27/)。先不要看代码，不管你是菜鸟还是牛逼，使用图表来思考可以使你更好地理解构建这些流的方法。

![Multiple clicks stream](http://i.imgur.com/HMGWNO5.png)

灰色框里面的函数会把一个流转换成另外一个流。首先我们把点击打包到list中，如果点击后消停了250毫秒，我们就重新打包一个新的list（显然`buffer(stream.throttle(250ms))`就是用来干这个的，不明白细节没有关系，反正是demo嘛）。我们在列表上调用`map()`，将列表的长度映射为一个整数的流。最后，我们通过`filter(x >= 2)`过滤掉整数`1`。哈哈：3个操作就生成了我们需要的流，现在我们可以订阅（监听）这个流，然后来完成我们需要的逻辑了。

通过这个例子，我希望你能感受到使用Reactive的牛逼之处了。这仅仅是冰山一角。你可以在不同地流上（比如API响应的流）进行同样的操作。同时，Reactive还提供了许多其他实用的函数。

## "那么请告诉我为啥我要在今后使用Reactive programming?"

Reactive Programming 提高了编码的抽象程度，你可以更好地关注在商业逻辑中各种事件的联系避免大量细节而琐碎的实现，使得编码更加简洁。

使用Reactive Programming，将使得数据、交互错综复杂的web、移动app开发收益更多。10年以前，与网页的交互仅仅是提交表单、然后根据服务器简单地渲染返回结果这些事情。App进化得越来越有实时性：修改表单中一个域可以同步地更新到后端服务器。“点赞”信息实时地在不同用户设备上同步。

现代App中大量的实时事件创造了更好的交互和用户体验，披荆斩棘需要利剑在手，Reactive Programming就是你手中的利剑。

## Reactive Programming编程思想（附实例）

我们将从实例可以深入Reactive Programming的编程思想，文章末尾，一个完整地实例应用会被构建，你也会理解整个过程。

我选择 **JavaScript** 和 **[RxJS](https://github.com/Reactive-Extensions/RxJS)** 作为构建的基础, 大多开发者都熟悉JavaScript语言。[Rx* library family](http://www.reactivex.io) 在各种语言和平台都是实现 ([.NET](https://rx.codeplex.com/), [Java](https://github.com/Netflix/RxJava), [Scala](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-scala), [Clojure](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-clojure),  [JavaScript](https://github.com/Reactive-Extensions/RxJS), [Ruby](https://github.com/Reactive-Extensions/Rx.rb), [Python](https://github.com/Reactive-Extensions/RxPy), [C++](https://github.com/Reactive-Extensions/RxCpp), [Objective-C/Cocoa](https://github.com/ReactiveCocoa/ReactiveCocoa), [Groovy](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-groovy), 等等)。无论你选择在哪个平台或者那种语言实践Reactive Programming，你都将从本教程中受益。

## 微博（Twitter）简易版“你可能感兴趣的人”推荐

微博主页，有一个组件会推荐给你那些你可能感兴趣的人。

![Twitter Who to follow suggestions box](http://i.imgur.com/eAlNb0j.png)

我们的Demo将使用这个场景，关注下面这些主要特性：

* 页面打开后，通过API加载数据展示3个你可能感兴趣的用户账号
* 点击“刷新”按钮，重新加载三个新的用户账号
* 在一个用户账号上点击'x' 按钮，清除当前这个账户，重新加载一个新的账户
* 每行展示账户的信息和这个账户主页的链接

其他特性和按钮我们暂且忽略，由于Twitter在最近关闭了公共API授权接口，我们选择Github作为代替，展示GitHub用户的账户。实例中我们使用该接口[获取GitHub用户](https://developer.github.com/v3/users/#get-all-users).

如果你希望先睹为快，完成后的代码已经发布在了[Jsfiddle](http://jsfiddle.net/staltz/8jFJH/48/)。

## Request and response

**How do you approach this problem with Rx?** Well, to start with, (almost) _everything can be a stream_. That's the Rx mantra. Let's start with the easiest feature: "on startup, load 3 accounts data from the API". There is nothing special here, this is simply about (1) doing a request, (2) getting a response, (3) rendering the response. So let's go ahead and represent our requests as a stream. At first this will feel like overkill, but we need to start from the basics, right?

On startup we need to do only one request, so if we model it as a data stream, it will be a stream with only one emitted value. Later, we know we will have many requests happening, but for now, it is just one.

```
--a------|->

Where a is the string 'https://api.github.com/users'
```

This is a stream of URLs that we want to request. Whenever a request event happens, it tells us two things: when and what. "When" the request should be executed is when the event is emitted. And "what" should be requested is the value emitted: a string containing the URL.

To create such stream with a single value is very simple in Rx*. The official terminology for a stream is "Observable", for the fact that it can be observed, but I find it to be a silly name, so I call it _stream_.

```javascript
var requestStream = Rx.Observable.just('https://api.github.com/users');
```

But now, that is just a stream of strings, doing no other operation, so we need to somehow make something happen when that value is emitted. That's done by [subscribing](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypesubscribeobserver--onnext-onerror-oncompleted) to the stream.

```javascript
requestStream.subscribe(function(requestUrl) {
  // execute the request
  jQuery.getJSON(requestUrl, function(responseData) {
    // ...
  });
}
```

Notice we are using a jQuery Ajax callback (which we assume you [should know already](http://devdocs.io/jquery/jquery.getjson)) to handle the asynchronicity of the request operation. But wait a moment, Rx is for dealing with **asynchronous** data streams. Couldn't the response for that request be a stream containing the data arriving at some time in the future? Well, at a conceptual level, it sure looks like it, so let's try that.

```javascript
requestStream.subscribe(function(requestUrl) {
  // execute the request
  var responseStream = Rx.Observable.create(function (observer) {
    jQuery.getJSON(requestUrl)
    .done(function(response) { observer.onNext(response); })
    .fail(function(jqXHR, status, error) { observer.onError(error); })
    .always(function() { observer.onCompleted(); });
  });
  
  responseStream.subscribe(function(response) {
    // do something with the response
  });
}
```

What [`Rx.Observable.create()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservablecreatesubscribe) does is create your own custom stream by explicitly informing each observer (or in other words, a "subscriber") about data events (`onNext()`) or errors (`onError()`). What we did was just wrap that jQuery Ajax Promise. **Excuse me, does this mean that a Promise is an Observable?**

&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;

![Amazed](http://www.myfacewhen.net/uploads/3324-amazed-face.gif)

Yes.

Observable is Promise++. In Rx you can easily convert a Promise to an Observable by doing `var stream = Rx.Observable.fromPromise(promise)`, so let's use that. The only difference is that Observables are not [Promises/A+](http://promises-aplus.github.io/promises-spec/) compliant, but conceptually there is no clash. A Promise is simply an Observable with one single emitted value. Rx streams go beyond promises by allowing many returned values.

This is pretty nice, and shows how Observables are at least as powerful as Promises. So if you believe the Promises hype, keep an eye on what Rx Observables are capable of.

Now back to our example, if you were quick to notice, we have one `subscribe()` call inside another, which is somewhat akin to callback hell. Also, the creation of `responseStream` is dependent on `requestStream`. As you heard before, in Rx there are simple mechanisms for transforming and creating new streams out of others, so we should be doing that. 

The one basic function that you should know by now is [`map(f)`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemapselector-thisarg), which takes each value of stream A, applies `f()` on it, and produces a value on stream B. If we do that to our request and response streams, we can map request URLs to response Promises (disguised as streams). 

```javascript
var responseMetastream = requestStream
  .map(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });
```

Then we will have created a beast called "_metastream_": a stream of streams. Don't panic yet. A metastream is a stream where each emitted value is yet another stream. You can think of it as [pointers](https://en.wikipedia.org/wiki/Pointer_(computer_programming)): each emitted value is a _pointer_ to another stream. In our example, each request URL is mapped to a pointer to the promise stream containing the corresponding response.

![Response metastream](http://i.imgur.com/HHnmlac.png)

A metastream for responses looks confusing, and doesn't seem to help us at all. We just want a simple stream of responses, where each emitted value is a JSON object, not a 'Promise' of a JSON object. Say hi to [Mr. Flatmap](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeflatmapselector-resultselector): a version of `map()` that "flattens" a metastream, by emitting on the "trunk" stream everything that will be emitted on "branch" streams. Flatmap is not a "fix" and metastreams are not a bug, these are really the tools for dealing with asynchronous responses in Rx.

```javascript
var responseStream = requestStream
  .flatMap(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });
```

![Response stream](http://i.imgur.com/Hi3zNzJ.png)

Nice. And because the response stream is defined according to request stream, **if** we have later on more events happening on request stream, we will have the corresponding response events happening on response stream, as expected:

```
requestStream:  --a-----b--c------------|->
responseStream: -----A--------B-----C---|->

(lowercase is a request, uppercase is its response)
```

Now that we finally have a response stream, we can render the data we receive:

```javascript
responseStream.subscribe(function(response) {
  // render `response` to the DOM however you wish
});
```

Joining all the code until now, we have:

```javascript
var requestStream = Rx.Observable.just('https://api.github.com/users');

var responseStream = requestStream
  .flatMap(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });

responseStream.subscribe(function(response) {
  // render `response` to the DOM however you wish
});
```

## The refresh button

I did not yet mention that the JSON in the response is a list with 100 users. The API only allows us to specify the page offset, and not the page size, so we're using just 3 data objects and wasting 97 others. We can ignore that problem for now, since later on we will see how to cache the responses.

Everytime the refresh button is clicked, the request stream should emit a new URL, so that we can get a new response. We need two things: a stream of click events on the refresh button (mantra: anything can be a stream), and we need to change the request stream to depend on the refresh click stream. Gladly, RxJS comes with tools to make Observables from event listeners.

```javascript
var refreshButton = document.querySelector('.refresh');
var refreshClickStream = Rx.Observable.fromEvent(refreshButton, 'click');
```

Since the refresh click event doesn't itself carry any API URL, we need to map each click to an actual URL. Now we change the request stream to be the refresh click stream mapped to the API endpoint with a random offset parameter each time.

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

Because I'm dumb and I don't have automated tests, I just broke one of our previously built features. A request doesn't happen anymore on startup, it happens only when the refresh is clicked. Urgh. I need both behaviors: a request when _either_ a refresh is clicked _or_ the webpage was just opened.

We know how to make a separate stream for each one of those cases:

```javascript
var requestOnRefreshStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
  
var startupRequestStream = Rx.Observable.just('https://api.github.com/users');
```

But how can we "merge" these two into one? Well, there's [`merge()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemergemaxconcurrent--other). Explained in the diagram dialect, this is what it does:

```
stream A: ---a--------e-----o----->
stream B: -----B---C-----D-------->
          vvvvvvvvv merge vvvvvvvvv
          ---a-B---C--e--D--o----->
```

It should be easy now:

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

There is an alternative and cleaner way of writing that, without the intermediate streams.

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  })
  .merge(Rx.Observable.just('https://api.github.com/users'));
```

Even shorter, even more readable:
```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  })
  .startWith('https://api.github.com/users');
```

The [`startWith()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypestartwithscheduler-args) function does exactly what you think it does. No matter how your input stream looks like, the output stream resulting of `startWith(x)` will have `x` at the beginning. But I'm not [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself) enough, I'm repeating the API endpoint string. One way to fix this is by moving the `startWith()` close to the `refreshClickStream`, to essentially "emulate" a refresh click on startup.  

```javascript
var requestStream = refreshClickStream.startWith('startup click')
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

Nice. If you go back to the point where I "broke the automated tests", you should see that the only difference with this last approach is that I added the `startWith()`.

## Modelling the 3 suggestions with streams

Until now, we have only touched a _suggestion_ UI element on the rendering step that happens in the responseStream's `subscribe()`. Now with the refresh button, we have a problem: as soon as you click 'refresh', the current 3 suggestions are not cleared. New suggestions come in only after a response has arrived, but to make the UI look nice, we need to clean out the current suggestions when clicks happen on the refresh.

```javascript
refreshClickStream.subscribe(function() {
  // clear the 3 suggestion DOM elements 
});
```

No, not so fast, pal. This is bad, because we now have **two** subscribers that affect the suggestion DOM elements (the other one being `responseStream.subscribe()`), and that doesn't really sound like [Separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns). Remember the Reactive mantra? 

&nbsp;
&nbsp;
&nbsp;
&nbsp;

![Mantra](http://i.imgur.com/AIimQ8C.jpg)

So let's model a suggestion as a stream, where each emitted value is the JSON object containing the suggestion data. We will do this separately for each of the 3 suggestions. This is how the stream for suggestion #1 could look like:

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  });
```

The others, `suggestion2Stream` and `suggestion3Stream` can be simply copy pasted from `suggestion1Stream`. This is not DRY, but it will keep our example simple for this tutorial, plus I think it's a good exercise to think how to avoid repetition in this case.

Instead of having the rendering happen in responseStream's subscribe(), we do that here:

```javascript
suggestion1Stream.subscribe(function(suggestion) {
  // render the 1st suggestion to the DOM
});
```

Back to the "on refresh, clear the suggestions", we can simply map refresh clicks to `null` suggestion data, and include that in the `suggestion1Stream`, as such:

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  })
  .merge(
    refreshClickStream.map(function(){ return null; })
  );
```

And when rendering, we interpret `null` as "no data", hence hiding its UI element.

```javascript
suggestion1Stream.subscribe(function(suggestion) {
  if (suggestion === null) {
    // hide the first suggestion DOM element
  }
  else {
    // show the first suggestion DOM element
    // and render the data
  }
});
```

The big picture is now:

```
refreshClickStream: ----------o--------o---->
     requestStream: -r--------r--------r---->
    responseStream: ----R---------R------R-->   
 suggestion1Stream: ----s-----N---s----N-s-->
 suggestion2Stream: ----q-----N---q----N-q-->
 suggestion3Stream: ----t-----N---t----N-t-->
```

Where `N` stands for `null`.

As a bonus, we can also render "empty" suggestions on startup. That is done by adding `startWith(null)` to the suggestion streams:

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  })
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

Which results in:

```
refreshClickStream: ----------o---------o---->
     requestStream: -r--------r---------r---->
    responseStream: ----R----------R------R-->   
 suggestion1Stream: -N--s-----N----s----N-s-->
 suggestion2Stream: -N--q-----N----q----N-q-->
 suggestion3Stream: -N--t-----N----t----N-t-->
```

## Closing a suggestion and using cached responses

There is one feature remaining to implement. Each suggestion should have its own 'x' button for closing it, and loading another in its place. At first thought, you could say it's enough to make a new request when any close button is clicked:

```javascript
var close1Button = document.querySelector('.close1');
var close1ClickStream = Rx.Observable.fromEvent(close1Button, 'click');
// and the same for close2Button and close3Button

var requestStream = refreshClickStream.startWith('startup click')
  .merge(close1ClickStream) // we added this
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

That does not work. It will close and reload _all_ suggestions, rather than just only the one we clicked on. There are a couple of different ways of solving this, and to keep it interesting, we will solve it by reusing previous responses. The API's response page size is 100 users while we were using just 3 of those, so there is plenty of fresh data available. No need to request more.

Again, let's think in streams. When a 'close1' click event happens, we want to use the _most recently emitted_ response on `responseStream` to get one random user from the list in the response. As such:

```
    requestStream: --r--------------->
   responseStream: ------R----------->
close1ClickStream: ------------c----->
suggestion1Stream: ------s-----s----->
```

In Rx* there is a combinator function called [`combineLatest`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypecombinelatestargs-resultselector) that seems to do what we need. It takes two streams A and B as inputs, and whenever either stream emits a value, `combineLatest` joins the two most recently emitted values `a` and `b` from both streams and outputs a value `c = f(x,y)`, where `f` is a function you define. It is better explained with a diagram:

```
stream A: --a-----------e--------i-------->
stream B: -----b----c--------d-------q---->
          vvvvvvvv combineLatest(f) vvvvvvv
          ----AB---AC--EC---ED--ID--IQ---->

where f is the uppercase function
```

We can apply combineLatest() on `close1ClickStream` and `responseStream`, so that whenever the close 1 button is clicked, we get the latest response emitted and produce a new value on `suggestion1Stream`. On the other hand, combineLatest() is symmetric: whenever a new response is emitted on `responseStream`, it will combine with the latest 'close 1' click to produce a new suggestion. That is interesting, because it allows us to simplify our previous code for `suggestion1Stream`, like this:

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

One piece is still missing in the puzzle. The combineLatest() uses the most recent of the two sources, but if one of those sources hasn't emitted anything yet, combineLatest() cannot produce a data event on the output stream. If you look at the ASCII diagram above, you will see that the output has nothing when the first stream emitted value `a`. Only when the second stream emitted value `b` could it produce an output value.

There are different ways of solving this, and we will stay with the simplest one, which is simulating a click to the 'close 1' button on startup:

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

## Wrapping up

And we're done. The complete code for all this was:

```javascript
var refreshButton = document.querySelector('.refresh');
var refreshClickStream = Rx.Observable.fromEvent(refreshButton, 'click');

var closeButton1 = document.querySelector('.close1');
var close1ClickStream = Rx.Observable.fromEvent(closeButton1, 'click');
// and the same logic for close2 and close3

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
// and the same logic for suggestion2Stream and suggestion3Stream

suggestion1Stream.subscribe(function(suggestion) {
  if (suggestion === null) {
    // hide the first suggestion DOM element
  }
  else {
    // show the first suggestion DOM element
    // and render the data
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


{% endif %}
