### 什么是 Fiber

1、每一个 ReactElement 对象都有一个对应的 Fiber 对象，用于记录节点的各种状态。比如 class component 的 props 和 state 都是记录在 Fiber 对象上的。

2、串联整个应用形成树结构。 

``` javascript
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // tag 的所有类型，参考 ReactWorkTags.md
  this.tag = tag;
  this.key = key;
  // ReactElement.type，即调用 createElement 的第一个参数 
  this.elementType = null;
  // 异步组件 resolve 之后返回的内容，一般是 function 或 class
  this.type = null;
  // 与当前 Fiber 相关的状态，如果是浏览器环境就是 DOM 节点
  // 如果是 functional component，则没有 stateNode
  // 如果是 class component，则对应 class instance
  this.stateNode = null;

  // FiberRoot.current = RootFiber
  // RootFiber.stateNode = FiberRoot

  // Fiber
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  // 我们传入的 ref 属性
  this.ref = null;

  // 执行 setState 导致的新的变动带来的新的 props
  this.pendingProps = pendingProps;
  // 上一次渲染完成后的 props
  this.memoizedProps = null;
  /**
   * const queue: UpdateQueue<State> = {
      baseState: fiber.memoizedState,
      baseQueue: null,
      shared: {
        pending: null,
      },
      effects: null,
    };
    this.updateQueue = queue
  */
  this.updateQueue = null;
  // 上一次渲染完成后的 state，执行 setState 得到新的 state 会覆盖 memoizedState
  this.memoizedState = null;
  this.dependencies = null;

  // 参考 ReactTypeOfMode.md
  // Fiber 被创建的时候会继承父 Fiber
  this.mode = mode;

  // Effects
  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;
  // 表示任务应该在未来的某个时间点被完成
  // 不包括其子节点所产生的任务
  this.expirationTime = NoWork;
  //  记录其子节点的过期时间
  this.childExpirationTime = NoWork;

  // 在 Fiber 更新过程中，每个 Fiber 都有一个与其对应的 Fiber
  // current <=> workInProgress，渲染完成之后交换位置，相当于是一个 patch 的过程
  // 也是一个对象池，不需要重复创建 current 和 workInProgress 对象，可重复利用
  // 在 React 中叫做 DoubleBuffer
  this.alternate = null;

  if (enableProfilerTimer) {
    // Note: The following is done to avoid a v8 performance cliff.
    //
    // Initializing the fields below to smis and later updating them with
    // double values will cause Fibers to end up having separate shapes.
    // This behavior/bug has something to do with Object.preventExtension().
    // Fortunately this only impacts DEV builds.
    // Unfortunately it makes React unusably slow for some applications.
    // To work around this, initialize the fields below with doubles.
    //
    // Learn more about this here:
    // https://github.com/facebook/react/issues/14365
    // https://bugs.chromium.org/p/v8/issues/detail?id=8538
    this.actualDuration = Number.NaN;
    this.actualStartTime = Number.NaN;
    this.selfBaseDuration = Number.NaN;
    this.treeBaseDuration = Number.NaN;

    // It's okay to replace the initial doubles with smis after initialization.
    // This won't trigger the performance cliff mentioned above,
    // and it simplifies other profiler code (including DevTools).
    this.actualDuration = 0;
    this.actualStartTime = -1;
    this.selfBaseDuration = 0;
    this.treeBaseDuration = 0;
  }

  // This is normally DEV-only except www when it adds listeners.
  // TODO: remove the User Timing integration in favor of Root Events.
  if (enableUserTimingAPI) {
    this._debugID = debugCounter++;
    this._debugIsCurrentlyTiming = false;
  }

  if (__DEV__) {
    this._debugSource = null;
    this._debugOwner = null;
    this._debugNeedsRemount = false;
    this._debugHookTypes = null;
    if (!hasBadMapPolyfill && typeof Object.preventExtensions === 'function') {
      Object.preventExtensions(this);
    }
  }
}
```