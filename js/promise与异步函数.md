## 异步编程

异步行为是为了优化因计算量大而 时间长的操作。如果在等待其他操作完成的同时，即使运行其他指令，系统也能保持稳定，那么这样做就是务实的。 重要的是，异步操作并不一定计算量大或要等很长时间。只要你不想为等待某个异步操作而阻塞线 程执行，那么任何时候都可以使用。

### 同步与异步

**同步行为**对应内存中顺序执行的处理器指令。每条指令都会严格按照它们出现的顺序来执行，而每条指令执行后也能立即获得存储在系统本地（如寄存器或系统内存）的信息。

这样的执行流程**容易分析程序在执行到代码任意位置时的状态（比如变量的值）**

同步操作的例子可以是执行一次简单的数学计算：`let x = 3;  x = x + 4;`

**在程序执行的每一步，都可以推断出程序的状态。这是因为后面的指令总是在前面的指令完成后才会执行**。

等到最后一条指定执行完毕，存储在 x 的值就立即可以使用。

相对地，**异步行为类似于系统中断**，即当前进程外部的实体可以触发代码执行。异步操作经常是必 要的，因为强制进程等待一个长时间的操作通常是不可行的（同步操作则必须要等）。如果代码要访问 一些高延迟的资源，比如向远程服务器发送请求并等待响应，那么就会出现长时间的等待

异步操作的例子可以是在定时回调中执行一次简单的数学计算

```
let x = 3; 
setTimeout(() => x = x + 4, 1000);
```

这段程序最终与同步代码执行的任务一样，都是把两个数加在一起，但这一次**执行线程不知道 x 值 何时会改变，因为这取决于回调何时从消息队列出列并执行**

异步代码不容易推断。这个例子第二个指令块（加操作及赋值操作）是由系统计时器触发的，这会生成一个入队执行的中断。到底什么时候会触发这个中断，这对 JavaScript 运行时来说是一个黑盒，因此实际上无法预知

**为了让后续代码能够使用 x，异步执行的函数需要在更新 x 的值以后通知其他代码。**

### 以往的异步编程模式

在早期的 JavaScript 中，只支持定义回调函数来表明异步操作完成。串联多个异步操作是一个常见的问题，通常需要深度嵌套的回调函数（俗称“回 调地狱”）来解决。

#### 异步返回值

假设 setTimeout 操作会返回一个有用的值。有什么好办法把这个值传给需要它的地方？广泛接受 的一个策略是给异步操作提供一个回调，这个回调中包含要使用异步返回值的代码（作为回调的参数）

```
function double(value, callback) { 
 setTimeout(() => callback(value * 2), 1000); 
} 
double(3, (x) => console.log(`I was given: ${x}`)); 
// I was given: 6（大约 1000 毫秒之后)
```

这里的 setTimeout 调用告诉 JavaScript 运行时在 1000 毫秒之后把一个函数推到消息队列上。这 个函数会由运行时负责异步调度执行。而位于函数闭包中的回调及其参数在异步执行时仍然是可用的

简单来说，这里的例子就是初始参数3，异步操作返回6，要调用这个6，就把回调函数传给异步操作，由异步操作来调用这个回调函数，这样就肯定能取到6

#### 失败处理

异步操作的失败处理在回调模型中也要考虑，因此自然就出现了成功回调和失败回调

```
function double(value, success, failure) { 
 setTimeout(() => { 
 try { 
 if (typeof value !== 'number') { 
 throw 'Must provide number as first argument'; 
 } 
 success(2 * value); 
 } catch (e) { 
 failure(e); 
 } 
 }, 1000); 
}
```

这种模式已经不可取了，因为必须在初始化异步操作时定义回调。异步函数的返回值只在短时间内存在，只有预备好将这个短时间内存在的值作为参数的回调才能接收到它

#### 嵌套异步回调

如果异步返值又依赖另一个异步返回值，那么回调的情况还会进一步变复杂。在实际的代码中，这 就要求嵌套回调

```
function double(value, success, failure) { 
 setTimeout(() => { 
 try { 
 if (typeof value !== 'number') { 
 throw 'Must provide number as first argument'; 
 } 
 success(2 * value); 
 } catch (e) { 
 failure(e); 
 } 
 }, 1000); 
 } 
const successCallback = (x) => { 
 double(x, (y) => console.log(`Success: ${y}`)); 
}; 
const failureCallback = (e) => console.log(`Failure: ${e}`); 
double(3, successCallback, failureCallback); 
// Success: 12（大约 1000 毫秒之后）
```

显然，随着代码越来越复杂，回调策略是不具有扩展性的。“回调地狱”这个称呼可谓名至实归。 嵌套回调的代码维护起来就是噩梦

## promise

promise是对尚不存在的结果的一个替身

### 期约基础

ECMAScript 6 新增的引用类型 Promise，可以通过 new 操作符来实例化。创建新期约时需要传入 执行器（executor）函数作为参数，下面的例子使用了一个空函数对象来应付一下解释器

```
let p = new Promise(() => {}); 
setTimeout(console.log, 0, p); // Promise <pending> 
```

#### 期约状态机

在把一个期约实例传给 console.log()时，控制台输出（可能因浏览器不同而略有差异）表明该 实例处于待定（pending）状态。如前所述，**期约是一个有状态的对象，可能处于如下 3 种状态之一**：

- 待定（pending）
- 兑现（fulfilled，有时候也称为“解决”，resolved） 
- 拒绝（rejected）

**待定（pending）是期约的最初始状态**。**在待定状态下，期约可以转换为代表成功的兑现 （fulfilled）状态，或者代表失败的拒绝（rejected）状态**。无论转换为哪种状态都是**不可逆的**。只要从待定转换为兑现或拒绝，期约的状态就不再改变。

而且，也**不能保证期约必然会脱离待定状态**。

组织合理的代码无论期约解决（resolve）还是拒绝（reject），甚至永远处于待定（pending）状态，都应该 具有恰当的行为

重要的是，**期约的状态是私有的，不能直接通过 JavaScript 检测到**。这主要是为了避免根据读取到的期约状态，以同步方式处理期约对象。另外，**期约的状态也不能被外部 JavaScript 代码修改**。这与不能读取该状态的原因是一样的：期约故意将异步行为封装起来，从而隔离外部的同步代码

#### 返回值或拒绝理由

期约主要有两大用途。

1、**抽象地表示一个异步操作**。期约的状态代表期约是否完成。“待定” 表示尚未开始或者正在执行中。“兑现”表示已经成功完成，而“拒绝”则表示没有成功完成。

某些情况下，这个状态机就是期约可以提供的最有用的信息。知道一段异步代码已经完成，对于其他代码而言已经足够了。

比如，假设期约要向服务器发送一个 HTTP 请求。请求返回 200~299 范围内的 状态码就足以让期约的状态变为“兑现”。类似地，如果请求返回的状态码不在 200~299 这个范围内， 那么就会把期约状态切换为“拒绝”。

2、期约封装的异步操作会**实际生成某个值**，而程序期待期约状态改变时可以访问 这个值。相应地，如果期约被拒绝，程序就会期待期约状态改变时可以拿到拒绝的理由。

为了支持这两种用例，每个期约只要状态切换为兑现，就会有一个私有的内部值（value）。类似地， 每个期约只要状态切换为拒绝，就会有一个私有的内部理由（reason）。无论是值还是理由，都是包含原 始值或对象的不可修改的引用。二者都是可选的，而且默认值为 undefined。在期约到达某个落定状 态时执行的异步代码始终会收到这个值或理由。

#### 通过执行函数控制期约状态

由于期约的状态是私有的，所以只能在内部进行操作。内部操作在期约的执行器函数中完成。

执行器函数主要有两项职责：**初始化期约的异步行为**和**控制状态的最终转换**。

其中，**控制期约状态的转换**是通过调用它的两个函数参数实现的。这两个函数参数通常都命名为 **resolve()**和 **reject()**。

调用 resolve()会把状态切换为兑现，调用 reject()会把状态切换为拒绝。另外，调用 reject()也会抛出错误

```js
let p1 = new Promise((resolve, reject) => resolve()); 
setTimeout(console.log, 0, p1); // Promise <resolved> 
let p2 = new Promise((resolve, reject) => reject()); 
setTimeout(console.log, 0, p2); // Promise <rejected>
// Uncaught error (in promise)
```

无论 resolve()和 reject()中的哪个被调用，状态转换都不可撤销了。于是继续修改状态会静默失败(即没有效果)

**为避免期约卡在待定状态，可以添加一个定时退出功能**。比如，可以通过 setTimeout 设置一个 10 秒钟后无论如何都会拒绝期约的回调

因为期约的状态只能改变一次，所以这里的超时拒绝逻辑中可以放心地设置让期约处于待定状态的最长时间。

**执行器函数是立即执行的**

```js
const b = new Promise((resolve) => { console.log("run!"); resolve(); });
b.then(()=>{ console.log("resolve") })
console.log("haha")
// run!
// haha
// resolve
```

#### Promise.resolve()

期约并非一开始就必须处于待定状态，然后通过执行器函数才能转换为落定状态。**通过调用 Promise.resolve()静态方法，可以实例化一个解决的期约**

下面两个期约实例实际上是一样的

```js
let p1 = new Promise((resolve, reject) => resolve()); 
let p2 = Promise.resolve(); 
```

**这个解决的期约的值对应着传给 Promise.resolve()的第一个参数**。使用这个静态方法，实际上可以把任何值都转换为一个期约

```js
setTimeout(console.log, 0, Promise.resolve()); 
// Promise <resolved>: undefined 
setTimeout(console.log, 0, Promise.resolve(3)); 
// Promise <resolved>: 3 
// 多余的参数会忽略
setTimeout(console.log, 0, Promise.resolve(4, 5, 6)); 
// Promise <resolved>: 4
```

对这个静态方法而言，如果传入的参数本身是一个期约，那它的行为就类似于一个空包装。

```js
let p = Promise.resolve(7); 
setTimeout(console.log, 0, p === Promise.resolve(p)); 
// true 
setTimeout(console.log, 0, p === Promise.resolve(Promise.resolve(p))); 
// true 
```

这个包装会保留传入期约的状态：

```
let p = new Promise(() => {}); 
setTimeout(console.log, 0, p); // Promise <pending> 
setTimeout(console.log, 0, Promise.resolve(p)); // Promise <pending> 
setTimeout(console.log, 0, p === Promise.resolve(p)); // true
```

这个静态方法能够包装任何非期约值，包括错误对象，并将其转换为解决的期约。

#### Promise.reject()

Promise.reject()会实例化一个拒绝的期约并抛出一个异步错误 （**这个错误不能通过 try/catch 捕获，而只能通过拒绝处理程序捕获**）

拒绝的期约的理由就是传给 Promise.reject()的第一个参数。这个参数也会传给后续的拒绝处理程序

```
let p = Promise.reject(3); 
setTimeout(console.log, 0, p); // Promise <rejected>: 3 
p.then(null, (e) => setTimeout(console.log, 0, e)); // 3 
```

**Promise.reject()并没有照搬 Promise.resolve()的幂等逻辑**。如果给它传一个期约对象，则这个期约会成为它返回的拒绝期约的理由

#### 同步/异步执行的二元性

期约真正的异步特性：它们是同步对象（在同步执行模式中使用），但**也是异步执行模式的媒介**

代码一旦开始以异步模式执行，则唯一与之交互 的方式就是使用异步结构——更具体地说，就是期约的方法

### 期约的实例方法

期约实例的方法是连接外部同步代码与内部异步代码之间的桥梁。**这些方法可以访问异步操作返回的数据，处理期约成功和失败的结果，连续对期约求值，或者添加只有期约进入终止状态时才会执行的代码**

#### Promise.prototype.then()

Promise.prototype.then()是为期约实例添加处理程序的主要方法。**这个 then()方法接收最多两个参数：onResolved 处理程序和 onRejected 处理程序。**这两个参数都是**可选**的，如果提供的话， 则会在期约**分别进入“兑现”和“拒绝”状态时执行**。

```js
function onResolved(id) { 
 setTimeout(console.log, 0, id, 'resolved');
 } 
function onRejected(id) { 
 setTimeout(console.log, 0, id, 'rejected'); 
} 
let p1 = new Promise((resolve, reject) => setTimeout(resolve, 3000)); 
let p2 = new Promise((resolve, reject) => setTimeout(reject, 3000)); 
p1.then(() => onResolved('p1'), () => onRejected('p1')); 
p2.then(() => onResolved('p2'), () => onRejected('p2')); 
//（3 秒后）
// p1 resolved 
// p2 rejected
```

因为期约只能转换为最终状态一次，所以**这两个操作一定是互斥的**

then()在状态改变时执行，但其返回值在定义时就已经返回了，这个返回值是一个新的promise实例，状态如何下方有说。**总之，then肯定是在状态改变时才执行，也只有在状态改变时，其返回值的状态才会发生相应的改变。不可能出现then的父promise是pending，而then的返回值是落定状态的情况**。如下代码：

```js
let p1 = new Promise(() => {}); 
let p2 = p1.then(() => Promise.resolve(3));
console.log(p2) // Promise {<pending>}

let p9 = new Promise((resolve) => { setTimeout(()=>{resolve()},  10000) }); 
let p10 = p9.then(() => Promise.resolve(3));
console.log(p10) // 十秒内pending，十秒后fulfilled:3
```

传给 then()的任何非函数类型的参数都会被静默忽略。如果想只提供 onRejected 参数，那就要在 onResolved 参数的位置上传入 undefined。

**Promise.prototype.then()方法返回一个新的期约实例**

```js
let p1 = new Promise(() => {}); 
let p2 = p1.then(); 
setTimeout(console.log, 0, p1); // Promise <pending> 
setTimeout(console.log, 0, p2); // Promise <pending> 
setTimeout(console.log, 0, p1 === p2); // false 
```

这个新期约实例**基于 onResovled 处理程序的返回值构建**。换句话说，**onResovled处理程序的返回值会通过 Promise.resolve()包装来生成新期约**。如果没有显式的返回语句，则 Promise.resolve()会包装默认的返回值 undefined

如果没有提供onResovled处理程序，则 Promise.resolve()就会**包装上一个期约解决之后的值**。

（这里，新实例是通过Promise.resolve()包装的，**而上方代码块中状态为pending的原因是包装会保留传入期约的状态**）

```js
let p1 = Promise.resolve('foo'); 
// 属于“如果没有提供onResovled处理程序，则 Promise.resolve()就会包装上一个期约解决之后的值”
let p2 = p1.then(); 
setTimeout(console.log, 0, p2); // Promise <resolved>: foo 
// 这些都一样，属于“如果没有显式的返回语句，则 Promise.resolve()会包装默认的返回值 undefined”
let p3 = p1.then(() => undefined); 
let p4 = p1.then(() => {}); 
let p5 = p1.then(() => Promise.resolve()); 
setTimeout(console.log, 0, p3); // Promise <resolved>: undefined 
setTimeout(console.log, 0, p4); // Promise <resolved>: undefined 
setTimeout(console.log, 0, p5); // Promise <resolved>: undefined 
```

如果有显式的返回值，则 Promise.resolve()会包装这个值：

```js
// 这些都一样
let p6 = p1.then(() => 'bar'); 
let p7 = p1.then(() => Promise.resolve('bar')); 
setTimeout(console.log, 0, p6); // Promise <resolved>: bar 
setTimeout(console.log, 0, p7); // Promise <resolved>: bar 
// Promise.resolve()保留返回的期约
let p8 = p1.then(() => new Promise(() => {})); 
let p9 = p1.then(() => Promise.reject()); 
// Uncaught (in promise): undefined 
setTimeout(console.log, 0, p8); // Promise <pending> 
setTimeout(console.log, 0, p9); // Promise <rejected>: undefined 
```

抛出异常会返回拒绝的期约：

```js
let p10 = p1.then(() => { throw 'baz'; }); 
// Uncaught (in promise) baz 
setTimeout(console.log, 0, p10); // Promise <rejected> baz 
```

onRejected 处理程序也与之类似：onRejected 处理程序返回的值也会被 Promise.resolve() 包装。乍一看这可能有点违反直觉，但是想一想，onRejected 处理程序的任务不就是捕获异步错误吗？ 因此，拒绝处理程序在捕获错误后不抛出异常是符合期约的行为，应该返回一个解决期约

```
let p1 = Promise.reject('foo'); 
// 调用 then()时不传处理程序则原样向后传
let p2 = p1.then(); 
// Uncaught (in promise) foo 
setTimeout(console.log, 0, p2); // Promise <rejected>: foo 
// 这些都一样
let p3 = p1.then(null, () => undefined); 
let p4 = p1.then(null, () => {}); 
let p5 = p1.then(null, () => Promise.resolve()); 
setTimeout(console.log, 0, p3); // Promise <resolved>: undefined 
setTimeout(console.log, 0, p4); // Promise <resolved>: undefined 
setTimeout(console.log, 0, p5); // Promise <resolved>: undefined 
...
```

#### Promise.prototype.catch()

Promise.prototype.catch()方法用于给期约添加拒绝处理程序。这个方法只接收一个参数： onRejected 处理程序。事实上，**这个方法就是一个语法糖，调用它就相当于调用 Promise.prototype.  then(null, onRejected)**

```js
let p = Promise.reject(); 
let onRejected = function(e) { 
 setTimeout(console.log, 0, 'rejected'); 
}; 
// 这两种添加拒绝处理程序的方式是一样的：
p.then(null, onRejected); // rejected 
p.catch(onRejected); // rejected 
```

Promise.prototype.catch()返回一个新的期约实例

在返回新期约实例方面，Promise.prototype.catch()的行为与 Promise.prototype.then() 的 onRejected 处理程序是一样的

#### Promise.prototype.finally()

Promise.prototype.finally()方法用于给期约添加 onFinally 处理程序，这个处理程序**在期约转换为解决或拒绝状态时都会执行**。这个方法**可以避免 onResolved 和 onRejected 处理程序中出现冗余代码**。但 **onFinally 处理程序没有办法知道期约的状态是解决还是拒绝**，所以这个方法主要用于添加清理代码。

```js
let p1 = Promise.resolve(); 
let p2 = Promise.reject(); 
let onFinally = function() { 
 setTimeout(console.log, 0, 'Finally!') 
} 
p1.finally(onFinally); // Finally 
p2.finally(onFinally); // Finally 
```

Promise.prototype.finally()方法返回一个新的期约实例：

```js
let p1 = new Promise(() => {}); 
let p2 = p1.finally();
setTimeout(console.log, 0, p1); // Promise <pending> 
setTimeout(console.log, 0, p2); // Promise <pending> 
setTimeout(console.log, 0, p1 === p2); // false
```

这个新期约实例不同于 then()或 catch()方式返回的实例。因为 onFinally 被设计为一个状态 无关的方法，所以在大多数情况下它将表现为父期约的传递。对于已解决状态和被拒绝状态都是如此

#### 非重入期约方法

当期约进入落定状态(resolve或reject)时，与该状态相关的处理程序仅仅会被排期，**而非立即执行**。**跟在添加这个处理程序的代码之后的同步代码一定会在处理程序之前先执行**。即使期约一开始就是与附加处理程序关联的状态(resolve或reject)，执行顺序也是这样的。这个特性由 JavaScript 运行时保证，被称为**“非重入”（non-reentrancy）** 特性。下面的例子演示了这个特性

```js
// 创建解决的期约
let p = Promise.resolve(); 
// 添加解决处理程序
// 直觉上，这个处理程序会等期约一解决就执行
p.then(() => console.log('onResolved handler')); 
// 同步输出，证明 then()已经返回
console.log('then() returns'); 
// 实际的输出：
// then() returns 
// onResolved handler
```

在这个例子中，在一个解决期约上调用 then()会**把 onResolved 处理程序推进消息队列**。但**这个处理程序在当前线程上的同步代码执行完成前不会执行**。因此，跟在 then()后面的同步代码一定先于 处理程序执行。

先添加处理程序后解决期约也是一样的。如果添加处理程序后，同步代码才改变期约状态，那么处理程序仍然会基于该状态变化表现出非重入特性

```js
let synchronousResolve; 
// 创建一个期约并将解决函数保存在一个局部变量中
let p = new Promise((resolve) => { 
 synchronousResolve = function() { 
 console.log('1: invoking resolve()'); 
 resolve(); 
 console.log('2: resolve() returns'); 
 }; 
}); 
p.then(() => console.log('4: then() handler executes')); 
synchronousResolve(); 
console.log('3: synchronousResolve() returns'); 
// 实际的输出：
// 1: invoking resolve() 
// 2: resolve() returns 
// 3: synchronousResolve() returns 
// 4: then() handler executes
```

执行顺序：promise的执行器函数=》synchronousResolve函数=》console.log(‘3’)=》then函数

非重入适用于 onResolved/onRejected 处理程序、catch()处理程序和 finally()处理程序

#### 邻近处理程序的执行顺序

如果给期约添加了多个处理程序，**当期约状态变化时，相关处理程序会按照添加它们的顺序依次执行**。无论是 then()、catch()还是 finally()添加的处理程序都是如此

#### 传递解决值和拒绝理由

**在执行函数中，解决的值和拒绝的理由是分别作为 resolve()和 reject()的第一个参数往后传的**。然后，**这些值又会传给它们各自的处理程序，作为 onResolved 或 onRejected 处理程序的唯一参数**。下面的例子展示了上述传递过程

```js
let p1 = new Promise((resolve, reject) => resolve('foo')); 
p1.then((value) => console.log(value)); // foo 
let p2 = new Promise((resolve, reject) => reject('bar')); 
p2.catch((reason) => console.log(reason)); // bar 
```

#### 拒绝期约与拒绝错误处理

拒绝期约类似于 throw()表达式，因为它们都代表一种程序状态，即需要中断或者特殊处理。**在期约的执行函数或处理程序中抛出错误会导致拒绝，对应的错误对象会成为拒绝的理由**。因此以下这些期约都会以一个错误对象为由被拒绝

```js
let p1 = new Promise((resolve, reject) => reject(Error('foo'))); 
let p2 = new Promise((resolve, reject) => { throw Error('foo'); }); 
let p3 = Promise.resolve().then(() => { throw Error('foo'); }); 
let p4 = Promise.reject(Error('foo')); 

setTimeout(console.log, 0, p1); // Promise <rejected>: Error: foo 
setTimeout(console.log, 0, p2); // Promise <rejected>: Error: foo 
setTimeout(console.log, 0, p3); // Promise <rejected>: Error: foo 
setTimeout(console.log, 0, p4); // Promise <rejected>: Error: foo
```

正常情况下，在通过 throw()关键字抛出错误时， JavaScript 运行时的错误处理机制会停止执行抛出错误之后的任何指令

```
throw Error('foo'); 
console.log('bar'); // 这一行不会执行
// Uncaught Error: foo
```

但是，**在期约中抛出错误时，因为错误实际上是从消息队列中异步抛出的，所以并不会阻止运行时继续执行同步指令**

```
Promise.reject(Error('foo')); 
console.log('bar'); 
// bar 
// Uncaught (in promise) Error: foo
```

### 期约连锁与期约合成

多个期约组合在一起可以构成强大的代码逻辑。这种组合可以通过两种方式实现：**期约连锁与期约合成。前者就是一个期约接一个期约地拼接，后者则是将多个期约组合为一个期约**

#### 期约连锁

把期约逐个地串联起来是一种非常有用的编程模式。之所以可以这样做，是**因为每个期约实例的方 法（then()、catch()和 finally()）都会返回一个新的期约对象，而这个新期约又有自己的实例方法**。这样连缀方法调用就可以构成所谓的“期约连锁”。

```
let p = new Promise((resolve, reject) => { 
 console.log('first'); 
 resolve(); 
}); 
p.then(() => console.log('second')) 
 .then(() => console.log('third')) 
 .then(() => console.log('fourth')); 
// first 
// second 
// third 
// fourth
```

这个实现最终执行了一连串同步任务，没什么用。

要真正执行异步任务，可以改写前面的例子，让每个执行器都返回一个期约实例。这样就可以让每个后续期约都等待之前的期约，也就是**串行化异步任务**。（串行类似于同步执行）

```
let p1 = new Promise((resolve, reject) => { 
 console.log('p1 executor'); 
 setTimeout(resolve, 1000); 
}); 
p1.then(() => new Promise((resolve, reject) => { 
 console.log('p2 executor'); 
 setTimeout(resolve, 1000); 
 })) 
 .then(() => new Promise((resolve, reject) => { 
 console.log('p3 executor'); 
 setTimeout(resolve, 1000); 
 })) 
 .then(() => new Promise((resolve, reject) => { 
 console.log('p4 executor'); 
 setTimeout(resolve, 1000); 
 })); 
// p1 executor（1 秒后）
// p2 executor（2 秒后）
// p3 executor（3 秒后）
// p4 executor（4 秒后）
```

**每个后续的处理程序都会等待前一个期约解决，然后实例化一个新期约并返回它**。**这种结构可以简洁地将异步任务串行化，解决之前依赖回调的难题**。

因为 then()、catch()和 finally()都返回期约，所以串联这些方法也很直观

```
let p = new Promise((resolve, reject) => { 
 console.log('initial promise rejects'); 
 reject(); 
}); 
p.catch(() => console.log('reject handler')) 
 .then(() => console.log('resolve handler')) 
 .finally(() => console.log('finally handler')); 
// initial promise rejects 
// reject handler 
// resolve handler 
// finally handler 
```

上方代码中，因为处理函数没有返回值，所以其他处理函数都能沿用到最开始的promise的返回值

#### Promise.all()和 Promise.race()

Promise 类提供两个将多个期约实例组合成一个期约的静态方法：Promise.all()和 Promise.race()。 而合成后期约的行为取决于内部期约的行为。