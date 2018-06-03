---
title: JS随写：数组排序去重
date: 2017-08-02 21:48:37
tags:
 - JavaScript
 - 随写
---
最近公司事情巨多，整日折腾公司的上古backbone框架，让我头疼不已，说好的最少1周1次blog的更新也多次delay，再努力吧，毕竟开头难，养成习惯也难。
### 数组对象
#### 按照数组中对象的某一属性大小进行排序
```javascript
array.sort(function(a,b){   //升序
	if(a.id === b.id){
		return 0;
	}else{
		return a.id < b.id ? -1 : 1;
	}
})
```
#### 按照数组中对象的某一属性是否重复进行去重
```javascript
//使用数组的reduce方法
var hash = {};
arr = arr.reduce(function(item, next) {
    hash[next.name] ? '' : hash[next.name] = true && item.push(next);
    return item
}, []);
// 拆解
var hash = {};
arr = arr.reduce(function(item,next){
	if(!hash[next.name]){
		hash[next.name] = true;
		item.push(next);
	}
	return item;
},[]);

//例子
var arr = [
	{
	    "name": "aaa",
	    "age": "16"
	}, 
	{
	    "name": "bbb",
	    "age": "20"
	}, 
	{
	    "name": "ccc",
	    "age": "15"
	}, 
	{
	    "name": "bbb",
	    "age": "30"
	}, 
	{
	    "name": "eee",
	    "age": "28"
	}];
var hash = {};
arr = arr.reduce(function(item, next) {
    hash[next.name] ? '' : hash[next.name] = true && item.push(next);
    return item
}, []);
console.log(arr);
// [{"name": "aaa","age": "16"},
//	{"name": "bbb","age": "20"},
//	{"name": "ccc","age": "15"},
//	{"name": "eee","age": "28"},
//]
```

### 纯数组
#### 排序
* js自带sort排序

```javascript
var arr = [1,8,3,4,5,9,3,6,];
arr.sort(function(a,b){
	return a - b;    //a-b升序，b-a降序
});
arr; //[1, 3, 3, 4, 5, 6, 8, 9]
```
<div class="tip">对于数值类型或者其valueOf()方法会返回数值类型的对象类型，可以使用以上方法 --《javascript高级程序设计》</div>

* 冒泡排序(选择排序？)

```javascript
var arr = [1,8,3,4,5,9,3,6,];
function bubbleSort(arr){
	for(var i=0;i<arr.length-1;i++){
		for(var j=i+1;j<arr.length;j++){
			if(arr[i]>arr[j]){
				var temp = arr[j];
				arr[j] = arr[i];
				arr[i] = temp;
			}
		}
	}
	return arr;
}
bubbleSort(arr);  // [1, 3, 3, 4, 5, 6, 8, 9]
```

* 快速排序

```javascript
var arr = [1,8,3,4,5,9,3,6,7];
function quickSort(arr){
	if(arr.length <= 1){  //数组剩下一个不比较，直接返回，不判断的话循环无法结束
		return arr;
	}
	var index = Math.floor(arr.length/2); //取中间数的index
	var cur = arr.splice(index,1);  //中间值，不能使用arr[index]替换
	var left = [],right = [];  //left存比中间小的，right存大的
	for(var i = 0;i<arr.length;i++){
		arr[i] < cur ? left.push(arr[i]) : right.push(arr[i]);
	}
	return quickSort(left).concat(cur,quickSort(right)); //递归，left和right自身多次比较排序
}
quickSort(arr);
```

* 插入排序

```javascript
var arr = [8,2,3,4,5,9,3,6];
function insersort(arr){
	for(i=1;i<arr.length;i++){
		temp = arr[i];
		j = i;
		while(j > 0 && arr[j-1] > temp){
			arr[j] = arr[j-1];
			j--;
		}
		arr[j] = temp;
	}
	return arr;
}
insersort(arr);
```
#### 去重

```javascript
var arr = [8,2,3,4,5,4,3,6,8];
function removeDup(arr){
	var newarr = [];
	for(var i = 0; i<arr.length; i++){
		if(newarr.indexOf(arr[i]) === -1){
			newarr.push(arr[i]);
		}
	}
	return newarr;
}
removeDup(arr);
```