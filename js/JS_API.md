### 跨上下文消息

跨文档消息，有时候也简称为 XDM（cross-document messaging），**是一种在不同执行上下文（如不同工作线程或不同源的页面）间传递信息的能力**。例如，`www.wrox.com` 上的页面想要与包含在内嵌窗格中的 `p2p.wrox.com` 上面的页面通信。在 XDM 之前，要以安全方式实现这种通信需要很多工作。XDM 以安全易用的方式规范化了这个功能

XDM 的核心是 `postMessage()`方法。除了 XDM，这个方法名还在 HTML5 中很多地方用到过， 但目的都一样，都是把数据传送到另一个位置

postMessage()方法接收 3 个参数：**消息、表示目标接收源的字符串和可选的可传输对象的数组**（只与工作线程相关）。第二个参数对于安全非常重要，其可以限制浏览器交付数据的目标。

```js
let iframeWindow = document.getElementById("myframe").contentWindow; 
iframeWindow.postMessage("A secret", "http://www.wrox.com"); 
```

最后一行代码尝试向内嵌窗格中发送一条消息，而且指定了源必须是"`http://www.wrox.com`"。 如果源匹配，那么消息将会交付到内嵌窗格；否则，postMessage()什么也不做。这个限制可以保护 信息不会因地址改变而泄露。**如果不想限制接收目标，则可以给 postMessage()的第二个参数传"*"， 但不推荐这么做**



**接收到 XDM 消息后，window 对象上会触发 message 事件**。这个事件是异步触发的，因此从消息发出到接收到消息（接收窗口触发 message 事件）可能有延迟。

传给 onmessage 事件处理程序的 event对象包含以下 3 方面重要信息

- data：作为第一个参数传递给 postMessage()的字符串数据
- origin：发送消息的文档源，例如"`http://www.wrox.com`"
- source：**发送消息的文档中 window 对象的代理**。这个代理对象主要**用于在发送上一条消息的窗口中执行 postMessage()方法**。如果发送窗口有相同的源，那么这个对象应该就是 window 对象(接收方的第三个参数并不对应发送方的第三个参数)

**接收消息之后验证发送窗口的源是非常重要的。与 postMessage()的第二个参数可以保证数据不 会意外传给未知页面一样，在 onmessage 事件处理程序中检查发送窗口的源可以保证数据来自正确的地方**。

```js
window.addEventListener("message", (event) => { 
 // 确保来自预期发送者
 if (event.origin == "http://www.wrox.com") { 
 // 对数据进行一些处理
 processMessage(event.data); 
 // 可选：向来源窗口发送一条消息
 event.source.postMessage("Received!", "http://p2p.wrox.com"); 
 } 
});
```

大多数情况下，event.source 是某个 window 对象的代理，而非实际的 window 对象。因此不能通过它访问所有窗口下的信息。最好只使用 postMessage()，这个方法永远存在而且可以调用。

XDM 有一些怪异之处。首先，postMessage()的第一个参数的最初实现始终是一个字符串。后来， 第一个参数改为允许任何结构的数据传入，不过并非所有浏览器都实现了这个改变。为此，**最好就是只 通 过 postMessage() 发送字符串。如果需要传递结构化数据，那么最好先对该数据调用 JSON.stringify()，通过 postMessage()传过去之后，再在 onmessage 事件处理程序中调用 JSON.parse()**

