---
title: RxJS学习总结-Observable
date: 2019-02-22 10:35:15
tags:
 - RxJS
 - 知识积累
---

 上篇博客我们使用`RxJS`的一些知识实现了一个贪吃蛇的小游戏，对`RxJS`有了一定的了解。接下来几篇博客会从基础开始，总结一下`RxJS`的概念、操作符、多播、调度器等知识点。

今天先来看一下`Observable`的几个相关概念。

## Observable可观察对象

### 概述

什么是`Observable`？讲道理我也不太清楚到底什么是`Observable`，硬要一个定义的话，可以说`Observable`是一个以推送`Push`方式产生一个或多个值（同步或者异步）的集合。我们以推送方式和产生值的数量来对`JavaScript`常见的几种概念来个分类：

|          | 单值SINGLE | 多值MULTIPLE |
| -------- | ---------- | ------------ |
| 拉取PULL | Function   | Iterator     |
| 推送PUSH | Promise    | Observable   |

`Function`函数，每个函数只能有一个`return`出来的值，而且当调用它时，会同步的发出值。生产者`foo()`的值是由消费者`x`主动`Pull`拉取来的，单值。

```javascript
function foo() {
  console.log('hello');
  return 47;
  return 48; //被忽略
}
const x = foo.call();  // 或者foo();
console.log(x);
// hello
// 47
```

`Iterator`迭代器，迭代器可以产生多个值，需要消费者自己`Pull`取用。想想`async/await`。

```javascript
const arr = [1, 2, 3];
const iterator = arr[Symbol.iterator]();

iterator.next();
// { value: 1, done: false }
iterator.next();
// { value: 2, done: false }
iterator.next();
// { value: 3, done: false }
iterator.next();
// { value: undefined, done: true }
```

`Promise`承诺(感觉怪怪的)，承诺以推送`Push`的方式发出（或者不发出）单值。承诺在创建的时候会立即计算出结果，而且只执行一次（跟`Observable`的区别）。

```javascript
const promise = new Promise((resolve, reject) => {
  console.log('hello');
  resolve('world');
})
// hello
promise.then(res => console.log(res));
// world
```

`Observable`可观察对象，今天的重点，以推送`Push`的方式同步或异步的发出一个或多个值，直接看代码：

```javascript
import { Observable } from 'rxjs';

const observable = new Observable(subscriber => {
  console.log('hello');  // 不订阅，不会打印
  subscriber.next(1);
  subscriber.next(2);
  setTimeout(() => {
    subscriber.next(3);   // 订阅后 1s 发出 3
    subscriber.complete();
  }, 1000);
})
// 创建后不会有任何值打印出来
```

想要调用`observable`并且看到值，我们需要订阅`subscribe`它：

```javascript
console.log('just before subscribe');
observable.subscribe({
  next(x) { console.log('got value ' + x); },
  error(err) { console.error('something wrong occurred: ' + err); },
  complete() { console.log('done'); }
});
console.log('just after subscribe');
// just before subscribe
// hello
// got value 1
// got value 2
// just after subscribe
// got value 3
// done
```

### Pull && Push

`Pull`拉取模式，消费者`Consumer`决定什么时候从生产者`Producer`处拿值。生产者不知道数据什么时候会发给消费者。如上面的`Function`和`Iterator`，只有当函数`foo()`调用或者变量`iterator`调用`next()`方法时，即“拉取`Pull`”时，才会“消费”值。

`Push`推送模式，生产者`Producer`决定什么时候发出值，消费者不知道数据什么时候过来。`Promise`是最常见的推送模式，promise推送`resolved`值到回调函数。`RxJS`的`Observable`也是推送模式，不过它可以推送多个值（同步或异步）。

### Observable & Function & EventEmitters & Promise 异同

`Observable`有点像无参数的函数`Function`，但是是可以产生多个值。先看下面：

```javascript
// Function 形式
function foo() {
  console.log('hello');
  return 47;
}
const x = foo.call();
console.log(x);  // hello 47
const y = foo.call();
console.log(x);  // hello 47

// Observable 形式
import { Observable } from 'rxjs';
const foo = new Observable(subscriber => {
  console.log('hello');
  subscriber.next(47);
});
 
foo.subscribe(x => {
  console.log(x);   // hello 47
});
foo.subscribe(y => {
  console.log(y);   // hello 47
});
```

`Function`和`Observable`都是延迟计算`lazy computation`，如果你不调用函数`foo()`，那么`console.log('hello')`就不会运行；同样，如果你不订阅（使用`subscribe`）`Observable`，那么`console.log('hello')`也不会运行。而且不管`调用`还是`订阅`都是一个独立的操作：多次调用或者多次订阅的副作用`side effects`是相互独立，互不影响的。

`Observable`和`EventEmitter`有些类似，但`Observable`适用范围更广，且数据在交付前可以预处理：

```javascript
// EventEmitter
function handler(e) {
  // 数据无法被预处理
  console.log('Clicked', e);
}
button.addEventListener('click', handler);
button.removeEventListener('click', handler);

// Observable
let clicks$ = fromEvent(buttonEl, 'click');
let subscription = clicks$.pipe(
  map(e => e.target.value)   // 预处理事件，直接发出value
).subscribe(val => console.log('Clicked', val))
subscription.unsubscribe();
// 除事件处理外，Observable还可以处理其他数据流
```

这里，`EventEmitter`更像是使用了`Subject`的`Observable`，后面会再讨论`Subject`。

`Observable`经常被拿来跟`Promise`进行比较。有一些关键点的不同：

1. `Observable`是声明式的，在被订阅之前，它不会开始执行。`Promise`是在创建时就立即执行的，具体见上面的例子。
2. `Observable`可以提供多个值，`Promise`只提供一个。`Observable`可以随着时间的推移获取多个值（流）。
3. `Observable`可以通过管道`pipe()`串联处理数据，`Promise()`只有`then()`语句。`Observable`可以使得数据在被订阅之前进行处理。
4. `Observable`的`subscribe(observer)`会负责处理错误`error`，`Promise`会把错误推给它的`子Promise`去处理。
5. 最重要一点，`Observable`是可以通过`unsubscribe()`取消的。取消订阅就会移除监听器，不再接收将来的值。

最后简单提一下`Observable`和数组`Array`在运算方式上的不同。先看下面：

```javascript
// 数组链式调用
let arr = [1, 2, 3, 4];
let result = arr.map(num => num * num).filter(num => num > 5);
console.log(result);  // [9, 16]

// Observable管道操作
import { from } from 'rxjs';
import { map, filter } from 'rxjs/operators';
from([1, 2, 3, 4]).pipe(
  map(num => num * num),
  filter(num => num > 5)
).subscribe(console.log);  // 9, 16
```

数组的链式调用，每一步都会等所有数据计算完，才继续走下一步。即`.map()`先返回一个数据`[1, 4, 9, 16]`，然后`.filter()`过滤后，返回`[9, 16]`；而`Observable`的管道，每一个值都会处理到底，再继续下一个值。即数字1先`.map()`，再`.filter()`，不符合；同样，2不符合；数字3进来，符合，打印出9；最后处理数字4，打印16。

## subscriber订阅者函数

当我们手动创建一个`Observable`的实例时，就需要传入一个订阅者函数`subscriber`。当有消费者调用`subscribe()`方法时，这个函数就会执行。

```javascript
import { Observable } from 'rxjs';
const observable = new Observable(function subscriber(subscriber) {
  try {
    subscriber.next(1);
    subscriber.next(2);
    subscriber.next(3);
    subscriber.complete();  // 完成
    subscriber.next(4);     // 不会发出
  } catch (err) {
    subscriber.error(err);  // 捕获错误
  }
})
```

不过，我们一般情况下不使用这种形式创建`Observable`，而是使用`creation functions`，如`of()`、`fromEvent()`、`interval()`等等。

> *Observables can be created with* `new Observable`*. Most commonly, observables are created using creation functions, like* `of`*,* `from`*,* `interval`*, etc.*

注意，`complete()`后，后面的值将不会再发出。自定义`subscriber`函数时，最好将代码用`try/catch`包裹，处理错误。

## 订阅Subscribing

只有当有人订阅 `Observable` 的实例时，它才会开始发布值。 订阅时要先调用该实例的 `subscribe()` 方法，并把一个观察者对象`observer`传给它，用来接收通知。`observer`定义了收到数据时的处理器`handler`。同时`subscribe()`的调用会返回一个`Subscription`对象，这个对象有一个`unsubscribe()`方法，用来取消订阅。

```javascript
import { Observable } from 'rxjs';
const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
});
// 观察者对象
const observer = {
  next: x => console.log('next value: ' + x),
  error: err => console.error('error: ' + err),
  complete: () => console.log('complete'),
}
// 第一次订阅
observable.subscribe(observer);
// next value: 1
// next value: 2
// next value: 3
// complete

// 第二次订阅
observable.subscribe(observer);
// next value: 1
// next value: 2
// next value: 3
// complete
```

订阅后，`Observable`发出值，`observer`接收值，处理值。注意：多次订阅互不影响。

> *Subscribing to an Observable is like calling a function, providing callbacks where the data will be delivered to.*

另外，订阅时还可以直接传入回调函数，`next` 处理器都是必要的，而 `error` 和 `complete` 处理器是可选的，注意传入顺序。

```javascript
observable.subscribe(
  x => console.log('next value: ' + x),
  err => console.error('error: ' + err),
  () => console.log('complete')
);
// next value: 1
// next value: 2
// next value: 3
// complete
```

每当调用`subscribe()`方法时，都会返回一个`Subscription`对象，这个对象有一个`unsubscribe()`方法，用来取消订阅。类似于`setInterval()`和`setTimeout()`返回`id`的形式。

```javascript
const subscription = observable.subscribe(observer);
// 之后取消订阅
subscription.unsubscribe();
```

## observer观察者

观察者对象`observer`定义了一些回调函数来处理可观察对象`Observable`可能发来的三种通知：

| 通知类型 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| next     | 必要。用来处理每个送达值。在开始执行后可能执行零次或多次。   |
| error    | 可选。用来处理错误通知。错误会中断这个可观察对象实例的执行过程。 |
| complete | 可选。用来处理执行完毕（complete）通知。当执行完毕后，这些值就会继续传给下一个处理器。 |



以上就是`Observable`的一些基础知识点，简单记忆就是：

```javascript
const subscription = new Observable(subscriber).subscribe(observer);
subscription.unsubscribe();
```

接下来我们会先总结一下`Observable`的创建函数和操作符，`RxJS`繁琐就在于`creation function`和`operators`特别多，需要费点时间去记忆和理解，但也正因为这样，`RxJS`在处理流数据的时候才能“因材施教”，得心应手。过完这些知识点后，我们再去总结`Subject`和`Scheduler`这两个难搞的点。

