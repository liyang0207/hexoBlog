---
title: 《TypeScript学习笔记-泛型》
date: 2018-01-15 20:41:57
tags:
 - 知识积累
 - TypeScript
---
今天来说一下TypeScript中的泛型和声明合并。

### 泛型
泛型（Generics）是指在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性。
<!-- more -->

#### 简单的例子
首先，我们来实现一个函数`createArray`，它可以创建一个指定长度的数组，同时将每一项都填充一个默认值：
```javascript
function createArray(length: number, value: any): Array<any> {
    let result = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}
createArray(3, 'x');   //['x', 'x', 'x']
```
上例不会报错，但一个缺陷就是：它并没有准确的定义返回值的类型：`Array<any>`允许数组的每一项都是任意类型，但我们的预期是，数组中的每一项都应该是输入的`value`的类型。
这时候，泛型就派上用场了。
```javascript
function createArray<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}
createArray<string>(3, 'x');   //['x', 'x', 'x']
```
上例中，我们在函数名后添加了`<T>`，其中`T`用来指代任意输入的类型，在后面的输入`value: T`和输出`Array<T>`中即可使用了。接着在调用的时候，可以指定它具体的类型为 string。当然，也可以不手动指定，而让类型推论自动推算出来：
```javascript
function createArray<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray(3, 'x'); // ['x', 'x', 'x']
```
#### 多个类型参数
定义泛型的时候，可以一次定义多个类型参数：
```javascript
function swap<T, U>(tuple: [T, U]): [U, T]{
    return [tuple[1], tuple[0]];
}
swap([7, 'seven']);  //['seven', 7]
```
#### 约束泛型
在函数内部使用泛型变量时，由于事先不知道它是什么类型，所以不能随意操作它的属性或方法：
```javascript
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);
    return arg;
}
// index.ts(2,19): error TS2339: Property 'length' does not exist on type 'T'.
```
上例中，泛型 T 不一定包含属性`length`，所以编译的时候报错了。
这时，我们可以对泛型进行约束，只允许这个函数传入那些包含`length`属性的变量。这就是泛型约束：
```javascript
interface Lengthwise {
    length: number;
}
function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}
```
上例中，我们使用了`extends`约束了泛型`T`必须符合接口`Lengthwise`的形状，也就是必须包含`length`属性。
此时如果调用`loggingIdentity`的时候，传入的`arg`不包含`length`，那么在编译阶段就会报错了：
```javascript
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}

loggingIdentity(7);

// index.ts(10,17): error TS2345: Argument of type '7' is not assignable to parameter of type 'Lengthwise'.
```
#### 泛型接口
之前说过，可以使用接口的方式来定义一个函数需要符合的形状：
```javascript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    return source.search(subString) !== -1;
}
```
当然也可以使用含有泛型的接口来定义函数的形状：
```javascript
interface CreateArrayFunc {
    <T>(length: number, value: T): Array<T>;
}
let createArray: CreateArrayFunc;
createArray = function<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}
createArray(3, 'x'); // ['x', 'x', 'x']
```
进一步，我们可以把泛型参数提前到接口名上：
```javascript
interface CreateArrayFunc<T> {
    (length: number, value: T): Array<T>;
}

let createArray: CreateArrayFunc<any>;
createArray = function<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray(3, 'x'); // ['x', 'x', 'x']
```
注意，此时在使用泛型接口的时候，需要定义泛型的类型。

#### 泛型类
与泛型接口类似，泛型也可以用于类的类型定义中：
```javascript
class GenericNumber<T {
    zeroValue: T;
    add: (x: T, y: T) => T;
}
let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```
#### 泛型参数的默认类型
在 TypeScript 2.3 以后，我们可以为泛型中的类型参数指定默认类型。当使用泛型时没有在代码中直接指定类型参数，从实际值参数中也无法推测出时，这个默认类型就会起作用。
```javascript
function createArray<T = string>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}
```

### 声明合并
如果定义了两个相同名字的函数、接口或类，那么它们会合并成一个类型：
#### 函数的合并
之前学习过，我们可以使用重载定义多个函数类型：
```javascript
function reverse(x: number): number;
function reverse(x: string): string;
function reverse(x: number | string): number | string {
    if (typeof x === 'number') {
        return Number(x.toString().split('').reverse().join(''));
    } else if (typeof x === 'string') {
        return x.split('').reverse().join('');
    }
}
```
#### 接口的合并
```javascript
interface Alarm {
    price: number;
}
interface Alarm {
    weight: number;
}
```
相当于：
```javascript
interface Alarm {
    price: number;
    weight: number;
}
```
注意，合并的属性的类型必须是唯一的：
```javascript
interface Alarm {
    price: number;
}
interface Alarm {
    price: number;  // 虽然重复了，但是类型都是 `number`，所以不会报错
    weight: number;
}
```
```javascript
interface Alarm {
    price: number;
}
interface Alarm {
    price: string;  // 类型不一致，会报错
    weight: number;
}

// index.ts(5,3): error TS2403: Subsequent variable declarations must have the same type.  Variable 'price' must be of type 'number', but here has type 'string'.
```
接口中方法的合并，与函数的合并一样：
```javascript
interface Alarm {
    price: number;
    alert(s: string): string;
}
interface Alarm {
    weight: number;
    alert(s: string, n: number): string;
}
```
相当于：
```javascript
interface Alarm {
    price: number;
    weight: number;
    alert(s: string): string;
    alert(s: string, n: number): string;
}
```
#### 类的合并
类的合并与接口的合并规则一致。