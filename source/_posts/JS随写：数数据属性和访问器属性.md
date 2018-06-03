---
title: JS随写：数据属性和访问器属性
date: 2017-08-17 21:38:09
tags:
  - JavaScript
  - 随写
---
今天简单来聊一聊对象的两种属性：数据属性和访问器属性

### 数据属性
数据属性包含一个数据值的位置，有4个描述其行为的特性。
* [[Configurable]] : 表示能否通过 delete 删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性。默认值为true。
* [[Enumerable]] : 是否可枚举，表示能否通过 for-in 循环返回属性。默认值为true。
* [[Writable]] : 表示能否修改属性的值。默认值为true。
* [[Value]] : 包含这个属性的数据值。读取属性值的时候，从这个位置读；写入属性值时，把新值保存在这个位置。默认值是 undefined。
<div class="tip">对于通过`new`关键字定义的对象的属性或者通过对象字面量直接在对象上定义的属性，它们的 [[Configurable]]、[[Enumerable]] 和 [[Writable]] 特性默认都被设置为 true，而 [[Value]] 特性被设置为指定的值。</div>

### Object.defineProperty( obj, prop, descriptor)
* obj：需要定义属性的对象
* prop：需定义或修改的属性的名字。
* descriptor：一个包含设置特性的描述符对象
如果要修改属性默认的特性，必须使用 ECMAScript 的 `Object.defineProperty()` 方法。该方法接受3个参数：属性所在的对象，属性的名字和一个描述符对象。描述符对象的属性必须是：configurable、enumerable、writable和value。

```javascript
var person = {
    name: 'liyang',
    age: '26'
}
Object.defineProperty(person,'name',{
    writable: false
});
console.log(person.name);  //liyang
person.name = "miaomiao";  //此时，严格模式下该赋值操作会抛出错误，非严格模式下会被忽略
console.log(person.name);  //liyang
Object.getOwnPropertyDescriptor(person,'name'); //Object {value: "liyang", writable: false, enumerable: true, configurable: true}
```
注：一旦把属性定义为不可配置的，那么就再也不能把属性定义回可配置的了。
```javascript
var person = {
    name: 'liyang',
    age: '26'
}
Object.defineProperty(person,'name',{
    configurable: false
});
Object.defineProperty(person,'name',{
    configurable: true
});   //Uncaught TypeError: Cannot redefine property: name

```
注：如果对一个空对象调用Object.defineProperty()方法时，如果不指定，configurable、enumerable和writable特性默认都将是false
```javascript
var person = {}
Object.defineProperty(person,'name',{value:"liyang"});
Object.getOwnPropertyDescriptor(person,'name');  //Object {value: "xx", writable: false, enumerable: false, configurable: false}
```
### Object.getOwnPropertyDescriptor( obj, prop)
该方法可以取得给定属性的描述符。接收两个参数：属性所在的对象和要读取其描述符的属性名称。如果是数据属性 ，则有configurable、enumerable、writable和value；若是访问器属性，则有configurable、enumerable、get和set;

### 访问器属性
访问器属性不包含数据值，它们包含一对儿`getter`和`setter`函数（不过，这两个函数都不是必需的）。在读取访问器属性时，会调用`getter`函数，在写入访问器属性时，又会调用`setter`函数并传入新值。
* [[Configurable]] : 表示能否通过 delete 删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为数据属性。默认值为true。
* [[Enumerable]] : 是否可枚举，表示能否通过 for-in 循环返回属性。默认值为true。
* [[Get]] : 在读取属性时调用的函数。默认值为 undefined。
* [[Set]] : 在写入属性时调用的函数。默认值为 undefined。
注：访问器属性不能直接定义，必须使用 Object.defineProperty() 来定义。
```javascript
var book = {
    _year : 2004,
    edition : 1
};
Object.defineProperty(book,"year",{ 
    get : function () {
        return this._year;
    },
    set : function (newValue) {
        if (newValue > 2004) {
            this._year = newValue;
            this.edition += newValue - 2004;
        }
    }
});
console.log(book.year);  // 2004
book.year = 2005;
console.log(book.edition);  //2
```
getter函数返回_year的值，setter函数通过计算来得到正确的edition值。属性year的值修改为2005会导致_year的值变为2005，而edition变成2。这就是访问器属性的常见方式，即设置一个属性的值会导致其他属性发生变化。
getter和setter不一定非得同时指定，只指定getter意味着属性不能写，尝试写入属性会被忽略。没有指定setter属性也不能读，否则在非严格模式下会返回undefined，严格模式下抛出错误。

### 定义多个属性 Object.defineProperties()
该方法可一次性定义多个属性。
```javascript
var book = {};
Object.defineProperties(book,{
    _year:{
        value: 2004
    },
    edition:{
        value: 1
    },
    year:{
        get:function(){
            return this._year;
        },
        set:function(){
            if (newValue > 2004) {
                this._year = newValue;
                this.edition += newValue - 2004;
            }
        }
    }
});
```

前几天百度编辑器bug修复之后，还是被市场同事抱怨编辑器屎一样难用。想了想决定把项目中的编辑器换掉，初步确定使用ckEditor编辑器，需要自定义一些功能：基于阿里OSS的图片上传功能和腾讯视频的iframe视频上传功能。而且编辑器编辑成文后是要在移动端展示，目标是做一个移动端的“所见即所得”编辑器，最近有得搞了。。。
















