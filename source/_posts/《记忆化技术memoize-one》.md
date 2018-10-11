---
title: 《记忆化技术memoize-one》
date: 2018-10-11 20:54:03
tags:
  - react
  - 知识积累
---

最近在努力学习react中，各种工具概念应接不暇：`react-redux`、`dva`、`umi`、`redux-saga`、`ant design pro`…每一个都需要耗费时间去接触熟悉。上个月常刷的`leetCode`刷到中等难度题也开始屡屡碰壁，果然罗马不是一天建成的，还是要不断跳出舒适区，不断折腾才能前进。

### 引入场景

平时在开发中，少不了前端对服务器传来的数据进行处理整合，然后丢给框架去`render`。`react`配合`redux`开发时，外层容器组件的`props`改变时，都会调用`render()`去`diff`渲染。假如我们在`render()`内进行了复杂的数据处理：

```javascript
import React, { PureComponent } from 'react';
class TableList extends PureComponent {
  state = { filterText: "" };
  handleChange = event => {
    this.setState({ filterText: event.target.value });
  };
  render() {
    const filteredList = this.props.list.filter(
      item => item.text.includes(this.state.filterText)
    )
    //过滤数据
    const filteredList = list.filter(item => item.name === this.state.filterName)
    return (
      <Fragment>
        <input onChange={this.handleChange} value={this.state.filterText} />
        <ul>{filteredList.map(item => <li key={item.id}>{item.text}</li>)}</ul>
      </Fragment>
    )
  }
}
```

每一次`render()`都会去过滤一次数据，假如我们这个组件的`title`经常变化，但是`list`数据不变，就会消耗大量计算时间，导致性能问题。

而在实际的开发中，数据结构往往更加复杂，有时候甚至会有多次的循环。有时候组件的更新**并不是**因为从服务器拿来的这一段数据结构发生变化造成的（组件中的其他部分更新造成的），但是这一段很重的逻辑因为是写在 render 中的，所以不可避免的在每次 render 会调用一次。如果这段逻辑在两次调用的时候，输入参数是一样的，那么输出结果必然一样，所以再次计算是一种十分浪费资源的行为。

接下来我们使用`记忆化技术`来优化。

### 记忆化库-memoize-one

根据`memoize-one`名字中的`one`可以知道，这个库的每个实例都缓存了一个结果，下一次不同的结果将覆盖上一次的。虽然只能缓存一个数据，但是用到合适的地方却能发挥很大的作用。

使用`npm`安装：`$ npm install memoize-one`，先看一下官方例子：

```javascript
import memoizeOne from 'memoize-one';
 
const add = (a, b) => a + b;
const memoizedAdd = memoizeOne(add);
 
memoizedAdd(1, 2); // 3
 
memoizedAdd(1, 2); // 3
// Add 函数并没有执行: 前一次执行的结果被返回
 
memoizedAdd(2, 3); // 5
// Add 函数再次被调用以获得新的结果
 
memoizedAdd(2, 3); // 5
// Add 函数并没有执行: 前一次执行的结果被返回
 
memoizedAdd(1, 2); // 3
// Add 函数再次被调用以获得新的结果
// 虽然之前调用过
// 但是不是上一次调用的，所以结果丢失了
```

`memoizeOne(resultFn, isEqual)`接收一个结果函数和一个对比函数，对比函数为空则默认使用`===`来进行入参的比较。

简单来讲就是，`memoizeOne()`在原来`resultFn()`函数外面包了一层，返回一个函数，然后每次调用的时候看新入参`newArgs`是否和上一次的入参`lastArgs`一致，参数不变，则直接返回缓存的结果，否则重新执行`resultFn(newArgs)`，缓存新结果。

简单改造一下上面的例子：

```javascript
import React, { Component } from 'react';
import memoizeOne from 'memoize-one';
class TableList extends Component {
  state = { filterText: "" };
  handleChange = event => {
    this.setState({ filterText: event.target.value });
  };
  filter = memoize(
    (list, filterText) => list.filter(item => item.text.includes(filterText))
  );
  render() {
    const { list, title } = this.props;
    //当list和filterName不变时，filteredList返回值不变
    const filteredList = this.filter(list, this.state.filterText);
    return (
      <Fragment>
        <input onChange={this.handleChange} value={this.state.filterText} />
      	<ul>{filteredList.map(item => <li key={item.id}>{item.text}</li>)}</ul>
      </Fragment>
    )
  }
}
```

这样就简单的实现了一个记忆优化。

### 源码解读

`memoize-one`记忆库巧妙的使用了`闭包`来实现，一般我是不看源码的，但是这个库的源码只有不到40行的代码，简单易懂，这里就简单看一下：

```javascript
//isEqual比较函数，用来判断参数是否一致，默认使用全等来判断
var simpleIsEqual = function simpleIsEqual(a, b) {
  return a === b;
};

function index (resultFn, isEqual) {
  //不传isEqual，使用默认的内置函数
  if (isEqual === void 0) {
    isEqual = simpleIsEqual;
  }

  var lastThis;
  var lastArgs = [];  //上一次的入参
  var lastResult;     //缓存的结果
  var calledOnce = false;  //是否调用过，区分第一次

  //判断两次入参是否相等，使用了every方法，这个是every方法的函数
  var isNewArgEqualToLast = function isNewArgEqualToLast(newArg, index) {
    return isEqual(newArg, lastArgs[index]);
  };

  var result = function result() {
    //将入参arguments按顺序一个个存入newArgs内
    for (var _len = arguments.length, newArgs = new Array(_len), _key = 0; _key < _len; _key++) {
      newArgs[_key] = arguments[_key];
    }

    //入参不变，直接返回缓存的结果lastResult
    if (calledOnce && lastThis === this && newArgs.length === lastArgs.length && newArgs.every(isNewArgEqualToLast)) {
      return lastResult;
    }

    lastResult = resultFn.apply(this, newArgs);  //apply到resultFn,传入参数newArgs，缓存结果
    calledOnce = true;
    lastThis = this;  //this？
    lastArgs = newArgs;  //新入参替换缓存的参数
    return lastResult;   //返回新计算的结果
  };

  return result;   //返回一个函数，闭包，不被GC
}

export default index;

```

源码还是比较容易看懂的。

### isEqual函数

因为对相等的理解，不同场景不一样，而且参数有时候是复杂的对象，所以我们不能仅仅通过比较操作符 `==` 或者 `===` 来判断。`memoize-one` 允许用户自定义传入判断是否相等的函数，比如我们可以使用 lodash 的 isEqual 来判断两次参数是否相等。

```javascript
import memoizeOne from 'memoize-one';
import deepEqual from 'lodash/isEqual';
 
const identity = x => x;
 
const defaultMemoization = memoizeOne(identity);
const customMemoization = memoizeOne(identity, deepEqual);
 
const result1 = defaultMemoization({foo: 'bar'});
const result2 = defaultMemoization({foo: 'bar'});
 
result1 === result2 // false - 索引不同
 
const result3 = customMemoization({foo: 'bar'});
const result4 = customMemoization({foo: 'bar'});
 
result3 === result4 // true - 参数通过 lodash 的 isEqual 判断是相等的
```

参考链接：[what-about-memoization](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization)，[记忆化技术介绍——使用闭包提升你的 React 性能](https://zhuanlan.zhihu.com/p/37913276)