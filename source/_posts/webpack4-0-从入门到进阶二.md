---
title: webpack4.0 从入门到进阶二
date: 2018-08-06 21:08:52
tags:
  - webpack
  - 随写
---

上一篇讲了webpack配置Html模板和css文件的方法，配置Html模板需要用到`html-webpack-plugin`插件，可以选择一个静态html文件作为模板；而配置css文件分为使用`<style></style>`标签插入样式和使用`link`方式引入css文件，这里面依次要使用到`css-loader`、`style-loader`和`less-loader`(根据css预处理语言的不同)。`webpack.config.js`文件的配置项比较多，比较复杂，需要我们根据不同的场景去查阅相应的文档去实现功能。今天这篇我们继续来讲`webpack4.0`的配置。

### 清理dist文件夹

开始之前我们先来配置`clean-webpack-plugin`插件，这个插件的作用是在每次生成`dist`文件的时候先将该文件夹清空，然后再生成新的静态文件：

```
npm i clean-webpack-plugin -D
```

安装完后进行简单的配置：

```javascript
...
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
  ...
  plugins: [
    ...
    // 打包前先清空dist文件
    new CleanWebpackPlugin('dist')
  ]
}
```

这样每次运行`npm run build`就不会重复在dist文件夹内生成文件了。

### webpack-dev-server

为了方便我们本地调试代码，我们抽两分钟时间来具体配置一下`webpack-dev-sever`，上一篇的时候我们已经安装好了`webpack-dev-server`，我们现在做以下配置：

```javascript
module.exports = {
  mode: 'development',
  devServer: {           //配置此静态文件服务器，可以用来预览打包后项目
    inline:true,         //打包后加入一个websocket客户端
    contentBase: path.resolve(__dirname, 'dist'),     //开发服务运行时的文件根目录
    host: 'localhost',   //主机地址
    port: 4200,          //端口号
    compress: true       //开发服务器是否启动gzip等压缩
    open:true,           // 自动打开浏览器
  },
  entry: './src/index.js',
  ...
}
```

具体其他配置可以去查看文档，以上的配置已经足够我们来使用。

还有一点要注意的是，`webpack4.0`新增了`mode`配置，可选`development`/`production`两种模式，还记得`vue`脚手架里面的`webpack.dev.conf.js`和`webpack.prod.conf.js`吗？就是为了区分不同的模式进行了配置的拆分，现在`4.0`加入`mode`选项，也是对社区最优选择的一种肯定，接下来会再写博客单独对这里进行探究。

### 图片资源的配置

处理图片的时候，我们需要另外的`loader`，先使用npm安装：

```
npm i url-loader file-loader -D
```

先来说下这两个`loader`的区别，`file-loader`可以解析项目中的url引入（不仅限于css），根据我们的配置，将图片拷贝到相应的路径，修改打包后文件引用路径，使之指向正确的文件。`url-loader`相当于`file-loader`的又一层封装，且`url-loader`可以单独使用，不必依赖于`file-loader`，`url-loader`的作用是在图片大小小于`limit`值的时候将图片转为`base64`图片，大于`limit`的时候使用`file-loader`来处理图片，这样可以减少图片的`http`请求数量。来看看例子：

```javascript
module: {
    rules: [
      ...
      {
        test: /\.(jpg|png|jpeg|gif)$/,
        loader: 'file-loader',
        options: {
          // 这里将图片资源统一放入dist/static/img内
          name: 'static/img/[name].[ext]'
        }
      },
      // 或者使用url-loader
      {
        test: /\.(jpg|png|jpeg|gif)$/,
        loader: 'url-loader',
        options: {
          // 小于8k的图片转为base64
          limit: 8192,
          name: 'static/img/[name].[ext]'
        }
      }, 
      // url-loader也可以使用query的形式带入配置参数
      {
        test: /\.(jpg|png|jpeg|gif)$/,
        loader: 'url-loader?limit=8192&name=static/img/[name].[ext]'
      }, 
    ]
  },
```

写到这里的时候其实卡了很久，卡在了`publicPath`和样式内引入图片的问题（background-image）,这个后面可能会单独拿出来讲一下`path`和`publicPath`的区别和作用，涉及到了后台部署前端代码的一些问题，需要好好研究一番。

不光图片资源，我们的视频文件和字体文件也使用到了`file-loader`/`url-loader`，可以参照`vue`脚手架的设置：

```javascript
...
{
	test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
	loader: 'url-loader',
	options: {
		limit: 10000,
		name: utils.assetsPath('media/[name].[hash:7].[ext]')
	}
},
{
	test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
	loader: 'url-loader',
	options: {
		limit: 10000,
		name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
	}
},
...
```

除了css内引入图片，我们也会在html文件内使用`<img src="img/a.png" />`或者内联样式的`background`来引入图片，这时候我们可以使用`html-withimg-loader`来处理这种情况：

```
npm i html-withimg-loader -D
```

在`webpack.config.js`内配置：

```javascript
{
 	test: /\.(html|htm)$/,
    loader: 'html-withimg-loader'
}
```

值得注意的是，该`loader`在`NPM`上已经有2年多没有更新了，而且在`vue`的脚手架内也没有找到这个`loader`的影子，那`vue`内是如何处理html内的图片的呢？留待研究。

![wp7](/images/wp7.png)

### ES6转义

说完了图片资源的处理，我们来说一下对`ES6`代码的处理，也就是最重要的`babel-core`、`babel-loader`和`babel-preset-env`。这3个包是用来将`ES6`代码转为`ES5`的，方便兼容低版本浏览器的同时，也不影响我们使用最新的`ES6`去撸代码。首先安装：

```
npm i babel-core babel-loader babel-preset-env -D
```

其中，`babel-preset-env`功能比较强大，可以指定我们的代码转义后需要兼容什么版本的浏览器，什么类型的浏览器等，配置项比较多，一般会将该配置单独另起一个`.babelrc`文件来存放。我们在根目录下新建一个`.babelrc`文件：

```javascript
{
  "presets": [
    ["env", {
      "modules": false,
      "targets": {
        "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
      }
    }]
  ]
}
```

可以看到，转义后的es6代码支持浏览器份额大于1%，最近两个版本，且不是ie8以下。

在`webpack.config.js`内：

```javascript
module: {
    rules: [
        ...
        {
            test: /\.js$/,
        	loader: 'babel-loader',
        	include: '/src/',           //只转义src目录下的js文件
        	exclude: '/node_modules/'   //排除掉node_modules目录
         }
    ]
}
```

注意，`exclude`优先级高于`include`。

写到这里，我们对`webpack4.0`的基本概念已经有了一个基本的认识，可以说我们已经入门了！不再一提`webpack`打包就一头雾水，手忙脚乱了。剩下的一些知识如`resolve`、`热更新`等概念，遇到了，不清楚，立马查文档，基础架子有了，剩下的就是加深理解了。

最后算下欠的账：一篇关于开发和生产模式的研究，一篇关于`path`和`publicPath`区别的博客，还有关于webpack更深入的一些知识总结，嗯，未完待续...

参考文章：[webpack4-用之初体验，一起敲它十一遍](https://juejin.im/post/5adea0106fb9a07a9d6ff6de)，[面试官：请手写一个webpack4.0配置](https://segmentfault.com/a/1190000015611030)


