---
title: VUE双向数据绑定原理及简单实现
date: 2018-06-03 11:12:18
tags:
 - JavaScript
 - Vue
---

Reactive Programming是一种编程形式，在很多场景都会见到，最近正在学习的RxJS是一个例子，当然Vue同样是一种Reactive Programming，就是当变量发生改变的时候，相关的变量和视图也会跟着改变，而我们开发者不需要自己去写代码来实现这个过程，我们只需要关心变量改变之后应该进行什么操作，更加关注于业务流程。

Vue的双向数据绑定是基于ES5的`Object.defineProperty()`的`getter`和`setter`，每当数据发生变化，就会执行`getter/setter`，结合发布者/订阅者的模式，通知订阅者这些变化，进而执行相应的回调函数。

今天我们来分析一下vue双向数据绑定的原理，同时我们自己用js来实现一个简单的双向数据绑定，首先看一下原理图：

![two-ways-binding](/images/two-ways-binding.png)

`MVVM`就是我们要实现的`vue`实例，简单讲述一下流程：

1. 首先通过一个`Observer`（监听器或者劫持器）去劫持`data`对象中的所有属性，方法就是使用`Object.defineProperty()`中的`getter/setter`，在属性`set`的时候通知`Dependency`（订阅器/容器）发布变化；
2. 实现一个`Watcher`（订阅者），这个`Watcher`就是说我收到数据变化的通知后，应该去执行什么操作（重新填充列表，填充值等等，即更新视图），一个data.message数据可能对应多个使用场景，比如`v-model="message"`、`v-text="message"`、`{ {message} }`等等，所以`Watcher`不止一个；
3. 上面说到`Watcher`不止一个，所以我们可以实现一个容器`Dependency`，里面存放data.message对应的所有`Watcher`，这样当`Observer`的`Setter`改变时，调用`Dependency`的`notify`方法，逐条去通知所有的`Watcher`；
4. 实现一个编译器`Complier`，编译器的作用是扫描和解析每一个节点`node`，先将节点转换为`fragment`（性能优化，一次性append所有节点至目标element内），再根据不同的节点类型`nodeType`，针对`v-model`、`v-text`、`{{message}}`做不同的处理，完成第一次的数据`message`填充（即初始化视图）；同时编译器还担当着初始化`Watcher`的任务，将`Watcher`添加到`Dependency`中去；

有了以上的思路，接下来就是编写代码时间，使用了ES6的`class`，首先我们来实现`Observer`:

```javascript
class Observer {
  constructor(data) {
    this.observer(data);
  }
  observer(data) {
    if (!data || typeof data !== 'object') {
      return false;
    } else {
      Object.keys(data).forEach((key) => {
        // 劫持data对象中的每一条数据
        this.defineReactive(data, key, data[key]);
      })
    }
  }
  defineReactive(obj, key, value) {
    let dep = new Dependency();
    Object.defineProperty(obj, key, {
      enumerable: true,
      configurable: false,
      get() {
        if (Dependency.target) {
          dep.addSub(Dependency.target);    // 添加订阅者watcher,应该是整个实例Watcher
        }
        return value;
      },
      set(newValue) {
        // 值未变化return回去
        if (newValue === value) { return false; }
        value = newValue;
        // 数据变化，通知dep里所有的watcher
        dep.notify();
      }
    })
  }
}
 // 第一次get值的时候不会添加Watcher到Dependency,实例化（调用）watcher时再添加
Dependency.target = null; 
```

接下来实现`Watcher`:

```javascript
class Watcher {
  constructor(vm, expr, callback) {
    this.vm = vm;
    this.expr = expr;           // data中的key值
    this.callback = callback;   // 值变化的时候执行什么回调
    this.value = this.get();    // 实例化watcher的时候将自己添加到Dependency
  }
  get() {
    Dependency.target = this;   // 缓存自己,就是这个Watcher实例
    let value = this.vm.$data[this.expr];  // 触发执行Observer中的get函数，将自己添加到Dep
    Dependency.target = null;   // 释放自己
    return value;
  }
  update() {
    // 值更新后，Observer的setter就会触发，就会执行dep.notify()，即通过Dep容器通知watcher根据callback去更新视图
    let newValue = this.vm.$data[this.expr];
    let oldValue = this.value;
    if (newValue !== oldValue) {
      // 新老值不一致，执行回调
      this.callback(newValue);
    }
  }
}
```

然后我们需要一个容器`Dependency`去储存data.message对应的所有`Watcher`:

```javascript
class Dependency {
  constructor() {
    this.subs = [];      // 容器数据，放watcher用
  }
  addSub(watch) {
    this.subs.push(watch);   // 将watcher添加到subs内
  }
  notify() {
    // 通知subs内的所有watcher更新回调
    this.subs.forEach((watch) => {
      watch.update();
    })
  }
}
```

下面是编译器`Complier`，编译器涉及的东西比较杂，判断的情况比较多，所以这里只考虑到了`v-model`、`v-text`、`{ {message} }`这3种情况的实现：

```javascript
class Complier {
  constructor(el, vm) {
    this.vm = vm;
    this.el = document.querySelector(el);
    if (this.el) {
      // 使用fragment储存元素，这时候#app内就没有节点了，因为已经被frag删除完了
      let fragment = this.nodeToFragment(this.el);    
      this.complie(fragment);                         // 编译fragment
      this.el.appendChild(fragment);                  // 将fragment放回#app内
    }
  }
  complie(node) {
    // 使用Array.from将类数组node.childNodes转换为真正的数组
    let nodeList = Array.from(node.childNodes);    
    nodeList.forEach((item) => {
      //根据nodeType判读节点类型，执行不同的编译
      switch (item.nodeType) {
        case 1:
          this.elementComplier(item);break;
        case 3:
          this.textComplier(item);break;
      }
    })
  }
  elementComplier(node) {
    // 元素节点编译器，处理属性v-model，v-text等
    let attrs = Array.from(node.attributes);
    attrs.forEach((attr) => {
      if (attr.name.indexOf('v-') > -1) {
        let type = attr.name.split('-')[1];    // 取到'model',即指令的类型
        complierUnits[type] && complierUnits[type](node, this.vm, attr.value);
      }
    })
  }
  textComplier(node) {
    // 文本节点编译器{{message}},跟v-text共用一个编译方法
    if ((/\{\{(.+)\}\}/).test(node.textContent)) {
      complierUnits.text(node, this.vm, RegExp.$1);
    }
  }
  nodeToFragment(node) {
    // 将node转换为fragment
    let frag = document.createDocumentFragment();
    let child;
    while (child = node.firstChild) {
      // fragment调用appendChild方法会删除node.firstChild节点
      frag.appendChild(child);
    }
    return frag;
  }
}

// 编译器工具箱
const complierUnits = {
  model (node, vm, expr) {
    let updateFn = this.updater.modelUpdater;
    // 初始化的时候取一次值填充，渲染页面数据
    updateFn && updateFn(node, vm.$data[expr]);
    // 实例化watcher(调用watcher),将watcher添加到Dep中，同时定义好回调函数（数据变化后干什么）
    new Watcher(vm, expr, function(newValue){
      updateFn && updateFn(node, newValue);
    });
    // 监听input值的变化，从视图到data
    node.addEventListener('input', (event) => {
      vm.$data[expr] = event.target.value;
    })
  },
  text (node, vm, expr) {
    let updateFn = this.updater.textUpdater;
    updateFn && updateFn(node, vm.$data[expr]);
    new Watcher(vm, expr, function(newValue){
      updateFn && updateFn(node, newValue);
    });
  },
  updater: {
    modelUpdater(node, value) {
      node.value = value;
    },
    textUpdater(node, value) {
      node.textContent = value;
    }
  }
};
```

还有入口`main.js`:

```javascript
class MVVM {
  constructor(options) {
    this.$el = options.el;
    this.$data = options.data;
    // 当视图存在时
    if (this.$el) {
      // 将属性添加进Observer，劫持数据
      new Observer(this.$data);
      // 编译页面
      new Complier(this.$el, this);
    }
  }
}
```

最后就是html调用了：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>vue原理简单实现</title>
  <script src="js/dependency.js"></script>
  <script src="js/observer.js"></script>
  <script src="js/watcher.js"></script>
  <script src="js/complier.js"></script>
  <script src="js/main.js"></script>
</head>
<body>
<div id="app">
  <span v-text="message"></span>
  <input type="text" v-model="message" />
  {{message}}
</div>

<script>
  let vm = new MVVM({
    el: '#app',
    data: {
      message: 'hello Vue!'
    }
  })
</script>
</body>
</html>
```

总结：实例化`MVVM`时，先使用`Object.defineProperty`劫持每一个data数据，为每一个属性实例化一个`Dependency；`在编译页面的时候为每一个需要更新message的地方添加一个`Watcher`，即`v-model="message"`、`v-text="message"`和`{ {message} }`，有一个算一个，将这些`Watcher`添加到`Dependency`中进行统一管理；在编译的时候我们还要为input添加一个事件监听`addEventListener`，这样input的输入值变化时，触发`setter`，在`setter`内调用`Dep`的`notify()`方法，循环调用每一个`Watcher`的`update`更新我们的视图（执行回调函数）。

以上代码都放到了我的github仓库[vue-principle](https://github.com/liyang0207/vue-principle)，欢迎查阅。

参考文章：[vue双向绑定原理分析](https://www.cnblogs.com/zhenfei-jiang/p/7542900.html)，[vue的双向绑定原理及实现](https://www.cnblogs.com/libin-1/p/6893712.html)