---
title: webpack4.0 从入门到进阶三
date: 2018-08-08 20:58:16
tags:  
  - Webpack
  - 随写
---

上两篇博客简单总结了webpack4.0的一些基础配置，写完之后对webpack有了一个入门的认识。这两天又继续对遗留的几个知识点进行了梳理，如两种开发模式的配置流程，`path`和`publicPath`的区别与作用等。但是看一些关于webpack运行原理的博客时还是会一脸懵逼，完全不知所云，而且目前来讲我并没有真正的从0配置一个webpack的项目，所有的这些知识点还都没进入实践去检验。接下来写完这一两篇博客后，会着手开始`react`框架的学习，到时候结合项目，再去深入的理解`webpack`。

### 生产模式&&开发模式

现代前端工程项目，不管是使用`angular-cli`还是`vue-cli`来搭建，都会有本地开发和线上生产两种模式，命令行运行`npm run dev`即可在本地起一个服务，方便本地开发调试。之前讲`webpack4.0`新增了`mode`模式，之后会简化两种模式的配置，但是我们还是先来看一下4.0之前是如何配置两种模式的配置文件的，这里以`vue`为例。

#### vue的最佳实践

`vue-cli`脚手架生成的工程项目结构非常清晰，值得反复看，反复理解，它关于生产和开发模式的配置主要是这些文件：

![](/images/wp9.png)

`webpack.base.conf.js`中是一些公共的配置，然后通过`webpack-merge`把这些公共配置项和环境特定的配置项 merge 起来，成为一个完整的配置项。比如在`webpack.dev.conf.js`中：

```javascript
'use strict'
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')

const devWebpackConfig = merge(baseWebpackConfig, {
   ...
})
```

通过这种方式，开发和生产两种模式就可以有各种独有的wenpack配置项，更关键的是，每个配置中用 `webpack.DefinePlugin` 向代码注入了 `NODE\_ENV` 这个环境变量。这个变量在不同环境下有不同的值，比如 dev 环境下就是 development。这些环境变量的值是在 config 文件夹下的配置文件中定义的。Webpack 首先从配置文件中读取这个值，然后注入。比如这样：

```javascript
//webpack.dev.conf.js
plugins: [
  new webpack.DefinePlugin({
    'process.env': require('../config/dev.env.js')
  }),
]
```

```javascript
//dev.env.js
'use strict'
const merge = require('webpack-merge')
const prodEnv = require('./prod.env')

module.exports = merge(prodEnv, {
  NODE_ENV: '"development"'
})
```

这样我们就可以根据当前环境做判断，执行不同逻辑的代码，比如这样：

```javascript
//utils.js
// 根据环境使用不同资源地址assetsSubDirectory
exports.assetsPath = function (_path) {
  const assetsSubDirectory = process.env.NODE_ENV === 'production'
    ? config.build.assetsSubDirectory
    : config.dev.assetsSubDirectory
  return path.posix.join(assetsSubDirectory, _path)
}
```

这种区分不同的环境，并给环境变量设置不同的值的实践，让我们开启了编译时按环境对代码进行针对性优化的可能。 

#### 模拟4.0之前

看的再多不如手写一遍。我们先来手动实现vue的模式。首先我们安装两个依赖：

```shell
npm i webpack-merge uglifyjs-webpack-plugin -D
```

然后新建`webpack.dev.conf.js`和`webpack.prod.conf.js`两个文件，同时将`webpack.conf.js`修改为`webpack.base.conf.js`:

![](/images/wp10.png)

我们将`devServer`配置移动到`webpack.dev.conf.js`中，同时新增一个`devtool`配置：

```javascript
const path = require('path');
const merge = require('webpack-merge');
const baseWebpackConfig = require('./webpack.base.conf')

module.exports = merge(baseWebpackConfig, {
  mode: 'development',
  devServer: {//配置此静态文件服务器，可以用来预览打包后项目
    inline:true,//打包后加入一个websocket客户端
    contentBase: path.resolve(__dirname, 'dist'),//开发服务运行时的文件根目录
    host: 'localhost',//主机地址
    port: 4200,//端口号
    compress: true,//开发服务器是否启动gzip等压缩
    open:true, // 自动打开浏览器
  },
  devtool: 'cheap-module-eval-source-map'
})
```

使用`webpack-merge`合并基础配置和开发配置，这里就简单这么配置。接下来配置一下`webpack.prod.conf.js`：

```javascript
const merge = require('webpack-merge');
const baseWebpackConfig = require('./webpack.base.conf.js');
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

module.exports = merge(baseWebpackConfig, {
  mode: 'production',
  devtool: '#source-map',
  plugins: [
    new UglifyJsPlugin({
      uglifyOptions: {
        compress: {
          warnings: false
        }
      },
      sourceMap: true,
      parallel: true
    })
  ]
})
```

这里我们配置了一个不同的`devtool`配置，然后新引入一个`uglifyjs-webpack-plugin`插件，用来压缩混淆js文件。最后我们为了方便测试效果，将`package.json`中的命令这么修改：

```json
...  
"scripts": {
   "dev": "webpack --config webpack.dev.conf.js",
   "build": "webpack --config webpack.prod.conf.js"
 },
...
```

接下来我们分别运行`npm run dev`和`npm run build`，观察生成的`dist`文件夹有什么不同：

![npm run dev](/images/wp11.png)

![npm run build](/images/wp12.png)

从这个简单的实践我们就实现了一个很粗糙的运行模式拆分。剩下的就是针对具体的项目需求来编写更加强大的`webpack`配置了，当然还是强烈推荐去看`vue`脚手架生成的项目结构，越看越觉得有收获。

#### webpack4.0模式初探

`webpack4.0`新增了`mode`配置，可选值有`development`/`production`/`none`，借用官方文档：

| 选项        | 配置                                                         |
| ----------- | ------------------------------------------------------------ |
| development | 会将 `process.env.NODE_ENV` 的值设为 `development`。启用 `NamedChunksPlugin` 和 `NamedModulesPlugin`。 |
| production  | 会将 `process.env.NODE_ENV` 的值设为 `production`。启用 `FlagDependencyUsagePlugin`, `FlagIncludedChunksPlugin`, `ModuleConcatenationPlugin`, `NoEmitOnErrorsPlugin`, `OccurrenceOrderPlugin`, `SideEffectsFlagPlugin` 和 `UglifyJsPlugin`. |
| none        | 不选用任何默认优化选项                                       |

在不同的模式下，`webpack`会自动执行不同的插件：

mode: development

```javascript
// webpack.dev.conf.js
module.exports = {
+ mode: 'development'
- plugins: [
-   new webpack.NamedModulesPlugin(),
-   new webpack.NamedChunksPlugin(),
-   new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("development") }),
- ]
}
```

mode: production

```javascript
// webpack.production.config.js
module.exports = {
+  mode: 'production',
-  plugins: [
-    new UglifyJsPlugin(/* ... */),
-    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("production") }),
-    new webpack.optimize.ModuleConcatenationPlugin(),
-    new webpack.NoEmitOnErrorsPlugin()
-  ]
}
```

mode: none

```javascript
// webpack.custom.config.js
module.exports = {
+  mode: 'none',
-  plugins: [
-  ]
}
```

如果你想要根据 *webpack.config.js* 中的 **mode** 变量去影响编译行为，那你必须将导出对象，改为导出一个函数：

```javascript
var config = {
  entry: './app.js'
  //...
};

module.exports = (env, argv) => {

  if (argv.mode === 'development') {
    config.devtool = 'source-map';
  }

  if (argv.mode === 'production') {
    //...
  }

  return config;
};
```

但是这个选项的加入，如何优雅的跟之前拆分文件的形式结合，还得看社区大神们的整合方式，不管怎样，以后遇到心里就有底了。

### path && publicPath

先来看一下现在我们`output`配置：

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: "[chunkhash].bundle.js",
    publicPath: '/'
  },
  ...
}
```

`output`配置中`path`指定了`webpack`打包后生成的文件输入到哪里，我们之前运行`npm run build`的时候，文件都是打包到了这里，这个比较好理解，不过需要注意的是，该路径必须为绝对路径，所以我们使用了`path.resolve`来输出。

简单了说完了，比较迷惑的`publicPath`来了，这个路径指什么呢？刚开始我也比较迷惑，`path`已经配置了，还要这个`publicPath`干什么？反正你静态资源（`css`、`image`）都是以此为基准嘛，等等，真的是这样吗？

我们先来运行一下`npm run dev`，注意，这时候我们把命令调回到`webpack-dev-server`，同时，我们设置`output`内`publicPath`的值为`"/"`，该值不设置的话默认为空字符串。

```shell
"dev": "webpack-dev-server --config webpack.dev.conf.js"
```

服务启动后，打开控制台，观察`html`、`css`和`image`的加载路径：

![](/images/wp13.png)

这里无法一次看完，手动写一下，第一个`localhost`即`document`文件路径是`http://localhost:4200/`，第二个`style.css`即`stylesheet`文件路径是`http://localhost:4200/static/css/style.css?4cb..`，第三个`wp7.png`即`png`文件路径是`http://localhost:4200/static/img/wp7.png`。

这里我们发现，所有的静态资源文件路径都是基于`/`，即我们设置的`publicPath`，这个配置指定了你上传所有打包文件的位置（相对于服务器根目录），不过要注意，有时候我们的项目可不一定就是在网站根目录下配置的啊，有时候我们做的项目可能在`xxx.com/project/`下面部署，如果你不相应的修改`publicPath`值为`/project/`，你会发现静态资源路径依然会基于`/`，即根目录，从而导致资源无法访问到。

我们来试一试，现在将`publicPath`改为`"/project/"`，再来运行`npm run dev`，发现页面空白，显示`Cannot GET/`，控制台内也是404，这时候我们访问`http://localhost:4200/project/`，页面加载成功，再看控制台：

![](/images/wp14.png)

所有的资源路径都是在`/project/`下。

这里面其实涉及到了项目打包部署的问题，我对这方面了解的比较少，所以前面才会一直搞不明白这个`publicPath`是干什么的，最近翻了翻`vue`脚手架项目，又反复测试，才渐渐有点理解。不过这里面确实需要不断的实践才能更好的理解，就如官方文档中举的例子：

```javascript
module.exports = {
  //...
  output: {
    // One of the below
    publicPath: 'https://cdn.example.com/assets/', // CDN（总是 HTTPS 协议）
    publicPath: '//cdn.example.com/assets/', // CDN（协议相同）
    publicPath: '/assets/', // 相对于服务(server-relative)
    publicPath: 'assets/', // 相对于 HTML 页面
    publicPath: '../assets/', // 相对于 HTML 页面
    publicPath: '', // 相对于 HTML 页面（目录相同）
  }
};
```

上面还涉及到了资源在CDN的情况，理解了这些，以后就可以愉快的配合运维同学一起部署上线了，再遇到问题，查看官方文档就有迹可循了。

`webpack`系列暂时告一段落，但是绝不是结束，后续的运行原理还等着我去研究和学习，到现在为止，感觉继续单纯的纸上谈兵已经不能有较大的提升了，接下来就是到项目中去，结合项目进行学习，巩固和调整。

参考文章：[官方中文文档](https://webpack.docschina.org/configuration/output/#output-path)，[Webpack 4 配置最佳实践](https://juejin.im/post/5b304f1f51882574c72f19b0)

