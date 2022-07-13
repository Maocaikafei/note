官网的定义：

> `componentDidMount()` 会在组件挂载后（插入 DOM 树中）立即调用。依赖于 DOM 节点的初始化应该放在这里。如需通过网络请求获取数据，此处是实例化请求的好地方。
>
> 这个方法是比较适合添加订阅的地方。如果添加了订阅，请不要忘记在 `componentWillUnmount()` 里取消订阅
>
> 你可以在 `componentDidMount()` 里直接调用 `setState()`。它将触发额外渲染，但此渲染会发生在浏览器更新屏幕之前。如此保证了即使在 `render()` 两次调用的情况下，用户也不会看到中间状态。请谨慎使用该模式，因为它会导致性能问题。通常，你应该在 `constructor()` 中初始化 state。如果你的渲染依赖于 DOM 节点的大小或位置，比如实现 modals 和 tooltips 等情况下，你可以使用此方式处理

这里有两部分要关注：

1、`componentDidMount()` 会在组件挂载后（插入 DOM 树中）立即调用

2、你可以在 `componentDidMount()` 里直接调用 `setState()`。它将触发额外渲染，但此渲染会发生在浏览器更新屏幕之前



`插入 DOM 树后`和 `此渲染会发生在浏览器更新屏幕之前` 看似有些冲突，但其实插入DOM树 !== 更新屏幕，插入DOM树是发生在更新屏幕之前的

而又因为`componentDidMount`属于js操作，因此会阻塞屏幕渲染

所以，在react新的Fiber架构中，useEffect选择了异步执行，就是为了防止同步执行时阻塞浏览器渲染