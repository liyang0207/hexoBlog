---
title: 百度编辑器UMeditor图片缩放bug
date: 2017-08-10 23:42:51
tags:
 - CSS
 - 工作积累
---
今天一早公司市场部同事就找过来说项目的编辑器图片无法缩放，心想着估计又是公司上古神兽“backbone”触发什么隐藏bug了，复现了一下bug：上传到百度编辑器的图片拖曳缩放的话会无法操作，或者会有一些奇奇怪怪的问题。查看了下百度编辑器模块的config文件和组件源码，却没发现什么明显的问题。

折腾半天，审核下元素看到图片继承的属性有box-sizing:border-box，随手勾掉，发现缩放竟然正常了。百度一下“百度编辑器+bootstrap”，还真有人遇到同样问题，两者一起使用就会出现图片缩放的bug。问题定位之后，直接在UMeditor样式文件：ueditor.min.css头部添加如下样式，将box-sizing属性复原即可。
```css
.edui-container *{-webkit-box-sizing: content-box;-moz-box-sizing: content-box;box-sizing: content-box;}  
.edui-container *:before,.edui-container *:after {-webkit-box-sizing: content-box;-moz-box-sizing: content-box;box-sizing: content-box;}
```