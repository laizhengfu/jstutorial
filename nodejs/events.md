---
title: events模块
layout: page
category: nodejs
date: 2014-10-20
modifiedOn: 2014-10-20
---

## 基本用法

events模块是node.js对“发布/订阅”模式（publish/subscribe）的部署。该模块通过EventEmitter属性，提供了一个构造函数。该构造函数的实例具有on方法，可以用来监听指定事件，并触发回调函数。任意对象都可以发布指定事件，被EventEmitter实例的on方法监听到。

下面是一个实例，先建立一个消息中心，然后通过on方法，为各种事件指定回调函数，从而将程序转为事件驱动型，各个模块之间通过事件联系。

{% highlight javascript %}

var EventEmitter = require("events").EventEmitter;
 
var ee = new EventEmitter();
ee.on("someEvent", function () {
        console.log("event has occured");
});
 
ee.emit("someEvent");

{% endhighlight %}

上面代码在加载events模块后，通过EventEmitter属性建立了一个EventEmitter对象实例，这个实例就是消息中心。然后，通过on方法为someEvent事件指定回调函数。最后，通过emit方法触发someEvent事件。

emit方法还接受第二个参数，用于向回调函数提供参数。

{% highlight javascript %}

ee.on("someEvent", function (data){
        console.log(data);
});
 
ee.emit("someEvent", data);

{% endhighlight %}

EventEmitter实例的emit方法，用来触发事件。它的第一个参数是事件名称，其余参数都会依次传入回调函数。

默认情况下，Node.js允许同一个事件最多可以指定10个回调函数。

{% highlight javascript %}

ee.on("someEvent", function () { console.log("event 1"); });
ee.on("someEvent", function () { console.log("event 2"); });
ee.on("someEvent", function () { console.log("event 3"); });

{% endhighlight %}

超过10个回调函数，会发出一个警告。这个门槛值可以通过setMaxListeners方法改变。 

{% highlight javascript %}

ee.setMaxListeners(20);

{% endhighlight %}

events模块的作用，还表示在其他模块可以继承这个模块，因此也就拥有了EventEmitter接口。

{% highlight javascript %}

var EventEmitter = require('events').EventEmitter;

function Dog(name) {
    this.name = name;
}

Dog.prototype.__proto__ = EventEmitter.prototype;

var simon = new Dog('simon');

simon.on('bark', function(){
    console.log(this.name + ' barked');
});

setInterval(function(){
    simon.emit('bark');
}, 500);

{% endhighlight %}

上面代码新建了一个构造函数Dog，然后让其继承EventEmitter，因此Dog就拥有了EventEmitter的接口。最后，为Dog的实例指定bark事件的监听函数。最后，使用EventEmitter的emit方法，触发bark事件。

## 事件类型

events模块默认支持一些事件。

- newListener事件：添加新的回调函数时触发。
- removeListener事件：移除回调时触发。

{% highlight javascript %}

ee.on("newListener", function (evtName){
	console.log("New Listener: " + evtName);
});

ee.on("removeListener", function (evtName){
	console.log("Removed Listener: " + evtName);
});

function foo (){}

ee.on("save-user", foo);
ee.removeListener("save-user", foo);

// New Listener: removeListener
// New Listener: save-user
// Removed Listener: save-user

{% endhighlight %}

上面代码会触发两次newListener事件，以及一次removeListener事件。

## EventEmitter对象的方法

**（1）once方法**

该方法类似于on方法，但是回调函数只触发一次。

{% highlight javascript %}

ee.once("firstConnection", function (){
		console.log("本提示只出现一次"); 
});

{% endhighlight %}

**（2）removeListener方法**

该方法用于移除回调函数。它接受两个参数，第一个是事件名称，第二个是回调函数名称。这就是说，不能用于移除匿名函数。

```javascript

var EventEmitter = require('events').EventEmitter;

var emitter = new EventEmitter;

emitter.on('message', console.log);

setInterval(function(){
    emitter.emit('message', 'foo bar');
}, 300);

setTimeout(function(){
    emitter.removeListener('message', console.log);
}, 1000);

```

上面代码每300毫秒触发一次message事件，直到1000毫秒后取消监听。

另一个例子是使用removeListener方法模拟once方法。

{% highlight javascript %}

var EventEmitter = require('events').EventEmitter;

var emitter = new EventEmitter;

function onlyOnce () {
	console.log("You'll never see this again");
	emitter.removeListener("firstConnection", onlyOnce);
}

emitter.on("firstConnection", onlyOnce);

{% endhighlight %}

**（3）removeAllListeners方法**

该方法用于移除某个事件的所有回调函数。

{% highlight javascript %}

var EventEmitter = require('events').EventEmitter;

var emitter = new EventEmitter;

// some code here

emitter.removeAllListeners("firstConnection");

{% endhighlight %}

如果不带参数，则表示移除所有事件的所有回调函数。

{% highlight javascript %}

emitter.removeAllListeners();

{% endhighlight %}

**（4）listener方法**

该方法接受一个事件名称作为参数，返回该事件所有回调函数组成的数组。

{% highlight javascript %}

function onlyOnce () {
	console.log(ee.listeners("firstConnection"));
	ee.removeListener("firstConnection", onlyOnce);
	console.log(ee.listeners("firstConnection"));
}

ee.on("firstConnection", onlyOnce)
ee.emit("firstConnection");
ee.emit("firstConnection");

// [ [Function: onlyOnce] ]
// []

{% endhighlight %}

上面代码显示两次回调函数组成的数组，第一次只有一个回调函数onlyOnce，第二次是一个空数组，因为removeListener方法取消了回调函数。
