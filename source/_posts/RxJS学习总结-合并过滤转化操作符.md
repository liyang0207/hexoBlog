---
title: RxJS学习总结-合并过滤转化操作符
date: 2019-03-03 20:43:09
tags:
 - RxJS
 - 知识积累
---

`RxJS`中，除了创建类操作符，其余所有操作符最重要的三类就是合并类、过滤类和转化类。今天我们来简单归总一下这三类操作符。各操作符具体的实践方式可以自行到[RxJS官方API](https://rxjs.dev/api)中查阅。

## 合并类

### concat

`concat<O extends ObservableInput<any>, R>(...observables: Array<O | SchedulerLike>): Observable<ObservedValueOf<O> | R>`

`concat`用来将多个`Observable`依次合并到一起。有静态方法和实例方法。`实例方法`可能会在下个版本弃用，推荐使用`静态方法`，如此引入：`import { concat } from 'rxjs'`。

### merge

`merge<T, R>(...observables: Array<ObservableInput<any> | SchedulerLike | number>): Observable<R>`

`merge`用来合并多个`Observable`数据，同时发出每个给定输入`Observable`的所有值，来一个数据就发一个数据。同样有静态方法和实例方法，下个版本`实例方法`将会弃用。

### zip

`zip<O extends ObservableInput<any>, R>(...observables: Array<O | ((...values: ObservedValueOf<O>[]) => R)>): Observable<ObservedValueOf<O>[] | R>`

`zip`是拉链的意思，将多个`Observable`发出的相同index位置的值像拉链一样组合到一起。同样有静态方法和实例方法，下个版本`实例方法`将会弃用。

### combineLatest

`combineLatest<O extends ObservableInput<any>, R>(...observables: (O | ((...values: ObservedValueOf<O>[]) => R) | SchedulerLike)[]): Observable<R>`

`combineLatest`组合多个`Observable`来创建一个`Observable`，它的值是根据每个输入`Observable`的最新值计算出来的。同样有静态方法和实例方法，下个版本`实例方法`将会弃用。

### withLatestFrom

`withLatestFrom<T, R>(...args: Array<ObservableInput<any> | ((...values: Array<any>) => R)>): OperatorFunction<T, R>`

`withLatestFrom`将源`Observable`与其他`Observable`组合以创建一个`Observable`，它的值是各`Observable`发出的最新值组合在一起，但只有当源`Observable`发出值时才会发出值。

### race

`race<T>(...observables: (Observable<any>[] | Oservable<any>)[]): Observable<T>`

`race`是个静态方法，返回的是第一个发出值（胜出）的`Observable`的镜像。也就是说谁先发出值，则`race`产生的值就完全采用它，其他的"败者"就会被抛弃。

### startWith

`startWith<T, D>(...array: Array<T | SchedulerLike>): OperatorFunction<T, T | D>`

`startWith`可以接收多个值，在源`Observable`发出值之前先发出这几个值。

### forkJoin

`forkJoin<T>(...sources: Array<ObservableInput<T> | ObservableInput<T>[] | Function>): Observable<T[]>`

这是个静态方法。将所有给定的`Observable`发出的最后一个值组合成数组发给下游。

### 处理高阶Observable

#### concatAll

`concatAll<T>(): OperatorFunction<ObservableInput<T>, T>`

`concatAll`将高阶`Observable`转换为一阶`Observable`发出，发出顺序遵循`concat`原则，一个完成再订阅下一个。

#### mergeAll

`mergeAll<T>(concurrent: number = Number.POSITIVE_INFINITY): OperatorFunction<ObservableInput<T>, T>`

`mergeAll`将高阶`Observable`转换为一阶`Observable`发出，发出顺序遵循`merge`原则。

#### zipAll

`zipAll<T, R>(project?: (...values: Array<any>) => R): OperatorFunction<T, R>`

`zipAll`也是用来处理高阶`Observable`的，类似`zip`。

#### combineAll

`combineAll`就是处理高阶`Observable`的`combineLatest`。

#### switchAll

`switchAll<T>(): OperatorFunction<ObservableInputM<T>, T>`

`switchAll`也是用来处理高阶`Observable`的，它的主要作用是"切换"，每当上游高阶`Observable`产生一个新的内部`Observable`，`switchAll`都会退订当前的`Observable`，来订阅这个最新的`Observable`。这样不断切换到最新的`Observable`。

#### exhaust

`exhaust<T>(): OperatorFunction<any, T>`

`exhaust`正好跟`switchAll`相反，在"耗尽"（即完成）上一个`Observable`前，不会订阅下一个`Observable`。

## 过滤类

### filter

`filter<T>(predicate: (value: T, index: number) => boolean, thisArg?: any): MonoTypeOperatorFunction<T>`

`filter`发出那些经过`predicate`过滤后的值。

### first

`first<T, D>(predicate?: ((value: T, index: number, source: Observable<T>) => boolean) | null, defaultValue?: D): OperatorFunction<T, T | D>`

`first`发出第一个值（或者第一个满足某个条件的值），都不满足时，发出默认值`defaultValue`（如果有）。

### last

`last<T, D>(predicate?: ((value: T, index: number, source: Observable<T>) => boolean) | null, defaultValue?: D): OperatorFunction<T, T | D>`

`last`发出最后一个值（或者最后一个满足某个条件的值），都不满足时，发出默认值`defaultValue`（如果有）。必须等`Observable`完结之后才会知道最后一个值。

### take

`take<T>(count: number): MonoTypeOperatorFunction<T>`

`take`只取`Observable`发出值的前几个值。

### takeLast

`takeLast<T>(count: number): MonoTypeOperatorFunction<T>`

`takeLast`取`Observable`的最后几个值，同步发出。

### takeWhile

`takeWhile<T>(predicate: (value: T, index: number) => boolean, inclusive: false = false): MonoTypeOperatorFunction<T>`

`takeWhile`发出那些满足`predicate`的值，一旦有值不满足条件，就会`complete`。

### takeUntil

`takeUntil<T>(notifier: Obervable<any>): MonoTypeOperatorFunction<T>`

`takeUntil`使用一个`Observable`来控制另外一个`Observable`产生数据。当`notifier Observable`发出值的时候，停止发出源`Observable`的值。

### skip, skipLast, skipWhile, skipUntil

`skip`用来跳过前几个值。这四个方法跟上面`take`的四个方法用法一致。

### throttle

`throttle`可以认为是"节流"数据，忽略那些由另外一个`Observable`控制的`duration`时间段内的数据。

### throttleTime

`throttleTime`从源`Observable`发出一个值，然后忽略持续时间毫秒的后续源值，然后重复此过程。

### debounce

`debounce`可以认为是"去抖"，只有在另一个`Observable`确定的时间跨度内没有其他源`Observable`发出值时，才会从源`Observable`发出一个值。

### debounceTime

`debounceTime`仅在经过特定时间跨度而没有其他源发射之后才从源`Observable`发出值。

### audit && auditTime

`audit`跟`throttle`类似，`throttle`取的是一段时间内的第一个值，而`audit`取的是一段时间内最后的一个值。`auditTime`则跟`throttleTime`类似，也是取最后一个值。

### distinct

`distinct<T, K>(keySelector?: (value: T) => K, flushes?: Observable<any>): MonoTypeOperatorFunction<T>`

`distinct`有点像一个`Set`，内部维持了一个集合，只有不在集合内的元素才会发出来，同时添加这个元素到集合内。第一个参数可以用来指定比较源`Observable`内的哪个属性值。第二个参数是一个`Observable`，用来清空内部的集合，避免数据过多内存溢出。

### distinctUntilChanged

`distinctUntilChanged<T, K>(compare?: (x: K, y: K) => boolean, keySelector?: (x: T) => K): MonoTypeOperatorFunction<T>`

`distinctUntilChanged`可以说是`distinct`的简化版，只会比较上一个值，所以不会有内存问题。

## 转化类

### map

`map<T, R>(project: (value: T, index: number) => R, thisArg?: any): OperatorFunction<T, R>`

`map`使用给定的`project`来处理每一个`Observable`发出的值，然后交给下游。

### mapTo

`mapTo<T, R>(value: R): OperatorFunction<T, R>`

`mapTo`将源`Observable`发出的值转化为一个常量发出。

### bufferTime

`bufferTime<T>(bufferTimeSpan: number): OperatorFunction<T, T[]>`

`bufferTime`用来缓存一个时间段内的数据，然后同步发出。

### bufferCount

`bufferCount<T>(bufferSize: number, startBufferEvery: number = null): OperatorFunction<T, T[]>`

`bufferCount`用来缓存一定数量的值，然后以数组形式同步发出。

### buffer

`buffer<T>(closingNotifier: Observable<any>): OperatorFunction<T, T[]>`

`buffer`缓存数据，直到`closingNotifier`这个`Observable`发出值，还是使用`Observable`来控制`Observable`。

### concatMap

`concatMap`相当于`map`加上`concatAll`，用来处理高阶`Observable`。每一个源`Observable`发出的值都产生了一个`Observable`，`concatMap`就是将这些`Observable`抚平，使值一个接一个发出来。

### mergeMap

同理，`mergeMap`就是`map`加上`mergeAll`，也是用来处理高阶`Observable`，同`mergeAll`一样，同时处理发出的值。

### switchMap

`switchMap`是`map`加上`switchAll`，一个适用场景就是，假如我们需要点击一次按钮发送一次`ajax`请求，则下一次请求发出时，需要取消上一次的`ajax`请求，这时就可以用`switchMap`来处理。

### exhaustMap

`exhaustMap`跟`switchMap`相反，先产生的内部`Observable`优先级最高，只有当前面的`Observable`完成，后产生的内部`Observable`才会被使用。

### scan

`scan<T, R>(accumulator: (acc: R, value: T, index: number) => R, seed?: T | R): OperatorFunction<T, R>`

`scan`是一个比较重要的操作符，逻辑类似于`reduce`方法，如果需要保存一个状态，则可以考虑使用它。

以上三类操作符基本上包括了`RxJS`中绝大部分常用的操作符。如何使用不是难事，查一下文档，看一些例子就八九不离十，难点在于如何正确的选择最合适的操作符，这个就只能在实际使用中不断摸索实验了。

