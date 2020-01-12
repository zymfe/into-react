在一个典型的 React 应用中，数据是通过 props 属性自上而下（由父及子）进行传递的，但这种做法对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI 主题），这些属性是应用程序中许多组件都需要的。Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props。

demo 如下：

``` javascript
// context.js
import React from 'react'

export const ColorContext = React.createContext("orange", (previous, current) => {
  console.log('previous: ', previous)
  console.log('current: ', current)
  return 1
});
```

``` javascript
// parent component
import React from 'react'
import ColorContext from './context.js'
import ChildComponent from './child'

class Parent extends React.component {
  constructor (props) {
    super(props)
    this.state = {
      color: 'red'
    }
  }

  render () {
    return (
      <ColorContext.Provider value={this.state.color}>
        <ChildComponent></ChildComponent>
        <button onClick={() => this.onClick}>change</button>
      </ColorContext.Provider>
    )
  }

  onClick () {
    this.setState((prevState, nextState) => ({
      color: prevState.color === 'red' ? 'orange' : 'red'
    }))
  }
}
```

``` javascript
// child component
import React from 'react'
import ColorContext from './context.js'
class Child extends React.component {
  constructor (props) {
    super(props)
    this.state = {
      color: 'red'
    }
  }

  render () {
    return (
      <ColorContext.Consumer>
      {color => <div>color: {color}</div>}
      </ColorContext.Consumer>
    )
  }
}
```

以上这种方式，无论组件之间的嵌套有多少层，都能起到数据通信的作用，而不用层层传递 props。

createContext 函数位于 /packages/react/src/ReactContext.js 文件中：

``` javascript
import {REACT_PROVIDER_TYPE, REACT_CONTEXT_TYPE} from 'shared/ReactSymbols';

import type {ReactContext} from 'shared/ReactTypes';

export function createContext<T>(
  defaultValue: T,
  calculateChangedBits: ?(a: T, b: T) => number,
): ReactContext<T> {
  if (calculateChangedBits === undefined) {
    calculateChangedBits = null;
  } else {
    if (__DEV__) {
      if (
        calculateChangedBits !== null &&
        typeof calculateChangedBits !== 'function'
      ) {
        console.error(
          'createContext: Expected the optional second argument to be a ' +
            'function. Instead received: %s',
          calculateChangedBits,
        );
      }
    }
  }

  const context: ReactContext<T> = {
    $$typeof: REACT_CONTEXT_TYPE,
    _calculateChangedBits: calculateChangedBits,
    // As a workaround to support multiple concurrent renderers, we categorize
    // some renderers as primary and others as secondary. We only expect
    // there to be two concurrent renderers at most: React Native (primary) and
    // Fabric (secondary); React DOM (primary) and React ART (secondary).
    // Secondary renderers store their context values on separate fields.
    _currentValue: defaultValue,
    _currentValue2: defaultValue,
    // Used to track how many concurrent renderers this context currently
    // supports within in a single renderer. Such as parallel server rendering.
    _threadCount: 0,
    // These are circular
    Provider: (null: any),
    Consumer: (null: any),
  };

  context.Provider = {
    $$typeof: REACT_PROVIDER_TYPE,
    _context: context,
  };

  let hasWarnedAboutUsingNestedContextConsumers = false;
  let hasWarnedAboutUsingConsumerProvider = false;

  if (__DEV__) {
    // ... 开发环境的配置
  } else {
    // 最终 context.Consumer = context，开发环境也是一样
    context.Consumer = context;
  }

  if (__DEV__) {
    // ... 开发环境的配置
  }

  return context;
}
```

### 注意

本文最后编辑于2020/01/12，技术更替飞快，文中部分内容可能已经过时，如有疑问，可在线提issue。