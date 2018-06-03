---
title: 《offset、scroll、client知识总结》
date: 2018-04-25 21:20:47
tags:
 - 知识积累
 - JavaScript
---

获取元素位置，滚动高度，实现懒加载效果等等与元素位置有关的问题时，总绕不开`scrollTop`、`scrollWidth`、`offsetTop`等等属性名词，有的时候只用到某几个属性，但是没有通盘总结了解一下这几个看起来相似的属性，今天就来总结一下，顺带感受一下灵魂画师的绘画技巧...
<!-- more -->

# 偏移量offset
与偏移量有关的四个属性是：`offsetHeight`、`offsetWidth`、`offsetTop`和`offsetLeft`，先上个图简单指示一下：
![偏移量图解](/images/offset.png)
灵魂画手，使用ipad备忘录画的。以上可以清楚的看到这四个offset属性各指什么，可以将偏移量offset理解为“元素块”，offset就是指元素的实际大小，包括边框，包括滚动条（如果窗口不够大，出现scrollBar）,而且这个大小跟是否有隐藏的部分是无关的，实际多大，`offsetWidth`和`offsetHeight`就是指多大。另外，`offsetTop`和`offsetLeft`指的是元素块的外边框跟它外出元素块的内边框间的距离。
有时候还有

# 客户区client
与客户区有关的四个属性是：`clientWidth`、`clientHeight`、`clientTop`和`clientLeft`，上个图简单指示一下：
![客户区图解](/images/client.png)
需要注意的是，`clientWidth`和`clientHeight`均不包含滚动条的宽度，同时也跟是否有隐藏的部分无关，实际多大的窗口，就是多大（没有scrollBar）；而且，所谓的`clientTop`和`clientLeft`就是指元素对应的`border`大小。

# 滚动区scroll
与客户区有关的四个属性是：`scrollWidth`、`scrollHeight`、`scrollTop`和`scrollLeft`，上个图简单指示一下：
![滚动区图解](/images/scroll.png)
滚动的`scrollWidth`和`scrollHeight`也是不包括滚动条的，`scrollTop`和`scrollLeft`也分别如图所示。

网上还流传着一张更复杂的图，一并放到这里作参考：
![图解](/images/offset_client_scroll.jpg)



