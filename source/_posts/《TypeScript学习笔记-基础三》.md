---
title: 《TypeScript学习笔记-基础三》
date: 2018-01-06 20:17:45
tags:
 - 知识积累
 - TypeScript
---
今天来说一下基础篇剩下的一些内容：类型断言、声明文件和内置对象

### 类型断言
类型断言（Type Assertion）可以用来手动指定一个值的类型。
语法：`<类型>值`或者`值 as 类型`
#### 将一个联合类型的变量指定为一个更加具体的类型
之前说，当 TypeScript 不确定一个联合类型的变量到底是哪个类型的时候，我们只能访问此联合类型的所有类型里共有的属性或方法：
```javascript
function getLenth(something: string | number): number{
    return something.length;
}
//编译错误
```
而有时候，我们确实需要在还不确定类型的时候就访问其中一个类型的属性或方法，比如：

```javascript
function getLength(something: string | number): number {
    if (something.length) {
        return something.length;
    } else {
        return something.toString().length;
    }
}
//编译错误
```
此时，可以使用类型断言，将`something`断言成`string`:
```javascript
function getLength(something: string | number): number {
    if((<string>something).length){
        return (<string>something).length;
    } else {
        return something.toString().length;
    }
}
```

#### 类型断言不是类型转换，断言成一个联合类型中不存在的类型是不允许的：
```javascript
function toBoolean(something: string | number): boolean {
    return <boolean>something;
}
//编译错误，Type 'string | number' cannot be converted to type 'boolean'.
```

### 声明文件
当使用第三方库时，我们需要引入它的声明文件。

#### 声明语句
假如我们想使用第三方库，比如jQuery，我们通常这样获取一个`id`是`foo`的元素：
```javascript
$('#foo');
//or
jQuery('#foo');
```
但TS并不知道`$`或者`jQuery`是什么东西，这时候我们就需要使用`declare`关键字来定义它的类型，帮助`TypeScript`判断我们的传入的参数类型对不对：
```javascript
declare var jQuery:(string) => any;
jQuery('#foo');
```
`declare`定义的类型只会用于编译时的检查，编译结果中会被删除。上面编译结果为：

```javascript
jQuery('#foo');
```

#### 声明文件
通常我们会把类型声明放到一个单独的文件中，这就是声明文件：
```javascript
// jQuery.d.ts
declare var jQuery: (string) => any;
```
我们约定声明文件以`.d.ts`为后缀。
然后在使用到的文件开头，用`三斜线指令`表示引用了声明文件：

```javascript
/// <reference path="./jQuery.d.ts" />
jQuery('#foo');
```

#### 第三方声明文件
当然，jQuery 的声明文件不需要我们定义了，已经有人帮我们定义好了：[jQuery in DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/jquery/index.d.ts)。
我们可以直接下载下来使用，但是更推荐的是使用工具统一管理第三方库的声明文件。
社区已经有多种方式引入声明文件，不过 TypeScript 2.0 推荐使用 @types 来管理。
@types 的使用方式很简单，直接用 npm 安装对应的声明模块即可，以 jQuery 举例：
```javascript
npm install @types/jquery --save-dev
```
可以在[这个页面](http://microsoft.github.io/TypeSearch/)搜索你需要的声明文件。

### 内置对象
JavaScript 中有很多[内置对象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects)，它们可以直接在 TypeScript 中当做定义好了的类型。
内置对象是指根据标准在全局作用域（Global）上存在的对象。这里的标准是指 ECMAScript 和其他环境（比如 DOM）的标准。
#### ECMAScript 的内置对象
ECMAScript 标准提供的内置对象有：`Boolean`、`Error`、`Date`、`RegExp` 等。我们可以在 TypeScript 中将变量定义为这些类型：
```javascript
let b: Boolean = new Boolean(1);
let e: Error = new Error('Error occurred');
let d: Date = new Date();
let r: RegExp = /[a-z]/;
```
更多的内置对象，可以查看[MDN的文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects)。
而他们的定义文件，则在[TypeScript核心库的定义文件](https://github.com/Microsoft/TypeScript/tree/master/src/lib)中。

#### DOM 和 BOM 的内置对象
DOM 和 BOM 提供的内置对象有：`Document`、`HTMLElement`、`Event`、`NodeList` 等。TypeScript 中会经常用到这些类型：
```javascript
let body: HTMLElement = document.body;
let allDiv: NodeList = document.querySelectorAll('div');
document.addEventListener('click', function(e: MouseEvent) {
  // Do something
});
```

#### TypeScript 核心库的定义文件
TypeScript 核心库的定义文件中定义了所有浏览器环境需要用到的类型，并且是预置在 TypeScript 中的。当你在使用一些常用的方法的时候，TypeScript实际上已经帮你做了很多类型判断的工作了，比如：
```javascript
Math.pow(10, '2');
// index.ts(1,14): error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.
```
上面的例子中，Math.pow 必须接受两个 number 类型的参数。事实上 Math.pow 的类型定义如下：
```
interface Math {
  /**
   * Returns the value of a base expression taken to a specified power.
   * @param x The base value of the expression.
   * @param y The exponent value of the expression.
   */
  pow(x: number, y: number): number;
}
```

#### 用 TypeScript 写 Node.js
Node.js 不是内置对象的一部分，如果想用 TypeScript 写 Node.js，则需要引入第三方声明文件：
```javascript
npm install @types/node --save-dev
```

