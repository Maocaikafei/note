### 什么是state

State 与 props 类似，但是 state 是私有的，并且完全受控于当前组件。

### 将生命周期方法添加到 Class 中

在具有许多组件的应用程序中，**当组件被销毁时释放所占用的资源是非常重要的**。

要实现一个自动刷新的时钟功能，需要用到计时器：

当 `Clock` 组件第一次被渲染到 DOM 中的时候，就为其[设置一个计时器](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval)。这在 React 中被称为“**挂载（mount）**”

同时，当 DOM 中 `Clock` 组件被删除的时候，应该[清除计时器](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/clearInterval)。这在 React 中被称为**“卸载（unmount）”**。

我们可以为 class 组件声明一些特殊的方法，当组件挂载或卸载时就会去执行这些方法，这些方法叫做**“生命周期方法”**

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {  }
  componentWillUnmount() {  }
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

`componentDidMount()` 方法会在组件已经被渲染到 DOM 中后运行，

一旦 `Clock` 组件从 DOM 中被移除，React 就会调用 `componentWillUnmount()` 生命周期方法

### setState()是如何实现更新页面的

浏览器每秒都会调用一次 `tick()` 方法。 在这方法之中，`Clock` 组件会通过调用 `setState()` 来计划进行一次 UI 更新。**得益于 `setState()` 的调用，React 能够知道 state 已经改变了，然后会重新调用 `render()` 方法来确定页面上该显示什么。这一次，`render()` 方法中的 `this.state.date` 就不一样了，如此以来就会渲染输出更新过的时间。React 也会相应的更新 DOM。**

### 正确地使用 State

#### 不要直接修改 State

例如，此代码不会重新渲染组件：`this.state.comment = 'Hello';`（确认下是修改了state但是页面没更新，还是无法修改？）

应该使用 `setState()`：`this.setState({comment: 'Hello'});`

构造函数是唯一可以给 `this.state` 赋值的地方

#### State 的更新可能是异步的

出于性能考虑，**React 可能会把多个 `setState()` 调用合并成一个调用**。

**因为 `this.props` 和 `this.state` 可能会异步更新，所以不要依赖他们的值来更新下一个状态。**

```js
// Wrong 此代码可能会无法更新计数器
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

要解决这个问题，可以**让 `setState()` 接收一个函数而不是一个对象。这个函数用上一个 state 作为第一个参数，将此次更新被应用时的 props 做为第二个参数**：

```js
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

确认下是不是执行这条语句后，“监听state、props，当出现下一次变化时，执行setState”

还是永久生效？每次state、props变化都执行setState

#### State 的更新会被合并

当你调用 `setState()` 的时候，React 会把你提供的对象合并到当前的 state

```js
this.state = {
      posts: [],      
      comments: []    
};

this.setState({
      posts: response.posts
});

this.setState({
      comments: response.comments
});
```

这里的合并是浅合并，所以 `this.setState({comments})` 完整保留了 `this.state.posts`， 但是完全替换了 `this.state.comments`。

### 数据是向下流动的

不管是父组件或是子组件都无法知道某个组件是有状态(state)的还是无状态的，并且它们也并不关心它是函数组件还是 class 组件

这就是为什么称 state 为局部的或是封装的的原因。**除了拥有并设置了它的组件，其他组件都无法访问。**

组件可以选择把它的 state 作为 props 向下传递到它的子组件中

这通常会被叫做“自上而下”或是“单向”的数据流。任何的 state 总是所属于特定的组件，而且从该 state 派生的任何数据或 UI 只能影响树中“低于”它们的组件。