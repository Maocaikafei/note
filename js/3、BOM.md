### 什么是BOM

浏览器对象模型（BOM，Browser Object Model）

BOM 提供了与网页无关的浏览器功能对象。

### window 对象

BOM 的核心是 window 对象，表示浏览器的实例。window 对象在浏览器中有两重身份，一个是 ECMAScript 中的 Global 对象，另一个就是浏览器窗口的 JavaScript 接口。这意味着网页中定义的所有 对象、变量和函数都以 window 作为其 Global 对象，都可以访问其上定义的 parseInt()等全局方法

#### Global 作用域

因为 window 对象被复用为 ECMAScript 的 Global 对象，所以通过 var 声明的所有全局变量和函数都会变成 window 对象的属性和方法

如果在这里使用 let 或 const 替代 var，则不会把变量添加给全局对象(window)

```js
let age = 29; 
const sayAge = () => alert(this.age); 
alert(window.age); // undefined 
sayAge(); // undefined 
window.sayAge(); // TypeError: window.sayAge is not a function 
```

#### 窗口关系

top 对象始终指向最上层（最外层）窗口，即浏览器窗口本身。而 parent 对象则始终指向当前窗 口的父窗口。如果当前窗口是最上层窗口，则 parent 等于 top（都等于 window）。最上层的 window 如果不是通过 window.open()打开的，那么其 name 属性就不会包含值

还有一个 self 对象，它是终极 window 属性，始终会指向 window。实际上，self 和 window 就 是同一个对象。之所以还要暴露 self，就是为了和 top、parent 保持一致

这些属性都是 window 对象的属性，因此访问 window.parent、window.top 和 window.self 都可以。这意味着可以把访问多个窗口的 window 对象串联起来，比如 window.parent.parent

#### 窗口位置与像素比

window 对象的位置可以通过不同的属性和方法来确定。现代浏览器提供了 screenLeft 和 screenTop 属性，用于表示窗口相对于屏幕左侧和顶部的位置 ，返回值的单位是 CSS 像素

可以使用 moveTo()和 moveBy()方法移动窗口。这两个方法都接收两个参数，其中 moveTo()接 收要移动到的新位置的绝对坐标 x 和 y；而 moveBy()则接收相对当前位置在两个方向上移动的像素数

```js
// 把窗口移动到左上角
window.moveTo(0,0); 
// 把窗口向下移动 100 像素
window.moveBy(0, 100); 
// 把窗口移动到坐标位置(200, 300) 
window.moveTo(200, 300); 
// 把窗口向左移动 50 像素
window.moveBy(-50, 0);
```

#### 窗口大小

在不同浏览器中确定浏览器窗口大小没有想象中那么容易。所有现代浏览器都支持 4 个属性： innerWidth、innerHeight、outerWidth 和 outerHeight。**outerWidth 和 outerHeight 返回浏览器窗口自身的大小**（不管是在最外层 window 上使用，还是在窗格中使用）。**innerWidth 和 innerHeight 返回浏览器窗口中页面视口的大小**（不包含浏览器边框和工具栏）

document.documentElement.clientWidth 和 document.documentElement.clientHeight 返回页面视口的宽度和高度。

可以使用resizeTo()和resizeBy()方法调整窗口大小。这两个方法都接收两个参数，resizeTo() 接收新的宽度和高度值，而 resizeBy()接收宽度和高度各要缩放多少

#### 视口位置

浏览器窗口尺寸通常无法满足完整显示整个页面，为此用户可以通过滚动在有限的视口中查看文 档。度量**文档相对于视口滚动距离**的属性有两对，返回相等的值：window.pageXoffset/window.  scrollX 和 window.pageYoffset/window.scrollY

可以使用 **scroll()、scrollTo()和 scrollBy()**方法滚动页面。这 3 个方法都接收表示相对视口距离的 x 和 y 坐标，这两个参数在前两个方法中表示要滚动到的坐标，在最后一个方法中表示滚动的距离

这几个方法也都接收一个 ScrollToOptions 字典，除了提供偏移值，还可以通过 behavior 属性告诉浏览器是否平滑滚动

```js
// 正常滚动 
window.scrollTo({ 
 left: 100, 
 top: 100, 
 behavior: 'auto' 
}); 
// 平滑滚动
window.scrollTo({ 
 left: 100, 
 top: 100, 
 behavior: 'smooth' 
});
```

#### 导航与打开新窗口

window.open()方法可以用于导航到指定 URL，也可以用于打开新浏览器窗口。

#### 定时器

JavaScript 在浏览器中是**单线程**执行的，但允许使用定时器指定在某个时间之后或每隔一段时间就执行相应的代码。

**setTimeout()用于指定在一定时间后执行某些代码。**

**而 setInterval()用于指定 每隔一段时间执行某些代码。**

##### setTimeout()

setTimeout()方法通常接收两个参数：要执行的代码和在执行回调函数前等待的时间（毫秒）。第一个参数可以是包含 JavaScript 代码的字符串（类似于传给 eval()的字符串）或者一个函数

第二个参数是要等待的毫秒数，**而不是要执行代码的确切时间**。JavaScript 是单线程的，所以每次只能执行一段代码。**为了调度不同代码的执行，JavaScript 维护了一个任务队列。其中的任务会按照添加到队列的先后顺序执行**。setTimeout()的第二个参数只是**告诉 JavaScript 引擎在指定的毫秒数过后 把任务添加到这个队列**。如果队列是空的，则会立即执行该代码。如果队列不是空的，则代码必须等待前面的任务执行完才能执行。 **调用 setTimeout()时，会返回一个表示该超时排期的数值 ID。这个超时ID是被排期执行代码唯一标识符，可用于取消该任务。**要取消等待中的排期任务，可以调用 **clearTimeout()**方法并传入超时ID

只要是在指定时间到达之前调用 clearTimeout()，就可以取消超时任务。

`所有超时执行的代码（函数）都会在全局作用域中的一个匿名函数中运行，因此函 数中的 this 值在非严格模式下始终指向 window，而在严格模式下是 undefined。如果给setTimeout()提供了一个箭头函数，那么 this 会保留为定义它时所在的作用域`

##### setInterval()

指定的任务会每隔指定时间就执行一次，直到取消循环定时或者页面卸载。

setInterval()同样可以接收两个参数：要执行的代码（字符 串或函数），以及把下一次执行定时代码的任务添加到队列要等待的时间（毫秒）。下面是一个例子： `setInterval(() => alert("Hello world!"), 10000);` 

这里的间隔时间，指的是**向队列添加新任务之前**等待的时间。比如，调用 setInterval()的时间为 01:00:00，间隔时间为 3000 毫秒。这意 味着 01:00:03 时，浏览器会把任务添加到执行队列。**浏览器不关心这个任务什么时候执行或者执行要花多长时间。因此，到了 01:00:06，它会再向队列中添加一个任务。**由此可看 出，执行时间短、非阻塞的回调函数比较适合 setInterval()。

setInterval()方法也会返回一个循环定时 ID，可以用于在未来某个时间点上取消循环定时。要取消循环定时，可以调用 clearInterval()并传入定时 ID。**相对于 setTimeout()而言，取消定时的能力对 setInterval()更加重要**。毕竟，如果一直不管它，那么定时任务会一直执行到页面卸载。

**使用setTimeOut()实现每隔一段时间执行某代码：**

```js
let num = 0; 
let max = 10; 
let incrementNumber = function() { 
 num++; 
 // 如果还没有达到最大值，再设置一个超时任务
 if (num < max) { 
 setTimeout(incrementNumber, 500); 
 } else { 
 alert("Done"); 
 } 
} 
setTimeout(incrementNumber, 500);
```

这个模式是设置循环任务的推荐做法。setIntervale()在实践中很少会在 生产环境下使用，因为一个任务结束和下一个任务开始之间的时间间隔是无法保证的，有些循环定时任务可能会因此而被跳过。而**像前面这个例子中一样使用 setTimeout()则能确保不会出现这种情况**。一 般来说，最好不要使用 setInterval()。(这样能保证是任务执行过后，再开始计算下一个任务的间隔)

### location 对象

location 是最有用的 BOM 对象之一，提供了当前窗口中加载文档的信息，以及通常的导航功能。 这个对象独特的地方在于，它既是 window 的属性，也是 document 的属性。也就是说， **window.location 和 document.location 指向同一个对象。location 对象不仅保存着当前加载文档的信息，也保存着把 URL 解析为离散片段后能够通过属性访问的信息**。这些解析后的属性在下表中有详细说明（location 前缀是必需的）

假设浏览器当前加载的 URL 是 `http://foouser:barpassword@www.wrox.com:80/WileyCDA/?q=  javascript#contents`，location 对象的内容如下表所示。

| 属 性             | 值                | 说 明                                          |
| ----------------- | ----------------- | ---------------------------------------------- |
| location.host     | "www.wrox.com:80" | 服务器名及端口号                               |
| location.hostname | "www.wrox.com"    | 服务器名                                       |
| location.href     | ...               | 当前加载页面的完整 URL。                       |
| location.pathname | "/WileyCDA/"      | URL 中的路径和（或）文件名                     |
| location.port     | "80"              | 请求的端口。如果 URL中没有端口，则返回空字符串 |
| location.protocol | "http:"           | 页面使用的协议。通常是"http:"或"https:"        |
| location.search   | "?q=javascript"   | URL 的查询字符串。这个字符串以问号开头         |

#### 操作地址

可以通过修改 location 对象修改浏览器的地址。

`location.href = "http://www.wrox.com";`

这行代码会立即启动导航到新 URL 的操作，同时在浏览器历史记录中增加一条记录。

在以前面提到的方式修改 URL 之后，浏览器历史记录中就会增加相应的记录。当用户单击“后退” 按钮时，就会导航到前一个页面。如果不希望增加历史记录，可以使用 replace()方法。这个方法接 收一个 URL 参数，但重新加载后不会增加历史记录。调用 replace()之后，用户不能回到前一页。

`location.replace("http://www.wrox.com/")`

最后一个修改地址的方法是 **reload()，它能重新加载当前显示的页面**。调用 reload()而不传 数，页面会以最有效的方式重新加载。也就是说，如果页面自上次请求以来没有修改过，浏览器可能会 从缓存中加载页面。如果想强制从服务器重新加载，可以像下面这样给 reload()传个 true：

location.reload(); // 重新加载，可能是从缓存加载 

location.reload(true); // 重新加载，从服务器加载 

脚本中位于 reload()调用之后的代码可能执行也可能不执行，这取决于网络延迟和系统资源等因 素。为此，最好把 reload()作为最后一行代码

### screen

window 的另一个属性 screen 对象，是为数不多的几个在编程中很少用的 JavaScript 对象。这个对 象中保存的纯粹是客户端能力信息，也就是浏览器窗口外面的客户端显示器的信息

### history 对象

history 对象表示当前窗口首次使用以来用户的导航历史记录。因为 history 是 window 的属性， 所以每个 window 都有自己的 history 对象。出于安全考虑，这个对象不会暴露用户访问过的 URL， 但可以通过它在不知道实际 URL 的情况下前进和后退

#### 导航

go()方法可以在用户历史记录中沿任何方向导航，可以前进也可以后退。这个方法只接收一个参数， 这个参数可以是一个整数，表示前进或后退多少步。负值表示在历史记录中后退（类似点击浏览器的“后 退”按钮），而正值表示在历史记录中前进（类似点击浏览器的“前进”按钮）。go()有两个简写方法：back()和 forward()。

`history.go(-1);`

`history.back();`

history 对象还有一个 length 属性，表示历史记录中有多个条目。这个属性反映了历史记录的数 量，包括可以前进和后退的页面。对于窗口或标签页中加载的第一个页面，history.length 等于 1。

#### 历史状态管理

hashchange 会在页面 URL 的散列变化时被触发，开发者可以在此时执行某些操作。而状态管理 API 则可以让开发者改变浏览器 URL 而不会加载新页面。

为此，可以使用 **history.pushState()**方 法。这个方法接收 3 个参数：一个 state 对象、一个新状态的标题和一个（可选的）相对 URL。例如： 

```js
let stateObject = {foo:"bar"};  
history.pushState(stateObject, "My title", "baz.html"); 
```

pushState()方法执行后，状态信息就会被推到历史记录中，浏览器地址栏也会改变以反映新的相对 URL。除了这些变化之外，即使 location.href 返回的是地址栏中的内容，浏览器也不会向服务器发送请求。第二个参数并未被当前实现所使用，因此既可以传一个空字符串也可以传一个短标题。第一个参数应该包含正确初始化页面状态所必需的信息。为防止滥用，这个状态的对象大小是有限制的，通 常在 500KB～1MB 以内

因为 pushState()会创建新的历史记录，所以也会相应地启用“后退”按钮。此时单击“后退” 按钮，就会触发 window 对象上的 popstate 事件。popstate 事件的事件对象有一个 state 属性，其 中包含通过 pushState()第一个参数传入的 state 对象：

```js
window.addEventListener("popstate", (event) => { 
 let state = event.state; 
 if (state) { // 第一个页面加载时状态是 null 
 processState(state); 
 } 
});
```

基于这个状态，应该把页面重置为状态对象所表示的状态（因为浏览器不会自动为你做这些，**在这里做自定义的状态保存操作**）

可以通过 history.state 获取当前的状态对象，也可以使用 replaceState()并传入与 pushState()同样的前两个参数来更新状态。更新状态不会创建新历史记录，只会覆盖当前状态

传给 pushState()和 replaceState()的 state 对象应该只包含可以被序列化的信息。因此， DOM 元素之类并不适合放到状态对象里保存。

**总而言之，就是用pushState()替换当前状态，同时存到历史记录，如果后续新开了一个页面，然后点击后退时，就可以取到之前存的数据，通过history.state**

使用 HTML5 状态管理时，要确保通过 pushState()创建的每个“假”URL 背后 都对应着服务器上一个真实的物理 URL。否则，单击“**刷新**”按钮会导致 404 错误。