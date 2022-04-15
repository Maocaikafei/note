Hook 是一些可以让你在函数组件里“钩入” React state 及生命周期等特性的函数，在不编写 class 的情况下使用 state 以及其他的 React 特性。

Hook 不能在 class 组件中使用

## 使用hook的动机

### **Hook 可在无需修改组件结构的情况下复用状态逻辑**

在组件之间复用状态逻辑很难。React 没有提供将可复用性行为“附加”到组件的途径（例如，把组件连接到 store）。如果你使用过 React 一段时间，你也许会熟悉一些解决此类问题的方案，比如 [render props](https://react.docschina.org/docs/render-props.html) 和 [高阶组件](https://react.docschina.org/docs/higher-order-components.html)。但是这类方案需要重新组织你的组件结构，这可能会很麻烦，使你的代码难以理解。

例如：自定义hook

### **Hook 将组件中相互关联的部分拆分成更小的函数**

(解决 class 中生命周期函数经常包含不相关的逻辑，但又把相关逻辑分离到了几个不同方法中的问题)

我们经常维护一些组件，组件起初很简单，但是逐渐会被状态逻辑和副作用充斥。每个生命周期常常包含一些不相关的逻辑。例如，组件常常在 `componentDidMount` 和 `componentDidUpdate` 中获取数据。但是，同一个 `componentDidMount` 中可能也包含很多其它的逻辑，如设置事件监听，而之后需在 `componentWillUnmount` 中清除。相互关联且需要对照修改的代码被进行了拆分(分别在didmount和unmount中)，而完全不相关的代码却在同一个方法中组合在一起。如此很容易产生 bug，并且导致逻辑不一致。

为了解决这个问题，**Hook 将组件中相互关联的部分拆分成更小的函数（比如设置订阅或请求数据）**，而并非强制按照生命周期划分。

例如：useEffect

### **Hook 使你在非 class 的情况下可以使用更多的 React 特性**

例如：state

## Hook 规则

Hook 本质就是 JavaScript 函数，但是在使用它时需要遵循两条规则

### 只在最顶层使用 Hook

**不要在循环，条件或嵌套函数中调用 Hook，** 确保总是在你的 React 函数的最顶层调用他们。遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 `useState` 和 `useEffect` 调用之间保持 hook 状态的正确。

### 只在 React 函数中调用 Hook

**不要在普通的 JavaScript 函数中调用 Hook。**你可以：

- ✅ 在 React 的函数组件中调用 Hook
- ✅ 在自定义 Hook 中调用其他 Hook

### React 怎么知道哪个 state 对应哪个 `useState`？

React 靠的是 Hook 调用的顺序

**只要 Hook 的调用顺序在多次渲染之间保持一致**，React 就能正确地将内部 state 和对应的 Hook 进行关联。但如果我们将一个 Hook (例如 `persistForm` effect) 调用放到一个条件语句中会发生什么呢？

```jsx
 // 🔴 在条件语句中使用 Hook 违反第一条规则
  if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
  }
```

在第一次渲染中 `name !== ''` 这个条件值为 `true`，所以我们会执行这个 Hook。但是下一次渲染时我们可能清空了表单，表达式值变为 `false`。此时的渲染会跳过该 Hook，Hook 的调用顺序发生了改变：

```
useState('Mary')           // 1. 读取变量名为 name 的 state（参数被忽略）
// useEffect(persistForm)  // 🔴 此 Hook 被忽略！
useState('Poppins')        // 🔴 2 （之前为 3）。读取变量名为 surname 的 state 失败
useEffect(updateTitle)     // 🔴 3 （之前为 4）。替换更新标题的 effect 失败
```

**React 不知道第二个 `useState` 的 Hook 应该返回什么。React 会以为在该组件中第二个 Hook 的调用像上次的渲染一样，对应得是 `persistForm` 的 effect**，但并非如此。从这里开始，后面的 Hook 调用都被提前执行，导致 bug 的产生。

**这就是为什么 Hook 需要在我们组件的最顶层调用。**如果我们想要有条件地执行一个 effect，可以将判断放到 Hook 的*内部*：

## State Hook

### 声明 State 变量

在 class 中，我们通过在构造函数中设置 `this.state` 为 `{ count: 0 }` 来初始化 `count` state 为 `0`

在函数组件中，我们没有 `this`，所以我们不能分配或读取 `this.state`。我们直接在组件中调用 `useState` Hook

**`调用 useState` 方法的时候做了什么?** 它定义一个 “state 变量”。我们的变量叫 `count`， 但是我们可以叫他任何名字，比如 `banana`。([数组解构](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring)的语法让我们在调用 `useState` 时可以给 state 变量取不同的名字。当然，这些名字并不是 `useState` API 的一部分。React 假设当你多次调用 `useState` 的时候，你能保证每次渲染时它们的调用顺序是不变的。)

**初始 state 参数只有在第一次渲染时会被用到**。`useState()` 方法里面唯一的参数就是初始 state

**`useState` 方法的返回值是什么？** 返回值为：当前 state 以及更新 state 的函数。这就是我们写 `const [count, setCount] = useState()` 的原因。这与 class 里面 `this.state.count` 和 `this.setState` 类似，唯一区别就是你需要成对的获取它们。（所以其实就是个数组解构）

` const [fruit, setFruit] = useState('banana');`等价于

```js
var fruitStateVariable = useState('banana'); // 返回一个有两个元素的数组
var fruit = fruitStateVariable[0]; // 数组里的第一个值
var setFruit = fruitStateVariable[1]; // 数组里的第二个值
```

我们声明了一个叫 `count` 的 state 变量，然后把它设为 `0`。**React 会在重复渲染时记住它当前的值，并且提供最新的值给我们的函数**。

## Effect Hook

在 React 组件中执行数据获取、订阅或者手动修改 DOM。这些操作被称为“副作用”，或者简称为“作用”。

`useEffect` 就是一个 Effect Hook，给函数组件增加了操作副作用的能力。它跟 class 组件中的 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 具有相同的用途，只不过被合并成了一个 API。（可以把 `useEffect` Hook 看做 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合）

当你调用 `useEffect` 时，就是在告诉 React 在完成对 DOM 的更改后（确认下，当state变了而页面元素不变时，会不会触发）运行你的“副作用”函数。由于副作用函数是在组件内声明的，所以它们可以访问到组件的 props 和 state。默认情况下，React 会在每次渲染后调用副作用函数 —— **包括**第一次渲染的时候。

副作用函数还可以通过返回一个函数来指定如何“清除”副作用。例如，在下面的组件中使用副作用函数来订阅好友的在线状态，并通过取消订阅来进行清除操作：

```jsx
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);    return () => {      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);    };  });
  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

在这个示例中，React 会在组件销毁时取消对 `ChatAPI` 的订阅，然后在后续渲染时重新执行副作用函数。（如果传给 `ChatAPI` 的 `props.friend.id` 没有变化，你也可以[告诉 React 跳过重新订阅](https://react.docschina.org/docs/hooks-effect.html#tip-optimizing-performance-by-skipping-effects)。）

通过使用 Hook，你可以把组件内相关的副作用组织在一起（例如创建订阅及取消订阅），而不用把它们拆分到不同的生命周期函数里。

### 与class的区别

在 class ，我们需要在两个生命周期函数中编写重复的代码。这是因为很多情况下，我们希望在组件加载和更新时执行同样的操作。从概念上说，我们希望它在每次渲染之后执行 —— 但 React 的 class 组件没有提供这样的方法。即使我们提取出一个方法，我们还是要在两个地方调用它。

### 每个 effect “属于”一次特定的渲染

传递给 `useEffect` 的函数在每次渲染中都会有所不同，这是刻意为之的。事实上这正是我们可以在 effect 中获取最新的 `count` 的值，而不用担心其过期的原因。每次我们重新渲染，都会生成*新的* effect，替换掉之前的。某种意义上讲，effect 更像是渲染结果的一部分 —— 每个 effect “属于”一次特定的渲染

### 何时执行？

每次渲染后执行（默认情况下，它在第一次渲染之后*和*每次更新之后都会执行）

### 需要清除的 effect

有一些副作用是需要清除的。例如**订阅外部数据源**。这种情况下，清除工作是非常重要的，可以防止引起内存泄露！

如果你的 effect 返回一个函数，React 将会在执行清除操作时调用它

```jsx
useEffect(() => {    
    function handleStatusChange(status) {      
        setIsOnline(status.isOnline);    
    }    
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);         // Specify how to clean up after this effect:    
    return function cleanup() { 
     ChatAPI.unsubscribeFromFriendStatus(props.friend.id,handleStatusChange);    };  
});
```

### **为什么要在 effect 中返回一个函数？** 

这是 effect 可选的清除机制。每个 effect 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。它们都属于 effect 的一部分。

### **React 何时清除 effect？** 

React 会在组件卸载的时候执行清除操作。正如之前学到的，effect 在每次渲染的时候都会执行。这就是为什么 **React *会*在执行当前 effect 之前对上一个 effect 进行清除**。

### **Hook 允许我们按照代码的用途分离他们，**

而不是像生命周期函数那样。**React 将按照 effect 声明的顺序依次调用组件中的*每一个* effect**。

### 为什么每次更新的时候都要运行 Effect

为什么 effect 的清除阶段在每次重新渲染时都会执行，而不是只在卸载组件的时候执行一次。

假设有三个时间段的state，分别为1、2、3。再假设清除函数需要用到state，那么如果只在卸载组件时调用清除函数，就无法清除state1与2。

所以清除函数的逻辑应该如下：

```jsx
function FriendStatus(props) {
  // ...
  useEffect(() => {
    // ...
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  
// Mount with { friend: { id: 100 } } props
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);     // 运行第一个 effect

// Update with { friend: { id: 200 } } props
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // 清除上一个 effect
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);     // 运行下一个 effect

// Update with { friend: { id: 300 } } props
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // 清除上一个 effect
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);     // 运行下一个 effect

// Unmount
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // 清除最后一个 effect
```

### 空数组（`[]`）

如果你传入了一个空数组（`[]`），effect 内部的 props 和 state 就会一直拥有其初始值。

```
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1); // 这个 effect 依赖于 `count` state    }, 1000);
    return () => clearInterval(id);
  }, []); // 🔴 Bug: `count` 没有被指定为依赖
  return <h1>{count}</h1>;
}
```

传入空的依赖数组 `[]`，意味着该 hook 只在组件挂载时运行一次，并非重新渲染时。但如此会有问题，在 `setInterval` 的回调中，`count` 的值不会发生变化。因为当 effect 执行时，我们会创建一个闭包，并将 `count` 的值被保存在该闭包当中，且初值为 `0`。每隔一秒，回调就会执行 `setCount(0 + 1)`，因此，`count` 永远不会超过 1。

指定 `[count]` 作为依赖列表就能修复这个 Bug，但会导致每次改变发生时定时器都被重置。事实上，每个 `setInterval` 在被清除前（类似于 `setTimeout`）都会调用一次。但这并不是我们想要的。要解决这个问题，我们可以使用 [`setState` 的函数式更新形式](https://react.docschina.org/docs/hooks-reference.html#functional-updates)。它允许我们指定 state 该 *如何* 改变而不用引用 *当前* state：

```
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1); // ✅ 在这不依赖于外部的 `count` 变量    }, 1000);
    return () => clearInterval(id);
  }, []); // ✅ 我们的 effect 不适用组件作用域中的任何变量
  return <h1>{count}</h1>;
}
```

### 延迟调用 `useEffect`

React 会等待浏览器完成画面渲染之后才会延迟调用 `useEffect`

## 自定义Hook

通过自定义 Hook，可以将组件逻辑提取到可重用的函数中

目前为止，在 React 中有两种流行的方式来共享组件之间的状态逻辑: [render props](https://react.docschina.org/docs/render-props.html) 和[高阶组件](https://react.docschina.org/docs/higher-order-components.html)，现在让我们来看看 Hook 是如何在让你不增加组件的情况下解决相同问题的

### 提取自定义 Hook

当我们想在两个函数之间共享逻辑时，我们会把它提取到第三个函数中。而组件和 Hook 都是函数，所以也同样适用这种方式

**自定义 Hook 是一个函数，其名称以 “`use`” 开头，函数内部可以调用其他的 Hook。**

```js
mport { useState, useEffect } from 'react';

function useFriendStatus(friendID) {  
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

与组件中一致，请确保只在自定义 Hook 的顶层无条件地调用其他 Hook。

与 React 组件不同的是，自定义 Hook 不需要具有特殊的标识。我们可以自由的决定它的参数是什么，以及它应该返回什么（如果需要的话）。换句话说，它就像一个正常的函数。但是它的名字应该始终以 `use` 开头，这样可以一眼看出其符合 [Hook 的规则](https://react.docschina.org/docs/hooks-rules.html)。

## 使用自定义 Hook

**自定义 Hook 是一种自然遵循 Hook 设计的约定，而并不是 React 的特性。**

**自定义 Hook 必须以 “`use`” 开头**]。这个约定非常重要。不遵循的话，由于无法判断某个函数是否包含对其内部 Hook 的调用，React 将无法自动检查你的 Hook 是否违反了 [Hook 的规则](https://react.docschina.org/docs/hooks-rules.html)。

**在两个组件中使用相同的 Hook 会共享 state 吗？**不会。自定义 Hook 是一种重用*状态逻辑*的机制(例如设置为订阅并存储当前值)，所以每次使用自定义 Hook 时，其中的所有 state 和副作用都是完全隔离的。