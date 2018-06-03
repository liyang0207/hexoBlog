---
title: 《ES6总结之解构赋值》
date: 2018-03-25 21:56:52
tags:
 - ES6
 - JavaScript
---

ES6中新增了变量的`解构`赋值语法，在一些新的框架内，我们可能很容易就看到解构的应用，语义简洁明了，方便我们将数组或者对象属性进行结构化的赋值，以前看到这样的代码，总有点云里雾里的感觉，这两天抽了点时间，将这个小知识点做了一个总结，以后会尽量尝试使用`解构`语法来书写代码。
<!-- more -->

# 解构

## spread/rest运算符
讲`解构`之前，我们先来看一个ES6中新增的运算符`...`，称为“spread/rest”运算符，主要根据使用的场景来区分叫法（展开/收集）。
### spread展开
`...`用到数组前面时，它会把这个变量展开为各个独立的值：
```javascript
function foo(a, b, c){
  console.log(a, b, c);
}
foo(...[1,2,3]);   // 1 2 3
```
`...[1,2,3]`将数组展开，带入函数参数中。`...`也可以在其他上下文中展开一个值：
```javascript
var a = [2, 3, 4];
var b = [1, ...a, 5];
console.log(b);   // [1, 2, 3, 4, 5]
```
这里，`...`代替了`concat(..)`，如：`[1].concat( a, [5] )`。

### rest收集
`...`也可以把一系列值收集到一起成为一个数组：
```javascript
function foo(x, y, ...z){
  console.log(x, y, z);
}
foo(1, 2, 3, 4, 5);  // 1 2 [3, 4, 5]
```
`...z`就是将剩下的参数收集到一起组成一个名为z的数组，但如果没有名称参数的话，`...`会收集所有参数：
```javascript
function foo(...args){
  console.log( args );
}
foo(1, 2, 3, 4, 5);   // [1,2,3,4,5]
```
`...args`通常称为“rest参数”，是`arguments`数组的替代品。
函数`foo(..)`申明中`...args`收集参数，`console.log(..)`中`...args`将其展开，很好的展示了运算符`...`对称而又相反的用法。

## 解构的默认值
解构允许默认值，而且默认值不仅可以是简单值，还可以是任意合法的表达式，甚至可以是函数调用。
```javascript
var [a=1,b] = [undefined];
console.log(a);  // 1

var [a=null] = [null];
console.log(a);  // null
```

默认表达式是惰性求值的，只有当参数省略或者为undefined的时候才会运行。注意，传值为null会仍会被识别为null。
```javascript
function bar(val){
  console.log('bar called');
  return y + val;
}
function foo(x = y+3, z = bar(x)){
  console.log(x, z);
}
var y = 5;
foo();  //  "bar called"  8 13

foo(10);  // "bar called"  10 15

y = 6;
foo(undefined, 10);   // 9 10

foo(null, 1);  // null 1
```
另一个需要注意的地方是，函数声明中的形参首先会匹配它们自己的作用域，也就是括号内，然后才会搜索外层作用域：
```javascript
var w = 1, z = 2;
function foo( x = w+1, y = x+1, z = z + 1 ){
  console.log(x,y,z);
}
foo();  // ReferenceError
```
`w`先在括号内寻找没有，接着在外层找到1，`x+1`时`x`已初始化，故`y`此时为3，到`z+1`时，`z`发现自己还没有初始化，所以直接报错，永远不会试图去外层作用域寻找`z`。

## 数组的解构赋值
先通过一些例子引入：
```javascript
var [a, [ b, c ]] = [1, [ 2, 3 ]];
console.log( a, b, c);  // 1 2 3

var [ , , a] = [1, 2, 3];
console.log(a);  // 3

var [a, ...b] = [1,2,3,4,5];
console.log(a, b);  // 1 [2,3,4,5]

var [a, b, ...c] = [1];
console.log(a, b, c);  // 1 undefined []
```
这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值，如果解构不成功，变量的值就等于undefined。
如果等号左边的模式，只匹配一部分的等号右边的数组，称之为“不完全解构”，这种情况下，解构依然可以成功。
```javascript
var [a, b] = [1,2,3,4,5];
console.log(a,b);  // 1 2
```
由于数组本质是特殊的对象，因此可以对数组进行对象属性的解构。
```javascript
let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;
first // 1
last // 3
```

## 对象的解构赋值
对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。
```javascript
var { bar, foo } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

var { baz } = { foo: "aaa", bar: "bbb" };
baz // undefined
```
之所以`{bar: bar, foo: foo}`可以简写为`{ bar, foo }`是因为变量名与属性名一致，若不一致，则不能简写：
```javascript
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"

let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world
```
另一个需要重点注意的是，`{bar:bar, foo:foo}`的简写省略的是前面的`bar:`和`foo:`，与对象的`target: source`正好相反，解构赋值的结构是`source: target`，后面才是要赋值的目标变量。一定要清楚简写省略的是什么，不要搞晕。

与数组一样，解构也可以用于嵌套结构的对象。
```javascript
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p: [x, { y }] } = obj;
x // "Hello"
y // "World"
```
注意，这时p是模式，不是变量，因此不会被赋值。如果p也要作为变量赋值，可以写成下面这样。
```javascript
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p, p: [x, { y }] } = obj;
x // "Hello"
y // "World"
p // ["Hello", {y: "World"}]
```
下面是对象的嵌套赋值：
```javascript
let obj = {};
let arr = [];

({ foo: obj.prop, bar: arr[0] } = { foo: 123, bar: true });   //注意用括号括起来

obj // {prop:123}
arr // [true]
```
注意，对于对象解构赋值来说，如果省略了`var/let/const`声明符，就必须把整个赋值表达式用`()`括起来，不然`{...}`部分就会被当作一个块语句，进而报错。

## 字符串的解构赋值
字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。
```javascript
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```
类似数组的对象都有一个length属性，因此还可以对这个属性解构赋值。
```javascript
let {length : len} = 'hello';
len // 5
```

## 重复赋值
对象解构形式允许多次列出同一个源属性：
```javascript
var { a: X, a: Y } = { a: 1 }
X;  // 1
Y;  // 1
```
也可以解构子对象/数组属性，同时捕获子对象/类的值本身：
```javascript
var { a:{x: X, x: Y}, a} = { a: {x: 1}};
X;  // 1
Y;  // 1
a;  // { x: 1}

({ a: X, a: Y, a: [Z]} = { a: [1]});
X.push(2);
Y[0] = 10;

X;  // [10, 2]
Y;  // [10, 2]
Z;  // 1
```
X，Y都指向同一数组，引用复制，故为[10, 2];

## 函数参数的解构赋值
函数的参数也可以使用解构赋值。
```javascript
function add([x, y]){
  return x + y;
}

add([1, 2]); // 3
```
上面代码中，函数add的参数表面上是一个数组，但在传入参数的那一刻，数组参数就被解构成变量x和y。对于函数内部的代码来说，它们能感受到的参数就是x和y。
函数参数的解构也可以使用默认值。
```javascript
function foo({ x = 10 } = {}, { y } = { y: 10 }){
  console.log( x, y );
}
foo();   // 10 10
foo(undefined, undefined);  // 10 10
foo({}, undefined);   // 10 10
foo({}, {});  // 10 undefined
foo(undefined, {});  // 10 undefined
foo( {x: 2}, {y: 3});  // 2 3
```
`undefined`会触发函数参数的默认值。

## 用途
解构赋值用途很多

### 交换变量的值
```javascript
let x = 1;
let y = 2;

[x, y] = [y, x];
```
上面代码交换变量x和y的值，这样的写法不仅简洁，而且易读，语义非常清晰。

### 从函数返回多个值
函数只能返回一个值，如果要返回多个值，只能将它们放在数组或对象里返回。有了解构赋值，取出这些值就非常方便。
```javascript
// 返回一个数组
function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象
function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

### 函数参数的定义
解构赋值可以方便地将一组参数与变量名对应起来。
```javascript
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

### 提取JSON数据
解构赋值对提取 JSON 对象中的数据，尤其有用。
```javascript
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```
上面代码可以快速提取 JSON 数据的值。

### 函数参数的默认值
```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
}) {
  // ... do stuff
};
```
指定参数的默认值，就避免了在函数体内部再写`var foo = config.foo || 'default foo';`这样的语句。


