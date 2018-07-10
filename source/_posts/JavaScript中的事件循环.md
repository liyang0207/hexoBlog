---
title: JavaScript中的事件循环
date: 2018-07-08 22:13:47
tags:
 - JavaScript
 - 知识积累
---

作为前端，提起JS，都知道它是一门`单线程`的语言，只能从上到下顺序执行JS任务，而任务又分为`同步任务`和`异步任务`，常见的如`Ajax`请求、`setTimeout`、`setInterval`、点击事件的回调函数等等都是异步任务。当我们的JS逻辑写的越来越复杂，可能会发现一些逻辑的执行顺序跟我们预想的不一样，或者压根就没执行，这时候深入理解一下JS中事件循环的逻辑就显得势在必行了，更不用提现今前端框架如此流行，弄清了事件循环机制，才能对框架的生命周期，数据更新策略等有更深入的理解。
<!-- more -->

## 写在前面
浏览器是`多进程`的（注意不是`线程`），这里面包括Browser进程（主进程）、第三方插件进程、GPU进程和浏览器渲染进程，而渲染进程又称为浏览器内核，这个就跟我们前端有比较大的联系了。当我们在浏览器中每开一个tab页面，就相当于起了一个新的进程，该页面的渲染、js执行、事件的执行都是在这一个进程内。而每个`渲染进程`内可以包含多个`线程`，主要有：`GUI渲染线程`、`JS引擎线程`、`事件触发线程`、`定时器触发线程`、`异步请求线程`等。其中，GUI渲染线程和JS引擎线程是互斥的，两者同时只能有一个在运行，这就是为什么常说JS的执行会卡住页面，影响页面渲染（JS可能会操作dom元素）。说了这些，其实还没涉及到今天的主角，而浏览器的进程机制又是一些更深入的知识点了，今天不展开（也展不开，还没弄明白），下面我们进入今天的主角，JS事件循环。

## 事件循环
这部分我们来搞清楚什么是一个完整的事件循环？每一个循环内的任务执行顺序是怎样的？下一次循环中的任务从哪里来？

首先，JS分为同步任务和异步任务，同步任务在主线程上进行，形成一个`执行栈`，异步任务进入`Event Table`并注册函数，将注册函数（回调函数）放入到`Event Queue`事件队列中，供下一次循环来取用。如下思维图：

![](/images/event-loop.jpg)

当执行栈内的任务执行完毕（执行栈为空），系统就会去`Event Queue`中读取最顶部的异步任务，添加到执行栈中进行执行。JS引擎存在`monitoring process`进程，会持续不断的检查主线程执行栈是否为空，一旦为空，就会去`Event Queue`那里检查是否有等待被调用的函数。 

我们结合一个例子来看：

```javascript
console.log('1');
setTimeout(() => {
   console.log('2'); 
}, 2000);
console.log('3');
```

执行顺序：

1. 整段scripts开始执行，首先同步执行`console.log('1')`；
2. setTimeout作为异步任务，进入`Event Table`，2s后将`console.log('2')`放进事件队列，待执行；
3. 执行`console.log('3')`，接下来看有没有`微任务`（下面马上讲），没有，第一次事件循环结束；
4. 执行栈为空，从`Event Queue`中将`console.log('2')`（宏任务）取过来，放到执行栈内执行，第二次事件循环结束；
5. 执行栈再次为空，`Event Queue`内没有其他任务，等待下一次检查...

`setTimeout`是作为异步任务进入到`Event Table`中的，所以即使说`setTimeout(fn, 0)`也是进入到`Event Table`中，等下一次循环再执行`fn`，不会立即执行。而且这样写的意思是一旦执行栈为空，就将回调`fn`置入主线程，无需等待，但根据HTML的标准，最低是4毫秒，也达不到0ms。

写到这里，我们还明白了为什么有时候明明写的延迟3s执行的`fn`却没有按时执行，那就是因为3s后虽然`fn`已经到了`Event Queue`中，但是执行栈并没有空下来，必须等到执行栈为空，才能将`fn`取出来放到执行栈去执行，这也是为什么进程内要单独开一个`定时器触发线程`来处理这类倒计时事件，这样才能保证延迟时间的准确性。

### 宏任务（macrotask）

宏任务主要包括主代码块，`setTimeout`、`setInterval`等，浏览器为了能够使得JS内部事件循环与DOM任务能够有序的执行，会在一个事件循环结束后，在下一个 task 执行开始前，对页面进行重新渲染 （task->渲染->task->...） ，setTimeout的作用是等待给定的时间后为它的回调产生一个新的宏任务。 

### 微任务（microtask）

微任务主要包括`promise.then()`，`process.nextTick`，`mutation observe的回调`等。微任务会在`这一次事件循环`中的宏任务执行完成后执行，比如对一系列动作做出反馈，或或者是需要异步的执行任务而又不需要分配一个新的 task，这样便可以减小一点性能的开销。只要执行栈中没有其他的js代码正在执行且每个宏任务执行完，微任务队列会立即执行。如果在微任务执行期间微任务队列加入了新的微任务，会将新的微任务加入队列尾部，之后也会被执行。 

简单总结下，宏任务（macrotask）事件都被放到了一个事件队列（Event Queue）中，这个队列由`事件触发线程`维护；微任务（microtask）事件放到了微任务队列（Job Queues）中，等待宏任务（在当前事件循环）执行完成后执行，这个队列由`JS引擎线程`维护。

总结下运行机制：

- 执行一个宏任务（没有的话就去事件队列中取一个任务）
- 执行过程中遇到异步任务（setTimeout,ajax），走`Event Table`-`Event Queue`流程
- 执行过程中遇到微任务，就先将微任务添加到微任务队列中
- 宏任务执行完毕后，立即执行当前循环中微任务队列里所有的微任务（按添加顺序依次执行）
- 当前事件循环执行完毕，开始检查渲染，然后GUI线程接管渲染
- 渲染完毕后，JS引擎线程继续接管，开始下一个宏任务（从事件队列中取）

![](/images/event-loop2.jpg)

来看几个例子：

```javascript
console.log('1');
setTimeout(function() {
    console.log('setTimeout');
})

new Promise(function(resolve) {
    console.log('promise');
}).then(function() {
    console.log('then');
})

console.log('2');
```

分析：

1. 代码块作为宏任务，进入进程栈，执行`console.log('1')`；
2. 遇到`setTimeout`，将其回调函数注册后分发到`Event Queue`（宏任务），等待下一次事件循环取用；
3. 遇到`promise`，注意，这里直接执行`console.log('promise')`，而`.then()`回调被分发到了该次循环的`Job Queue`（微任务），等待执行；
4. 遇到`console.log('2')`，打印；
5. 该次循环宏任务执行完毕，然后在微任务列表发现了`console.log('then')`，执行；
6. 第一次事件循环执行完毕，进程栈为空，开始去`Event Queue`拉取任务，进行第二次事件循环，`console.log('setTimeout')`进入进程栈，执行，第二次事件循环宏任务执行完毕，没有微任务，循环结束。
7. 所以打印顺序为`'1'->'promise'->'2'->'then'->'setTimeout'`；

再来看一个复杂的例子：

```javascript
console.log('1');

setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
})
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})

setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
})
```

分析：

1. 首先整段scripts作为宏任务进入第一次事件循环，执行`console.log('1')`。
2. 遇到`setTimeout`，将其回调函数注册后分发到`Event Queue`（宏任务），等待下一次事件循环取用。
3. 遇到微任务`process.nextTick`，将其放入第一次事件的微任务队列。
4. 遇到`promise`，直接执行`console.log('7')`，`.then()`回调作为微任务被放入微任务队列。
5. 又遇到`setTimeout`，不管，将其回调函数注册后分发到`Event Queue`（宏任务），等待取用，注意这时候事件队列`Event Queue`已有两个宏任务。
6. 第一次事件循环的宏任务执行完毕，开始执行微任务列表，依次执行`console.log('6')`、`console.log('8')`，微任务执行完毕，第一次事件循环结束，所以第一次事件循环执行过程：`'1'->'7'->'6'->'8'`。
7. 进程栈为空，开始从事件队列拉取任务，首先取出来第一个任务，作为宏任务进入第二次事件循环。
8. 先遇到`console.log('2')`，直接执行。
9. 接着遇到微任务`process.nextTick`，将其放入第二次事件的微任务队列。
10. 遇到`promise`，直接执行`console.log('4')`，`.then()`回调作为微任务被放入微任务队列。
11. 第二次循环宏任务执行完毕，开始执行微任务队列任务，依次执行`console.log('3')`、`console.log('5')`，微任务执行完毕，第二次事件循环结束，所以第二次事件循环执行过程：`'2'->'4'->'3'->'5'`。
12. 进程栈为空，继续从事件队列拉取任务，取出下一个任务，作为宏任务进入第三次事件循环。
13. 先遇到`console.log('9')`，直接执行。
14. 接着遇到微任务`process.nextTick`，将其放入第三次事件的微任务队列。
15. 遇到`promise`，直接执行`console.log('11')`，`.then()`回调作为微任务被放入微任务队列。
16. 第三次循环宏任务执行完毕，开始执行微任务队列任务，依次执行`console.log('10')`、`console.log('12')`，微任务执行完毕，第二次事件循环结束，所以第二次事件循环执行过程：`'9'->'11'->'10'->'12'`。
17. 整个事件循环结束。

说到这里，基本上我们对事件循环就有了一个比较清晰的了解，当然更复杂的情况下就需要我们抽丝剥茧一点一点分析，再查阅更详细的资料去理解了，下面有两个比较复杂的demo，是从外文文章里找到的，有兴趣可以看一下，具体地址在文章末尾。

## 进阶Demo
示例1：

```html
<div class="outer">
  <div class="inner"></div>
</div>
```

```javascript
// Let's get hold of those elements
var outer = document.querySelector('.outer');
var inner = document.querySelector('.inner');

// Let's listen for attribute changes on the
// outer element
new MutationObserver(function() {
  console.log('mutate');
}).observe(outer, {
  attributes: true
});

// Here's a click listener…
function onClick() {
  console.log('click');

  setTimeout(function() {
    console.log('timeout');
  }, 0);

  Promise.resolve().then(function() {
    console.log('promise');
  });

  outer.setAttribute('data-random', Math.random());
}

// …which we'll attach to both elements
inner.addEventListener('click', onClick);
outer.addEventListener('click', onClick);
```

当点击内层`inner`div块时：

1. 因为冒泡，会将outer的点击事件放入到事件队列`Event Queue`，等待取用执行；
2. 进程栈（JS stack）执行`onClick()`函数，直接打印`console.log('click')`；
3. 接下来遇到`setTimeout`，将其回调注册到`Event Queue`，等待取用执行；
4. 遇到微任务`promise.then()`，放入微任务队列；
5. 接下来的`MutationObserver`也是微任务，放入微任务队列；
6. 宏任务执行完，依次执行微任务，本次循环结束，打印顺序：`'click'->'promise'->'mutate'`；
7. 接下来从事件队列中取第二次点击事件任务，再次执行2-6；
8. 此时进程栈为空，事件队列里还有两次的`setTimeout`回调事件，依次执行，完毕；
9. 打印顺序：`'click'->'promise'->'mutate'->'click'->'promise'->'mutate'->'timeout'->'timeout'`；

示例2：

```html
<div class="outer">
  <div class="inner"></div>
</div>
```

```javascript
// Let's get hold of those elements
var outer = document.querySelector('.outer');
var inner = document.querySelector('.inner');

// Let's listen for attribute changes on the
// outer element
new MutationObserver(function() {
  console.log('mutate');
}).observe(outer, {
  attributes: true
});

// Here's a click listener…
function onClick() {
  console.log('click');

  setTimeout(function() {
    console.log('timeout');
  }, 0);

  Promise.resolve().then(function() {
    console.log('promise');
  });

  outer.setAttribute('data-random', Math.random());
}

// …which we'll attach to both elements
inner.addEventListener('click', onClick);
outer.addEventListener('click', onClick);

inner.click();
```

示例2跟示例1基本一样，只有一点不同：示例2最后多了一行`inner.click()`，通过脚本来执行点击事件。这个就不分析了，可以去看[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/?utm_source=html5weekly)，里面有详细的步骤动画，一步步分析。

参考链接：[这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)，[从浏览器多进程到JS单线程，JS运行机制最全面的一次梳理](https://juejin.im/post/5a6547d0f265da3e283a1df7)，[译文：JS事件循环机制（event loop）之宏任务、微任务](https://segmentfault.com/a/1190000014940904)。本文图片和例子很多都来自于此，多谢各位牛人的分享，受益匪浅。