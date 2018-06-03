---
title: 《DOM知识点总结》
date: 2018-04-17 20:17:56
tags:
 - 知识积累
 - JavaScript
---

DOM(文档对象模型)是针对HTML和XML文档的一个API，描绘了一个层次化的节点树，允许我们添加、移除和修改页面的某一部分。目前流行的各种前端框架，对于页面结构的调整更新构建都是基于DOM的原生API来实现的，而我们平时在进行前端开发时也或多或少会用到DOM原生操作，尤其是“抛弃”`jQuery`这个框架之后，原生的DOM操作越来越重要，今天就来总结一些常见的DOM属性和操作方法。
<!-- more -->

# Node类型
DOM将文档描绘成了一个多层次节点构成的结构，节点有很多的类型，每个类型又有不同的特点、数据和方法，节点之间也存在着相互的关系，同级关系，子父级关系等等。整个文档就是由一个个节点构建起来的树，就是我们经常说的DOM树。
每个节点（node）都有一个`nodeType`属性，表明节点的类型，下面先简单列举一下常见的节点类型：

* Node.ELEMENT_NODE(1)-----------元素节点，nodeType值为1
* Node.ATTRIBUTE_NODE(2)---------属性节点，nodeType值为2
* Node.TEXT_NODE(3)--------------文本节点，nodeType值为3
* Node.COMMENT_NODE(8)-----------注释节点，nodeType值为8
* Node.DOCUMENT_NODE(9)----------文档节点，nodeType值为9
* Node.DOCUMENT_TYPE_NODE(10)----doctype节点，nodeType值为10

可以通过节点的`nodeType`值来判断节点类型
```javascript
if(someNode.nodeType == 1){
  console.log("Node is an element.")
}
```
## 节点关系
节点之间存在着这样或那样的关系，每个node都有一个`childNodes`属性，其中保存着一个`NodeList`对象，有`length`属性，但不是一个`Array`实例，可以像下面一样访问里面存着的node:
```javascript
var fistChild = someNode.childNodes[0];
var secondChild = someNode.childNodes.item(1);
var count = someNode.childNodes.length;
```
还有一些属性：
* `someNode.nextSibling`         同级下一个节点
* `someNode.previousSibling`     同级前一个节点
* `someNode.firstChild`          节点的第一个子节点
* `someNode.lastChild`           节点的最后一个子节点
* `someNode.parentNode`          节点的父节点

## 节点操作方法
### 常见的操作方法
* `someNode.appendChild(newNode)`--------------在节点的childNodes末尾添加一个新节点，返回new
* `someNode.insertBefore(newNode, whoNode)`----在who前面添加一个new节点，返回new
* `someNode.replaceChild(newNode, whoNode)`----将who替换为new节点，返回who
* `someNode.removeChild(whoNode)`--------------将who移除，返回who

以上方法都是在父节点上的操作，没有子节点节点调用这些方法会报错。

### 两个特例方法
这俩方法有没有子节点都可以使用：
#### `cloneNode()`
这个方法用来复制调用它的节点，可以传一个参数true来进行深复制；
```javascript
//浅复制，只复制节点本身，不会复制节点内部
var copyNode = someNode.cloneNode(false);

//深复制，复制节点内部子节点
var deepCopy = someNode.cloneNode(true);
```
要注意的一点是，`cloneNode`的默认参数在不同的规范下，取true和false都有可能，建议强制写明参数，不使用默认值（其实也不知道默认值到底是啥）。

#### `normalize()` 
这个方法是文本节点(Text_Node)的专用方法，在someNode上调用时，如果在其后代节点找到了空文本节点，就删除这个空文本节点；如果找到了相邻的文本节点（一般来说直接写的html都只有一个文本节点，除非我们用createTextNode()这种方法创建文本节点，才有可能出现相邻的文本节点），则合并为一个节点。

# 元素节点ELement_Node
其实元素节点简单来说就是html中的标签，什么`p`标签，`h1`标签等等，先说一下它的特性：
* `nodeType`值为1；
* `nodeName`值为元素标签名；也可以使用`tagName`获取，注意大小写问题
* `nodeValue`值为null；它也没啥值好用的
* `parentNode`可能为`Document`或`Element`；文档的子节点或者其他`element`的子节点
* 子节点可能是：`Element`、`Text`、`Comment`等等；

## HTML元素
所有的HTML元素都是由HTMLElement类型表示，它就是继承自Element，然后添加了一些属性，其实感觉跟ELement一个东西，主要有以下属性：
* id;------------就是div的id么，标识符一个
* title;---------就是那个title
* lang和dir;-----很少用
* className;-----class的名字结合

其中`className`属性，这个注意一下，返回的是一个class名称的字符串合集，比如"item active test"，需要自己用字符串方法拆分操作它，后来HTML5就新增了一个`classList`属性，返回的是个class数组，比['item', 'active', 'test']，可以用数组的方式操作class名称，就很方便了。

`classList`属性有自己的几个操作方法：
* `add(value)`;----------将给定的字符串添加到列表，如果有，则不添加
* `contains(value)`;-----列表里面有没有给定的值，返回true/false
* `remove(value)`;-------删除列表中给定的值
* `toggle(value)`;-------没有就添加，有就删除

```javascript
div.classList.remove('item');
div.classList.add('current');
div.classList.toggle('active');
```

## 操作特性
操作div元素的属性，就是常见的几种方法，列举一下：
* `getAttribute()`;
* `setAttribute()`;
* `removeAttribute()`;

我们还经常自定义属性，比如在标签里定义个`id='1'`，HTML5规范建议自定义属性加上`data-`前缀，表明是自定的属性，这样可以使用`div.dataset.id`取得属性值。
还有一个`Attributes`属性，不展开说了，不怎么用到。

## 创建元素节点
可以使用`document.createElement()`方法创建元素节点。再配合node的操作方法，进行各种穿插。

# 属性节点Attribute_Node
属性节点不被认为是DOM文档树的一部分，简单说一下它的特性：
* `nodeType`值为2；
* `nodeName`值为特性的名称；
* `nodeValue`值为特性的值；
* `parentNode`为null
* 没有子节点

操作属性一般用上面的`getAttribute()`、`setAttribute()`、`removeAttribute()`。

# 文本节点Text_Node
文本节点简单说就是标签里的内容，比如`<div>hello world</div>`中的'hello world'，先说一下它的特性：
* `nodeType`值为3；
* `nodeName`值为`#text`;
* `nodeValue`值为节点包含的文本；
* `parentNode`可能`Element`；
* 没有子节点；

开始和结束标签之间只要有内容，就会创建一个文本节点：
```javascript
//没有文本节点
<div></div>

//有空格，故有一个文本节点
<div> </div>

//有内容，故有一个文本节点
<div>hello world</div>
var textNode = div.firstChild;
textNode.nodeValue;  // hello world
```

## 创建文本节点
可以通过`document.createTextNode('hello world')`创建一个文本节点，再配合node的操作方法，进行各种穿插。
一般来讲，一个元素只有一个文本节点，但是你要是硬搞几个相邻节点，也是可行的，然后就有了`normalize()`和`spliteText()`这俩方法：
```javascript
var element = document.createElement('div');
element.className = 'message';

var textNode = document.createTextNode('hello world');
element.appendChild(textNode);

var anotherTextNode = document.createTextNode('Yep!');
element.appendChild(anotherTextNode);   //这样就有了俩文本节点

alert(element.childNodes.length);   // 2
element.normalize();
alert(element.childNodes.length);      // 1
alert(element.firstChild.nodeValue);   // 'hello worldYep!'
```
`spliteText()`是将文本节点进行分割，具体细节不想讲了，碰见了再查。

# 文档节点Document_Node
文档节点表示整个HTML页面，`document`对象也是`window`对象的一个属性，说一下它的特性：
* `nodeType`值为9；
* `nodeName`值为`#document`;
* `nodeValue`值为null；
* `parentNode`值为null；
* 子节点可能是一个`DocumentType`、`Element`、`Comment`；

## 子节点
document的子节点也就是常见的`html`、`head`、`body`、`doctype`等，可以用下面的属性取得它们的引用：
```javascript
var html = document.documentElement;
var head = document.head;            // HTML5新定义的
var body = document.body;
var doctype = document.doctype;
```

## 文档信息
仅做了解：
```javascript
var originTitle = document.title;   // 取得页面的title
document.title = 'New Title';       // 设置页面title

var url = document.URL;             // 取得页面的完整URL
var domin = document.domin;         // 取得域名
var referrer = document.referrer;   // 取得来源页面的URL
```

## 查找元素
### `document.getElementById()`
根据id获取元素的引用，如果找不到，就返回null，只能使用`document`调用。

### `getElementsByTagName()`
根据标签名获取一个`HTMLCollection`元素列表，跟`NodeList`对象类似。可以使用`document`或者`element`调用
```javascript
var images = document.getElementsByTagName('img');   // document调用
images.length;    // 数量
images[0].src;
images.item(0).src;

var test = element.getElementsByTagName('a');    // element调用
```
还可以传入一个`*`来获取文档中的所有元素。

### `getElementsByClassName()`
根据class类名查找，可以使用`document`或者`element`调用
```javascript
var test = document.getElementsByClassName("item active");
var test1 = element.getElementsByClassName("item");
```

## DOM的扩展
DOM还有一个`SelectorsAPI`的扩展，另一个是`HTML5`，下面说一下`SelectorsAPI`，都可以通过`element`或者`document`调用。

### `querySelector()` 
接受一个CSS选择符，返回匹配到的第一个元素，没有找到就返回null。
```javascript
var body = document.querySelector('body');
var myDiv = document.querySelector('#myDiv');
```

### `querySelectorAll()` 
同样接受一个CSS选择符，返回一个`NodeList`实例。
```javascript
var ems = document.getElementById('myDiv').querySelectorAll('em');
var strongs = document.querySelectorAll('p strong');
strongs.item(1);
strongs[2];
```

## 文档写入
* `wirte()`;
* `wirteIn()`;-----跟`write()`一样，只不过会在写入的字符串最后添加一个'/n'
* `close()`;
* `open()`;

HTML5新增了`innerHTML()`和`outerHTML()`两个方法，不同的是`outerHTML()`会包含调用元素本身。另外一个需要了解的方法是`contains()`，用来判断某个节点是不是调用节点的子节点：
```javascript
document.documentElement.contains(document.body);   // true
```

本次先总结这些，下次我们来总结一下原生操作样式`div.style.color`和关于`offsetWidth`、`scrollTop`等的知识点。





































