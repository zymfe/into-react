createRef 源码位于 /packages/react/src/ReactCreateRef.js 文件中：

``` javascript
import type {RefObject} from 'shared/ReactTypes';

// an immutable object with a single mutable value
export function createRef(): RefObject {
  const refObject = {
    current: null,
  };
  if (__DEV__) {
    // 封闭一个对象，阻止添加新属性并将所有现有属性标记为不可配置。
    // 当前属性的值只要原来是可写的就可以改变
    Object.seal(refObject);
  }
  return refObject;
}
```

那么它有什么作用呢？看下 demo：

``` javascript
import React from 'react'
class MyComponent extends React.Component {
  constructor(props) {
    super(props)
    this.inputRef = React.createRef()
  }

  render() {
    return <input type="text" ref={this.inputRef} />;
  }

  componentDidMount() {
    this.inputRef.current.focus();
  }
}
```

React.createRef 创建一个能够通过 ref 属性附加到 React 元素的 ref。通过 ref 可以获取到子组件实例，能在父组件中方便的执行子组件方法。

### 注意

本文最后编辑于2020/01/12，技术更替飞快，文中部分内容可能已经过时，如有疑问，可在线提issue。