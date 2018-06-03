---
title: 《TypeScript学习笔记-基础一》
date: 2018-01-03 20:00:41
tags:
 - 知识积累
 - TypeScript
---
17年前端的发展逐渐趋于理性，由盲目的追求广度转向探求语言和框架的深度。TS作为JS的超集，日后定会成为前端的必备技能之一，以前对ts只是懵懵懂懂的，不成体系，不求细节，趁着最近公司要上Angular的项目，抓紧时间系统学习一下TS的相关知识，该系列博客作为日后查漏补缺之用，内容来源于[xcatliu:TypeScript入门教程](https://www.gitbook.com/book/xcatliu/typescript-tutorial/details)。

### 什么是TypeScript?
TypeScript 是 JavaScript 的一个超集，主要提供了类型系统和对 ES6 的支持，它由 Microsoft 开发，代码开源于 GitHub 上。它可以编译成纯 JavaScript。编译出来的 JavaScript 可以运行在任何浏览器上。同时TypeScript 编译工具可以运行在任何服务器和任何系统上。

### TS编译
使用`npm`全局安装`typescript`
```javascript
$ npm install -g typescript
```
新建`hello.ts`文件，在当前目录下执行
```javascript
$ tsc hello.ts
```
即可编译

### 原始数据类型
JavaScript的原始类型包括：boolean,number,string,null,undefined及ES6中的新类型Symbol。
* 布尔值

```javascript
let boo: boolean = false;
//编译通过
```
注意，使用构造函数`Boolean`创造的对象不是布尔值，`new Boolean()`返回的是一个`Boolean`对象
```javascript
let newboo: boolean = new Boolean(1);
//编译失败
```
以下两种调用方式可行：
```javascript
let newboo: Boolean = new Boolean(1);
let newboo: boolean = Boolean(1);
```

总结：js的原始类型基本上都遵循以上规则（除null和undefined之外）

* 数值

```javascript
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
// ES6 中的二进制表示法
let binaryLiteral: number = 0b1010;
// ES6 中的八进制表示法
let octalLiteral: number = 0o744;
let notANumber: number = NaN;
let infinityNumber: number = Infinity;
```
编译结果：
```javascript
var decLiteral = 6;
var hexLiteral = 0xf00d;
// ES6 中的二进制表示法
var binaryLiteral = 10;
// ES6 中的八进制表示法
var octalLiteral = 484;
var notANumber = NaN;
var infinityNumber = Infinity;
```
总结：其中 0b1010 和 0o744 是 ES6 中的二进制和八进制表示法，它们会被编译为十进制数字。

* 字符串

```javascript
let myName: string = 'Tom';
let myAge: number = 20;
//模板字符串
let sentence: string = `my name is ${myName}, 
my age is ${myAge}`;
```
编译结果：
```javascript
var myName = 'Tom';
var myAge = 25;
// 模板字符串
var sentence = "my name is " + myName + ",\nmy age is " + myAge;
```

* 空值

js没有空值（Void）的概念，在ts中，用void表示没有任何返回值的函数：
```javascript
function getName(): void {
    alert('liyang');
}
```
申明一个`void`类型的变量没有什么用，因为只能赋值给他`undefined`和`null`:
```javascript
let test: void = undefined;
```

* Null和Undefined

在TS中，可以使用`null`和`undefined`来定义这两个原始的数据类型：
```javascript
let u: undefined = undefined;
let n: null = null;
```
`undefined`类型的变量只能被赋值为`undefined`,  `null`类型的变量只能被赋值为`null`
注意：与`void`区别是：`undefined`和`null`是所有类型的子类型，也就是说`undefined`类型的变量可赋值给`number`类型的变量：
```javascript
let num: number = undefined;
//编译通过

let u: undefined;
let num: number = u;
//编译通过
```
而`void`类型不能赋值给`number`类型的变量：
```
let u: void;
let num: number = u;
//编译错误
```

### 任意值
任意值用来表示允许赋值为任意类型。
* 普通类型在赋值过程中不允许改变类型：

```javascript
let number: string = 'three';
number = 12;
//编译错误
```
但若是`any`类型，则允许赋值时改变类型：
```javascript
let number: any = 'three';
number = 34;
//编译成功
```
* 在任意值上访问任何属性都是允许的：

```javascript
let anything: any = 'liyang';
console.log(anyThing.myName);
```
也可以调用任何方法：
```javascript
let anything: any = 'liyang';
anything.setName('tom');
anything.setName('tom').sayHello();
```
注：声明一个变量为任意值之后，对它的任何操作，返回的内容的类型都是任意值。

* 变量如果在声明时未指定其类型，则会被识别未任意值类型：
```javascript
let something;
something = 'seven';
something = 7;

something.setName('Tom');
```

### 类型推论
如果没有明确的指定类型，那么 TypeScript 会依照类型推论（Type Inference）的规则推断出一个类型。

```javascript
let number = 'three';
number = 12;
//编译报错
```
以上等价于：
```javascript
let number: string = 'three';
number = 12;
```
注：如果定义的时候没有赋值，不管之后有没有赋值，都会被推断成 any 类型而完全不被类型检查：
```javascript
let number;
number = 'three';
number = 13;
//编译成功
```

### 联合类型

* 联合类型表示取值可以是多种类型中的一种

```javascript
let num: string | number;
num = 'three';
num = 123;
//编译成功

let num: string | number;
num = 'three';
num = ture;
//编译错误
```
* 访问联合类型的属性或方法
当 TypeScript 不确定一个联合类型的变量到底是哪个类型的时候，我们只能访问此联合类型的所有类型里共有的属性或方法：

```javascript
function getLength(something:string | number): number{
    return something.length;
}
//编译错误，number没有length属性

function getLength(something:string | number): number{
    return something.toString();
}
//编译成功
```
* 联合类型的变量在被赋值的时候，会根据类型推论的规则推断出一个类型：

```javascript
let num: string | number;
num = 'three';
console.log(num.length);  // 成功，返回5
num = 123;
console.log(num.length);  // 错误
```
第四行的 num 被推断成了 number，访问它的 length 属性时就报错了。
