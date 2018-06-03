---
title: 《this探究》
date: 2018-03-31 22:06:15
tags:
 - JavaScript
 - 随写
---

今天来对`this`这个JavaScript中很重要，有时候也会让很多人绕晕的关键字做一个总结。首先我们得明白`this`既不指向函数自身也不指向函数的词法作用域，它实际上是在函数被调用时发生的绑定，指向哪里完全取决于函数在哪里被调用。

<!-- more -->

# this关键字

## 绑定规则
分析`this`的绑定规则时，我们要先找到函数的调用位置，再判断需要应用下面的哪条规则。

### 默认绑定
默认绑定是最常见的一种，分为两种情况：
#### 非严格模式下
在非严格模式，函数的独立调用下，`this`指向全局对象。
```javascript
function foo(){
  console.log(this.a);
}
var a = 2;
foo();    // 2
```
`foo()`是直接使用不带任何修饰的函数引用进行调用的，此时应用`默认绑定`，`foo()`中的`this`指向全局对象。

#### 严格模式下
在严格模式下，不能将`this`指向全局，因此`this`会被绑定到`undefined`。
```javascript
funciton foo(){
  "use strict";

  console.log(this.a);
}
var a = 2;
foo();   // TypeError: this is undefined
```
`this`指向了`undefined`，`this.a`会报错。这里需要注意的一点是，如果`foo()` `运行`在非严格模式下，但在严格模式下`调用`它，则不影响默认绑定，`this`还是会绑定到全局对象。
```javascript
funciton foo(){
  console.log(this.a);
}
var a = 2;
(function (){
  "use strict";
  foo();   // 2
})();
```
不过一般来说，最好不要在代码中混合使用`strict`模式和`非strcit`模式。

### 隐式绑定
如果函数调用的位置有上下文对象，或者说是否被某个对象`拥有`或者`包含`，这个时候隐式绑定规则会把函数调用中的`this`绑定到这个上下文对象。不严谨可以说：谁调用函数，`this`就指向谁。
```javascript
function foo(){
  console.log(this.a);
}
var obj = {
  a: 2,
  foo: foo
}

obj.foo();  // 2
```

对象属性引用链上只有上一层或者说最后一层在调用位置中起作用：
```javascript
function foo(){
  console.log(this.a);
}

var obj2 = {
  a: 42,
  foo: foo
}

var obj1 = {
  a: 2,
  obj2: obj2
}

obj1.obj2.foo();  // 42
```

#### 隐式丢失
一个常见的`this`绑定问题是被`隐式绑定`的函数会丢失绑定对象，这个时候它会应用`默认绑定`，将`this`绑定到全局对象或者`undefined`上。
```javascript
function foo(){
  console.log(this.a);
}
var obj = {
  a: 2,
  foo: foo
}
var bar = obj.foo;   //函数别名

var a = 'oops, global';

bar();   // "oops, global"
```
虽然`bar`是`obj.foo`的一个引用，但实际上，它引用的是函数`foo`本身，因此这个时候`bar()`其实是一个不带修饰符的函数调用，故应用`默认绑定`。
在传入回调函数时，也就是将函数作为参数传入到另一个函数中，也会发生这种情况：
```javascript
function foo(){
  console.log(this.a);
}

function doFoo(fn){
  fn();   // 调用位置
}

var obj = {
  a: 2,
  foo: foo
}

var a = 'oops, global';
doFoo( obj.foo );  //  "oops, global"
```
在定时器中，情况依旧：
```javascript
function foo(){
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo
}

var a = 'oops, global';
setTimeOut( obj.foo, 100 );  //  "oops, global"
```
定时器的实现类似如下代码：
```javascript
function setTimeOut(fn, delay){
  // 等待delay毫秒
  fn();  //调用位置
}
```
隐式丢失的问题可以使用`var that = this`或者ES6中的`箭头函数=>`来修复。

### 显式绑定
如果不想在对象内部包含函数引用，而想在某个对象上强制调用函数，就可以使用`call(..)`和`apply(..)`方法：它们的第一个参数是一个对象，是给`this`准备的，接着在调用函数时，将这个对象绑定到`this`，`this`就指向了这个对象。第二个参数为传入的参数列表，`call()`和`apply()`略有不同：
```javascript
fn.call(thisArg, arg1, arg2, ...);   // .call()接受的是若干参数的列表，用`,`号分隔

fn.apply(thisArg[, arg1[, arg2[, ...]]]);  // .apply()接受的是一个包含若干参数的数组，将参数“抹平”，类似于ES6中的`spread/rest`运算符`...`

fn.bind(thisArg[, arg1[, arg2[, ...]]]);  // .bind()与apply一致，不过它返回的是一个新函数
```

来看一个例子：
```javascript
function foo(){
  console.log(this.a);
}

var obj = {
  a: 2
}

foo.call( obj );  // 2
```
通过`foo.call(..)`，在调用`foo`时强制把它的`this`绑定到了`obj`上。

#### 显式绑定中的硬绑定
硬绑定可以解决绑定丢失问题：
```javascript
function foo(){
  console.log(this.a);
}

var obj = {
  a: 2
}

var bar = function() {
  foo.call( obj );
}
bar();  // 2
setTimeOut( bar, 100 );  // 2

//硬绑定的bar不能再修改它的this
bar.call( window );  // 2
```
ES5提供了内置的方法 `Function.prototype.bind`来实现硬绑定：
```
function foo(something) {
  console.log( this.a, something );
  retrun this.a + something;
}
var obj = {
  a: 2
}

var bar = foo.bind( obj );
var b = bar(3);   // 2 3
console.log(b);   // 5
```
`bind(..)`会返回一个硬编码的新函数，它会把你指定的参数设置为`this`的上下文并调用原始函数。

#### API调用中的“上下文”
第三方库的许多函数，以及JavaScript语言和内置函数，都提供了一个可选的参数，通常被称为“上下文”，其作用和`bind(..)`一样，确保回调函数使用制定的`this`，例如`forEach`方法：
```javascript
arr.forEach( callback( item, index, array), thisArg );
// thisArg可选，当执行回调函数时用作this的值（参考对象）
```

### new绑定
在Javascript中，所谓的构造函数只是一些使用`new`操作符时被调用的函数，并不会属于某个类，也不会实例化一个类，它们只是被`new`调用的普通函数而已。
使用`new`来调用函数，或者说发生构造函数时，会自动执行一下操作：
1. 创建（或者说构造）一个全新的对象。
2. 这个新对象会被执行`[[Prototype]]`链接。
3. 这个新对象会绑定到函数调用的`this`。
4. 如果函数没有返回其他对象，那么`new`表达式中的函数调用会自动返回这个新对象。

```javascript
function foo(a){
  this.a = a;
}
var bar = new foo(2);
console.log( bar.a );  // 2
```

## 优先级
如果某个调用位置应用了多条规则，则需要有一定的顺序来判断绑定优先级：

1. 函数是否在`new`中调用？如果是的话`this`绑定的是创建的对象。
`var bar = new foo()`
2. 函数是否通过`call`、`apply`（显式绑定）或者硬绑定调用？如果是的话，`this`绑定的是指定的对象。  
`var bar = foo.call(obj2)`
3. 某个函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，`this`绑定的是那个上下文对象。
`var bar = obj1.foo()`
4. 如果都不是的话，使用默认绑定，如果在严格模式下，就绑定到`undefined`，否则绑定到全局对象。

## 例外情况
如果把`null`或者`undefined`作为`this`的绑定对象传入`call`、`apply`或者`bind`，这些值在调用时会被忽略，实际应用的是`默认绑定`规则：
```javascript
function foo(){
  console.log(this.a);
}
var a = 2;
foo.call( null );  // 2
```
当使用`apply(..)`来展开一个数组，并当作参数传入一个函数，或者使用`bind()`对参数进行柯里化时，我们会传入`null`来实现：
```javascript
function foo(a, b){
  console.log("a:" + a + ", b:" + b);
}
// 把数组展开成参数
foo.apply(null, [2, 3]);  // a:2, b:3

// 使用bind(..)进行柯里化
var bar = foo.bind(null, 2);
bar(3);  // a:2, b:3
```
但在一些情况下，使用`null`来忽略`this`绑定可能产生一些副作用，比如某个函数确实使用了`this`，那么默认绑定就会把`this`指向全局对象，这将导致不可预计的后果（比如修改全局对象），这个时候我们可能需要一个“真正”的空对象来进行`this`绑定。
在Javascript中创建一个空对象最简单的方法是`Object.create(null)`，它和`{}`很像，但是不会创建`Object.prototype`这个委托，故它比`{}`更空：
```javascript
function foo(a, b){
  console.log("a:" + a + ", b:" + b);
}

var empty = Object.create(null);

foo.apply(empty, [2,3]);  // a:2, b:3

var bar = foo.bind(empty, 2);
bar(3);  // a:2, b:3
```

## 箭头函数=>
箭头函数不使用`this`的四种规则，而是根据外层（函数或者全局）作用域来决定`this`。
```javascript
function foo(){
  return (a) => {
    console.log(this.a);
  }
}

var obj1 = { a: 2 };
var obj2 = { a: 3 };
var bar = foo.call(obj1);
bar.call(obj2);  // 2, 不是3
```
`foo()`内部创建的箭头函数会捕获调用时`foo()`的this。由于`foo()`的`this`绑定到`obj1`，`bar`的`this`也会绑定到`obj1`，箭头函数的绑定无法被修改。

箭头函数最常用于回调函数内，例如事件处理器或者定时器：
```javascript
function foo(){
  setTimeOut( ()=>{
    // this在词法上继承自foo()
    console.log(this.a);
  }, 100);
}
var obj = { a: 2 };
foo.call(obj);  // 2
```
如果在ES5中，则是我们常用的`that`：
```javascript
function foo(){
  var that = this;
  setTimeOut( ()=>{
    // this在词法上继承自foo()
    console.log(that.a);
  }, 100);
}
var obj = { a: 2 };
foo.call(obj);  // 2
```

箭头函数在后面可能会单独写blog另说，到时候再深入讨论。





