---
title: js随写：正则表达式总结
date: 2017-09-05 20:38:33
tags:
 - JavaScript
 - 随写
---
### 概览
正则表达式是个比较深的知识点，往深的说、细的说能写本书。近日看了一篇大神对正则的总结和介绍，自己对正则这块又有了一些新的理解和认识。刚巧自己也想把正则的一些知识点整理一下，汇总一下，方便自己以后查阅。这里基本上只写结论，例子会比较少，日后工作中多用，多写慢慢熟悉就好。

正则是匹配模式，要么匹配字符，要么匹配位置。这个理念很重要
### 匹配字符
关键字：两种模糊匹配，字符组，量词
#### 模糊匹配
* 横向匹配：正则可匹配的字符串长度不确定，使用量词来实现。如：{m,n},*,?,+

```javascript
var reg = /ab{2,3}/g;  //b重复2次到3次
```
* 纵向匹配：正则匹配字符串，具体到某一位字符时，可以是不确定的字符。使用字符组来实现。如：[abc]表示abc中的任意一个

```javascript
var regex = /a[123]b/g;
var string = "a0b a1b a2b a3b a4b";
console.log( string.match(regex) ); //["a1b", "a2b", "a3b"]
```
#### 字符组
字符组只能匹配当中的某一个字符，如[abc]表示a、b、c中的一个。
* 范围表示法。如：[a-z],[0-9],[a-m]等等。
* 排除字符法。在字符组内添加^符号，表示取反。如[^abc]表示除abc3个字符外的所有其他字符。
* 常见的简写形式：
    `\d`就是`[0-9]`，表示数字
    `\D`就是`[^0-9]`，表示除数字外的任意字符
    `\w`就是`[a-zA-Z0-9_]`，表示数字，大小写字母，下划线。也称单词字符
    `\W`就是`[^a-zA-Z0-9_]`，表示除数字，大小写字母，下划线外的其他任意字符
    `\s`就是`[ \t\v\r\n\f]`，包括空格、水平制表符、垂直制表符、回车符、换行符、换页符。
    `\S`就是`\s`取反，非空白符号
    `.`就是`[^\n\r\u2028\u2029]`，通配符，表示几乎任意字符，换行符、回车符、行分隔符和段分隔符除外
如果要匹配任意字符，可以使用`[\w\W]`表示

##### 量词
`{m,n}`表示连续出现最少m次，最多n次
`{m,}`表示至少出现m次
`{m}`等价于`{m,m}`，出现m次
`?`表示出现0次或1次
`*`表示出现0次或任意多次
`+`表示出现1次或任意多次

#### 多选分支
一个模式可以实现横向和纵向模糊匹配。而多选分支可以支持多个子模式任选其一。
具体形式如下：(p1|p2|p3)，其中p1、p2和p3是子模式，用|（管道符）分隔，表示其中任何之一。

### 匹配位置
位置是相邻字符之间的位置。如'hello','h'和'e'之间就是一个位置
在ES5中，共有6个锚字符：
`^`（脱字符）匹配开头，在多行匹配中匹配行开头。
`$`（美元符号）匹配结尾，在多行匹配中匹配行结尾。
`\b`表示是单词边界，具体就是`\w`和`\W`之间的位置，也包括`\w`和`^`之间的位置，也包括`\w`和`$`之间的位置。
`\B`就是`\b`的反面的意思，非单词边界。
`(?=p)`其中p是一个子模式，即p前面的位置。
`(?!p)`就是`(?=p)`的反面意思。

```javascript
var result = "hello".replace(/^|$/g, '#');  //把字符串的开头和结尾用"#"替换
console.log(result); // => "#hello#"

var result = "I\nlove\njavascript".replace(/^|$/gm, '@');  //多行匹配模式时，二者是行的概念
console.log(result);
//  @I@
//  @love@
//  @javascript@

var result = "[JS] Lesson_01.mp4".replace(/\b/g, '#');  // \b之间替换为#
console.log(result); // => "[#JS#] #Lesson_01#.#mp4#"

var result = "[JS] Lesson_01.mp4".replace(/\B/g, '#');  // 非\b之间替换为#
console.log(result); // => "#[J#S]# L#e#s#s#o#n#_#0#1.m#p#4"

var result = "hello".replace(/(?=l)/g, '#');  //l前面的位置
console.log(result); // => "he#l#lo"

var result = "hello".replace(/(?!l)/g, '#');  //非l前面的位置
console.log(result); // => "#h#ell#o#"
```
"hello"字符串等价于如下的形式:
`"hello" == "" + "h" + "" + "e" + "" + "l" + "" + "l" + "o" + "";`
或者：
`"hello" == "" + "" + "hello"`
<div class="tip">对于位置的理解，我们可以理解成空字符""。字符之间的位置，可以写成多个。</div>

### 正则分组，括号的作用
* 数据提取，进行替换操作

```javascript
var regex = /(\d{4})-(\d{2})-(\d{2})/;
var string = "2017-06-12";
console.log( string.match(regex) ); 
// => ["2017-06-12", "2017", "06", "12", index: 0, input: "2017-06-12"]
```
`match`返回的一个数组，第一个元素是整体匹配结果，然后是各个分组（括号里）匹配的内容，然后是匹配下标，最后是输入的文本。（注意：如果正则是否有修饰符g，match返回的数组格式是不一样的，带g返回一个结果数组）。
另外也可以使用正则对象的exec方法:
```javascript
var regex = /(\d{4})-(\d{2})-(\d{2})/;
var string = "2017-06-12";
console.log( regex.exec(string) ); 
// => ["2017-06-12", "2017", "06", "12", index: 0, input: "2017-06-12"]
```
进行过正则操作后，就可以使用$1-$9来提取分组数据
```javascript
var regex = /(\d{4})-(\d{2})-(\d{2})/;
var string = "2017-06-12";

regex.test(string); // 正则操作即可，例如
//regex.exec(string);
//string.match(regex);

console.log(RegExp.$1); // "2017"
console.log(RegExp.$2); // "06"
console.log(RegExp.$3); // "12"
```

* 反向引用：可以使用\1、\2、\3等来指代正则中的分组

```javascript
var regex = /\d{4}(-|\/|\.)\d{2}\1\d{2}/;
var string1 = "2017-06-12";
var string2 = "2017/06/12";
var string3 = "2017.06.12";
var string4 = "2016-06/12";
console.log( regex.test(string1) ); // true
console.log( regex.test(string2) ); // true
console.log( regex.test(string3) ); // true
console.log( regex.test(string4) ); // false
```
\1表示的引用之前的那个分组(-|\/|\.)。不管它匹配到什么（比如-），\1都匹配那个同样的具体某个字符。

### 正则的方法
#### String字符串方法
* search
* split
* replace
* match
`match`返回结果的格式，与正则对象是否有修饰符g有关。

```javascript
var string = "2017.06.27";
var regex1 = /\b(\d+)\b/;
var regex2 = /\b(\d+)\b/g;
console.log( string.match(regex1) );
console.log( string.match(regex2) );
// => ["2017", "2017", index: 0, input: "2017.06.27"]
// => ["2017", "06", "27"]
```
没有g，返回的是标准匹配格式，即，数组的第一个元素是整体匹配的内容，接下来是分组捕获的内容，然后是整体匹配的第一个下标，最后是输入的目标字符串。
有g，返回的是所有匹配的内容。

#### RegExp方法
* test
注：test整体匹配时需要使用^和$
* exec
`exec`方法不管有没全局标志g,每次都只会返回一个匹配项。没有g，在同一字符串上多次调用exec()将始终返回第一个匹配项的信息；有g，每次调用exec()都会在字符串中继续查找新的匹配项，这是跟`match`方法明显的一个不同点。

```javascript
var string = "2017.06.27";
var regex2 = /\b(\d+)\b/g;
console.log( regex2.exec(string) );
console.log( regex2.lastIndex);
console.log( regex2.exec(string) );
console.log( regex2.lastIndex);
console.log( regex2.exec(string) );
console.log( regex2.lastIndex);
console.log( regex2.exec(string) );
console.log( regex2.lastIndex);
// => ["2017", "2017", index: 0, input: "2017.06.27"]
// => 4
// => ["06", "06", index: 5, input: "2017.06.27"]
// => 7
// => ["27", "27", index: 8, input: "2017.06.27"]
// => 10
// => null
// => 0
```

### 正则修饰符
* g 全局匹配，即找到所有匹配的，单词是global
* i 忽略字母大小写，单词ingoreCase
* m 多行匹配，只影响^和$，二者变成行的概念，即行开头和行结尾。单词是multiline

本文基本上是基于掘金大神@老姚发布的正则火拼系列文章而来，我自己精简了一些内容，挑选了一些我目前能看懂希望掌握的知识点。原文还涉及许多正则的深入知识，待以后工作中接触后再逐步学习。文章传送门：[JS正则表达式完整教程（略长）](https://juejin.im/post/5965943ff265da6c30653879)





















    