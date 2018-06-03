---
title: 《ES6总结之Generator与Iterator下》
date: 2018-03-17 20:00:47
tags:
 - ES6
 - JavaScript
---

上篇博客归纳总结了一些关于`Generator`生成器的知识点，这些天又查阅了许多资料和书籍，包括《你不知道的JavaScript》及阮一峰老师写的《ECMAScript 6 入门》，着重理解和思考了关于`Iterator`迭代器的知识点，总结于此，方便以后查阅。
<!-- more -->

# Iterator 迭代器
迭代器（iterator）是一个结构化的模式，用于从“源”以一次一个的方式提取数据，是一种有序的、连续的、基于拉取的用于“消耗数据”的组织方式。通俗的来讲，`Iterator`为不同的数据结构（例如数组，字符串等）提供了一个统一的访问机制，可以对数据结构进行遍历操作（只要该数据结构部署了Iterator接口）。
其作用主要有3个：
1. 为各种数据结构，提供一个统一的、简便的访问接口；
2. 使得数据结构的成员能够按某种次序排列；
3. ES6 创造了一种新的遍历命令`for..of`循环，`Iterator` 接口主要供`for..of`消费。

## 接口
首先要明确一点，`Iterable`和`Iterator`两个概念是不同的，`Iterable`是一种类型，意为“可迭代的”，`Array`、`String`、`Map`和`Set`等类型都原生实现了`@@iterator`方法，也就是说这些类型可以迭代，相比于`Object`就必须我们自己去实现迭代的方法（后面再讲）。

### 接口要求
《你不知道的JavaScript》一书中简单介绍了一下`Iterator`的接口，标准是否调整我并没有查到相关资料，但是列在这里可以更清晰的理解`Iterator`的接口，其要求如下：
```javascript
Iterator [required]
  next() {method}: 取得下一个IteratorResult
```
有些迭代器还扩展支持两个可选成员：
```javascript
Iterator [optional]
  return() {method}: 停止迭代器并返回IteratorResult
  throw() {method}: 报错并返回IteratorResult
```
IteratorResult接口的指定如下：
```javascript
IteratorResult
  value() {property}: 当前迭代值或者最终返回值
  done() {property}: 布尔值，指示完成状态
```
还有一个`Iterable`接口，用来表述必需能够提供生成器的对象：
```javascript
Iterable
  @@iterator() {method}: 产生一个 Iterator
```

`IteratorResult`接口指定了从任何迭代器操作返回的值必须是下面这种形式的对象：
```javascript
{ value: .. , done: true/false }
```
内置的迭代器总是返回这种形式的值，自定义迭代器可以在结果对象上增加额外的元数据（如数据的来源，获取数据的时间长度，缓存过期时长，下次请求的适当频率等...）

### 接口目的
`Iterator`接口的目的，就是为所有数据结构，提供了一种统一的访问机制，即`for..of`循环（详见下文）。当使用`for..of`循环遍历某种数据结构时，该循环会自动去寻找`Iterator`接口。一种数据结构只要部署了`Iterator`接口，我们就称这种数据结构是“可遍历的”（iterable）。ES6规定，默认的`Iterator`接口部署在数据结构的`Symbol.iterator`属性上，只要一个数据接口有`Symbol.iterator`属性，就可以认为该数据接口“可遍历”，`Symbol.iterator`属性本身是一个函数，执行这个函数，就返回一个遍历器。属性名`Symbol.iterator`需要放在方括号内：
```javascript
var obj = {
  [Symbol.iterator]: function(){
    return {
      next() { 
        return {
          value: 1,
          done: false
        }
      }   
    }
  }
}
```
上面代码中，对象obj是可遍历的（iterable），因为具有`Symbol.iterator`属性。执行这个属性，会返回一个遍历器对象。该对象的根本特征就是具有`next`方法。每次调用`next`方法，都会返回一个代表当前成员的信息对象，具有`value`和`done`两个属性。

### 数据结构
ES6中一些数据结构原生具备了`Iterator`接口，直接调用这个接口，就会返回一个遍历器对象，这些数据结构有：
* Array
* Map
* Set
* String
* TypedArray
* 函数的arguments对象
* NodeList对象

例如数组的`Symbol.iterator`属性:
```javascript
var arr = [1,2,3,4];
var it = arr[Symbol.iterator]();
it.next();   // {value: 1, done: false}
it.next();   // {value: 2, done: false}
it.next();   // {value: 3, done: false}
it.next();   // {value: 4, done: false}
it.next();   // {value: undefined, done: true}
```
变量arr是一个数组，原生就具有遍历器接口，部署在arr的`Symbol.iterator`属性上面。所以，调用这个属性，就得到遍历器对象。

对于原生部署`Iterator`接口的数据结构，不用自己写遍历器生成函数，`for..of`循环会自动遍历它们。除此之外，其他数据结构（主要是对象）的`Iterator`接口，都需要自己在`Symbol.iterator`属性上面部署，这样才会被`for..of`循环遍历。
对象（Object）之所以没有默认部署`Iterator`接口，是因为对象的哪个属性先遍历，哪个属性后遍历是不确定的，需要开发者手动指定。一个对象如果要具备可被`for..of`循环调用的`Iterator` 接口，就必须在`Symbol.iterator`的属性上部署遍历器生成方法（原型链上的对象具有该方法也可）。
```javascript
//实现一个斐波那契序列
var Fib = {
  [Symbol.iterator]() {
    var n1 = 1, n2 = 1;
    return {
      //使迭代器成为iterable
      [Symbol.itertor]() { return this; },
      next() {
        var current = n2;
        n2 = n1;
        n1 = n1 + current;
        return { value: current, done: false };
      },

      return(v) {
        console.log("Fibonacci sequence abandoned.");
        return { value: v, done: true };
      }
    }
  }
};

for( var v of Fib ){
  console.log(v);
  if(v > 50) break;
}
// 1 1 2 3 5 8 13 21 34 55
// Fibonacci sequence abandoned.
```
调用`Fib[Symbol.iterator]()`方法时，会返回带有 `next()`和 `return()`方法的迭代器对象。

一个需要注意的地方是类数组对象调用`Symbol.iterator`的情况：
```javascript
let iterable = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
};
for( let item of iterable ){
  console.log(item);   // 'a' 'b' 'c'
}
```

## 使用场合
### 解构赋值
对数组和 Set 结构进行解构赋值时，会默认调用`Symbol.iterator`方法。结构赋值会有专门的一篇博客来总结。
```javascript
let set = new Set().add('a').add('b').add('c');

let [x,y] = set;
// x='a'; y='b'

let [first, ...rest] = set;
// first='a'; rest=['b','c'];
```

### 扩展预算符
扩展运算符（...）也会调用默认的`Iterator`接口。
```javascript
// 例一
var str = 'hello';
[...str] //  ['h','e','l','l','o']

// 例二
let arr = ['b', 'c'];
['a', ...arr, 'd']
// ['a', 'b', 'c', 'd']
```
实际上，这提供了一种简便机制，可以将任何部署了`Iterator`接口的数据结构，转为数组。也就是说，只要某个数据结构部署了`Iterator`接口，就可以对它使用扩展运算符，将其转为数组。

### yield*
如果`yield*`后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口。
```javascript
let generator = function* () {
  yield 1;
  yield* [2,3,4];
  yield 5;
};

var iterator = generator();

iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }
```

### 其他场合
由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。下面是一些例子。
* for...of
* Array.from()
* Map(), Set(), WeakMap(), WeakSet()（比如new Map([['a',1],['b',2]])）
* Promise.all()
* Promise.race()

## 可选的return(..)和throw(..)
`return`方法的使用场合是，如果`for..of`循环提前退出（通常是因为出错，或者有`break`语句或`continue`语句），就会调用`return`方法。如果一个对象在完成遍历前，需要清理或释放资源，就可以部署`return`方法。
```javascript
function readLinesSync(file) {
  return {
    [Symbol.iterator]() {
      return {
        next() {
          return { done: false };
        },
        return() {
          file.close();
          return { done: true };
        }
      };
    },
  };
}
```
上面代码中，函数`readLinesSync`接受一个文件对象作为参数，返回一个遍历器对象，其中除了`next`方法，还部署了`return`方法。下面的三种情况，都会触发执行`return`方法。
```javascript
// 情况一
for (let line of readLinesSync(fileName)) {
  console.log(line);
  break;
}

// 情况二
for (let line of readLinesSync(fileName)) {
  console.log(line);
  continue;
}

// 情况三
for (let line of readLinesSync(fileName)) {
  console.log(line);
  throw new Error();
}
```
上面代码中，情况一输出文件的第一行以后，就会执行`return`方法，关闭这个文件；情况二输出所有行以后，执行`return`方法，关闭该文件；情况三会在执行`return`方法关闭文件之后，再抛出错误。
注意，`return`方法必须返回一个对象，这是`Generator`规格决定的。

`throw(..)`方法用于向迭代器报告一个异常/错误，主要是配合`Generator`函数使用。

## 迭代器循环
ES6的`for..of`循环直接消耗一个符合规范的iterable。如果一个迭代器iterator也是一个iterable，那么它可以直接用于`for..of`循环，可以通过为迭代器提供一个`Symbol.iterator`方法简单的返回这个迭代器本身使它成为iterable:
```javascript
var it = {
  //使迭代器成为iterable
  [Symbol.iterator]() { return this; },
  next() { .. },
  ..
}
it[Symbol.iterator]() === it;  // true
```
现在可以使用`for..of`循环消耗这个it迭代器：
```javascript
for(var v of it){
  console.log(v);
}
```

参照下面的例子和上面的`Fib`对象对比来看：
```javascript
class RangeIterator {
  constructor(start, stop){
    this.value = start;
    this.stop = stop;
  }

  [Symbol.iterator]() { return this; }

  next() {
    var value = this.value;
    if(value < this.stop){
      this.value ++;
      return { value: value, done: false };
    }
    return { value: undefined, done: true };
  }
}

function range(start, stop){
  //隐藏细节
  return new RangeIterator(start, stop);
}
for( var value of range(0, 3) ){
  console.log(value);   // 0, 1, 2
}
```

## for..of循环
一个数据结构只要部署了`Symbol.iterator`属性，就被视为具有`iterator`接口，就可以用`for..of`循环遍历它的成员。也就是说，`for..of`循环内部调用的是数据结构的`Symbol.iterator`方法。
`for..of`循环可以使用的范围包括`数组`、`Set` 和`Map`结构、某些`类似数组的对象`（比如`arguments对象`、DOM `NodeList对象`）、后文的 `Generator对象`，以及`字符串`。

### 数组
`JavaScript`原有的`for..in`循环，只能获得对象的键名，不能直接获取键值。ES6提供`for..of`循环，允许遍历获得键值。
```javascript
var arr = ['a', 'b', 'c', 'd'];

for (let k in arr) {
  console.log(k); // 0 1 2 3
}

for (let v of arr) {
  console.log(v); // a b c d
}
```
`for..of`循环调用遍历器接口，数组的遍历器接口只返回具有`数字索引`的属性。这一点跟`for..in`循环也不一样。
```javascript
let arr = [3, 5, 7];
arr.foo = 'hello';

for (let k in arr) {
  console.log(k); // "0", "1", "2", "foo"
}

for (let v of arr) {
  console.log(v); //  "3", "5", "7"
}
```
上面代码中，`for..of`循环不会返回数组arr的foo属性。

### 类似数组的对象
类似数组的对象包括好几类。下面是`for..of`循环用于`字符串`、DOM `NodeList对象`、`arguments对象`的例子。
```javascript
//字符串
let str = 'hello';
for (let s of str){
  console.log(s);  // h e l l o
}

//DOM NodeList对象
let paras = document.querySelectorAll("p");
for(let p of paras){
  p.classList.add('test');
}

// arguments对象
function printArgs() {
  for (let x of arguments) {
    console.log(x);
  }
}
printArgs('a', 'b');
// 'a'
// 'b'
```
对于字符串来说，`for..of`循环还有一个特点，就是会正确识别 32 位 UTF-16 字符。
```javascript
for (let x of 'a\uD83D\uDC0A') {
  console.log(x);
}
// 'a'
// '\uD83D\uDC0A'
```

并不是所有类似数组的对象都具有`Iterator`接口，一个简便的解决方法，就是使用`Array.from`方法将其转为数组。
```javascript
let arrayLike = { length: 2, 0: 'a', 1: 'b' };

// 报错
for (let x of arrayLike) {
  console.log(x);
}

// 正确
for (let x of Array.from(arrayLike)) {
  console.log(x);
}
```

### 对象
对于普通的对象，`for..of`结构不能直接使用，会报错，必须部署了`Iterator`接口后才能使用。但是，这样情况下，`for..in`循环依然可以用来遍历键名。
```javascript
let es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
};

for (let e in es6) {
  console.log(e);
}
// edition
// committee
// standard

for (let e of es6) {
  console.log(e);
}
// Uncaught TypeError: es6 is not iterable
```

一种解决方法是，使用`Object.keys`方法将对象的键名生成一个数组，然后遍历这个数组。
```javascript
for (var key of Object.keys(someObject)) {
  console.log(key + ': ' + someObject[key]);
}
```

另一个方法是使用`Generator`函数将对象重新包装一下。
```javascript
function* entries(obj) {
  for (let key of Object.keys(obj)) {
    yield [key, obj[key]];
  }
}

for (let [key, value] of entries(obj)) {
  console.log(key, '->', value);
}
// a -> 1
// b -> 2
// c -> 3
```

### 与其他遍历语法的比较
以数组为例，`JavaScript`提供多种遍历语法。最原始的写法就是for循环:
```javascript
for (var index = 0; index < myArray.length; index++) {
  console.log(myArray[index]);
}
```
这种写法比较麻烦，因此数组提供内置的`forEach`方法。
```javascript
myArray.forEach(function (value) {
  console.log(value);
});
```
这种写法的问题在于，无法中途跳出`forEach`循环，`break`命令或`return`命令都不能奏效。

`for..in`循环可以遍历数组的键名。
```javascript
for (var index in myArray) {
  console.log(myArray[index]);
}
```
`for..in`循环有几个缺点：
* 数组的键名是数字，但是for...in循环是以字符串作为键名“0”、“1”、“2”等等；
* `for..in`循环不仅遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键；
* 某些情况下，`for..in`循环会以任意顺序遍历键名。

总之，`for..in`循环主要是为遍历对象而设计的，不适用于遍历数组。
而`for..of`循环相比上面几种做法，有一些显著的优点:
* 有着同`for..in`一样的简洁语法，但是没有`for..in`那些缺点。
* 不同于`forEach`方法，它可以与`break`、`continue`和`return`配合使用，如上面的`Fib`对象
* 提供了遍历所有数据结构的统一操作接口。
