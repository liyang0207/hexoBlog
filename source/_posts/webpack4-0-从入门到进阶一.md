---
title: webpack4.0 从入门到进阶一
date: 2018-08-04 22:46:15
tags:
  - Webpack
  - 随写
---

## webpack4.0 从入门到进阶一

好久没写博客了，一方面是最近在努力刷LeetCode题目，想通过做题来提升一下自己在算法方面的弱势，另一方面确实有点懒散了，公司的事情不多但是很杂。不管怎么说，这两天又静下来，准备深度学习一下webpack的相关知识，写几篇心得。说来惭愧，`vue`项目写过一两个，`angular`项目也写过几个，可对`webpack`基本上还是0的认知，`vue-cli`生成的项目基本上不需要自己去过多的配置，`angular`呢又找不到明显`webpack`配置文件，这就少了挑战自己的机会（还是懒），这次可要好好补上这个技能点。

### webpack安装

首先新建一个文件夹，命名`webpack4.0`，在文件内命令行运行：

```javascript
npm init -y
```

先生成一个默认配置的`package.json`，然后运行：

```javascript
npm install webpack webpack-cli --save-dev
```

这一步我们安装`webpack`和`webpack-cli`，需要注意的是`webpack4+`将`webpack-cli`单独拆了出来，所以要安装两个。这样就给项目安装好了`webpack`。

### webpack.config.js

接下来在目录下新建`webpack.config.js`配置文件，这里面就是`webpack`的基础和核心配置了，大致有这些配置：

```javascript
module.exports = {
    entry: '',				//入口文件
    output: {},  			//出口
    module: {},				//处理模块，loader都在这里
    plugin: {},				//强大的插件配置
    devServer: {},		    //开发服务器配置
    mode: '',                //模式配置， webpack4新增特性，有development/production
    ...
}
```

新建一个文件夹`src`，在里面新建一个`index.js`文件，作为我们的入口文件。这个文件其实不陌生，就是`vue`项目里的`src->main.js`文件，其实我们想想，项目的一切都是从这个文件开始的，`main.js`内`import`了许多文件（js，css...），然后这些`js`文件内又引入了其他文件，这样一层一层下去，所以这个文件就是一切开始的地方，`webpack`将从这里开始，一层一层将依赖找下去。

这个时候我们还需要一个`devServer`，来跑起来本地服务，运行以下命令：

```javascript
npm install webpack-dev-server --save-dev
```

接下里配置两条`npm scripts`，在`package.json`中：

```javascript
...
"scripts": {
  "dev": "webpack-dev-server",
  "build": "webpack"
},
...
```

这样就开发环境`run dev`，生成线上打包资源`run build`。

现在准备工作差不多了，来从零手写`webpack.config.js`！

### entry && output

我们先从最基础的地方入手，先来看看入口和出口的配置：

```javascript
const path = require('path');
module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[chunkhash].bundle.js'
    }
}
```

大部分情况下，可以直接使用字符串的形式来实现单入口，路径基于当前项目根目录。如果有多个入口，则可使用`entry: { app: './src/app.js', vendor: './src/vendor.js'}`这种对象形式。

入口可以多个，但是输出配置只能指定一个。`filename`指定了输出文件的文件名，有多种形式可选：

| 模板        | 描述                                        |
| ----------- | ------------------------------------------- |
| [hash]      | 模块标识符(module identifier)的 hash        |
| [chunkhash] | chunk 内容的 hash                           |
| [name]      | 模块名称                                    |
| [id]        | 模块标识符(module identifier)               |
| [query]     | 模块的 query，例如，文件名 `?` 后面的字符串 |

可以根据情况随心所欲的拼接。而输出目录`path`必须为绝对路径，这里使用了`node`的`path`（又一个盲点）来指定输出，这里指根目录下新建一个`dist`文件夹存放输出文件。

接下来就可以运行试一试了，我们先随便往`index.js`文件内写一些代码：

```javascript
const str = 'Hello Webpack';
console.log(str);
```

现在我们的目录长这样：

![wp1](/images/wp1.png)

命令行运行`npm run build`，可以看到在根目录下生成了`dist`文件夹，里面有根据我们在`output`内制定的文件名称格式的bundle文件：

![wp2](/images/wp2.png)

### 配置Html模板

可以想象，我们最后打包好的文件都将是`js`、`css`、`image`形式的，我们需要一个承载这些资源的文件，就是我们的`html`文件，我们需要实现`html`打包功能，我们引入`html-webpack-plugin`插件：

```javascript
npm i html-webpack-plugin -D
```

这是一个插件，我们在`plugins`内引用一下，同时在根目录下新建一个`index.html`文件作为模板：

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: "[chunkhash].bundle.js"
  },
  plugins: [
    new HtmlWebpackPlugin({
      // 选择一个html文件作为模板
      template: './index.html',
      // 在打包好的bundle.js后面添加hash
      hash: true
    })
  ]
}
```

再次运行`npm run build`，看看`dist`内生成的`index.html`文件。

![wp3](/images/wp3.png)

可以看到页面内引入了`chunkhash.bundle.js`文件，同时页面也是使用我们在根目录下建的`index.html`作为模板。

现在来回想一下使用`vue-cli`起的脚手架，是不是有点感悟呢？

### 配置CSS文件

这里有两点需要知道，首先，如果我们想要将css文件单独拆出来，需要使用`extract-text-webpack-plugin`插件；其次，`webpack`打包css需要使用能解析css的`loader`，不同的`css`编译语言需要使用不同的`loader`，如`less-loader`、`sass-loader`等。

我们先来安装两个基础的样式`loader`:

```javascript
npm i css-loader style-loader -D
```

这两个`loader`各有分工：`css-loader`负责解释(interpret) `@import` 和 `url()` ，会 `import/require()` 后再解析(resolve)它们，`style-loader`负责将样式放入`style`标签内，放入`DOM`内。所以，我们需要先引入`css-loader`，再使用`style-loader`。

#### css文件css-loader&style-loader&less-loader

在`src`文件夹内新建`css->reset.css`文件，里面随便写点样式，在`src/index.js`内通过import './css/reset.css'引入，然后我们来配置`loader`:

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: "[chunkhash].bundle.js"
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      // 选择一个html文件作为模板
      template: './index.html',
      // 在打包好的bundle.js后面添加hash
      hash: true
    })
  ]
}
```

`module`内的`rules`是创建模块时，匹配请求的规则数组，里面是对不同类型文件的处理方式，即对什么类型的文件使用什么`loader`，`test`用来匹配文件，`use`是使用的`loader`，可以多个`loader`同时生效，顺序从右向左。这里`loader`有多种写法，比较灵活：

```javascript
module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.css$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              ...
            }
          }
        ]
      },
      {
        test: /\.css$/,
        loader: 'style-loader',
        options: {
          ...
        }
      }
    ]
  },
```

单个`laoder`可以直接写出来，多个`loader`可以使用`use`方法，同时也可以根据是否有`options`选项来决定怎么写。

这些搞定后，运行`npm run build`，使用浏览器打开`dist/index.html`，切到控制台，可以看到样式以`<style>...</style>`形式引入了：

![wp4](/images/wp4.png)

接下来我们再来个`less示例`，在`src->css`文件夹内新建`style.less`文件，里面随便写点样式，在`src/index.js`内通过import './css/style.less'引入，同时安装`less-loader`来解析`.less`文件：

```javascript
npm i less less-loader -D
```

接下来编写`rule`，这里要主要`loader`的顺序，需要先使用`less-loader`解析`.less`样式：

```javascript
module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.less$/,
        use: ['style-loader', 'css-loader', 'less-loader']
      }
    ]
  },
```

运行`npm run build`，来浏览器里看看结果：

![wp5](/images/wp5.png)

看到我们的`.less`文件也引入到了页面中。

#### 拆分css

上面的样式都是通过`<style>`标签引入的，如果想以样式文件的方式引入，就需要用到`extract-text-webpack-plugin`插件，这里要注意，到目前为止，该插件的稳定版还不支持`webpack4.0`，需要安装它的beta版：

```
npm i extract-text-webpack-plugin@next -D
```

引入插件：

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: "[chunkhash].bundle.js"
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract({
          // 将css用link的方式引入就不再需要style-loader了
          use: 'css-loader'
        }),
      },
      {
        test: /\.less$/,
        use: ['style-loader', 'css-loader', 'less-loader']
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      // 选择一个html文件作为模板
      template: './index.html',
      // 在打包好的bundle.js后面添加hash
      hash: true
    }),
    // 拆分后会把css文件放到dist目录下的css/style.css
    new ExtractTextPlugin('css/reset.css')
  ]
}
```

运行`npm run build`，看看`dist`内生成的文件，再看看浏览器：

![wp6](/images/wp6.1.png)

![wp6.2](/images/wp6.2.png)

`.less`没有拆分，故还是以标签的形式引入页面。当然我们也可以同时拆分成多个css：

```javascript
// 正常写入的less
let styleLess = new ExtractTextWebpackPlugin('css/style.css');
// reset
let resetCss = new ExtractTextWebpackPlugin('css/reset.css');

module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: resetCss.extract({
                    use: 'css-loader'
                })
            },
            {
                test: /\.less$/,
                use: styleLess.extract({
                    use: 'css-loader'
                })
            }
        ]
    },
    plugins: [
        styleLess,
        resetCss
    ]
}
```

未完待续。。。



