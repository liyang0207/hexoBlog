---
title: JS随写：类型识别
date: 2017-09-15 22:06:12
tags:
 - JavaScript
 - 随写
---
javascript是个弱类型的语言，工作中难免会遇到对数据的类型判断的问题（不过一般都是跟php确认一下，让他们返回正确的数据类型）。想了想还是把类型识别的常用方法总结一下，方便以后查阅。

### typeof
先说结论：
 * 可以识别基本数据类型（Null除外），即Undefined,Boolean,Number,String
 * 不能识别具体的对象类型（Function除外）

```javascript
typeof "tommmy"; //"string"
typeof 123;  //"number"
typeof true;  //"boolean"
typeof undefined; //"undefined"
typeof null;  //"object"
typeof {name:"tommy"};  //"object"

typeof function(){};  //"function"
typeof [];  //"object"
typeof /\d/;  //"object"
typeof new Date;  //"object"
//自定义对象
function Person(){};
typeof new Person;  //  "object"
```

### instanceof
先说结论：
 * 可以判断内置对象类型
 * 不能判断基本数据类型
 * 可以判断自定义对象类型

```javascript
['a','b'] instanceof Array;  //true
/\d/ instanceof RegExp;  //true
{a:1} instanceof Object;  //true
new Date instanceof Date;  //true

null instanceof Object;  //false
'abc' instanceof String;  //false
123 instanceof Number;  //false

function man(){age:20};
function person(){height:170};
man.prototype = new person();
man.prototype.constructor = man;
var xiaoming = new man();
xiaoming instanceof man;  //true
xiaoming instanceof person;  //true
```

### Object.prototype.toString.call
先说结论：
 * 可以识别基本数据类型及内置的对象类型
 * 不能识别自定义对象类型

```javascript
Object.prototype.toString.call("abc");  //[object String];
Object.prototype.toString.call(123);  //[object Number];
Object.prototype.toString.call(null);  //[object Null];

//为方便测试，来一个测试方法
function type(obj){
    return (Object.prototype.toString.call(obj).slice(8,-1));
}
type(1);  //"Number"
type("abc");  //"String"
type(true);  //"Boolean"
type(null);  //"Null"
type(undefined);  //"Undefined"
type({});  //"Object"
type([]);  //"Array"
type(new Date);  //"Date"
type(/\d/);  //"RegExp"
type(function(){});  //"Function"

//自定义对象类型不能识别
function abc(x){
    this.x = x;
}
type(new abc(3));  //返回"Object"，应为Function abc?
```

### constructor原型链判断__proto__
先说结论：
 * 可以判断基本数据类型（Undefined和Null除外，因为他俩没构造函数）
 * 可以判断内置对象类型
 * 可以判断自定义对象类型
首先我们可以在浏览器里打印 console.dir(new Number),查看其返回：
![image.png](http://upload-images.jianshu.io/upload_images/6589697-7070071c984a3204.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在其__proto__内看到其constructor为function Number(),故可以此来判断
```javascript
"abc".constructor === String;  //true
(123).constructor === Number;  //true
true.constructor === Boolean;  //true
new Date().constructor === Date;  //true

//方便测试，来个测试方法
function constructorName(obj){
    //排除null和undefined
    return obj && obj.constructor && obj.constructor.toString().match(/function\s*([^(]*)/)[1];
}
constructorName(1);  //"Number"
constructorName("abc");  //"String"
constructorName(true);  //"Boolean"
constructorName(null);  //null
constructorName(undefined);  //undefined
constructorName({});  //"Object"
constructorName([]);  //"Array"
constructorName(new Date);  //"Date"
constructorName(/\d/);  //"RegExp"
constructorName(function(){});  //"Function"

//自定义对象类型可以判断
function abc(x){
    this.x = x;
}
constructorName(new abc11(3));  //"abc"
```

### 简单总结
 * 对于除null之外的其他4个基本数据类型，可以使用`typeof`，方便快捷
 * 如果想判断内置的对象类型，比如常见的Array,Object,正则日期等，直接是使用`instanceof`来判断
 * 对于恼人的null来讲，只好请出`Object.prototype.toString.call(null)`来返回`[object Null]`来做精准判断了
 * 还有用`constructor`来判断？看着就头大，不到万不得已，还是不用了吧