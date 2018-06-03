---
title: 《微信小程序开发之自定义tips组件》
date: 2017-12-03 15:45:50
tags:
 - 小程序
 - 工作积累
---
目前小程序官方提供的交互反馈组件只有Toast、Loading、Modal这3种。
#### wx.showToast()
默认有success和loading两种图标可选，也可以自定义图标，设置好duration持续时间后可自动消失，或者主动调用wx.hideToast()来关闭。title设置过长会被隐藏部分内容。
```javascript
wx.showToast({
  title: '成功',
  icon: 'success',
  duration: 2000
})
```
#### wx.showLoading()
loading组件无图标可选，只有转圈圈的图标可用，而且必须调用wx.hideLoading()来关闭。同样title设置过长文字会被隐藏。
```javascript
wx.showLoading({
  title: '加载中',
})
...
wx.hideLoading()
```
#### wx.showModal()
Modale组件功能就更丰富了一些。可以设置title、content、confirm和cancel按钮的内容和颜色，还有各自的回调函数等等。title和content设置过长也会被完整显示。在上一篇的《微信小程序开发之授权逻辑》中就用到了这种弹窗来让用户进行选择操作。
```javascript
wx.showModal({
  title: '提示',
  content: '这是一个模态弹窗',
  success: function(res) {
    if (res.confirm) {
      console.log('用户点击确定')
    } else if (res.cancel) {
      console.log('用户点击取消')
    }
  }
})
```
不过在开发过程中，总有一些提示性的信息需要一个更舒服的提示组件来展示。比如提示用户电话号码输入错误、搜索内容不能为空、无法打开地图等等。这些提示如果用上述的3种组件来展示，会很难受，也不优美，所以需要我们自己去开发一个新的提示组件，来满足我们项目的需求。
#### 个人提示组件myTopTips
网上其实有很多大厂开发的微信组件ui，可以直接集成到项目中，或者只提取出我们需要的部分，以下我使用的topTips组件来源于有赞开源的组件，[zanui-weapp](https://github.com/youzan/zanui-weapp)，基于其topTips组件，我将其样式做了修改，添加了3种不同的提示类型：success、warning、error，基本上满足了我在项目中的使用。
项目中新建一个page，命名为mytips，记得将其引用路径从`app.json`中删除，不然会有很多错误。参看我的另一篇博客《微信小程序开发之踩坑记录》。然后编辑`mytips.js`文件
```javascript
<!-- mytips.js文件 -->
module.exports = {
  showMyTips(content = '', icon = '', options = {}) {
    let MyTips = this.data.MyTips || {};
    // 如果已经有一个计时器在了，就清理掉先
    if (MyTips.timer) {
      clearTimeout(MyTips.timer);
      MyTips.timer = undefined;
    }
    if (typeof options === 'number') {
      options = {
        duration: options
      };
    }
    // options参数默认参数扩展
    options = Object.assign({
      duration: 3000
    }, options);
    // 设置定时器，定时关闭topTips
    let timer = setTimeout(() => {
      this.setData({
        'MyTips.show': false,
        'MyTips.timer': undefined
      });
    }, options.duration);
    // 展示出topTips
    this.setData({
      MyTips: {
        show: true,
        content,
        icon,
        options,
        timer
      }
    });
  }
};
```
接下来编辑wxml和wxss文件
```html
<!-- mytips.wxml文件 -->
 <template name='mytips'>
  <view class="mytips {{ MyTips.show ? 'mytips-show' : '' }}">
    <view class='mytips-box'>
      <text class='mytips-{{MyTips.icon}}'>{{ MyTips.content}}</text>
    </view>
  </view>
</template> 
```
```css
<!-- mytips.wxss文件 -->
<!-- 图标使用了base64的背景图 -->
.mytips{position:fixed;top:10rpx;left:0;right: 0;z-index: 999;display: flex;justify-content: center;-webkit-transform:translateZ(0) translateY(-150%);transition:all .3s ease}
.mytips-show{-webkit-transform:translateZ(0) translateY(0)}
.mytips .mytips-box{padding: 12rpx 28rpx;background: #fff;box-shadow:0 2px 8px rgba(0,0,0,.2);border-radius: 8rpx;height:36rpx;}
.mytips .mytips-box text{padding-left:40rpx; font-size: 24rpx;line-height: 36rpx;height: 36rpx;display: block}
.mytips .mytips-box text.mytips-success{background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABwAAAAcCAYAAAByDd+UAAADh0lEQVRIia2WPYhcVRTHf+fc92ZXO4PBRkgsYiOaQIxEEXaJ4LIrCTbpIgQJiYQtLaIgBCstbCyCBsRCbbYJISQbP4gs+AEaggS2iIKspQEVETTz3rvnbzGzyc5zZrNj5hQP3rn3/H/349x7j00vA4CZY14CggiyQd3JlE2BZz+gFPOG7THZLmBbL4rfZfpJ6AfLvhwprtRFQ1klkgB3wFDUSNHjjAYKJTuZwk8a9phhCDHM1tuEVrPHGcs6k2RbA0oC6VGMs25phpGYYeDeN5RXEMcx+9FsEOiDIQJYMPfrZj6jMWDr0UKY+Yy5XwcWaCm0gMyZ2UVgagzOKJvqa82NAu4EX54AqGW+3NPu/3WnoeoY5sUSyCaBMBl3hGTmxVLVMbrT4Erg+FHH900K1njzeiaeA/4CcHyf40eVwMovCsrwNcN2jJMgw8zldFP1du35VKcpKfCbwHYDhH6pPXZ6yj5r+ARgRjdV79RFPlU0hSfsG4Pt0MtTw3ek7LOuFAftHmkmo0r1W3XKrxZVaR2lrx17WgN9QCkOeor0ZNwD0TCyxwdVmV9D0In0lRn724phIkXa68CO/w+DkD6uLR9DMB3leTOe2WT4DzvwYMv5pyz2GizaJqekB+PDypuXcspM1eX5IqdD2ny1HiqGOLPgWmNxzbHHHTvRlliHdb15OcrMVFOeK6M4FBabwQCSA7+1nNtc6dtMwz9F9UqWPt04TwMy+uTWbVhxrszFi1uAAfzqwFrba7C/UPoIoPGYB1bXoULLQkfcxVRTLI0BA1jz7Pmqt/ZKCCcduS933nRQtjgg1AhdDbSQMKZz+V6Ri8NbzXCXkT1fc8t+YVRuJPkbKfzYrdTcrFLzhIlnvXdLnhWcuEuCDE7CwLJfuOvV5jL+LqrnMX1+f90hTO+DHd8yid6+377a6k5DlfLpjff74MjEVBQXy5wOC94dF9YDGlXKp+tOg9mXYGFM5/I7sKEvhvXIjLOErWF/fyvVT8k1UNPsNO/8PKk38Y4ZiuoRKdZg8MVfg1iYLAwg5thw9No1zWVJLwDdCZC6SPPAZxudLaABXJJitxQrxqhUGm5Gv0ZVrEixG7hMS6E9w16QdCMsZmtvFgOt0hcaDeq1BVqtvVkMi1mTbgzt+5/KWwIF2aFOmVL9Ut9iwbA9hu0CHujH/yH6pb78UqS4UltDmRMpAHNoFcL/AsvcwfKVBFWTAAAAAElFTkSuQmCC) no-repeat left center/28rpx 28rpx;} 
.mytips .mytips-box text.mytips-warning{background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABwAAAAcCAYAAAByDd+UAAACRElEQVRIib3WT2sVZxTH8c95xpCWFhQVuilo6Z9dVUQQRBClatNFuyt05dK2vom8iuJWXGRtF9WWkGwtFFp1JdJE6KoLS7spGuc+x8Xca9Jrkju3cvODgZnnnPN8zzwzz3lO5JKXqhVJFCLxHPvI4oJqQTiBD3FwGPIXHkm/KW5HtaLFHIKanVMpm4zYFTjwbRZfCx+LzratRrb0IKrrGt9NC/xIuo7zOyAmaTXC1ZoejQPLK67hU+n+a8DgfKZ7uDRuGAdeCm5jfpfJ2kjHo3VGGuzi92bwIz7ZOrh1Sd/Lam1i7vyDAxLhb+yf4F+j+ADrUDzFBlkt7R63mSQaRTO8n6SS1ZINPKUYkFwRTvcEdtrpj90+xdPJFQOKeeyzONUE0yqHjHlKNs7h6AxxIx3Nxrli4PNeX+J1FRj4oihOzXQ5R0oUJwuO7AFupHcLDu8h8J1XS9ts1RQ82UPgnwWP9xD4uKh+mXJbdCdld9XeUYHq16Lx/ZTb4m2cxUm8NVWajVuRN8li3eyrzVpU7xfP0FqcabUpGFi0oTtkItzAzz3D2+BIdAWj7RVR3Y3qpqB4A3NE+Eq/nyAzNZka/Q6pNlpfqnTATa0nl3tMMCesCWtdqhOyYwF/jJ7LmHU50wKe9QBP0r+ZLmN56+B2pe2OcFxY/V+YrkddVh0Tfho371RLH0a6oLqGB1Pg7qm+iXRR+n3bfP7TCA+bvlHnnc+JZtjqp8+EY7pW/9Aw5Imu1b8v/BDVStYuPpNshm/V6upS8AJ22r+NeHvxEQAAAABJRU5ErkJggg==) no-repeat left center/28rpx 28rpx;}
.mytips .mytips-box text.mytips-error{background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABwAAAAcCAYAAAByDd+UAAACD0lEQVRIibWWz0tVQRTHP5NCJLoIqk3/RxBEUGKLQCjaFrgICWwhlBKC4KqNv/BHqzZFIChoi1qGC1dCLdpFi+CBYaKIiIqrOafFnZvXaaY743sNfLmPmXPO587c884co/w9yjlTPC4o9AI9wHXgKtBWMd8G1g2sKnwwsFPGMIHYaEBS6LzAoMCGgCZqX2FUoUsisWPAWwI/MkC+fgrcSQX2NQE6JYUndcBHrYJVoANBoMC1VsMq0B4f2Caw6Rl+FVg6A+CjwGdv7lCLbP+zu8GA44J7mRepMAszzud1YP1lCewQ2I8cxUgqtAIbjthYhcsI3K85/1EpAj1vAlaqH4E3CW8/5aDPAmtzsbWA3iPwLfX7iLcLC/MZMBXYQGAvJykcYM7Covs9lOovcITAUYaDWli2J3+n+RxfgYP2UEGvGca4Ag9cCt4I/xq5R6onR7pgAQuTGTs8ROB7JqyaNNMOOpEIbCDwNgEWTX0LUw46nQBcROBBDexVDFaxGXfQmRrgQwQ6Y5lqYVYSU9/CRM1OjxUulu1EKOCy1JS0AHTSxov3WPV6Oiew7Rl80YSyF9A7hU/e3N6p68ldUTfOEDxJCt2xFuPxf4A9rWui+lsIG/Djx9rEboFGE7CGwt2cvhQpOoFhgV8ZoC2BEYGOUFwFjAbqqzfXCdwDbgM3gSue+S6wBqwaWFE4gEibD/wGOu7Ykd5d0vYAAAAASUVORK5CYII=) no-repeat left center/28rpx 28rpx;}
```
文件编辑完之后，就是页面的引用了，其实现在来看，引入自定义组件还是比较麻烦的一件事，每个用到的页面都需单个引入，异常麻烦。不过我现在也没有更好的解决方案，只能先这样使用了。
假如我们要在index页面中使用这tips组件，则需要分别在`index.wxml`和`index.js`中引入该组件的内容，样式文件`mytips.wxss`直接放到了`app.wxss`中进行引入，所以该组件的样式会在所有小程序页面中覆盖到，建议取的class类名个性一点，防止影响页面样式。
```css
<!-- app.wxss文件中全局引入mytips.wxss样式 -->
@import "/component/mytips/mytips.wxss";
...
```
```html
<!-- index.wxml中引入mytips的模板 -->
<import src="path/to/component/mytips/mytips"/>
<template is="mytips" data="{{ MyTips }}"></template>
<view class='container'>
...
<view>
```
```javascript
<!-- index.js中需要这样引入，修改Page({})为如下，其他数据不会受到影响 -->
const MyTips = require('path/to/component/mytips/mytips');
Page(Object.assign({}, MyTips, {
  data:{  },
  onLoad: function(){  }
}))
```
如上引入完毕后，在页面中要使用该tips提示时，只需在js方法中
```javascript
<!-- 第二个参数可选为：success,warning,error；第三个参数为显示的时间 -->
showTips: function(){
  this.showMyTips('这里填需要提示内容', 'warning', 2000);
}
```
最后的提示效果如下：
![warning.png](http://upload-images.jianshu.io/upload_images/6589697-74b7491f6c846995.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![error.jpg](http://upload-images.jianshu.io/upload_images/6589697-c61685043f471b59.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
还有一个success的样式没有截到（因为这个很少使用到，就不找了），为一个绿色背景的白色对勾图标。
