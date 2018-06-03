---
title: JS随写：Date类型
date: 2017-07-12 20:25:41
tags:
 - JavaScript
 - 随写
---
### Date
要创建一个日期对象，用`new`操作符和Date构造函数

### 获取系统时间
* 获取当前时间：直接new Date()
* 根据毫秒数获取时间：传入毫秒数
* 创建指定时间：传入指定时间

```javascript
var time1 = new Date();  //获取系统当前时间
time1;  //Sat Jul 08 2017 14:55:09 GMT+0800 (中国标准时间)

var time2 = new Date(1499496964457);  //传入毫秒数获取指定时间，一般后台数据会以时间戳给你，注意时间戳是秒，不是毫秒
time2;  //Sat Jul 08 2017 14:56:04 GMT+0800 (中国标准时间)

var time3 = new Date(2017,6,8,15,02,30);  //传入指定时间来创建标准时间格式，注意月份从0开始
time3;  /Tue Jul 08 2017 15:02:30 GMT+0800 (中国标准时间)
```
以上可以使用Date类型的方法:
```javascript
time.getTime(); // 1499496964457, 毫秒数，与valueOf()方法返回的值相同
time.getFullYear(); // 2017, 年份
time.getMonth(); // 6, 月份，注意月份范围是0~11，6表示七月
time.getDate(); // 08, 表示8号
time.getDay(); // 6, 表示星期六
time.getHours(); // 15, 24小时制
time.getMinutes(); // 02, 分钟
time.getSeconds(); // 30, 秒
time.getMilliseconds(); // 875, 毫秒数
```

### 获取时间戳
* `Date.parse()` 接受几个指定格式的`字符串`参数
 - 月/号/年；     例如：7/8/2017
 - 英文月 号,年;     例如：July 8,2017
 - 标准格式；     例如：Tue Jul 08 2017 15:02:30 GMT+0800
 - ECMA5支持格式YYYY-MM-DDTHH:mm:ss.sssZ；  例如：2017-07-08T15:02:30

```javascript
var d = Date.parse('7/8/2017');  //毫秒数1499443200000
var d = Date.parse('July 8,2017');  //毫秒数1499443200000
var d = Date.parse('Tue Jul 08 2017 15:02:30 GMT+0800');  //毫秒数1499497350000
var d = Date.parse('2017-07-08T15:02:30');  //毫秒数1499497350000
```
*  `Date.UTC()` 直接接受参数(年,基于0的月,日,时,分,秒,毫秒)

```javascript
var d = Date.UTC(2017,7,8,15,02,30);   //毫秒数 1502204543000
//注意，时间戳是基于1970年1月1日凌晨，全世界都一样，只是转化成本地时间会有时区差别
var time = new Date(1502204543000);  //Tue Aug 08 2017 23:02:23 GMT+0800 (中国标准时间)
```

### 调用时的当前时间
ECMAScript 5新增 `Date.now()`方法，返回调用这个方法时的日期和时间的`毫秒数`

```javascript
var start = Date.now();  //取得开始时间
dosomething();   //调用函数
var end = Date.now();  //取得结束时间
var result = end - start;  //毫秒数
```
### 杂七杂八
* 若直接将表示日期的`字符串`传给Date构造函数也会在后台调用Date.parse()进行转换。

```javascript
var time = new Date('7/8/2017'); //等同于new Date(Date.parse('7/8/2017'));
time;  // Sat Jul 08 2017 00:00:00 GMT+0800 (中国标准时间)
```
* 上面的 `var time3 = new Date(2017,6,8,15,02,30)` 同理使用了`Date.UTC()`转化