## what

术语 [“render prop”](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) 是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术

具有 render prop 的组件接受一个返回 React 元素的函数，并在组件内部通过调用此函数来实现自己的渲染逻辑。

```
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```

**render prop 是一个用于告知组件需要渲染什么内容的函数 prop，**能够动态决定什么需要渲染

这种技术和高阶组件其实很像，可以使用带有 render prop 的常规组件来实现大多数[高阶组件](https://react.docschina.org/docs/higher-order-components.html) (HOC)

高阶组件其实就是包装：返回一个新的包装元素，该元素会处理传入的参数，将处理结果传入被包装的元素，然后将其作为结果返回

render prop：其实也是包装，用新的包装旧的，还是用旧的包装prop？都行

 ***任何*被用于告知组件需要渲染什么内容的函数 prop 在技术上都可以被称为 “render prop”**