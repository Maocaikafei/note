笔记内容：`Fiber节点`是如何被创建并构建`Fiber树`的

`render阶段`（Reconciler执行的阶段）开始于`performSyncWorkOnRoot`或`performConcurrentWorkOnRoot`方法的调用。这取决于本次更新是同步更新还是异步更新

在这两个方法中会调用如下两个方法

```js
// performSyncWorkOnRoot会调用该方法
function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}

// performConcurrentWorkOnRoot会调用该方法
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

可以看到，他们唯一的区别是是否调用`shouldYield`。**如果当前浏览器帧没有剩余时间，`shouldYield`会中止循环，直到浏览器有空闲时间后再继续遍历。**(`shouldYield`是由Scheduler提供的)

`workInProgress`代表当前已创建的`workInProgress fiber`

**`performUnitOfWork`方法会创建下一个`Fiber节点`并赋值给`workInProgress`，并将`workInProgress`与已创建的`Fiber节点`连接起来构成`Fiber树`**

# performUnitOfWork

我们知道`Fiber Reconciler`是从`Stack Reconciler`重构而来，通过遍历的方式实现可中断的递归，**所以`performUnitOfWork`的工作可以分为两部分：（遍历地）“递”和（遍历地）“归”**

## “递”阶段

首先从`rootFiber`开始向下深度优先遍历。为遍历到的每个`Fiber节点`调用[beginWork方法 (opens new window)](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L3058)。

该方法会根据传入的`Fiber节点`创建`子Fiber节点`，并将这两个`Fiber节点`连接起来。

当遍历到叶子节点（即没有子组件的组件）时就会进入“归”阶段

## “归”阶段

在“归”阶段会调用[completeWork (opens new window)](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCompleteWork.new.js#L652)处理`Fiber节点`。

当某个`Fiber节点`执行完`completeWork`，**如果其存在`兄弟Fiber节点`（即`fiber.sibling !== null`），会进入其`兄弟Fiber`的“递”阶段**。

**如果不存在`兄弟Fiber`，会进入`父级Fiber`的“归”阶段。**

“递”和“归”阶段会交错执行直到“归”到`rootFiber`。至此，`render阶段`的工作就结束了

```js
function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}

function performUnitOfWork(unitOfWork: Fiber): void {
  // The current, flushed, state of this fiber is the alternate. Ideally
  // nothing should rely on this, but relying on it here means that we don't
  // need an additional field on the work in progress.
  const current = unitOfWork.alternate;
  setCurrentDebugFiberInDEV(unitOfWork);

  let next;
  if (enableProfilerTimer && (unitOfWork.mode & ProfileMode) !== NoMode) {
    startProfilerTimer(unitOfWork);
    next = beginWork(current, unitOfWork, subtreeRenderLanes);
    stopProfilerTimerIfRunningAndRecordDelta(unitOfWork, true);
  } else {
    next = beginWork(current, unitOfWork, subtreeRenderLanes);
  }
  // 上方返回的next就是新生成的Fiber节点
  resetCurrentDebugFiberInDEV();
  unitOfWork.memoizedProps = unitOfWork.pendingProps;
  // 如果返回的next节点为空，说明当前节点没有子节点了，开始“归”操作
  if (next === null) { 
    // If this doesn't spawn new work, complete the current work.
    completeUnitOfWork(unitOfWork);
  } else {
    // 如果返回的next节点不为空，说明当前节点还有子节点，将子节点赋值给workInProgress，进行workLoopConcurrent的下一个循环
    workInProgress = next;
  }

  ReactCurrentOwner.current = null;
}
```

## 举例

```js
function App() {
  return (
    <div>
      i am
      <span>KaSong</span>
    </div>
  )
}
ReactDOM.render(<App />, document.getElementById("root"));
```

对应的`Fiber树`结构：

![Fiber架构](assets/fiber-16510443267979.png)

`render阶段`会依次执行：

```sh
1. rootFiber beginWork
2. App Fiber beginWork
3. div Fiber beginWork
4. "i am" Fiber beginWork
5. "i am" Fiber completeWork
6. span Fiber beginWork
7. span Fiber completeWork
8. div Fiber completeWork
9. App Fiber completeWork
10. rootFiber completeWork
```

之所以没有 “KaSong” Fiber 的 beginWork/completeWork，是因为作为一种性能优化手段，针对只有单一文本子节点的`Fiber`，`React`会特殊处理。

# beginwork

`beginWork`的工作是根据传入的`Fiber节点`，创建`子Fiber节点`，并将这两个节点连接起来

### 从传参看方法执行

```js
function beginWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  // ...省略函数体
}
```

- current：当前组件对应的`Fiber节点`对应的`current Fiber树`中的`Fiber节点`，即`workInProgress.alternate`
- workInProgress：当前组件对应的`Fiber节点`
- renderLanes：优先级相关

 组件`mount`时（除[`rootFiber`](https://react.iamkasong.com/process/doubleBuffer.html#mount时)以外），由于是首次渲染，不存在当前组件对应的`Fiber节点`在上一次更新时的`Fiber节点`，因此`mount`时`current === null`。

组件`update`时，由于之前已经`mount`过，所以`current !== null`。

所以我们可以通过`current === null ?`来区分组件是处于`mount`还是`update`

基于此原因，`beginWork`的工作可以分为两部分：

- `update`时：如果`current`存在，在满足一定条件时可以复用`current`节点，这样就能克隆`current.child`作为`workInProgress.child`，而不需要新建`workInProgress.child`。
- `mount`时：除`fiberRootNode`以外，`current === null`。会根据`fiber.tag`不同，创建不同类型的`子Fiber节点`

## update时（current  !== null）

满足如下情况时，认为该Fiber是可复用的

1. `oldProps === newProps && workInProgress.type === current.type`，即`props`与`fiber.type`不变
2. `!includesSomeLane(renderLanes, updateLanes)`，即当前`Fiber节点`优先级不够

此时，会将`didReceiveUpdate`置为`false`，否则置为true

`didReceiveUpdate`：收到更新提示了吗

## mount时（current  === null）

`didReceiveUpdate`为`false`

无论是update还是mount，都会根据`fiber.tag`不同，进入不同类型`Fiber`的创建逻辑

```js
switch (workInProgress.tag) {
    case FunctionComponent: {
      const Component = workInProgress.type;
      const unresolvedProps = workInProgress.pendingProps;
      const resolvedProps =
        workInProgress.elementType === Component
          ? unresolvedProps
          : resolveDefaultProps(Component, unresolvedProps);
      return updateFunctionComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderLanes,
      );
    }
    case ClassComponent: {
      const Component = workInProgress.type;
      const unresolvedProps = workInProgress.pendingProps;
      const resolvedProps =
        workInProgress.elementType === Component
          ? unresolvedProps
          : resolveDefaultProps(Component, unresolvedProps);
      return updateClassComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderLanes,
      );
    }
    case HostComponent:
      return updateHostComponent(current, workInProgress, renderLanes);
  }
```

对于我们常见的组件类型，如（`FunctionComponent`/`ClassComponent`/`HostComponent`），最终会进入[reconcileChildren (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L233)方法

## reconcileChildren

- 对于`mount`的组件，他会创建新的`子Fiber节点`
- 对于`update`的组件，他会将当前组件与该组件在上次更新时对应的`Fiber节点`比较（也就是俗称的`Diff`算法），将比较的结果生成新`Fiber节点`

```js
export function reconcileChildren(
  current: Fiber | null,
  workInProgress: Fiber,
  nextChildren: any,
  renderLanes: Lanes,
) {
  if (current === null) {
    // If this is a fresh new component that hasn't been rendered yet, we
    // won't update its child set by applying minimal side-effects. Instead,
    // we will add them all to the child before it gets rendered. That means
    // we can optimize this reconciliation pass by not tracking side-effects.
    workInProgress.child = mountChildFibers(
      workInProgress,
      null,
      nextChildren,
      renderLanes,
    );
  } else {
    // If the current child is the same as the work in progress, it means that
    // we haven't yet started any work on these children. Therefore, we use
    // the clone algorithm to create a copy of all the current children.

    // If we had any progressed work already, that is invalid at this point so
    // let's throw it out.
    workInProgress.child = reconcileChildFibers(
      workInProgress,
      current.child,
      nextChildren,
      renderLanes,
    );
  }
}
```

和`beginWork`一样，他也是通过`current === null ?`区分`mount`与`update`。

不论走哪个逻辑，最终他会生成新的子`Fiber节点`并赋值给`workInProgress.child`，

这个`workInProgress.child`将作为本次`beginWork`的返回值，也就是performUnitOfWork中的next，然后，作为workLoopConcurrent循环中的下一个workInprogress，开始下一次的循环

> 值得一提的是，`mountChildFibers`与`reconcileChildFibers`这两个方法的逻辑基本一致。唯一的区别是：`reconcileChildFibers`会为生成的`Fiber节点`带上`effectTag`属性，而`mountChildFibers`不会

其实，`mountChildFibers`与`reconcileChildFibers`这两个方法都是调用 `ChildReconciler(shouldTrackSideEffects)` 方法（此方法会返回一个新方法），只是参数不同。

`reconcileChildFibers`传的是true，会为生成的`Fiber节点`带上`effectTag`属性。

## effectTag

我们知道，`render阶段`的工作是在内存中进行，当工作结束后会通知`Renderer`需要执行的`DOM`操作。要执行`DOM`操作的具体类型就保存在`fiber.effectTag`中。

例如

```js
// DOM需要插入到页面中
export const Placement = /*                */ 0b00000000000010;
// DOM需要更新
export const Update = /*                   */ 0b00000000000100;
// DOM需要插入到页面中并更新
export const PlacementAndUpdate = /*       */ 0b00000000000110;
// DOM需要删除
export const Deletion = /*                 */ 0b00000000001000;
```

1、render阶段：从workLoopConcurrent方法开始进行关于workInProgress的循环，每次循环调用performUnitOfWork（workInProgress）。（循环到最深处，再从最深处循环回来）
2、performUnitOfWork方法会创建下一个Fiber节点并赋值给workInProgress，开始下一次循环
3、performUnitOfWork可分为递和归两部分，递：从`rootFiber`开始向下深度优先遍历。为遍历到的每个`Fiber节点`调用beginwork方法，该方法会根据传入的`Fiber节点`创建`子Fiber节点`，并将这两个`Fiber节点`连接起来，然后返回新建的子节点。对于mount操作，新建节点。对于update操作，可能能够复用，通过diff算法比较并生成新节点
4、当遍历到叶子节点（即没有子组件的组件）时就会进入“归”阶段。归：调用completeWork处理Fiber节点。递归交错直至rootFiber节点，至此render阶段结束。