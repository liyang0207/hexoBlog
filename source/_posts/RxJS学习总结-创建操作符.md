---
title: RxJS学习总结-创建操作符
date: 2019-02-25 14:20:09
tags:
 - RxJS
 - 知识积累
---

上一篇博客总结了一下`Observable`可观察对象，主要使用了`new Observable(subscriber)`来创建`Observable`流。但这种数据流的创建方式，一是比较繁琐，需要自己去定义发出值，错误处理等，不够简洁；二是一旦繁杂起来，手写`subscriber`极容易出错。`RxJS`中就抽出了若干固定模式，称为创建类操作符，方便我们来使用。

创建类操作符是数据流的源头，大部分（并不是全部）都是静态操作符。按照同步/异步，可以分为创建同步数据流和异步数据流。

## 创建同步数据流

创建同步的`Observable`对象，需要关心产生了什么数据，数据之间的先后顺序如何，不需要关心时间。

### of 列举数据

`of<T>(...args: Array<T | SchedulerLike>): Observable<T>`

`of`操作符是将参数转化为`Observable`对象的序列。这是最简单的一个创建操作符。

```javascript
import { of } from 'rxjs';

of(1,3,4,5).subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
);
// next:1
// next:3
// next:4
// next:5
// complete
```

数据发送完，会调用`complete()`函数。注意，`of`发出的数据是同步发出的，只有先后顺序，没有时间间隔。

### range 指定范围

`range(start: number = 0, count?: number, scheduler?: SchedulerLike): Observable<number>`

`range`操作符用来产生一个范围内的正整数序列。接收的第一个参数是`number`类型，表示起始值，第二个参数是发出的数据个数，每个数据间隔为1。如果只传入一个参数，则默认起始数字为0。

```javascript
import { of, range } from 'rxjs'; 

range(2, 3).subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
);
// next:2
// next:3
// next:4
// complete

range(2).subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
);
// next:0
// next:1
// complete
```

### generate 循环创建

`generate<T, S>(initialStateOrOptions: S | GenerateOptions<T, S>, condition?: ConditionFunc<S>, iterate?: IterateFunc<S>, resultSelectorOrObservable?: (ResultFunc<S, T>) | SchedulerLike, scheduler?: SchedulerLike): Observable<T>`

`generate`操作符类似一个for循环，设定一个初始值，每次递增这个值，直到满足某个条件的时候才终止循环，同时，循环体内可以根据当前值产生数据。如果我们需要使用for循环产生一组数据，那么就适合使用`generate`操作符。

### empty 立即完成

`empty(scheduler?: SchedulerLike)`

`empty`操作符用来产生一个立即完结的`Observable`对象，不产生任何数据，直接`complete`。

```javascript
import { empty } from 'rxjs'; 

empty().subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// complete
```

### never 永不完成

`never()`

`never`操作符永远不会完结，即不产生数据，也不产生错误。

```javascript
import { never } from 'rxjs'; 

never().subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// 什么也没有
```

### throwError 扔出错误

`throwError(error: any, scheduler?: SchedulerLike): Observable<never>`

`throw`操作符一开始就直接抛出错误，不会产生任何数据。

```javascript
import { throwError } from 'rxjs'; 

throwError('Oop!').subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// error:Oop!
```

## 创建异步数据流

创建异步的`Observable`对象，重点需要关心产生数据之间的时间间隔。

### from

`form<T>(input: ObservableInput<T>, scheduler?: SchedulerLike): Observable<T>`

`from`操作符接收数组、类数组对象、`Promise`、`iterable object`或者`Observable-like object`来创建一个`Observable`对象。

```javascript
import { from } from 'rxjs'; 

//数组
from([1, 2, 3]).subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// next:1
// next:2
// next:3
// complete

//字符串
from('abc').subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// next:a
// next:b
// next:c
// complete

//promise
from(new Promise((resolve, reject) => {
  setTimeout(() => {
      resolve('Hello RxJS!');
    },3000)
})).subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// next:Hello RxJS!
// complete
```

### fromEvent

`fromEvent<T>(target: FromEventTarget<T>, eventName: string, options?: EventListenerOptions | ((...args: any[]) => T), resultSelector?: ((...args: any[]) => T)): Observable<T>`

`formEvent`经常被用来将页面中的`DOM`事件转化为`Observable`对象中的数据，是连接`DOM`和`RxJS`的桥梁，产生`Observable`对象之后，就可以交由`RxJS`来进行后续的处理。

```javascript
import { from } from 'rxjs'; 

fromEvent(document, 'click').subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// 点击页面，打印 next:[object MouseEvent]，发出一个点击事件流
```

### interval 定时产生数据

`interval(period: 0 = 0, scheduler: SchedulerLike = async): Observable<number>`

`interval`接收一个数值类型的参数，代表产生数据的间隔毫秒数，返回从0开始，按这个间隔递增的整数`Observable`序列。是一个无限序列。

```javascript
import { interval } from 'rxjs'; 

interval(1000).subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// ...1000ms
// next:0
// ...1000ms
// next:1
// ...1000ms
// next:2
// ...
```

### timer 定时产生数据

`timer(dueTime: number | Date = 0, periodOrScheduler?: number | SchedulerLike, scheduler?: SchedulerLike): Observable<number>`

`timer`第一个参数可以是毫秒数值，或者一个`Date`类型的对象，在经过这个毫秒或者到达这个`Date`时间过发出0后结束。如果传入第二个参数，则会产生一个持续发出递增数据的`Observable`对象，类似于`interval`操作符。

```javascript
import { timer } from 'rxjs'; 

timer(1000, 2000).subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// ...1000ms
// next:0
// ...2000ms
// next:1
// ...2000ms
// next:2
// ...
```

可以简单推出，`timer(1000, 1000)`相当于`interval(1000)`。

### repeat 重复订阅

`repeat<T>(count: number = -1): MonoTypeOperatorFunction<T>`

`repeat`从它的功能上来讲是可以看做创建类操作符的。但是它并不是一个静态方法（相比于上面的操作符），它是`Observable`的实例方法。`repeat`承接的是上游的数据流，通过退订->重新订阅的方法向下游发出数据。`repeat`只有等到上游`Observable`对象完结之后才会重新订阅，上游不完结，永远不会重新订阅。订阅次数是传入的参数次数。

```javascript
import { timer } from 'rxjs'; 
import { repeat } from 'rxjs/operators';

of(1).pipe(
  repeat(2)
).subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// next:1
// next:1
// complete
```

注意，`repeat`相当于将上游的流重复了`count`次发了出去，`observer`订阅到的流只是这`count`次的流。

### repeatWhen 有条件的重复订阅

`repeatWhen<T>(notifier: (notifications: Observable<any>) => Observable<any>): MonoTypeOperatorFunction<T>`

`repeatWhen`接收一个函数`notifier`作为参数，这个函数在上游第一次产生异常的时候调用，然后这个函数应该返回一个`Observable`对象，当这个`Observable`发出一个数据的时候，`repeatWhen`就会退订上游并重新订阅。

```javascript
import { of, timer } from 'rxjs'; 
import { repeatWhen } from 'rxjs/operators';

of(1, 2, 3).pipe(
  repeatWhen(() => {
    timer(2000)
  })
).subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// next:1
// next:2
// next:3
// 等待2s
// next:1
// next:2
// next:3
// complete
```

如果上游产生的是异步数据，不知道什么时候结束，那么什么重复就比较难判断。这时候可以为`notifier`函数传入一个参数`notifications`，是一个`Observable`对象，当上游完成的时候，`notifications`就会发出一个数据。

```javascript
import { interval } from 'rxjs'; 
import { repeatWhen, take } from 'rxjs/operators';

interval(1000).pipe(
  take(3),
  repeatWhen((notification) => {
    return notification.pipe(delay(3000))
  })
).subscribe(
  val => console.log('next:' + val),
  err => console.log('error:' + err),
  () => console.log('complete')
)
// next:0
// next:1
// next:2
// 等待3s
// next:0
// next:1
// next:2
// 等待3s
// next:0
// next:1
// next:2
// ...
```



以上就是`RxJS`中最常用的几个创建操作符。还有几个操作符如`ajax`、`defer`等用的比较少，用到的时候查文档即可。不过写到这里感觉这个系列博客如果只是重复机械式的把几十个操作符列举一遍实在没有任何意义，总结的再详细也只是官方文档的搬运工，而且也不会比官方例子更详实。所以接下来会简要概括，将操作符进行分类，方便记忆和日后查阅时能找准方向。最后会将重点放在`Subject`和`Scheduler`的理解上。

