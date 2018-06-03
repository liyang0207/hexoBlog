---
title: 《微信小程序开发之自定义轮播dot》
date: 2017-12-15 20:55:46
tags:
 - 小程序
 - 工作积累
---

### 概述
小程序内置了`swiper`组件，组件的dots默认为黑灰色的，简单的使用还可以，但是无法满足我们更深入的需求，改变一下样式、交互效果之类时就需要自定义dots。

#### 解决方式
首先将`swiper`的`indicator-dots`属性设为false，接下来直接贴代码
```html
<!-- index.html -->
<view class='swiper-container'>
  <swiper class="swiper" indicator-dots="{{false}}" autoplay="{{true}}" interval="3000" circular="{{true}}" bindchange="swiperChange">
    <block wx:for="{{banners}}" wx:key="{{item.id}}">
      <swiper-item>
        <image src="{{item.images}}" class="slide-image" mode="aspectFill"/>
      </swiper-item>
    </block>
  </swiper>
  <view class="dots">  
    <block wx:for="{{banners}}" wx:key="{{item.id}}">
      <view class="dot {{index == current ? 'active' : ''}}"></view>
    </block>
  </view>  
</view>
```
```css
<!-- index.css -->
.swiper-container{position: relative;}
.swiper-container swiper{height: 350rpx;width: 100%;} 
.swiper-container swiper image{width: 100%;height: 100%;display: block}
/* 轮播点样式 */
.swiper-container .dots{position: absolute;left: 0;right: 0;bottom: 18rpx;display: flex;justify-content: center;}
.swiper-container .dots .dot{margin: 0 6rpx;width: 12rpx;height: 12rpx;background: #fff;border-radius: 8rpx;transition: all .6s;}
.swiper-container .dots .dot.active{width: 26rpx;background: #fd2c4e;}
```
```javascript
<!-- index.js -->
Page({
  data: {
    banners: [],
    current: 0
  },
  onLoad: function(){
    //获取主页banner图
    wx.request({
      url: 'https://xxxx.xxxx',
      method: 'GET',
      success: res => {
        _this.setData({
          banners: res.data
        })
      }
    })
  },
  // 轮播图切换事件
  swiperChange: function (e) {
    this.setData({
      current: e.detail.current
    })
  },
})
```
