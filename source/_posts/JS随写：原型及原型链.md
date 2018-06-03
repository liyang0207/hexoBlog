---
title: JS随写：原型及原型链
date: 2017-06-25 17:10:33
tags:
 - JavaScript
 - 随写
---

原型链这个话题算是老生常谈了，随意一搜，网上大把文章来论述这个话题。

<!-- more -->

### 知识准备
先给个经典的person构造函数：

```javascript
function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function(){
        console.log(this.name);
    };
}

var person1 = new Person("liyang", 26, "programmer");
var person2 = new Person("xiaoming", 24, "designer");
```

在js中，万物皆对象，原型`Person.prototype`也是对象。

#### 理解构造函数、实例和原型的概念。

* 构造函数: 构造函数其实就是函数，但任何函数如果使用 `new` 操作符来调用，那它就可以作为构造函数；而任何函数，如果不通过 `new` 操作符来调用，那它跟普通函数也不会有什么两样。按照惯例，构造函数始终都应该以一个大写字母开头，而非构造函数则应该使用一个小写字母开头。
```javascript
// 当作构造函数使用
var person = new Person("liyang", 26, "programmer");
person.sayName(); // "liyang"

// 作为普通函数调用
Person("xiaoming", 24, "designer"); // 添加到 window
window.sayName(); // "xiaoming"

// 在另一个对象的作用域中调用
var o = new Object();
Person.call(o, "Tommy", 3, "Baby");
o.sayName(); // "Tommy"
```

* 实例：使用 `new` 操作符来创建Person的新实例 `person1` 和 `person2` 。
```javascript
var person1 = new Person("liyang", 26, "programmer");
var person2 = new Person("xiaoming", 24, "designer");
```
* 原型: 简单来说， `Person.prototype` 就是Person原型。注意，原型也是对象，且所有函数的默认原型都是Object的实例，所以 `Person.prototype` 同时也是Object的一个实例。

### 理解原型对象
函数一创建，就会按照一组特定的规则（暂时搞不清）来给这个函数创建一个 `prototype` 属性，其实就是函数 `Person()` 创建了，它的原型 `Person.prototype` 就有了。我们再来new一个Person的实例 person1：
```javascript
var person1 = new Person("liyang", 26, "programmer");
```
实例person1的内部会包含一个官方ECMA5称为 `[[Prototype]]` ，Firefox、Safari和Chrome称为 `__proto__` 的指针，而这个指针 `__proto__` 就会指向构造函数的原型对象 `Person.prototype` ，注意：这个指向是存在于实例 `person1` 和 原型对象 `Person.prototype` 之间的，跟构造函数 `Person()` 是没半毛钱关系的。

扩展一下：实例 `person1` 有一个 `__proto__` 的指针指向了原型对象 `Person.prototype` ，前面说过原型也是个实例，那它同样也会有个 `__proto__` 的指针，指向了谁呢？指向了 `Object.prototype` 这个原型。最后 `Object.prototype` 原型的 `__proto__` 指向了原型链的终点 `null` 。 网上盗图一波：

![01.jpg](http://upload-images.jianshu.io/upload_images/6589697-2191c71af9d9a337.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### why原型?
为什么要用原型？因为使用原型可以让所有对象的实例共享它所包含的属性和方法，先来个代码：
```javascript
function Person(){}

Person.prototype.name = "xiaohong";
Person.prototype.age = 25;
Person.prototype.job = "teacher";
Person.prototype.sayName = function(){
    console.log(this.name);
};

var person1 = new Person();
person1.sayName();   // "xiaohong"

var person2 = new Person();
person2.sayName();   // "xiaohong"

console.log(person1.sayName == person2.sayName);  // true
```

当调用 `person1.sayName()` 的时候，解析器会先在person1里找，找不到的话就接着在上面的 `Person.prototype` 里找，找到就返回，找不到就接着往上，在 `Object.prototype` 里面找，这样一级一级的找下去。

### 杂七杂八
* `Person.prototype.construtor === Person;  //ture `
* 使用 `hasOwnProperty()` 检测一个属性是存在于实例中，还是存在于原型中。
```javascript
Person.name = "xiaohong";  //ture  name属性可访问到
Person.hasOwnProperty("name");  //false  name属性不在实例中
```
* 实例重新定义属性会屏蔽原型属性，可使用 `delete` 操作符删除实例属性，但不可删除原型上的属性。
若以对象字面量的方式来封装原型的功能，那么就相当于重写了整个原型对象，constructor属性指向将会改变，可以通过属性设置将constructor属性重新指向，此时constructor属性将会[[Enumerable]]可枚举（默认原生constructor不可枚举）。
```javascript
function Person(){};
var person1 = new Person();  //重写原型对象之前实例化
Person.prototype = {
	construtor: Person,  //重新指向Person
	name: 'xiaowang',
	age:'30',
	sayName: function(){
		alert(this.name);
	}
};
var person2 = new Person();  //重写原型对象之后实例化
console.log(person1.name);  //undefined
console.log(person2.name);  //xiaowang
```
* 若如上重写原型对象，会切断现有原型与任何之前已经存在的对象实例之间的联系，所以必须在其之后再创建实例。
* 让原型对象等于另一个类型的实例，来实现原型链的继承。
```javascript
son.prototype = new Father(); //将son的原型作为Father()的实例
```

