React.Children 有 map、forEach、count、toArray、only 等5个属性，它们对应5个函数。具体作用可以参考官网 API：https://zh-hans.reactjs.org/docs/react-api.html#reactchildren

``` javascript
const React = {
  Children: {
    map,
    forEach,
    count,
    toArray,
    only,
  }
}
```

以 React.Children.map 为例，写个 demo：

``` javascript
import React from 'react'
import ChildComponent from './Child'

class App extends React.Component {
  render () {
    return (
      <ChildComponent>
        <div index="0">a</div>
        <div index="1">
          <div>b-1</div>
          <div>b-2</div>
        </div>
        <div index="2">c</div>
      </ChildComponent>
    )
  }
}
```

``` javascript
import React from 'react'

class ChildComponent extends React.Component {
  return (
    <React.Fragment>
      {
        (() => {
          const mappedChildren = React.Children.map(this.props.children, (el, index) => [el, [el, [el]], [el]])
          console.log('mappedChildren: ', mappedChildren)
          return mappedChildren
        })()
      }
    </React.Fragment>
  )
}
```

看下页面渲染结果和控制台打印结果。

在 children 里的每个直接子节点上调用一个函数。如果 children 是一个数组，它将被遍历并为数组中的每个子节点调用该函数。如果子节点为 null 或是 undefined，则此方法将返回 null 或是 undefined，而不会返回数组。

上面 demo 中，map 方法的第二个参数是一个函数，该函数返回一个多维嵌套的数组。但是页面渲染结果是一维的，控制台中的打印结果也是一维的。

看下它的源码，在 /packages/react/src/ReactChildren.js 文件中：

``` javascript
function mapChildren(children, func, context) {
  if (children == null) {
    return children;
  }
  // 这个 result 就是上面 demo 执行完后在控制台中看到的结果
  // 这个地方可以打断点看下执行流程：
  debugger
  const result = [];
  mapIntoWithKeyPrefixInternal(children, result, null, func, context);
  return result;
}
```

看下 mapIntoWithKeyPrefixInternal 函数：

``` javascript
function mapIntoWithKeyPrefixInternal(children, array, prefix, func, context) {
  let escapedPrefix = '';
  if (prefix != null) {
    // 添加分割标识符，表示这是一个子组件，即 this.props.children[n]
    escapedPrefix = escapeUserProvidedKey(prefix) + '/';
  }
  // 从 TraverseContextPool 中 pop 一个元素，后面会用到
  const traverseContext = getPooledTraverseContext(
    array,
    escapedPrefix,
    func,
    context,
  );
  // 重点逻辑都在 traverseAllChildren 中
  traverseAllChildren(children, mapSingleChildIntoContext, traverseContext);
  // 释放 traverseContext，重置 TraverseContextPool
  releaseTraverseContext(traverseContext);
}
```

traverseAllChildren 函数：

``` javascript
function traverseAllChildren(children, callback, traverseContext) {
  if (children == null) {
    return 0;
  }
  // 通过 if 条件块内代码可知，traverseAllChildrenImpl 返回一个 number 类型的数值
  return traverseAllChildrenImpl(children, '', callback, traverseContext);
}
```

重点来了：

``` javascript
function traverseAllChildrenImpl(
  children,
  nameSoFar,
  callback,
  traverseContext,
) {
  const type = typeof children;

  if (type === 'undefined' || type === 'boolean') {
    // All of the above are perceived as null.
    children = null;
  }

  let invokeCallback = false;

  if (children === null) {
    invokeCallback = true;
  } else {
    switch (type) {
      case 'string':
      case 'number':
        invokeCallback = true;
        break;
      case 'object': 
        switch (children.$$typeof) {
          case REACT_ELEMENT_TYPE:
          case REACT_PORTAL_TYPE:
            invokeCallback = true;
        }
    }
  }
  // 如果是 REACT_ELEMENT_TYPE 或 REACT_PORTAL_TYPE，会执行 callback
  // 这里的 callback 是 mapSingleChildIntoContext
  if (invokeCallback) {
    callback(
      traverseContext,
      children,
      // If it's the only child, treat the name as if it was wrapped in an array
      // so that it's consistent if the number of children grows.
      nameSoFar === '' ? SEPARATOR + getComponentKey(children, 0) : nameSoFar,
    );
    return 1;
  }

  let child;
  let nextName;
  let subtreeCount = 0; // Count of children found in the current subtree.
  const nextNamePrefix =
    nameSoFar === '' ? SEPARATOR : nameSoFar + SUBSEPARATOR;

  // 这里的 children 有两种情况
  // 1、以上 demo 中 map 函数的第一个参数
  // 2、以上 demo 中 map 函数的返回值或其子元素（可能是多维数组）
  if (Array.isArray(children)) {
    for (let i = 0; i < children.length; i++) {
      child = children[i];
      nextName = nextNamePrefix + getComponentKey(child, i);
      // 递归调用，这里的入参 callback 可能有两种情况
      // 1、mapSingleChildIntoContext
      // 2、下面 mapSingleChildIntoContext 函数中继续执行 mapIntoWithKeyPrefixInternal 函数传入的最后一个参数
      subtreeCount += traverseAllChildrenImpl(
        child,
        nextName,
        callback,
        traverseContext,
      );
    }
  } else {
    // ... 省略
  }

  return subtreeCount;
}
```

看下上面提到的 mapSingleChildIntoContext 函数：

``` javascript
function mapSingleChildIntoContext(bookKeeping, child, childKey) {
  const {result, keyPrefix, func, context} = bookKeeping;

  // func 就是上面 demo 中 map 函数的第二个参数
  let mappedChild = func.call(context, child, bookKeeping.count++);
  // mappedChild 可能是一个数组，如上面 demo
  if (Array.isArray(mappedChild)) {
    // 如果开发者使用 map 函数返回一个 array，则继续执行 mapIntoWithKeyPrefixInternal 函数
    // 那么这个 mappedChild 就是上面提到的 children 有两种情况的第二种
    mapIntoWithKeyPrefixInternal(mappedChild, result, childKey, c => c);
  } else if (mappedChild != null) {
    // 递归执行到底了，将包装好的 mappedChild 放到 result 中
    if (isValidElement(mappedChild)) {
      mappedChild = cloneAndReplaceKey(
        mappedChild,
        // Keep both the (mapped) and old keys if they differ, just as
        // traverseAllChildren used to do for objects as children
        keyPrefix +
          (mappedChild.key && (!child || child.key !== mappedChild.key)
            ? escapeUserProvidedKey(mappedChild.key) + '/'
            : '') +
          childKey,
      );
    }
    result.push(mappedChild);
  }
}
```

整个流程比较绕，首先我们需要根据 demo，看下控制台和页面中的执行结果，理解 map 函数最终的执行结果是什么，然后再通过 debugger 的方式一步步验证。

模仿 React.Children.map 方法的执行流程，写了个 map 方法，代码参考：https://github.com/zymfe/test-code/blob/master/test130.js

React.Children 下还有其他4个函数，但都很简单了，mapChildren 的流程繁琐些。

### 注意

本文最后编辑于2020/01/12，技术更替飞快，文中部分内容可能已经过时，如有疑问，可在线提issue。