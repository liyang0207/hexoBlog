---
title: 《ES6总结之Map和Set》
date: 2018-04-08 21:53:07
tags:
 - ES6
 - JavaScript
---

ES6新增了`Map`、`Set`两种新的数据结构，解决了Javascript中对象和数组结构的一些痛处：比如`Map`类似于对象结构，但是允许非字符串值作为键值，`Set`则类似于数组结构，但是不允许有重复的`value`，我们可以用这个特性来进行去重操作等等。这两种数据结构都有自己的一个变体：`WeakMap`和`WeakSet`，都涉及到`GC（垃圾回收）`机制的工作方式。总体而言，这两种数据结构都方便了我们对数据进行更优雅、更强大的操作，今天来总结一些两者的操作方法以及`GC`方面的相关知识。

# Map
`Map`数据结构类似于对象结构，但是允许非字符串来作为键值，也就是“值-值”的对应方式。

## 操作方法和属性
```javascript
var m = new Map();
var x = { a: 1 }, y = { b: 2 };

m.set( x, "foo" );
m.set( y, "bar" );

m.get(x);  // "foo"
m.get(y);  // "bar"
```
`Map`结构可以通过`set(..)`和`get(..)`方法来进行数据的添加和获取。另外还有`has(..)`、`delete(..)`、`clear()`来对`Map`进行操作：
```javascript
m.size;         // 2  返回map的长度

m.has(x);       // true  是否有x元素
m.delete(x);    //  删除x元素，返回布尔值，是否删除成功
m.has(x);       // false

m.clear();      //清空map
m.size;         // 0
```

如果对`map`的一个键多次赋值，后面的将覆盖前面的：
```javascript
m.set( x, "foo" ).set( x, "baz");
m.get(x);  // "baz"
```
注意，只有对同一对象的引用，`Map`结构才会将其视为同一键值：
```javascript
var m = new Map();
m.set( ['1'], "foo" );
m.get( ['1'] );  // undefined
```
上面代码的`set`和`get`方法，表面是针对同一个键，但实际上这是两个值，内存地址是不一样的，因此`get`方法无法读取该键，返回`undefined`。其实，`Map`的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。这就解决了同名属性碰撞（clash）的问题，我们扩展别人的库的时候，如果使用对象作为键名，就不用担心自己的属性与原作者的属性同名。

如果`Map`的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，`Map`将其视为一个键，比如`0`和`-0`就是一个键，布尔值`true`和`字符串true`则是两个不同的键。另外，`undefined`和`null`也是两个不同的键。虽然`NaN`不严格相等于自身，但`Map`将其视为同一个键。
```javascript
let map = new Map();

map.set(-0, 123);
map.get(+0) // 123

map.set(true, 1);
map.set('true', 2);
map.get(true) // 1

map.set(undefined, 3);
map.set(null, 4);
map.get(undefined) // 3

map.set(NaN, 123);
map.get(NaN) // 123
```

我们还可以在`Map(..)`构造器中手动指定一个项目列表（键/值数组的数组）：
```javascript
var x = { a: 1 }, y = { b: 2 };
var m = new Map([
    [ x, "foo" ],
    [ y, "bar" ]
  ])

m.get(x);  // "foo"
m.get(y);  // "bar"
```

## 遍历方法
`Map`结构原生提供三个遍历器生成函数和一个遍历方法：
* `keys()`：返回键`名`的遍历器。
* `values()`：返回键`值`的遍历器。
* `entries()`：返回所有成员的遍历器。
* `forEach()`：遍历`Map`的所有成员。

注意，`Map`的遍历顺序就是插入顺序，可以使用ES6的`...`扩展操作符将`Map`结构转为数组结构：

```javascript
var x = { a: 1 }, y = { b: 2 };
var m = new Map([
    [ x, "foo" ],
    [ y, "bar" ]
  ])

var vals = [...m.values()];
console.log(vals);  // ["foo", "bar"]

var keys = [...m.keys()];
console.log(keys);  // [{a:1}, {b:2}]

var entries = [...m.entries()];
console.log(entries);  // [ [{a:1}, "foo"], [{b:2}, "bar"]]
entries[0][0];  // {a:1}
entries[0][1];  // "foo"
entries[1][0];  // {b:2}
entries[1][1];  // "bar"
```
`Map`的`forEach`方法，与数组的`forEach`方法类似，也可以实现遍历。
```javascript
map.forEach(function(value, key, map) {
  console.log("Key: %s, Value: %s", key, value);
});
```
结合数组的`map方法`、`filter方法`，可以实现`Map`的遍历和过滤（`Map`本身没有`map`和`filter`方法）。
```javascript
var map0 = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c');

var map1 = new Map(
  [...map0].filter(([k, v]) => k < 3)
);
// 产生 Map 结构 {1 => 'a', 2 => 'b'}

var map2 = new Map(
  [...map0].map(([k, v]) => [k * 2, '_' + v])
    );
// 产生 Map 结构 {2 => '_a', 4 => '_b', 6 => '_c'}
```

# WeakMap
`WeakMap`是`Map`的变体，二者的多数外部特性是一样的，主要有两点区别：
首先，`WeakMap`只接受对象作为键名（`null`除外），不接受其他类型的值作为键名。
```javascript
var m = new WeakMap();
map.set(1, 2)
// TypeError: Invalid value used as weak map key
map.set(Symbol(), 2)
// TypeError: Invalid value used as weak map key
map.set(null, 2)
// TypeError: Invalid value used as weak map key
```
其次，`WeakMap`的键名所指向的对象，不计入垃圾回收机制，而且只弱持有键，而不弱持有值，如果键（这个对象）本身被`GC`，在`WeakMap`中的这个项目也会被移除；返过来，如果值被`GC`，则在`WeakMap`中的这个项目不会受到影响。
```javascript
var m = new WeakMap();
var x = { id: 1 },
    y = { id: 2 },
    z = { id: 3 },
    w = { id: 4 };

m.set( x, y );
x = null;   // 将 变量x GC
m.get(x);   // undefined，项目x: y已经被GC

m.set( z, w );
w = null;   // 将 变量w GC
m.get(z);   // { id: 4 }  未被GC
m;          // { {id: 3}: {id: 4} }  该项依旧存在
```
当我们需要释放某些对象所占用的内存，我们就必须手动删除这些引用，让垃圾回收机制释放内存，同时存在`WeakMap`内对应的项会自动消失，不用手动删除引用。所以如果你要往对象上添加数据，又不想干扰垃圾回收机制，就可以使用 WeakMap。
总之，WeakMap的专用场合就是，它的键所对应的对象，可能会在将来消失。WeakMap结构有助于防止内存泄漏。
另外，`WeakMap`没有`size`属性和`clear()`方法，也不存在遍历操作（`key()`、`values()`、`entries()`）。

# Set
`Set`数据结构类似于数组结构，但是其值唯一，重复值会被忽略。

## 操作方法和属性
`Set`的API和`Map`类似，但没有`get(..)`方法，同时用`add(..)`方法替换了`set(..)`：
```javascript
var s = new Set();
var x = { id: 1 }, y = { id: 2 };
s.add(x);
s.add(y);
s.add(x);

s.size;        // 2
s.delete(y);   // 删除是否成功的布尔值
s.size;        // 1

s.has(x);      // true
s.clear();
s.size;        // 0
```
向`Set`加入值的时候，不会发生类型转换，所以`5`和`"5"`是两个不同的值；`Set`内部判断两个值是否不同，使用的算法叫做`Same-value-zero equality`，它类似于精确相等运算符（`===`），主要的区别是`NaN`等于自身，而精确相等运算符认为`NaN`不等于自身；两个对象也总是不相等的。

```javascript
var s1 = new Set();
var a = NaN, b = NaN;
s1.add(a);
s1.add(b);
s1;            // Set {NaN}

var s2 = new Set();
s2.add({});
s2.size;       // 1
s2.add({});
s2.size;       // 2
```

## 遍历方法
`Set`结构原生提供三个遍历器生成函数和一个遍历方法，与`Map`相同：
* `keys()`：返回键`名`的遍历器。
* `values()`：返回键`值`的遍历器。
* `entries()`：返回所有成员的遍历器。
* `forEach()`：遍历`Set`的所有成员。

`Set`结构没有键名，只有键值（或者说键名和键值是同一个值），而且`Set`的遍历顺序就是插入顺序，ES6的`...`扩展操作符也可以用于`Set`结构：

```javascript
var s = new Set(['a', 'b', 'c']);

var vals = [...s.values()];
console.log(vals);  // ['a', 'b', 'c']

var keys = [...s.keys()];
console.log(keys);  // ['a', 'b', 'c']

var entries = [...s.entries()];
console.log(entries);  // [ [ 'a', 'a' ], [ 'b', 'b' ], [ 'c', 'c' ] ]
entries[0][0];  // 'a'
entries[0][1];  // 'a'
entries[1][0];  // 'b'
entries[1][1];  // 'b'
entries[2][0];  // 'c'
entries[2][1];  // 'c'
```
`Set`结构的实例与数组一样，也拥有`forEach`方法，用于对每个成员执行某种操作，没有返回值。
```javascript
var s = new Set([1,2,3]);
s.forEach((value, key) => console.log(key + ' : ' + value));
```
同样，数组的`map`和`filter`方法也可以间接用于`Set`了：
```javascript
var set = new Set([1, 2, 3]);
set = new Set([...set].map(x => x * 2));
// 返回Set结构：{2, 4, 6}

var set = new Set([1, 2, 3, 4, 5]);
set = new Set([...set].filter(x => (x % 2) == 0));
// 返回Set结构：{2, 4}
```
一个技巧，使用`Set`可以很容易地实现并集（Union）、交集（Intersect）和差集
```javascript
var s1 = new Set([1, 2, 3]);
var s2 = new Set([4, 3, 2]);

// 并集
let union = new Set([...s1, ...s2]);
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...s1].filter(x => s2.has(x)));
// set {2, 3}

// 差集
let difference = new Set([...s1].filter(x => !s2.has(x)));
// Set {1}
```

# WeakSet
`WeakSet`是`Set`的变体，也是不重复的值的集合，主要有两点区别：
首先，`WeakSet`的值也必须是对象，不能是其他类型的值；
```javascript
var ss = new WeakSet();
ss.add(1);
// TypeError: Invalid value used as weak set key
ss.add(Symbol());
// TypeError: Invalid value used as weak set key
ss.add(null);
// TypeError: Invalid value used as weak set key
```
其次，`WeakSet`对于其内的对象，也是弱持有的，只要对象在外部消失，它在`WeakSet`里面的引用就会自动消失，由于这个特点，`WeakSet`的成员是不适合引用的，因为它会随时消失。另外，由于`WeakSet`内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一样的，而垃圾回收机制何时运行是不可预测的，因此ES6规定`WeakSet`不可遍历。
```javascript
var a = { id: 1 }, b = { id: 2 };
var s = new WeakSet();
s.add(a);
s.add(b);

a = null;      // a可以被GC
b = null;      // b可以被GC
//注意，浏览器对于GC有一定的机制，此时访问s，可能内部还有值
```
还有一点注意，`WeakSet`只有`has()`、`add()`和`delete()`这3个操作方法，没有`size`属性和`clear()`方法。`WeakSet`也不能遍历，因为成员都是弱引用，随时可能消失，遍历机制无法保证成员的存在，很可能刚刚遍历结束，成员就取不到了。`WeakSet` 的一个用处，是储存`DOM`节点，而不用担心这些节点从文档移除时，会引发内存泄漏。

