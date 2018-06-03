---
title: 《TypeScript学习笔记-基础二》
date: 2018-01-04 20:22:39
tags:
 - 知识积累
 - TypeScript
---
今天主要来说一下TS中对象、数组和函数的类型。

### 对象的类型-接口
在TS中，使用接口（Interface）来定义对象的类型。
#### 什么是接口
接口是对行为的抽象，而具体如何行动需要由类（classes）去实现（implements）。TS中接口除了可用于`对类的一部分行为进行抽象`以外，也常用于对`对象的形状`进行描述。

#### 例子

```javascript
interface Person {
    name: string;
    age: number;
}
let tom: Person = {
    name: 'tom',
    age: 25
};
```
先定义一个接口`Person`，接着定义一个变量`tom`，他的类型是`Person`。这样，我们就约束了`tom`的形状必须和接口`Person`一致。接口一般首字母大写。
注意：赋值的时候，变量的形状必须跟接口形状一致，属性不能多也不能少。
```javascript
interface Person {
    name: string;
    age: number;
}
let tom: Person = {
    name: 'tom',
    age: 25,
    gender: 'male'
};
//编译错误

interface Person {
    name: string;
    age: number;
}
let tom: Person = {
    name: 'tom'
};
//编译错误
```
#### 可选?属性
可选属性表示该属性在变量中可有可无。
```javascript
interface Person {
    name: string;
    age?: number;
}
let tom: Person = {
    name: 'tom'
};
//编译成功
interface Person {
    name: string;
    age?: number;
}
let tom: Person = {
    name: 'tom',
    age: 12
};
//编译成功
```
注意：此时仍不允许添加未定义属性，即：不能多

#### 任意属性
任意属性，即变量可有任意的属性，且一旦定义了任意属性，那么确定属性和可选属性都必须是它的子属性：
```javascript
interface Person {
    name: string;
    age?: number;
    [propName: string]: any;
}

let tom: Person = {
    name: 'Tom',
    gender: 'male'
};
//编译成功

interface Person {
    name: string;
    age?: number;
    [propName: string]: string;
}

let tom: Person = {
    name: 'Tom',
    age: 25,
    gender: 'male'
};
//编译失败
```

#### 只读属性
有时候我们希望对象中的一些字段只能在创建的时候被赋值，那么可以用 readonly 定义只读属性：
```javascript
interface Person {
    readonly id: number;
    name: string;
    age?: number;
    [propName: string]: any;
}
let tom: Person = {
    id: 89757,
    name: 'Tom',
    gender: 'male'
};
tom.id = 9527;
//编译错误
```
注意：只读的约束存在于`第一次给对象赋值`的时候，而不是`第一次给只读属性赋值`的时候：
```javascript
interface Person {
    readonly id: number;
    name: string;
    age?: number;
    [propName: string]: any;
}
let tom: Person = {
    name: 'Tom',
    gender: 'male'
};
tom.id = 89757;
//编译错误
```

### 数组的类型
在TS中，数组的类型有多种定义方式，比较灵活。
#### 类型+方括号表示法
最简单的办法是通过`类型+方括号`来表示数组：

```javascript
let fibonacci: number[] = [1, 1, 2, 3, 5];
```
数组的项中不允许出现其他的类型。
```javascript
let fibonacci: number[] = [1, '1', 2, 3, 5];
//编译错误
```
数组的一些方法的参数也会根据数组在定义时约定的类型进行限制：
```javascript
let fibonacci: number[] = [1, 1, 2, 3, 5];
fibonacci.push('8');
//编译错误
```

#### 数组泛型表示法
也可以使用数组泛型（Array Generic） `Array<elemType>`来表示数组：
```javascript
let fibonacci: Array<number> = [1, 1, 2, 3, 5];
```
泛型到后面会再展开描述

#### 用接口表示数组
接口也可以用来表示数组：
```javascript
interface numArray {
    [index: number]: string
}
let arr: numArray = ['a', 'b', 'c'];
```
`numArray`表示：只要`index`的类型是`number`，那么值的类型必须是`string`。

### 函数的类型
在 JavaScript 中，有两种常见的定义函数的方式——函数声明（Function Declaration）和函数表达式（Function Expression）。一个函数有输入和输出，要在 TypeScript 中对其进行约束，需要把输入和输出都考虑到。
#### 函数声明
```javascript
function sum(x: number, y: number): number {
    return x + y;
}
sum(1, 2);
```
注意，输入多的或少的参数，是不被允许的：
```javascript
function sum(x: number, y: number): number {
    return x + y;
}
sum(1, 2, 3);
//编译错误
function sum(x: number, y: number): number {
    return x + y;
}
sum(1);
//编译错误
```

#### 函数表达式
如果我们现在写一个对函数表达式（Function Expression）的定义，可能会写成这样：
```javascript
let mySum = function(x: number, y: number): number {
    return x + y;
}
```
这是可以通过编译的，不过事实上，上面的代码只对等号右侧的匿名函数进行了类型定义，而等号左边的 mySum，是通过赋值操作进行类型推论而推断出来的。如果需要我们手动给 mySum 添加类型，则应该是这样：
```javascript
let mySum: (x: number, y: number) => number = function(x: number, y: number): number {
    return x + y;
}
```
注意不要混淆了TS中的 `=>` 和ES6中的 `=>`。在 TypeScript 的类型定义中，`=>`用来表示函数的定义，左边是输入类型，需要用括号括起来，右边是输出类型，在 ES6 中，`=>` 叫做箭头函数，应用十分广泛。

#### 用接口定义函数形状
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

#### 可选参数
前面提到，输入多余的（或者少于要求的）参数，是不允许的。那么如何定义可选的参数呢？与接口中的可选属性类似，我们用 ? 表示可选的参数：
```javascript
function buildName(firstName: string, lastName?: string) {
    if (lastName) {
        return firstName + ' ' + lastName;
    } else {
        return firstName;
    }
}
let tomcat = buildName('Tom', 'Cat');
let tom = buildName('Tom');
```
注意：可选参数必须接在必需参数后面。换句话说，可选参数后面不允许再出现必须参数了：
```javascript
function buildName(firstName?: string, lastName: string) {
    if (firstName) {
        return firstName + ' ' + lastName;
    } else {
        return lastName;
    }
}
let tomcat = buildName('Tom', 'Cat');
let tom = buildName(undefined, 'Tom');
//编译错误
```

#### 参数默认值
在 ES6 中，我们允许给函数的参数添加默认值，TypeScript 会将添加了默认值的参数识别为可选参数：
```javascript
function buildName(firstName: string, lastName: string = 'Cat'){
    return firstName + ' ' + lastName;
}
let tomcat = buildName('Tom', 'Cat');
let tom = buildName('Tom');
```
此时就不受「可选参数必须接在必需参数后面」的限制了：
```javascript
function buildName(firstName: string = 'Tom', lastName: string) {
    return firstName + ' ' + lastName;
}
let tomcat = buildName('Tom', 'Cat');
let cat = buildName(undefined, 'Cat');
```

#### 剩余参数
ES6中，可以使用`...rest`的方式获取函数中的剩余参数（rest参数）：
```javascript
function push(array, ...items){
    item.forEach(function(item){
        array.push(item);
    })
}
let a = [];
push(a, 1, 2, 3);
```
注意，rest 参数只能是最后一个参数
