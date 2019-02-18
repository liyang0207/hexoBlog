---
title: 使用RxJS实现一个贪吃蛇小游戏
date: 2019-02-18 14:57:11
tags:
 - RxJS
 - 知识积累
---

最近几周趁年底年初工作事情少，把`RxJS`的基础知识学习了一下，算是简单入了个门，近期准备写几篇博客总结一下`RxJS`的相关知识点，给自己找点事情做。

## 概述

什么是`RxJS`？简单来说`RxJS`是响应式编程技术的`JavaScript`实现。响应式编程`Reactive Extension`，也叫`ReactiveX`，简称`Rx`，用官网上的一句话解释就是： 

>  An API for asynchronous programming with observable streams

其中，`Observable`是一种模式，有点像`Observer Pattern观察者模式`和`Iterator Pattern迭代器模式`的结合，而`streams`指的是数据流。Rx最早的实现是`Rx.NET`，之后又陆陆续续有了`RxJava`、`RxPy`等语言的实现。那`RxJS`当然就是Rx的`JavaScript`实现了。

不过今天先不细说`RxJS`的一些具体细节，这两天在网上找到一篇比较有趣的博客：使用`RxJS`来简单实现一个“贪吃蛇”小游戏，感觉很有趣，就由此作为`RxJS`入门学习的引子，看看如何用`RxJS`的思维方式去思考这个游戏的实现。先看一下游戏运行的预览图：

![](/images/snake/snake.gif)

## 设计&思考

首先可以确定的是整个游戏界面都是使用`canvas`来进行绘制的，游戏中的“蛇”、“苹果”(红色块儿)和分数(动图未展示)都是`canvas`用进来的数据流`streams`来进行绘制的，而这些流的产生及走向就是我们要思考的。在响应式编程中，编程无外乎数据流及输入数据流。从概念上来说，当响应式编程执行时，它会建立一套可观察的管道，可以根据变化采取行动。我们需要理清数据源头，找到它，生成它，进而处理整合它。先来概览一下这个游戏的“流”：

![snake流](/images/snake/snake-streams.png)

我们反向来进行思考。首先，canvas绘制的数据流来自于`scene$流`，这个流是由`snake$蛇`、`apples$苹果`和`score$分数`三个流合成的，但其中`apples$`流又来源于`snake$`。其次，`snake$`流受到键盘的操作输入`direction$`、蛇本身的速度`ticks$`和蛇长度`snakeLength$`的影响，这三者构成了`snake$`流的数据。再看`score$`分数流，每当蛇的长度改变时，`score$`就会改变，发出数据；最后，当我们判断到蛇吃到了苹果，`apples$`流就会发出数据，同时通过`appleEaten$`使`length$`执行`length$.next(point)`，让蛇长度`snakeLength$`进行改变，进而触发一系列的数据发出。

仔细看，里面的源头只有`direction$键盘方向事件`、`snakeLength$`和`ticks$定时器`三个流。

## 实际操作

### canvas游戏布局

这部分主要涉及到使`canvas`绘制一个游戏区域，细节略过，简单贴一下代码，博客最后有超链接指向源博客，有详细的说明。

```javascript
// html
// <div id="snake"></div>
const COLS = 30;
const ROWS = 30;
const GAP_SIZE = 1;
const CELL_SIZE = 10;
const CANVAS_WIDTH = COLS * (CELL_SIZE + GAP_SIZE);
const CANVAS_HEIGHT = ROWS * (CELL_SIZE + GAP_SIZE);

function createCanvasElement() {
  const canvas = document.createElement('canvas');
  canvas.width = CANVAS_WIDTH;
  canvas.height = CANVAS_HEIGHT;
  return canvas;
}

const canvas = createCanvasElement();
let ctx = canvas.getContext('2d');
document.getElementById('snake').appendChild(canvas);
```

### 键盘事件direction$流

`RxJS`提供了一个`fromEvent()`操作符用来将事件转换为数据流，我们就用它来将玩家按键输入转为流数据，同时使用一些`map()`、`filter()`等操作符，来处理一下，确保只输出上下左右按键的值：

```javascript
import { fromEvent } from 'rxjs';
import { map, filter } from 'rxjs/operators';
interface Point2D {
  x: number;
  y: number;
}
interface Directions {
  [key: number]: Point2D;
}
const DIRECTIONS: Directions = {
  37: { x: -1, y: 0 },
  39: { x: 1, y: 0 },
  38: { x: 0, y: -1 },
  40: { x: 0, y: 1 }
};
let keydown$ = fromEvent(document, 'keydown');
let direction$ = keydown$.pipe(
  map((event: KeyboardEvent) => DIRECTIONS[event.keyCode]),
  filter(direction => !!direction),
)
// 临时订阅一下direction$，简单在console里看下效果
direction$.subscribe(console.log);
```

页面中现在什么也没有，随便点按键盘按键，上下左右会输出值，其他键的输入会被忽略：

![](/images/snake/snake01.png)

这里有个问题，我们的蛇是不能进行反方向运动的，所以我们需要忽略掉反向的输入；同时对于连续相同方向的输入，我们也应该忽略掉。这样的话我们就需要记录下键盘最后的输入状态，不过这里最好还是不要引入外部变量，全部交由Observable管道去处理，这里就要用到`scan`操作符，类似于js中的`reduce()`方法。同时对于相同方向的输入，我们使用`distinctUntilChangede()`来处理：

```javascript
function nextDirection(previous, next) {
  let isOpposite = (previous: Point2D, next: Point2D) => {
    return next.x === previous.x * -1 || next.y === previous.y * -1;
  };
  return isOpposite(previous, next) ? previous : next;
}
let direction$ = keydown$.pipe(
  map((event: KeyboardEvent) => DIRECTIONS[event.keyCode]),
  filter(direction => !!direction),
  startWith(INITIAL_DIRECTION),    // 这里给direction$一个初始方向值
  scan(nextDirection),						 // 使用处理函数处理方向流
  distinctUntilChanged()					 // 当本次发出的数据跟前一个不同时，才会发出这个值
)
```

这次，相同方向或者相反方向的输入都已经被处理（忽略）掉了。看一下`direction$`的`Marbles`图：

![](/images/snake/snake02.png)

### snakeLength$蛇长流

我们再回头看一下上面的`snake流图`，`snakeLength$`记录的是蛇当前的长度，发出的是蛇的长度的数字流。这个流基于（或者说来源于）`length$`流，`length$`记录的是蛇每次吃到苹果时增加的长度（也是增加的分数），这时候我们需要手动把值emit出去，使用`BehaviorSubject()`来实现。当`length$`发出值，即使用`length$.next()`来给`Subject()`提供值，这个值就会在`snakeLength$`上发出。我们需要`snakeLength$`的值累加上`length$`的值：

```javascript
const SNAKE_LENGTH = 5;
let length$ = new BehaviorSubject<number>(SNAKE_LENGTH);  // SNAKE_LENGTH为蛇长的初始值
let snakeLength$ = length$.pipe(
  scan((snakeLength, step) => snakeLength + step),
  share()
);
```

注意，这里使用`share()`来允许多次订阅`Observable`，否则每次订阅都会重新创建源`Observable`。我们的`snakeLength$`同时被`snake$`流和`score$`流订阅，需要确保这两个流的获取的数据是同步的。

### snake$流

到目前，`direction$`和`snakeLength$`都已确定，还差一个`ticks$`就可以组成`snake$`流。`ticks$`是一个类计时器的东西，限定蛇的移动速度，抽象来讲就是我们需要一个按时间规律发出值的`Observable`，`interval()`操作符可以完成这个任务：

```javascript
const SPEED = 120;
let ticks$ = interval(SPEED);     // 每隔120ms依次发出...0...1...2...
```

从`ticks$`到最终的`snake$`，我们需要继续思考：每当`ticks`发出值，蛇继续前进还是增加身长，取决于是否吃到了苹果，我们可以使用`scan()`操作符去积累蛇身的长度。但接下来如何组合`direction$`、`snakeLength$`和`ticks$`？

`withLatestFrom()`可以完美实现我们的需求，我这里直接贴上官方文档的英文释义，通俗易懂：

> Combines the source Observable with other Observables to create an Observable whose values are calculated from the latest values of each, only when the source emits.

`withLatestFrom()`组合多个`Observables`，并且具有主从关系，非常适合这里当蛇移动的时候去组合`direction$`和`snakeLength$`：

```javascript
import { move, generateSnake } from './utils';
let snake$ = ticks$.pipe(
	withLatestFrom(direction$, snakeLength$, (_, direction, snakeLength) => [direction,snakeLength]),
  scan(move, generateSnake()),   // 根据方向流和蛇身长流来产生蛇流，最终发出[{x: 1, y: 2}, ...]数组对象形式的蛇流
  share()  // snake$被scene$和apples$使用，需要保持一致性，使用Subject()多播
);

// utils.ts
// 处理数据，积累蛇身的数组
export function move(snake, [direction, snakeLength]) {
  let nx = snake[0].x;
  let ny = snake[0].y;
  nx += 1 * direction.x;
  ny += 1 * direction.y;
  let tail;
  if (snakeLength > snake.length) {
    tail = { x: nx, y: ny };
  } else {
    tail = snake.pop();
    tail.x = nx;
    tail.y = ny;
  }
  snake.unshift(tail);
  return snake;
}

// 生成初始蛇数据，数组对象形式
export function generateSnake() {
  let snake: Array<Point2D> = [];
  for (let i = SNAKE_LENGTH - 1; i >= 0; i--) {
    snake.push({ x: i, y: 0 });
  }
  return snake;
}
```

我们主要的源 `Observable` 是 `ticks$`，每当管道上有新值发出，我们就取 `direction$` 和 `snakeLength$` 的最新值。注意，即使辅助流频繁地发出值(例如,玩家头撞键盘上)，也只会在每次定时器发出值时处理数据。

此外，我们给 `withLatestFrom` 传入了选择器函数，当主要的流产生值时才会调用此函数。此函数是可选的，如果不传，将会生成包含所有元素的列表。来看下`Marbles`图：

![](/images/snake/snake03.png)

可以看到，使用了`withLatestFrom()`后，只有当`ticks$`发出值时，才会去`direction$`和`snakeLength$`中取最近的值一起发出，然后经过`scan()`处理，生成蛇身的数组对象形式的流。

### score$流

玩家的比分比较好处理，有了`snakeLength$`我们直接使用`scan()`积累一下分数即可，每当`snakeLength$`发出值，累加一次分数：

```javascript
const POINTS_PER_APPLE = 1;     // 每次一分
let score$ = snakeLength$.pipe(
  startWith(0),  // 初始0分
  scan((score, _) => score + POINTS_PER_APPLE)
);
```

来看下`Marbles`图：

![](/images/snake/snake04.png)

这里看似简单，其实有点小坑。可以看到`score$`中没有出现`snakeLength$`流中数据`5`对应的分数，这是因为`snakeLength$`使用了`share()`来允许多次订阅它，同时`snake$`已经先行订阅了`snakeLength$`，这时候`score$`的订阅就只会先发出`startWith(0)`初始化的数据`0`，只有等到`length$.next(POINT)`发出值，`snakeLength$`发出`6`，`score$`才会同步发出分数`1`，仔细体会一下。

### apples$流

`snake$`和`score$`已经搞定，剩下的就是`apples$`流。先确定一下`apples$`的数据格式，我们需要在`canvas`中同时显示两个苹果，同时，如果蛇吃到了苹果，则会在页面中`非蛇身`的位置随机产生另外一个苹果。所以我们使用`[{x: *, y: *}, {x: **, y: **}]`数组对象的格式来表示苹果。每次蛇移动时都检查是否有碰撞。如果有碰撞，我们就生成一个新的苹果并返回一个**新的**数组。这样的话我们便可以利用 `distinctUntilChanged()` 来过滤掉完全相同的值。

```javascript
let apples$ = snake$.pipe(
  scan(eat, generateApples()),
  distinctUntilChanged(),  // apples$会随着snake$发出值而发出，过滤重复值
  share()  // apples$也会被applesEaten$和scene$订阅，使用share()保持数据一致
);

// utils.ts
export function eat(apples: Array<Point2D>, snake) {
  let head = snake[0];
  for (let i = 0; i < apples.length; i++) {
    if (checkCollision(apples[i], head)) {
      apples.splice(i, 1);
      return [...apples, getRandomPosition(snake)];
    }
  }
  return apples;
}
export function generateApples(): Array<Point2D> {
  let apples = [];
  for (let i = 0; i < APPLE_COUNT; i++) {
    apples.push(getRandomPosition());
  }
  return apples;
}
// 获取随机位置
export function getRandomPosition(snake: Array<Point2D> = []): Point2D {
  let position = {
    x: getRandomNumber(0, COLS - 1),
    y: getRandomNumber(0, ROWS - 1)
  };
  if (isEmptyCell(position, snake)) {
    return position;
  }
  return getRandomPosition(snake);
}
// 检测是否碰撞，即两点是否一致
export function checkCollision(a, b) {
  return a.x === b.x && a.y === b.y;
}
function isEmptyCell(position: Point2D, snake: Array<Point2D>): boolean {
  return !snake.some(segment => checkCollision(segment, position));
}
function getRandomNumber(min, max) {
  return Math.floor(Math.random() * (max - min + 1) + min);
}

```

每当 `apples$` 产生一个新值时，我们就可以假定蛇吞掉了一个苹果。剩下要做的就是增加比分，还要将此事件通知给其他流，比如 `snake$`，它从 `snakeLength$` 中获取最新值，以确定是否将蛇的身体变长。

### appleEaten$流

我们在上面的`eat`方法中检测到了“碰撞”，即蛇吃到了苹果。这时候我们应该调用`length$.next(POINTS_PER_APPLE)`去增加蛇的长度`snakeLength$`，但是我们把`eat`工具方法整合到了`utils.ts`文件内，无法调用到`length$.next()`，这个时候我们可以引入一个中间流，让它帮我们通知。

`appleEaten$`扮演了这个角色，首先，因为`appleEaten$`订阅的是`apples$`流，`apples$`刚开始就有一个初始值发出，我们需要跳过这个初始值，不然一开局比分就会增加，显然不行；其次，`appleEaten$`只负责扮演通知者的角色，它只负责通知其他的流，而不会有观察者来订阅它。因此，我们需要**手动**订阅。

```javascript
let appleEaten$ = apples$.pipe(
  skip(1),   // 跳过第一个apples值，即初始值
  tap(() => length$.next(POINTS_PER_APPLE))   // 每次apples发出值，都让length$ next出去一分
).subscribe();
```

### scene$流

回头再看一次上面的`snake流图`，基本上所有的流都已完成，而且形成了一整套闭环。只差之后的的整合流`scene$`了。先来看下代码：

```javascript
let scene$ = combineLatest(snake$, apples$,score$, (snake, apples, score) => ({ snake, apples, score }));
```

与 `withLatestFrom` 不同的是，我们不会限制辅助流，我们关心每个输入 `Observable` 产生的新值。最后一个参数还是选择器函数，我们将所有数据组合成一个表示**游戏状态**的对象，并将对象返回。游戏状态包含了 canvas 渲染所需的所有数据。

![](/images/snake/snake05.png)

### 性能维护

无论是游戏，还是 Web 应用，性能都是我们所追求的。性能的意义重大，但就我们的游戏而言，我们希望每秒重绘整个场景 60 次。所以我们需要另外一个定时器去限制渲染频率：

```javascript
// interval 接收以毫秒为单位的时间周期，这也就是为什么我们要用 1000 来除以 FPS
const FPS = 60;
interval(1000 / FPS);
```

问题是 JavaScript 是单线程的。最糟糕的情况是，我们阻止浏览器执行任何操作，导致其锁定。换句话说，浏览器可能无法快速处理所有这些更新。原因是浏览器正在尝试渲染一帧，然后立即被要求渲染下一帧。作为结果，它会抛下当前帧以维持速度。这时候动画就开始看上去有些不流畅了。

幸运的是，我们可以使用 `requestAnimationFrame` 来允许浏览器对任务进行排队，并在最合适的时间执行任务。但是，我们如何在 Observable 管道中使用呢？好消息是包括 `interval()` 在内的众多操作符都接收 `Scheduler` (调度器) 作为最后的参数。总而言之，`Scheduler` 是一种调度将来要执行的任务的机制。

虽然 RxJS 提供了多种调度器，但我们关心的是名为 `animationFrame` 的调度器。此调度器在 `window.requestAnimationFrame`触发时执行任务。

```javascript
const game$ = Observable.interval(1000 / FPS, animationFrame)
```

现在`interval`大概每 16ms 发出一次值，从而保持 FPS 在 60 左右。

### 场景渲染

最后，我们把`game$`和`scene$`组合起来。`game$`作为主要流，每隔`1000/FPS`毫秒时，根据`scene$`数据渲染一次页面，所以使用`withLatestFrom()`组合：

```javascript
const game$ = interval(1000 / FPS, animationFrame)
  .withLatestFrom(scene$, (_, scene) => scene)
  .takeWhile(scene => !isGameOver(scene))  // 蛇身蛇头相碰，则游戏结束
  .subscribe({
    next: (scene) => renderScene(ctx, scene),  // canvas渲染页面
    complete: () => renderGameOver(ctx)   // 渲染游戏结束页面
  });
// isGameOver、renderScene和renderGameOver方法在稍后的超链接中可以找到，这里略去
```

至此，我们就使用`RxJS`完成了整个游戏的核心部分，完全使用响应式编程，没有依赖任何外部状态。

这里是在线试玩的 [demo](https://reactive-snake.stackblitz.io/)。

## 总结&资料引入

在花了两三周时间把`RxJS`的基础概念和操作符都过了一遍之后，急需一个实例demo来将自己所学的`RxJS`知识进行转化，只有真正的操作过，使用过，才能更深入的理解`Observable`和响应式编程的理念。最后在知乎上找到了一篇`RxJS 游戏之贪吃蛇`的专栏文章，自己跟着做了一遍，反复思考其中流的起源、转换和走向。这篇博客基本上是按照[RxJS 游戏之贪吃蛇](https://zhuanlan.zhihu.com/p/35457418)一文中的思路整理的，其中加入了我自己的思考过程，极力推荐阅读原文。以下是本文的参考资料：

[RxJS 游戏之贪吃蛇](https://zhuanlan.zhihu.com/p/35457418)、[RxJS官方api文档](https://rxjs.dev/api)、[30 天精通 RxJS](https://ithelp.ithome.com.tw/users/20103367/ironman/1199)、[RxJS Marbles](https://rxmarbles.com/#from)、[学习 RxJS 操作符](https://rxjs-cn.github.io/learn-rxjs-operators/about/)、[贪吃蛇源码](https://github.com/thoughtram/reactive-snake)

以上的链接也是我最近几周所看所学的来源。接下来会写几篇博客进行知识点的总结，掺杂一些自己的思考，巩固自己所学，加深印象。



