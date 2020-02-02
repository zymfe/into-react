``` javascript
export function createUpdate(
  expirationTime: ExpirationTime,
  suspenseConfig: null | SuspenseConfig,
): Update<*> {
  let update: Update<*> = {
    expirationTime,
    suspenseConfig,

    // 0
    tag: UpdateState,
    // setState 接收的第一个参数（对象或方法），或应用首次渲染时传入的 element 
    payload: null,
    // 对应的回调，render 和 setState 都有
    callback: null,
    // updateQueue 是一个单向链表，next 指向下一个更新
    next: (null: any),
  };
  update.next = update;
  if (__DEV__) {
    update.priority = getCurrentPriorityLevel();
  }
  return update;
}
```