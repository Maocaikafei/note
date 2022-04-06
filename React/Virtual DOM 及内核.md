### 什么是 Virtual DOM？

Virtual DOM 是一种编程概念。在这个概念里， UI 以一种理想化的，或者说“虚拟的”表现形式被保存于内存中，**并通过如 ReactDOM 等类库使之与“真实的” DOM 同步**。这一过程叫做[协调](https://react.docschina.org/docs/reconciliation.html)。（React 的 “diffing” 算法）

这种方式赋予了 React 声明式的 API：您告诉 React 希望让 UI 是什么状态，React 就确保 DOM 匹配该状态。这使您可以从属性操作、事件处理和手动 DOM 更新这些在构建应用程序时必要的操作中解放出来。