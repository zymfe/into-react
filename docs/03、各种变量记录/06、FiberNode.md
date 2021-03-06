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
  // 当前Fiber的状态（比如浏览器环境就是DOM节点）  
  // 不同类型的实例都会记录在stateNode上  
  // 比如DOM组件对应DOM节点实例  
  // ClassComponent对应Class实例  
  // FunctionComponent没有实例，所以stateNode值为null  
  // state更新了或props更新了均会更新到stateNode上
  this.stateNode = null;

  // FiberRoot.current = RootFiber
  // RootFiber.stateNode = FiberRoot

  // Fiber
  this.return = null;
  // 指向自己的第一个子节点
  this.child = null;
  this.sibling = null;
  this.index = 0;

  // 我们传入的 ref 属性
  this.ref = null;

  // 执行 setState 导致的新的变动带来的新的 props，即 nextProps
  this.pendingProps = pendingProps;
  // 上一次渲染完成后的 props，即 prevProps
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
 // 该 Fiber 对应的组件，所产生的 update，都会放在该队列中
  this.updateQueue = null;
  // 上一次渲染完成后的 state，执行 setState 得到新的 state 会覆盖 memoizedState
  // 新的state由updateQueue计算得出
  this.memoizedState = null;
  // 一个列表，存在该 Fiber 依赖的 contexts，events
  this.dependencies = null;

  // 参考 ReactTypeOfMode.md
  // mode 有 conCurrentMode 和 strictMode  
  // 用来描述当前Fiber和其他子树的Bitfield  
  // 共存的模式表示这个子树是否默认是异步渲染的  
  // Fiber刚被创建时，会继承父Fiber  
  // 其他标识也可以在创建的时候被设置，但是创建之后不该被修改，特别是它的子Fiber创建之前
  this.mode = mode;

  // Effects
  // 副作用是 标记组件哪些需要更新的工具、标记组件需要执行哪些生命周期的工具
  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;
  // 表示任务应该在未来的某个时间点被完成
  // 不包括其子节点所产生的任务
  this.expirationTime = NoWork;
  // 快速确定子树中是否有 update
  // 如果子节点有update的话，就记录应该更新的时间
  this.childExpirationTime = NoWork;

  // 在 Fiber 更新过程中，每个 Fiber 都有一个与其对应的 Fiber(alternate)
  // current <=> workInProgress，渲染完成之后交换位置
  // diff 出的变化记录在这个 fiber 上
  // The alternate of the current fiber is the work-in-progress, and the alternate of the work-in-progress is the current fiber.
  // A fiber's alternate is created lazily using a function called cloneFiber. Rather than always creating a new object, cloneFiber will attempt to reuse the fiber's alternate if it exists, minimizing allocations.
  // You should think of the alternate field as an implementation detail, but it pops up often enough in the codebase that it's valuable to discuss it here.
  // flush: To flush a fiber is to render its output onto the screen.
  // work-in-progress: A fiber that has not yet completed; conceptually, a stack frame which has not yet returnedre
  // At any time, a component instance has at most two fibers that correspond to it: the current, flushed fiber, and the work-in-progress fibe
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
}
```

Fiber 的作用：

1、单向遍历

2、props.children连接

3、子指父

4、doubleBuffer