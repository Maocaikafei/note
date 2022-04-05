推荐使用组合而非继承来实现组件间的代码重用。

### 包含关系

有些组件无法提前知晓它们子组件的具体内容。在 `Sidebar`（侧边栏）和 `Dialog`（对话框）等展现通用容器（box）的组件中特别容易遇到这种情况。

我们建议这些组件使用一个特殊的 `children` prop 来将他们的子组件传递到渲染结果中

这使得别的组件可以通过 JSX 嵌套，将任意组件作为子组件传递给它们。

```jsx
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}    
    </div>
  );
}

function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```

`` JSX 标签中的所有内容都会作为一个 `children` prop 传递给 `FancyBorder` 组件。因为 `FancyBorder` 将 `{props.children}` 渲染在一个 `` 中，被传递的这些子组件最终都会出现在输出结果中。

**注意**：如果没有在 `FancyBorder`中显式调用 `{props.children}`，则在 `WelcomeDialog`中的 `FancyBorder`内部的组件将不会被渲染！（可以把这些东西当成传递给 `FancyBorder`的参数，参数需要被调用才能生效）