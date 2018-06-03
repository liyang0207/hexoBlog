---
title: 《ES6总结之Generator与Iterator上》
date: 2018-03-11 17:12:29
tags:
 - ES6
 - JavaScript
---

在ES6之前，JavaSript中默认来讲，一个函数一旦开始执行，就会一直运行到函数结束，期间不会有其他代码能够打断它并插入其间。虽然JavaScript是单线程运行，但是如果如果有多个函数“并行”运行，基于运行环境的复杂性及函数异步同步问题，相同的代码在多次运行中可能会出现不同的运行顺序，导致运行结果的不确定。这种函数的不确定性就是通常所说的**竞态条件**，两个函数相互竞争，看谁先运行。好在ES6为我们带来了`Generator`和`Iterator`,也就是所谓的**生成器**和**迭代器（遍历器）**，使我们能够实现函数的启停控制，遍历数据等等新的操作。
<!-- more -->

## Generator生成器
### 总览
`Generator`形式上就是一个普通的函数，不过有两个特征：一是`function`关键字与函数名之间有个`*`号，`*`的位置可以靠左也可以靠右；二是函数体内会使用`yield`表达式，定义不同的内部状态，函数运行到`yield`处会暂停，同时`yield`也实现了生成器内外的消息通讯功能。
```javascript
function *foo(){
  var y = x * (yield);
  return y;
}
var it = foo(6);   //构造一个迭代器it，同时传入参数6
it.next();   //启动生成器

var res = it.next(7);   //将参数7传入*foo内的yield处，继续运行，直到下一个yield或者return
res.value;   // 42
```

1. `it = foo(6)`并没有执行生成器`*foo()`，只是构造了一个**迭代器**，迭代器控制它的执行；
2. 第一次调用迭代器的`next()`方法，启动生成器，一直运行到`yield`处；
3. 再次调用`next()`方法，同时传入参数7至yield处，继续运行；
4. `next()`的调用结果返回一个对象`{value: str, done: boolean}`,其中的value属性保存着从`*foo()`返回的值（如果有的话）；

**有几个需要注意的点：**
1. 消息是双向传递的，`yield..`作为一个表达式可以发出消息响应`next(..)`的调用，`next(..)`也可以向暂停的`yield`表达式发送值。
2. 只有暂停的`yield`才能接收一个通过`next(..)`传递的值，也就是说作为启动的第一个`next()`还没有`yield`来接收它传入的值，所以浏览器会默默丢弃传递给第一个`next()`的任何东西，所以一定不要给迭代器的一个`next()`方法传入值。

### 多个迭代器
每次构建一个迭代器，实际上就隐式的创建了生成器的一个实例，通过这个迭代器来控制的是这个生成器的实例。同一个生成器的实例可以同时运行，也可以彼此交互运行：
```javascript
function *foo(){
  var x = yield 2;
  z++;
  var y = yield( x * z );
  console.log( x, y, z);
}
var z = 1;

var it1 = foo();
var it2 = foo();

var val1 = it1.next().value;      //启动生成器，返回的是第一个yield处的值：2
var val2 = it2.next().value;      //启动生成器，返回的是第一个yield处的值：2

val1 = it1.next( val2 * 10 ).value;    //将 val2 * 10 也就是20传入第一个yield处，将x赋值为20，z++为2，返回x*z 40
val2 = it2.next( val1 * 5 ).value;     //将 val1 * 5 也就是200传入第一个yield处，将x赋值为200，z++为3，返回x*z 600

it1.next( val2 / 2 );    //继续运行，将val2 / 2也就是300传入第二个yield处，将y赋值为300，打印出来 20，300，3
it2.next( val1 / 4 );    //继续运行，将val1 / 2也就是10传入第二个yield处，将y赋值为300，打印出来 200，10，3
```

### yield表达式
`yield`表达式在生成器函数中表示暂停，只有继续调用迭代器的`next()`方法才会遍历下一个内部状态，迭代器的`next()`方法运行逻辑如下：
1. 遇到`yield`表达式，就暂停执行后面的操作，并将紧跟在`yield`后面的那个表达式的值，作为返回的对象的value属性值。
2. 下一次调用`next`方法时，再继续往下执行，直到遇到下一个`yield`表达式。
3. 如果没有再遇到新的`yield`表达式，就一直运行到函数结束，直到return语句为止，并将return语句后面的表达式的值，作为返回的对象的value属性值。
4. 如果该函数没有return语句，则返回的对象的value属性值为`undefined`。

`yield`表达式与`return`语句既有相似之处，也有区别。相似之处在于，都能返回紧跟在语句后面的那个表达式的值。区别在于每次遇到`yield`，函数暂停执行，下一次再从该位置继续向后执行，而`return`语句不具备位置记忆的功能。一个函数里面，只能执行一次（或者说一个）`return`语句，但是可以执行多次（或者说多个）`yield`表达式。正常函数只能返回一个值，因为只能执行一次`return`；**Generator** 函数可以返回一系列的值，因为可以有任意多个`yield`。从另一个角度看，也可以说**Generator** 生成了一系列的值，这也就是它的名称的来历（英语中，`generator` 这个词是“生成器”的意思）。

`Generator`函数如果不使用`yield`表达式，就变成了一个暂缓执行函数：
```javascript
function *foo(){
  console.log('执行');
}
var it = foo();

setTimeout(function(){
  it.next();
}, 2000)
```
如果`foo`为普通函数，那么在为`it`赋值时就会执行，但是`foo`是一个`generator`函数，只有调用it的`next()`方法时，函数才会执行。
注意：`yield`表达式只能用在`generator`函数中，用在其它地方都会报错。

### Generator.prototype.throw()
`Generator` 函数返回的迭代器对象，都有一个`throw`方法，可以在函数体外抛出错误，然后在`Generator`函数体内捕获。
```javascript
var g = function* () {
  try {
    yield;
  } catch (e) {
    console.log('内部捕获', e);
  }
};

var it = g();
it.next();

try {
  it.throw('a');
  it.throw('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 内部捕获 a
// 外部捕获 b
```
上面代码中，迭代器对象it连续抛出两个错误。第一个错误被 `Generator` 函数体内的`catch`语句捕获。it第二次抛出错误，由于 `Generator`函数内部的`catch`语句已经执行过了，不会再捕捉到这个错误了，所以这个错误就被抛出了`Generator`函数体，被函数体外的`catch`语句捕获。
`throw`方法可以接受一个参数，该参数会被`catch`语句接收，建议抛出`Error`对象的实例。
```javascript
var g = function* () {
  try {
    yield;
  } catch (e) {
    console.log(e);
  }
};

var it = g();
it.next();
it.throw(new Error('出错了！'));
// Error: 出错了！(…)
```
注意，不要混淆迭代器对象的`throw`方法和全局的`throw`命令。上面代码的错误，是用迭代器对象的`throw`方法抛出的，而不是用`throw`命令抛出的。后者只能被函数体外的`catch`语句捕获。
```javascript
var g = function* () {
  while (true) {
    try {
      yield;
    } catch (e) {
      if (e != 'a') throw e;
      console.log('内部捕获', e);
    }
  }
};

var it = g();
it.next();

try {
  throw new Error('a');
  throw new Error('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 外部捕获 [Error: a]
```
上面代码之所以只捕获了a，是因为函数体外的`catch`语句块，捕获了抛出的a错误以后，就不会再继续`try`代码块里面剩余的语句了。

如果`Generator`函数内部没有部署`try...catch`代码块，那么`throw`方法抛出的错误，将被外部`try...catch`代码块捕获。
```javascript
var g = function* () {
  while (true) {
    yield;
    console.log('内部捕获', e);
  }
};

var it = g();
it.next();

try {
  it.throw('a');
  it.throw('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 外部捕获 a
```
上面代码中，`Generator`函数g内部没有部署`try...catch`代码块，所以抛出的错误直接被外部`catch`代码块捕获。

如果`Generator`函数内部和外部，都没有部署`try...catch`代码块，那么程序将报错，直接中断执行。
```javascript
var gen = function* gen(){
  yield console.log('hello');
  yield console.log('world');
}

var g = gen();
g.next();
g.throw();
// hello
// Uncaught undefined
```
上面代码中，g.throw抛出错误以后，没有任何try...catch代码块可以捕获这个错误，导致程序报错，中断执行。

throw方法被捕获以后，会附带执行下一条yield表达式。也就是说，会附带执行一次next方法。
```javascript
var gen = function* gen(){
  try {
    yield console.log('a');
  } catch (e) {
    // ...
  }
  yield console.log('b');
  yield console.log('c');
}

var g = gen();
g.next() // a
g.throw() // b
g.next() // c
```
上面代码中，`g.throw`方法被捕获以后，自动执行了一次`next`方法，所以会打印b。另外，也可以看到，只要`Generator` 函数内部部署了`try...catch`代码块，那么遍历器的`throw`方法抛出的错误，不影响下一次遍历。

### Generator.prototype.return()
`Generator`函数返回的遍历器对象，还有一个`return`方法，可以返回给定的值，并且终结遍历`Generator`函数。
```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }
```
上面代码中，遍历器对象g调用`return`方法后，返回值的value属性就是return方法的参数foo。并且，`Generator`函数的遍历就终止了，返回值的done属性为true，以后再调用`next`方法，done属性总是返回true。

如果return方法调用时，不提供参数，则返回值的value属性为undefined。
```javscript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return() // { value: undefined, done: true }
```

如果`Generator`函数内部有`try...finally`代码块，那么return方法会推迟到finally代码块执行完再执行。
```javascript
function* numbers () {
  yield 1;
  try {
    yield 2;
    yield 3;
  } finally {
    yield 4;
    yield 5;
  }
  yield 6;
}
var g = numbers();
g.next() // { value: 1, done: false }
g.next() // { value: 2, done: false }
g.return(7) // { value: 4, done: false }
g.next() // { value: 5, done: false }
g.next() // { value: 7, done: true }
```
上面代码中，调用return方法后，就开始执行finally代码块，然后等到finally代码块执行完，再执行return方法。

### next()、throw()、return() 的共同点
`next()`、`throw()`、`return()`这三个方法本质上是同一件事，可以放在一起理解。它们的作用都是让`Generator` 函数恢复执行，并且使用不同的语句替换`yield`表达式。

`next()`是将`yield`表达式替换成一个值：
```javascript
const g = function* (x, y) {
  let result = yield x + y;
  return result;
};

const gen = g(1, 2);
gen.next();         // Object {value: 3, done: false}

gen.next(1);        // Object {value: 1, done: true}
// 相当于将 let result = yield x + y
// 替换成 let result = 1;
```
上面代码中，第二个`next(1)`方法就相当于将`yield`表达式替换成一个值1。如果next方法没有参数，就相当于替换成undefined。

`throw()`是将yield表达式替换成一个throw语句。
```javascript
gen.throw(new Error('出错了'));           // Uncaught Error: 出错了
// 相当于将 let result = yield x + y
// 替换成 let result = throw(new Error('出错了'));
```

`return()`是将yield表达式替换成一个return语句。
```javascript
gen.return(2);          // Object {value: 2, done: true}
// 相当于将 let result = yield x + y
// 替换成 let result = return 2;
```

未完待续...
