---
title: 《TypeScript学习笔记-类与接口》
date: 2018-01-09 20:09:29
tags:
 - 知识积累
 - TypeScript
---

今天的进阶篇主要来说一下类与接口。

### 概述
传统方法中，JavaScript通过构造函数实现类的概念，通过原型链实现继承。而在ES6中有了`class`。
TypeScript除了实现了所有ES6中的类的功能外，还添加了一些新的用法。
### 类的概念
先说下几个重要的概念：
* 类(class): 定义了一件事件的抽象特点，包含它的属性和方法
* 对象(Object): 类的实例，通过`new`生成
* 面向对象(OPP)的三大特点: 封装、继承、多态
* 封装(Encapsulation): 将对数据的操作细节隐藏起来，只暴露对外的接口。外界调用端不需要（也不可能）知道细节，就能通过对外提供的接口来访问该对象，同时也保证了外界无法任意更改对象内部的数据。
* 继承(Inheritance): 子类继承父类，子类除了有父类的所有特性外，还有一些更具体的特性
* 多态(Polymorphism): 由继承而产生了相关的不同的类，对同一个方法可以有不同的响应。比如`Cat`和`Dog`都是继承自`Animal`,但是分别实现了自己的`eat`方法。此时针对某一个实例，我们无需了解它是`Cat`还是`Dog`，就可以直接调用`eat`方法，程序会自动判断出来应该如何执行`eat`
* 存取器(getter & setter): 用以改变属性的读取和赋值行为
* 修饰符(Modifiers): 修饰符是一些关键字，用于限定成员或类型的性质。比如`public`表示公有属性或方法
* 抽象类(Abstract Class): 抽象类是供其他类继承的基类，抽象类不允许被实例化。抽象类中的抽象方法必须在子类中实现
* 接口(Interfaces): 不同类之间公有的属性或者方法，可以抽象成一个接口。接口可以被实现(implements)。一个类只能继承自另一个类，但是可以实现多个接口

### ES6中类的用法
先简单介绍一下ES6中类的用法，具体可参照阮一峰老师的[ECMAScript 6 入门 - Class](http://es6.ruanyifeng.com/#docs/class)
#### 属性和方法
使用`class`定义类，使用`constructor`定义构造函数。
通过`new`生成新实例的时候，会自动调用构造函数。

```javascript
class Animal {
    constructor(name){
        this.name = name;
    }
    sayHi() {
        return `My name is ${this.name}`;
    }
}

let a = new Animal('Jack');
console.log(a.sayHi());  //My name is Jack
```
#### 类的继承
使用`extends`关键字实现继承，子类中使用`super`关键字来调用父类的构造函数和方法。
```javascript
class cat extends Animal {
    constructor(name){
        super(name);
        console.log(this.name);
    }
    sayHi() {
        retrun 'Meow,' + super.sayHi();
    }
}
let c = new Cat('Tom'); // Tom
console.log(c.sayHi()); // Meow, My name is Tom
```
#### 存取器
使用`getter`和`setter`可以改变属性的赋值和读取行为:
```javascript
class Animal {
    constructor(name){
        this.name = name;
    }
    get name(){
        return 'Jack';
    }
    set name(){
        console.log('setter:' + value);
    }
}
let a = new Animal('kitty');  //setter: kitty
a.name = 'Tom';  //setter: Tom
console.log(a.name);  //Jack
```

#### 静态方法
使用`static`修饰符修饰的方法称为静态方法，它们不需要实例化，而是直接通过类来调用：
```javascript
class Animal {
  static isAnimal(a) {
    return a instanceof Animal;
  }
}

let a = new Animal('Jack');
Animal.isAnimal(a); // true
a.isAnimal(a); // TypeError: a.isAnimal is not a function
```

### ES7中类的用法
ES7 中有一些关于类的提案，TypeScript 也实现了它们，这里做一个简单的介绍。
#### 实例属性
ES6 中实例的属性只能通过构造函数中的 this.xxx 来定义，ES7 提案中可以直接在类里面定义：
```javascript
class Animal {
    name = 'Jack';

    constructor(){
        //...
    }
}
let a = new Animal();
console.log(a.name);  // Jack
```
#### 静态属性
ES7 提案中，可以使用 static 定义一个静态属性：
```javascript
class Animal {
    static num =  42;

    constructor () {
        //...
    }
}
console.log(Animal.name);  // 42
```

### TypeScript中类的用法

#### public private 和 protected
TypeScript 可以使用三种访问修饰符（Access Modifiers），分别是 public、private 和 protected。
* public修饰的属性或方法是公有的，可以在任何地方被访问到，默认所有的属性和方法都是`public`的
* private修饰的属性或方法私有的，不能在声明它的类的外部访问
* protected修饰的属性或方法是受保护的，它和`private`类似，区别是它在子类中也是允许访问的

```javascript
class Animal {
  public name;
  public constructor(name) {
    this.name = name;
  }
}

let a = new Animal('Jack');
console.log(a.name); // Jack
a.name = 'Tom';
console.log(a.name); // Tom
```
上面的例子中，`name`被设置为了`public`，所以直接访问实例的`name`属性是允许的。
很多时候，我们希望有的属性是无法直接存取的，这时候就可以用`private`了：

```javascript
class Animal {
  private name;
  public constructor(name) {
    this.name = name;
  }
}

let a = new Animal('Jack');
console.log(a.name); // Jack
a.name = 'Tom';

// index.ts(9,13): error TS2341: Property 'name' is private and only accessible within class 'Animal'.
// index.ts(10,1): error TS2341: Property 'name' is private and only accessible within class 'Animal'.
```
需要注意的是，TypeScript 编译之后的代码中，并没有限制`private`属性在外部的可访问性。
上面的例子编译后的代码是：
```javascript
var Animal = (function () {
    function Animal(name) {
        this.name = name;
    }
    return Animal;
}());
var a = new Animal('Jack');
console.log(a.name);
a.name = 'Tom';
```
使用`private`修饰的属性或方法，在子类中也是不允许访问的：

```javascript
class Animal {
  private name;
  public constructor(name) {
    this.name = name;
  }
}

class Cat extends Animal {
  constructor(name) {
    super(name);
    console.log(this.name);
  }
}

// index.ts(11,17): error TS2341: Property 'name' is private and only accessible within class 'Animal'.
```

而如果是用`protected`修饰，则允许在子类中访问：
```javascript
class Animal {
  protected name;
  public constructor(name) {
    this.name = name;
  }
}

class Cat extends Animal {
  constructor(name) {
    super(name);
    console.log(this.name);
  }
}
```

#### 抽象类
`abstract`用于定义抽象类和其中的抽象方法。
首先，抽象类不允许被实例化：

```javascript
abstract class Animal {
  public name;
  public constructor(name) {
    this.name = name;
  }
  public abstract sayHi();
}

let a = new Animal('Jack');

// index.ts(9,11): error TS2511: Cannot create an instance of the abstract class 'Animal'.
```
上面的例子中，我们定义了一个抽象类`Animal`，并且定义了一个抽象方法`sayHi`。在实例化抽象类的时候报错了。
其次，抽象类中的抽象方法必须被子类实现：
```javascript
abstract class Animal {
  public name;
  public constructor(name) {
    this.name = name;
  }
  public abstract sayHi();
}

class Cat extends Animal {
  public eat() {
    console.log(`${this.name} is eating.`);
  }
}

let cat = new Cat('Tom');

// index.ts(9,7): error TS2515: Non-abstract class 'Cat' does not implement inherited abstract member 'sayHi' from class 'Animal'.
```
上面的例子中，我们定义了一个类 Cat 继承了抽象类 Animal，但是没有实现抽象方法 sayHi，所以编译报错了。
下面是一个正确使用抽象类的例子：
```javascript
abstract class Animal {
  public name;
  public constructor(name) {
    this.name = name;
  }
  public abstract sayHi();
}

class Cat extends Animal {
  public sayHi() {
    console.log(`Meow, My name is ${this.name}`);
  }
}

let cat = new Cat('Tom');
```
上面的例子中，我们实现了抽象方法 sayHi，编译通过了。
需要注意的是，即使是抽象方法，TypeScript 的编译结果中，仍然会存在这个类。

#### 类的类型
给类加上 TypeScript 的类型很简单，与接口类似：
```javascript
class Animal {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  sayHi(): string {
    return `My name is ${this.name}`;
  }
}

let a: Animal = new Animal('Jack');
console.log(a.sayHi()); // My name is Jack
```

### 类与接口

#### 类实现接口
实现（implements）是面向对象中的一个重要概念。一般来讲，一个类只能继承自另一个类，有时候不同类之间可以有一些共有的特性，这时候就可以把特性提取成接口（interfaces），用`implements`关键字来实现。这个特性大大提高了面向对象的灵活性。
举例来说，门是一个类，防盗门是门的子类。如果防盗门有一个报警器的功能，我们可以简单的给防盗门添加一个报警方法。这时候如果有另一个类，车，也有报警器的功能，就可以考虑把报警器提取出来，作为一个接口，防盗门和车都去实现它：
```javascript
interface Alarm {
    alert();
}

class Door {}

class SecurityDoor extends Door implements Alarm {
    alert(){
        console.log('SecurityDoor alert');
    }
}

class Car implements Alarm {
    alert() {
        console.log('Car alert');
    }
}
```
一个类可以实现多个接口：
```javascript
interface Alarm {
    alert();
}
interface Light {
    lightOn();
    ligntOff();
}

class Car implements Alarm, Light {
    alert() {
        console.log('Car alert');
    }
    lightOn() {
        console.log('Car light on');
    }
    lightOff() {
        console.log('Car light off');
    }
}
```
上例中，`Car`实现了`Alarm`和`Light`接口，既能报警，也能开关车灯。

#### 接口继承接口
接口与接口之间可以是继承的关系：
```javascript
interface Alarm {
    alert();
}
interface LightableAlarm extends Alarm {
    lightOn();
    lightOff();
}
```
上例中，我们使用`extends`使`LightableAlarm`继承`Alarm`。

#### 接口继承类
接口也可以继承类：
```javascript
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z:number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3}
```

#### 混合类型
可以使用接口的方式来定义一个函数需要符合的形状：
```javascript
interface SearchFunc {
    (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    return source.search(subString) !== -1;
}
```
有时候，一个函数还可以有自己的属性和方法：
```javascript
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```