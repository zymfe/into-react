### 什么是 FiberRoot

1、它是整个应用的起点

2、包含应用挂载的目标节点
 
3、记录整个应用更新过程的各种信息

其属性类型定义在 /react-reconciler/src/ReactFiberRoot.js 文件中

``` javascript
type PendingInteractionMap = Map<ExpirationTime, Set<Interaction>>;

type BaseFiberRootProperties = {|
  // The type of root (legacy, batched, concurrent, etc.)
  tag: RootTag,

  // Any additional information from the host associated with this root.
  containerInfo: any,
  // Used only by persistent updates.
  pendingChildren: any,
  // The currently active root fiber. This is the mutable root of the tree.
  current: Fiber,

  pingCache:
    | WeakMap<Thenable, Set<ExpirationTime>>
    | Map<Thenable, Set<ExpirationTime>>
    | null,

  finishedExpirationTime: ExpirationTime,
  // A finished work-in-progress HostRoot that's ready to be committed.
  finishedWork: Fiber | null,
  // Timeout handle returned by setTimeout. Used to cancel a pending timeout, if
  // it's superseded by a new one.
  timeoutHandle: TimeoutHandle | NoTimeout,
  // Top context object, used by renderSubtreeIntoContainer
  context: Object | null,
  pendingContext: Object | null,
  // Determines if we should attempt to hydrate on the initial mount
  +hydrate: boolean,
  // Node returned by Scheduler.scheduleCallback
  callbackNode: *,
  // Expiration of the callback associated with this root
  callbackExpirationTime: ExpirationTime,
  // Priority of the callback associated with this root
  callbackPriority: ReactPriorityLevel,
  // The earliest pending expiration time that exists in the tree
  firstPendingTime: ExpirationTime,
  // The earliest suspended expiration time that exists in the tree
  firstSuspendedTime: ExpirationTime,
  // The latest suspended expiration time that exists in the tree
  lastSuspendedTime: ExpirationTime,
  // The next known expiration time after the suspended range
  nextKnownPendingLevel: ExpirationTime,
  // The latest time at which a suspended component pinged the root to
  // render again
  lastPingedTime: ExpirationTime,
  lastExpiredTime: ExpirationTime,
|};
```

``` javascript
function FiberRootNode(containerInfo, tag, hydrate) {
  // legacy root = 0，参考 02、RootTag.md
  this.tag = tag;
  // 当前应用对应的 Fiber 对象，这里是 Root Fiber
  // 每个 ReactElement 都会有一个与之对应的 Fiber 对象
  // 这里的 Fiber 是整个应用的起点（树结构的顶点、链表结构的起点）
  this.current = null; 
  // render 方法中接收的第二个参数 container，即整个应用挂载的节点
  this.containerInfo = containerInfo;
  this.pendingChildren = null;
  this.pingCache = null;
  this.finishedExpirationTime = NoWork;
  // 记录在一次更新渲染过程中，已完成的更新任务
  // 整个应用会存在多个更新任务，每个任务的更新优先级都不一样
  // 每个任务更新完成之后，它就是一个 finishedWork，被标记在应用的 FiberRootNode 上
  // 然后从 FiberRootNode 上读取 finishedWork，最后输出到对应的 DOM 节点上
  this.finishedWork = null;
  this.timeoutHandle = noTimeout;
  // 顶层 context 对象，只有主动调用 renderSubtreeIntoContainer 时才会有
  this.context = null;
  this.pendingContext = null;
  this.hydrate = hydrate;
  this.callbackNode = null;
  this.callbackPriority = NoPriority;
  this.firstPendingTime = NoWork;
  this.firstSuspendedTime = NoWork;
  this.lastSuspendedTime = NoWork;
  this.nextKnownPendingLevel = NoWork;
  this.lastPingedTime = NoWork;
  this.lastExpiredTime = NoWork;

  if (enableSchedulerTracing) {
    this.interactionThreadID = unstable_getThreadID();
    this.memoizedInteractions = new Set();
    this.pendingInteractionMap = new Map();
  }
  if (enableSuspenseCallback) {
    this.hydrationCallbacks = null;
  }
}
```