---
title: 《微信小程序开发之new Date()参数格式问题》
date: 2017-12-15 20:26:43
tags:
 - 小程序
 - 工作积累
---

### 概述
小程序开发中遇到时间对比的情况，根据接口返回的时间来判断订单是否过期。后台返回的时间是这样的字符串格式: `2017-12-15 20:30:47`,以下为比较代码
```javascript
let now_time = new Date();
let end_time = new Date(res.data.end_time);
if(now_time.getTime() > end_time.getTime()){
  <!-- 结束时间小于当前时间 -->
  ...
}
```
在微信开发者工具中进行调试运行没有问题，判断是正确的。但是到真机上调试判断却没有生效，尝试了多次，才发现微信对于`2017-12-15`这样的格式可以正常识别，但是加上时间就会出现无法识别的问题。故对时间格式进行了一步转化，转为`yyyy/mm/dd hh:mm:ss`格式。
```javascript
let now_time = new Date();
let end_time = new Date(res.data.end_time.replace(/-/g, '/'));
if(now_time.getTime() > end_time.getTime()){
  <!-- 结束时间小于当前时间 -->
  ...
}
```
再次测试，开发工具和真机调试都可以正常进行判断。
<div class="tip">基于微信web页面的开发，此问题一样存在</div>