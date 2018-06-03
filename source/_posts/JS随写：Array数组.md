---
title: JS随写：Array数组
date: 2017-07-02 22:31:57
tags:
 - JavaScript
 - 随写
---
### 创建数组的两种方式：
#### 1.使用Array构造函数 `var colors = new Array()` , `new` 操作符可省略
给构造函数传递一个值的话，若为数值，则该值为新数组的长度；若为其他类型，则该值为新数组的第一个元素，且数组长度为1。
```javascript
var colors = new Array(3);  //创建一个包含3项的数组，值都为undefined
var colors = new Array("liyang");  //创建一个包含1项，即字符串'liyang'的数组
```
根据此特性创建一个重复某字符串n次的方法：
```javascript
function repeatString(str,n){
    return new Array(n+1).join(str);   //创建一个长度为n+1的空数组，使用str来连接这个空数组的元素,即undefined
}
repeatString("a",4);    // 返回  "aaaa"
repeatString("3",3);   //返回 "333"
```
#### 2.使用数组字面量表示法
```javascript
var colors = ['red','yellow','green'];
var values = [1,2,];  //不推荐，IE8及以下会视为长度为3，最后一个值为undefined
var options = [,,,];  //不推荐，IE8会视为长度为4，值都是undefined
```
直接给`Array`的length赋一个新的值会导致`Array`大小的变化;
`Array`可以通过索引把对应的元素修改为新的值;
如果通过索引赋值时，索引超过了范围，同样会引起`Array`大小的变化

### 检测数组
* value instanceof Array 执行环境会对其判断有影响
* ECMAScript5新增的方法 `Array.isArray()` 来确定某个值到底是不是数组

### 转换方法
所有对象都有 toLocaleString()、toString()和valueOf()方法，数组的调用如下：
* `toString()`:返回由数组中每个值的字符串形式拼接而成的一个以逗号分隔的 `字符串`；
* `toLocaleString()`:同样返回一个逗号分隔的字符串，不同在于会调用数组每一项的toLocaleString()方法，而不是toString()方法；
* `valueOf()`: 返回数组；

### shift,unshift和push,pop
* `shift()` 方法删除 `Array` 的第一个元素，返回的是删掉的元素
* `unshift()` 方法向 `Array` 的头部添加若干值，返回的是新数组的长度
* `push()` 方法向 `Array` 的尾部添加若干值，返回的是新数组的长度
* `pop()` 方法删除 `Array` 的最后一个元素，返回的是删掉的元素

```javascript
var arr = [1,2,3,4];
arr.shift();  // 返回'1'
arr; // [2,3,4]
arr.unshift('a','b');  //返回 5
arr;  //['a','b','2','3','4']

arr.push('c','d');  //返回 7
arr;  //['a','b','2','3','4','c','d']
arr.pop();  //返回 'd'
arr;  //['a','b','2','3','4','c']
```
### reverse和sort
* `reverse()` 方法用于反转数组项的顺序
* `sort()` 方法用于排序数组，默认按照升序排序
 `sort()` 方法会调用每个数组项的 `toString()` 方法，然后比较字符串进行排序，可以接受一个比较函数作为参数。

```javascript
var arr = [13,15,5,2,6];
arr.sort();
arr;  //[13,15,2,5,6]
arr.sort(function(a,b){
	return a - b;  //升序
});
arr;  //[2,5,6,13,15]

arr.sort(function(a,b){
	return b - a;  //降序
});
arr;  //[15,13,6,5,2]
```
### slice和splice
* `slice()` 方法截取数组的部分元素，根据索引来截取。对应字符串的 `substring()` 方法
* `splice()` 方法可以从指定的索引处开始删除n个元素，然后再从该位置添加m个元素，是数组的万能方法

```javascript
var arr = ['red','yellow','green','blue'];
arr.slice(1,3);  //从1开始截取到3，不包括3。返回截取的数组['yellow','green']
arr;  //数组本身不变：['red','yellow','green','blue']

var arr2 = ['red','yellow','green','blue'];
arr2.slice(2);  //从索引2开始到结束。['green','blue']
arr2;  //数组本身不变：['red','yellow','green','blue']

var arr3 = ['red','yellow','green','blue'];
var copy = arr3.slice();  //不传值相当于复制数组
copy;  //返回：['red','yellow','green','blue']
copy === arr;  // false

var arr4 = ['red','yellow','green','blue'];
arr4.slice(1,-1);  // ['yellow','green']  -1索引指最后一个元素，-2指倒数第二个
---------------
var arr1 = ['a','b','c','d'];
//从索引2开始删除1个元素，然后再添加两个元素
arr1.splice(2,1,'red','yellow');  //返回删除的元素['c']
arr1;  // ['a','b','red','yellow','d']
//只删除，不添加
arr1.splice(1,2);  //返回删除的元素数组 ['b','red']
arr1;  //  ['a','yellow','d']
//只添加，不删除
arr1.splice(1,0,'green');  //返回空[],因为没有删除元素
arr1;   // ['a','green','yellow','d']  注意是在索引1的<前面>添加新元素
```

### concat和join
* `concat()` 方法把当前的Array和另一个Array连接起来，返回一个新的数组
* `join()` 方法把当前Array的每个元素都用指定的字符串连接起来，然后返回连接后的字符串

```javascript
var arr = ['a','b','c'];
var added = arr.concat([1,2,3]);
added;  // ['a','b','c',1,2,3]
arr;  //['a','b','c']  原数组并未改变
arr.concat([1,2],3);  // ['a','b','c',1,2,3] 数组元素会被拉平
arr.concat([1,2,[3,4]]);  // ['a','b','c',1,2,[3,4]] 多重数组不会被拉平

var arr2 = ['a','b','c',1,3]
arr.join('-');   //返回字符串 a-b-c-1-3
arr;  //['a','b','c',1,3]  原数组并未改变
arr.toString();  //返回字符串 a,b,c,1,3
arr.join(',');  //返回字符串 a,b,c,1,3 同toString()方法
```
### indexOf和lastIndexOf
两者都接收两个参数：要查找的项和表示查找地点的索引（可选）；indexOf()从数组开头开始向后查找，lastIndexOf()从数组的末尾向前查找。若查找的项没找到则返回-1
```javascript
var arr = [1,2,3,2,1];
arr.indexOf(2);  //1
arr.indexOf(99);  //-1
arr.indexOf(1,1);  //4
arr.indexOf(1,-3);  //4
arr.indexOf(2,-1);  //-1
arr.lastIndexOf(2); //3
arr.lastIndexOf(2,-2)  //3
arr.lastIndexOf(2,-3)  //1
```
### 数组的迭代方法
ECMAScript5定义了5个数组的迭代方法,每个方法都接收2个参数：要在每一项上运行的函数和运行该函数的作用域对象-影响this的值（可选）。  函数会接收3个参数：数组项的值item，item项的索引index和该数组对象本身。

#### every()和some()
`every()` ：数组中每一项运行函数，都返回true，才返回true。
`some()` ：数组中每一项运行函数，有一项返回true，就返回true。

#### filter()
数组中每一项运行函数，返回 函数运行为true的项 组成的数组。

#### map()
数组中每一项运行函数，返回 每次调用函数返回的值 组成的数组，跟原数组一一对应。

#### forEach()
对数组中每一项运行函数，没有返回值

### 数组的归并方法
#### reduce()和reduceRight()
ECMAScript5定义了2个数组的归并方法，每个方法都接收2个参数：在每一项调用的值和作为归并的初始值（可选）。  函数会接收4个参数：前一个值，当前值，项的索引和数组对象本身。这个函数的返回值会作为第一个参数自动传个下一项。  第一次迭代发生在数组第二项上，因此第一个参数是数组第一项，第二个参数是数组第二项。
```javascript
var values = [1,2,3,4,5];
var sum = values.reduce(function(prev,cur,index,array){
	return prev + cur;
});
sum;  // 15
```
第一次执行回调函数，prev是1，cur是2。第二次执行，prev是返回值3，cur是第三项3。过程直至数组最后一项。
```javascript
var values = [1,2,3,4,5];
var sum = values.reduce(function(prev,cur,index,array){
	return prev + cur;
},6);
sum;  // 21
```
若传入第二个参数作为初始值（可为任意值），则第一次prev是6，cur是数组第一项1。
reduceRight()是反过来从最后开始执行。
