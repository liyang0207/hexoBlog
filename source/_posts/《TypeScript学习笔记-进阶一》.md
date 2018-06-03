---
title: 《TypeScript学习笔记-进阶一》
date: 2018-01-07 14:17:13
tags:
 - 知识积累
 - TypeScript
---
今天进入进阶篇的学习和总结，先来说一下类型别名、字符串字面量类型和元祖。

### 类型别名
类型别名用来给一个类型起个新名字

```javascript
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResoler): Name {
    if(typeof n === 'string'){
        return n;
    } else {
        return n();
    }
}
```
我们使用 `type` 创建类型别名，常用于联合类型。

### 字符串字面量类型
字符串字面量类型用来约束取值只能是某几个字符串中的一个。
```javascript
type EventNames = 'click' | 'scroll' | 'mouseover';
function handleEvent(ele: Element, event: EventNames){
    //do something;
}
handleEvent(document.getElementById('hello'), 'scroll');  //编译成功
handleEvent(document.getElementById('world'), 'dbclick'); //编译错误
```
上面我们使用`type`定义了一个字符串字面量类型`EventNames`，它只能取三种字符串中的一种。
注意：`类型别名和字符串子面量类型都是使用type进行定义`。

### 元组(Tuple)
数组合并了相同类型的对象，而元组合并了不同类型的对象。元组起源于函数编程语言（如 F#）,在这些语言中频繁使用元组。

#### 简单的例子
定义一对值分别为`string`和`number`的元祖：
```javascript
let tom:[string, number] = ['tom', 20];
```
当赋值或访问一个已知索引的元素时，会得到正确的类型：
```javascript
let tom:[string, number];
tom[0] = 'tom';
tom[1] = 20;

tom[0].slice(1);
tom[1].toFixed(2);
```
也可以只赋值其中一项：
```javascript
let tom: [string, number];
tom[0] = 'tom';
```
但当直接对元祖类型的变量进行初始化或者赋值的时候，需要提供所有元祖类型中指定的项。
```javascript
let tom: [string, number];
tom = ['tom', 20];
//编译成功

let tom: [string, number] = ['tom'];
//编译错误

let tom: [string, number];
tom = ['tom'];
tom[1] = 20;
//编译错误
```

### 越界的元素
当赋值给越界的元素时，它的类型会被限制为元祖中每个类型的联合类型：
```javascript
let tom: [string, number];
tom = ['tom', 20, 'male'];
```
上面的例子中，数组的第三项满足联合类型 string | number。
```javascript
let tom: [string, number];
tom = ['tom', 20];
tom.push('male');
tom.push(true);
// index.ts(4,14): error TS2345: Argument of type 'boolean' is not assignable to parameter of type 'string | number'.
//   Type 'boolean' is not assignable to type 'number'.
```
当访问一个越界的元素，也会识别为元组中每个类型的联合类型：
```javascript
let tom: [string, number];
tom = ['tom', 20, 'male'];

console.log(tom[2].slice(1));
// index.ts(4,24): error TS2339: Property 'slice' does not exist on type 'string | number'.
```
之前提到过，`如果一个值是联合类型，我们只能访问此联合类型的所有类型里共有的属性或方法`。
