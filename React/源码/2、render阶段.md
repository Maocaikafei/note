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

个人总结：深度优先递到最底层，然后开始往上归，如果遇到没处理过的兄弟节点，就开始兄弟节点的深度优先递操作，然后再归到原本的父节点，如此往复，一次遍历即可遍历所有的节点

## “递”阶段

首先从`rootFiber`开始向下深度优先遍历。为遍历到的每个`Fiber节点`调用[beginWork方法 (opens new window)](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L3058)。

该方法会根据传入的`Fiber节点`创建`子Fiber节点`，并将这两个`Fiber节点`连接起来。

当遍历到叶子节点（即没有子组件的组件）时就会进入“归”阶段

## “归”阶段

在“归”阶段会调用[completeWork (opens new window)](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberCompleteWork.new.js#L652)处理`Fiber节点`。（completeWork主要就是处理dom）

当某个`Fiber节点`执行完`completeWork`，**如果其存在`兄弟Fiber节点`（即`fiber.sibling !== null`），会进入其`兄弟Fiber`的“==递==”阶段**。

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

- `update`时：如果`current`存在，在满足一定条件时可以复用`current`节点，这样就能克隆`current.child`作为`workInProgress.child`，而不需要新建`workInProgress.child`。（这里的.child就是这次beginwork最终输出的结果，即传入的参数fiber的子fiber）
- `mount`时：除`fiberRootNode`以外，`current === null`。会根据`fiber.tag`不同，创建不同类型的`子Fiber节点`

## update时（current  !== null）

满足如下情况时，认为该Fiber是可复用的

1. `oldProps === newProps && workInProgress.type === current.type`，即`props`与`fiber.type`不变
2. `!includesSomeLane(renderLanes, updateLanes)`，即当前`Fiber节点`优先级不够

此时，会将`didReceiveUpdate`置为`false`，否则置为true

`didReceiveUpdate`：收到更新提示了吗

可复用时直接复用，不可复用时和mount一样调用reconcileChildren方法，在方法内部会通过diff算法比较生成fiber

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

> 通过二进制表示`effectTag`，可以方便的使用位操作为`fiber.effectTag`赋值多个`effect`。

那么，如果要通知`Renderer`将`Fiber节点`对应的`DOM节点`插入页面中，需要满足两个条件：

1. `fiber.stateNode`存在，即`Fiber节点`中保存了对应的`DOM节点`
2. `(fiber.effectTag & Placement) !== 0`，即`Fiber节点`存在`Placement effectTag`

我们知道，`mount`时，`fiber.stateNode === null`，且在`reconcileChildren`中调用的`mountChildFibers`不会为`Fiber节点`赋值`effectTag`。那么首屏渲染如何完成呢？

针对第一个问题，`fiber.stateNode`会在`completeWork`中创建，我们会在下一节介绍。

**第二个问题的答案十分巧妙：假设`mountChildFibers`也会赋值`effectTag`，那么可以预见`mount`时整棵`Fiber树`所有节点都会有`Placement effectTag`。那么`commit阶段`在执行`DOM`操作时每个节点都会执行一次插入操作，这样大量的`DOM`操作是极低效的。**

**为了解决这个问题，在`mount`时只有`rootFiber`会赋值`Placement effectTag`，在`commit阶段`只会执行一次插入操作。**



1、render阶段：从workLoopConcurrent方法开始进行关于workInProgress的循环，每次循环调用performUnitOfWork（workInProgress）。（循环到最深处，再从最深处循环回来）
2、performUnitOfWork方法会创建下一个Fiber节点并赋值给workInProgress，开始下一次循环
3、performUnitOfWork可分为递和归两部分，递：从`rootFiber`开始向下深度优先遍历。为遍历到的每个`Fiber节点`调用beginwork方法，该方法会根据传入的`Fiber节点`创建`子Fiber节点`，并将这两个`Fiber节点`连接起来，然后返回新建的子节点。对于mount操作，新建节点。对于update操作，可能能够复用，不能复用的通过diff算法比较并生成新节点。这里，新的fiber节点并没有创建对应的dom，dom的创建在归阶段中
4、当遍历到叶子节点（即没有子组件的组件）时就会进入“归”阶段。归：调用completeWork处理Fiber节点。递归交错直至rootFiber节点，至此render阶段结束。

总之，就是遍历构建workInProgress-Fiber树，其中mount的节点是新建的，update的节点可能可以复用

# completeWork

类似`beginWork`，`completeWork`也是针对不同`fiber.tag`调用不同的处理逻辑

```js
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  const newProps = workInProgress.pendingProps;

  switch (workInProgress.tag) {
    case IndeterminateComponent:
    case LazyComponent:
    case SimpleMemoComponent:
    case FunctionComponent:
    case ForwardRef:
    case Fragment:
    case Mode:
    case Profiler:
    case ContextConsumer:
    case MemoComponent:
      return null;
    case ClassComponent: {
      // ...省略
      return null;
    }
    case HostRoot: {
      // ...省略
      updateHostContainer(workInProgress);
      return null;
    }
    case HostComponent: {
      // ...省略
      return null;
    }
  // ...省略
```

## 处理HostComponent

我们重点关注页面渲染所必须的`HostComponent`（即原生`DOM组件`对应的`Fiber节点`）

和`beginWork`一样，我们根据`current === null ?`判断是`mount`还是`update`。

同时针对`HostComponent`，判断`update`时我们还需要考虑`workInProgress.stateNode != null ?`（即该`Fiber节点`是否存在对应的`DOM节点`）

```js
case HostComponent: {
  popHostContext(workInProgress);
  const rootContainerInstance = getRootHostContainer();
  const type = workInProgress.type;

  if (current !== null && workInProgress.stateNode != null) {
    updateHostComponent(
      current,
      workInProgress,
      type,
      newProps,
      rootContainerInstance,
    );

    if (current.ref !== workInProgress.ref) {
      markRef(workInProgress);
    }
  } else {
    // mount的情况
    // ...省略
  }
  return null;
}
```

### update时

当`update`时，`Fiber节点`已经存在对应`DOM节点`，所以不需要生成`DOM节点`。需要做的主要是处理`props`，比如：

- `onClick`、`onChange`等回调函数的注册
- 处理`style prop`
- 处理`children prop`

在`updateHostComponent`内部，被处理完的`props`会被赋值给`workInProgress.updateQueue`，并最终会在`commit阶段`被渲染在页面上

```
workInProgress.updateQueue = (updatePayload: any);
```

其中`updatePayload`为数组形式，他的偶数索引的值为变化的`prop key`，奇数索引的值为变化的`prop value`。

### mount时

同样，我们省略了不相关的逻辑。可以看到，`mount`时的主要逻辑包括三个：

- 为`Fiber节点`生成对应的`DOM节点`
- 将子孙`DOM节点`插入刚生成的`DOM节点`中（这些子孙dom节点是之前已经生成完的，因为归操作是自底向上的）
- 与`update`逻辑中的`updateHostComponent`类似的处理`props`的过程

```js
// mount的情况

// ...省略服务端渲染相关逻辑

const currentHostContext = getHostContext();
// 为fiber创建对应DOM节点
const instance = createInstance(
    type,
    newProps,
    rootContainerInstance,
    currentHostContext,
    workInProgress,
  );
// 将子孙DOM节点插入刚生成的DOM节点中
appendAllChildren(instance, workInProgress, false, false);
// DOM节点赋值给fiber.stateNode
workInProgress.stateNode = instance;

// 与update逻辑中的updateHostComponent类似的处理props的过程
if (
  finalizeInitialChildren(
    instance,
    type,
    newProps,
    rootContainerInstance,
    currentHostContext,
  )
) {
  markUpdate(workInProgress);
}
```

还记得[上一节](https://react.iamkasong.com/process/beginWork.html#effecttag)我们讲到：`mount`时只会在`rootFiber`存在`Placement effectTag`。那么`commit阶段`是如何通过一次插入`DOM`操作（对应一个`Placement effectTag`）将整棵`DOM树`插入页面的呢？

原因就在于`completeWork`中的`appendAllChildren`方法。

**由于`completeWork`属于“归”阶段调用的函数，每次调用`appendAllChildren`时都会将已生成的子孙`DOM节点`插入当前生成的`DOM节点`下。那么当“归”到`rootFiber`时，我们已经有一个构建好的离屏`DOM树`**。（这是一个自底向上的过程）

# effectList

至此`render阶段`的绝大部分工作就完成了。

还有一个问题：作为`DOM`操作的依据，`commit阶段`需要找到所有有`effectTag`的`Fiber节点`并依次执行`effectTag`对应操作。难道需要在`commit阶段`再遍历一次`Fiber树`寻找`effectTag !== null`的`Fiber节点`么

这显然是很低效的。

为了解决这个问题，**在`completeWork`的上层函数`completeUnitOfWork`中，每个执行完`completeWork`且存在`effectTag`的`Fiber节点`会被保存在一条被称为`effectList`的单向链表中**。

类似`appendAllChildren`，在“归”阶段，所有有`effectTag`的`Fiber节点`都会被追加在`effectList`中，最终形成一条以`rootFiber.firstEffect`为起点的单向链表

这样，在`commit阶段`只需要遍历`effectList`就能执行所有`effect`了。

借用`React`团队成员**Dan Abramov**的话：`effectList`相较于`Fiber树`，就像圣诞树上挂的那一串彩灯

至此，`render阶段`全部工作完成。在`performSyncWorkOnRoot`函数中`fiberRootNode`被传递给`commitRoot`方法，开启`commit阶段`工作流程

```js
commitRoot(root);
```