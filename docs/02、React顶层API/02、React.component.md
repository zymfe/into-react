作为类组件最常用的一个 API：React.component 帮助我们创建一个 React 组件，其源码位于 packages/react/src/ReactBaseClasses.js 文件中，它是一个类，代码结构如下：

``` javascript
/**
 * Base class helpers for the updating state of a component.
 */
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  // We initialize the default updater but the real one gets injected by the
  // renderer.
  this.updater = updater || ReactNoopUpdateQueue;
}

Component.prototype.isReactComponent = {};

Component.prototype.setState = function(partialState, callback) {
  // ...
};

Component.prototype.forceUpdate = function(callback) {
  // ...
};
```

除了以上初始化的默认方法，在渲染的时候还有陆续添加 render 函数、生命周期函数、开发者各种自定义的函数。我们可以写 demo 测试下：

``` javascript
import React from 'react'

class App extends React.component {
  render () {
    console.log(this)
    return <div></div>
  }
}
```

在控制台中，可以看到 React.component 的实例对象，其对象本身及原型对象上有很多属性和方法。

本节笔记重点是从宏观上了解 React.Component 的结构，对于比如 setState 具体做了什么事情，会在 ReactDOM 中详细学习。

### 注意

本文最后编辑于2020/01/12，技术更替飞快，文中部分内容可能已经过时，如有疑问，可在线提issue。