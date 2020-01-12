React 本身专注于视图层，通过执行 this.setState 更新数据，然后驱动视图变化。

一个应用整体是一个非常复杂的组件树，嵌套关系非常复杂，互相之间是包含与被包含的关系。如果父组件的数据更新了，子组件通过 props 引用了父组件的数据，那么子组件也会变化，看个 demo：

``` javascript
// parent component
import React from 'react'
import ChildComponent1 from './child1'
import ChildComponent2 from './child2'

class ParentComponent extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      count: 0
    }
  }

  render () {
    console.log('parent component re-render')
    return (
      <React.Fragment>
        <ChildComponent1 count={this.state.count}></ChildComponent1>
        <ChildComponent2></ChildComponent2>
        <button onClick={() => this.onClick}>add</button>
      </React.Fragment>
    )
  }

  onClick () {
    this.setState((prevState, nextState) => ({
      count: prevState.count + 1
    }))
  }
}
```

``` javascript
// child component 1
import React from 'react'

class ChildComponent extends React.Component {
  constructor (props) {
    super(props)
    this.state = {}
  }

  render () {
    console.log('child component1 re-render')
    return <div onClick={() => this.onClick}>child component 1</div>
  }

  onClick () {
    console.log(this.props.count)
  }
}
```

``` javascript
// child component 2
import React from 'react'

class ChildComponent extends React.Component {
  constructor (props) {
    super(props)
    this.state = {}
  }

  render () {
    console.log('child component2 re-render')
    return <div>child component 2</div>
  }
}
```

点击父组件的 add 按钮，修改完 count 值之后，ChildComponent1 和 值之后，ChildComponent1 都会执行自己的 render 方法，重新渲染。

但是，ChildComponent1 的 render 函数中并没有依赖 count 属性，也就是说，父组件的 count 值改变之后，子组件并不需要立即渲染，只是在子组件执行 onClick 的时候才会用到 count。

而 ChildComponent2 根本就没有依赖父组件的任何数据，即子组件不应该跟着父组件更新。

当页面内容足够多的时候，无谓的重复渲染将导致页面性能下降，有两种解决方法：

1、对于 ChildComponent1，可以在 shouldComponentUpdate 函数中，通过一系列的逻辑判断，返回 true 或 false，返回 false，则当前组件不执行 render 函数。

2、对于 ChildComponent2，React 为我们提供了 React.PureComponent 这个顶层 API，以上 ChildComponent 可以修改为：

``` javascript
import React from 'react'

class ChildComponent2 extends React.PureComponent {
  constructor (props) {
    super(props)
    this.state = {}
  }

  render () {
    console.log('child component2 re-render')
    return <div>child component 2</div>
  }
}
```

再次点击父组件的 add 按钮，子组件将不再重复渲染。

下面来看看 PureComponent 的源码，打开 packages/react/src/ReactBaseClasses.js 文件：

``` javascript
function ComponentDummy() {}
ComponentDummy.prototype = Component.prototype;

/**
 * Convenience component with default shallow equality check for sCU.
 */
function PureComponent(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}

const pureComponentPrototype = (PureComponent.prototype = new ComponentDummy());
pureComponentPrototype.constructor = PureComponent;
// Avoid an extra prototype jump for these methods.
Object.assign(pureComponentPrototype, Component.prototype);
pureComponentPrototype.isPureReactComponent = true;
```

代码很简单，通过原型继承的方式，PureComponent 继承了 Component，然后在原型上添加 isPureReactComponent 属性为 true。

组件渲染前，React 会以【shallow equal】的方式，判断当前组件是否需要更新。当对象包含复杂的数据结构时，可能就不灵了，对象深层的数据已改变却没有触发 render。

来看下 shallowEqual 方法是如何实现的：

``` javascript
// 关于 Object.is 的逻辑，可以参考下面的链接，主要是考虑到了 NaN 及 0 和 -0 的场景
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
function is(x, y) {
  return x === y && (x !== 0 || 1 / x === 1 / y) || x !== x && y !== y // eslint-disable-line no-self-compare
  ;
}

var is$1 = typeof Object.is === 'function' ? Object.is : is;

var hasOwnProperty$2 = Object.prototype.hasOwnProperty;

// shallowEqual 的核心就是判断两个值是否【浅相等】
// 如果相等，说明数据没有变化，返回 true，组件不更新
// 如果不想等，说明数据变了，返回 false，组件重新渲染
// 它和 shouldComponentUpdate 要的事情一样，但是返回值正好相反
function shallowEqual(objA, objB) {
  if (is$1(objA, objB)) {
    return true;
  }

  if (typeof objA !== 'object' || objA === null || typeof objB !== 'object' || objB === null) {
    return false;
  }

  var keysA = Object.keys(objA);
  var keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) {
    return false;
  } // Test for A's keys different from B.

  // 获取 keysA 的所有属性，只遍历了一层，浅相等
  for (var i = 0; i < keysA.length; i++) {
    if (!hasOwnProperty$2.call(objB, keysA[i]) || !is$1(objA[keysA[i]], objB[keysA[i]])) {
      return false;
    }
  }

  return true;
}
```

本节笔记重点是从宏观上了解 React.PureComponent 的结构和 shallowEqual 函数，具体渲染过程，在 ReactDOM 中详细学习。

### 注意

本文最后编辑于2020/01/12，技术更替飞快，文中部分内容可能已经过时，如有疑问，可在线提issue。