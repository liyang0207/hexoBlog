---
title: 由-0与NaN的判断来看Array的元素识别
date: 2018-06-15 20:42:47
tags:
 - JavaScript
 - 知识积累
---

今天无聊在翻看MDN开发文档时，看到这么一个方法[Array.prototype.includes()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/includes)，之前在书上看到过，说是ES6修订文档（可以称为ES7）中新增的一个方法，用来判断一个数组中是否包含一个指定的值，返回一个布尔类型。当时也没有深究新增的这个方法与[Array.prototype.indexOf()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf)到底有什么不同，闲来无事，深究了一下，结果扯出来一些关于值判断的问题，自己都把自己问蒙了。下面就简单总结一下。
<!-- more -->

先说两个方法：

## Array.prototype.includes(searchElement, fromIndex)

用来判断一个数组是否包含一个指定的值，根据情况，如果包含则返回 true，否则返回false，`fromIndex` 表示从该索引处开始查找，如果为负数，则按升序从`arr.length`+`fromIndex`的索引开始搜索。默认为0。

```javascript
var arr = [1, 2, 3, NaN, 0, -0];
console.log(arr.includes(1));       //true
console.log(arr.includes(NaN));     //true
console.log(arr.includes(3, 3));    //false
console.log(arr.includes(2, -2));   //false
```

## Object.is(value1, value2)

用来判断两个值是否相等，返回布尔值，下面情况则认为是相等的：

- 两个值都是`undefined`
- 两个值都是`null`
- 两个值都是`true`或`false`
- 两个值是由相同个数的字符按照相同的顺序组成的字符串
- 两个值指向同一个对象
- 两个值都是数字并且都是正零+0、都是负-0、都是NaN、都是除零和NaN外的其他同一个数字

```javascript
Object.is(undefined, undefined);   //true
Object.is(null, null);             //true
Object.is(0, -0);                  //false
Object.is(NaN, NaN);               //true
```

从上面可以看出，`Object.is()`与全等`===`不同的地方在于，`===`认为正零`+0`和负零`-0`相等，而且`NaN`不自等：

```javascript
console.log(0 === -0);      // true
console.log(NaN === NaN);   // false
```

## includes VS indexOf

以往我们在判断某一项是否存在于一个数组内时，都是使用`arr.indexOf(item)`方法，根据返回的索引值是否为`-1`来进行判断：

```javascript
var arr = [1, 2, 3, 4, 0, -0, NaN];
if (arr.indexOf(5) > -1) {
    //不会执行
}
if (arr.indexOf(NaN) > -1) {
    //不会执行
}
console.log(arr.indexOf(NaN));  // -1
console.log(arr.indexOf(-0));   // 4
console.log(arr.indexOf(0));    // 4
```

方法可行，但是这种方法一来无法判断`NaN`，另一方面既然我们在条件判断语句中使用，为什么不能有一个返回布尔类型的方法呢？`includes()`方法应运而生。

```
var arr = [1, 2, 3, 4, 0, NaN];
if (arr.includes(5)) {
    //不会执行
}
if (arr.includes(NaN)) {
    //执行
}
console.log(arr.includes(NaN));      //true
console.log(arr.includes(4));        //true
console.log(arr.includes(-0));       //true，与indexOf一样，都无法区分+0和-0
```

这样我们在进行元素存在判断的时候就不用再跟`-1`去比较了，直接返回布尔值，岂不美哉！

## Object.is() VS ===

除了值‘在不在’的判断，我们日常还会跟值‘等不等’来打交道。现在随便问个Front-Ender都知道使用全等`===`来进行‘等不等’的判断，当然这里就涉及到我之前博客里讲的隐式类型转换的知识了，不再重复。如果我要判读的值是`NaN`或者`-0`呢？全等`===`还能胜任吗？

```javascript
console.log(NaN === NaN);    // false, 都知道NaN‘不自等’
console.log(0 === -0);       // true, 无法区分+0和-0
```

有个小误区，其实`NaN`应该跟`NaN`相等才是正常的，长久以来，一直被`NaN不自等`所误导，反而认为`NaN`不相等。现在有了`Object.is()`，可以愉快的判断`NaN`和`-0`了：

```javascript
console.log(Object.is(NaN, NaN));    // true
console.log(Object.is(0, -0));       // false
console.log(Object.is(-0, -0));      // true
```

还有一种粗暴的方式来判断`-0`：

```javascript
var x = -0;
if (x == 0 && 1 / x === -Infinity) {
    // -0的情况
}
//或者
if (x == 0 && 1 / x < 0) {
    // -0的情况
}
```

## Number.isNaN()  VS isNaN()

说起`NaN`的判断，怎么能不提`isNaN()`和`Number.isNaN()`，`Number.isNaN()`是一个更强大的方法，该方法不会强制将参数转换为数字，只有在参数是真正的数字类型，并且值为`NaN`的时候才会返回`true`。

```javascript
Number.isNaN(NaN);        // true
Number.isNaN(0 / 0);      // true
Number.isNaN('NaN');      // true
Number.isNaN({});         // false
Number.isNaN(undefined);  // false
Number.isNaN('test');     // false
Number.isNaN('37');       // fasle
```

而`isNaN()`会首先尝试将这个参数转换为数值，然后才会对转换后的结果是否是`NaN`进行判断。 对于能被强制转换为有效的非`NaN`数值来说（空字符串和布尔值分别会被强制转换为数值0和1），返回`false`值也许会让人感觉莫名其妙。

```javascript
isNaN('');                // false
isNaN(true);              // false
isNaN('NaN');             // true
isNaN({});                // true
isNaN(undefined);         // true
isNaN('test');            // true
isNaN('37');              // false
```

其实这里面又涉及到了类型转换的问题，可以看一下之前的博客。

## 总结

1. 当需要判断数组内是否有`NaN`时，可以考虑使用`includes()`方法，当然要考虑兼容性问题；
2. 当需要判断一个值是否是`NaN`时，可以使用`Object.is()`和`Number.isNaN()`，优先使用`Number.isNaN()`；
3. 如果非要考虑`-0`的情况，那就可以使用`Object.is()`或者`x == 0 && 1 / x < 0`来进行判断；
