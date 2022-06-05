### var

#### var 声明作用域

使用 var 操作符定义的变量**会成为包含它的函数的局部变量**。比如，使用 var 在一个函数内部定义一个变量，就意味着该变量将在函数退出时被销毁

不过，**在函数内定义变量时省略 var 操作符，可以创建一个全局变量**

```js
function test() { 
 var message = "hi"; // 局部变量
 message2 = "hi"; // 全局变量
} 
test(); 
console.log(message); // 出错！
console.log(message2); // "hi" 
console.log(window.message2); // "hi" 
```

在严格模式下，如果像这样给未声明的变量赋值，则会导致抛出 ReferenceError

#### var 声明提升

使用var关键字声明的变量会自动提升到函数作用域顶部（**提升只是提升声明，并没有提升赋值**）

```js
function foo() { 
 console.log(age); 
 var age = 26; 
} 
foo(); // undefined

// 相当于-------------------------------------
function foo() { 
 var age; 
 console.log(age); 
 age = 26; 
} 
foo(); // undefined 
```

**反复多次使用 var 声明同一个变量是没有问题的**

```js
function foo() { 
 var age = 16; 
 var age = 26; 
 var age = 36; 
 console.log(age); 
} 
foo(); // 36 
```

### let 声明

let 跟 var 的作用差不多，但有几个区别。

1、**let声明不会提升**

2、**let 声明的范围是块作用域， 而 var 声明的范围是函数作用域**

```js
if (true) { 
 var name = 'Matt'; 
 console.log(name); // Matt 
} 
console.log(name); // Matt
if (true) { 
 let age = 26; 
 console.log(age); // 26 
} 
console.log(age); // ReferenceError: age 没有定义
```

块作用域是函数作用域的子集

**let 不允许同一个块作用域中出现冗余声明。这样会导致报错**

```js
var name; 
var name; 
let age; 
let age; // SyntaxError；标识符 age 已经声明过了
```

嵌套使用相同的标识符不会报错，而这是因为同一个块中没有重复声明

```js
let age = 30; 
console.log(age); // 30 
if (true) { 
 let age = 26; 
 console.log(age); // 26 
} 
```

**对声明冗余报错不会因混用 let 和 var 而受影响**。这两个关键字声明的并不是不同类型的变量， **它们只是指出变量在相关作用域如何存在**

```js
var name; 
let name; // SyntaxError 
let age; 
var age; // SyntaxError
```

#### 暂时性死区

let 声明的变量不会在作用域中被提升

```js
// age 不会被提升
console.log(age); // ReferenceError：age 没有定义
let age = 26; 
```

在解析代码时，JavaScript 引擎也会注意出现在块后面的 let 声明，只不过在此之前不能以任何方式来引用未声明的变量。**在 let 声明之前的执行瞬间被称为“暂时性死区”（temporal dead zone）**，在此阶段引用任何后面才声明的变量都会抛出 ReferenceError。

#### 全局声明

与 var 关键字不同，使用 let **在全局作用域**中声明的变量不会成为 window 对象的属性（var 声 明的变量则会）

不过，let 声明仍然是在全局作用域中发生的，相应变量会在页面的生命周期内存续。

```js
var name = 'Matt'; 
console.log(window.name); // 'Matt' 
let age = 26; 
console.log(window.age); // undefined
console.log(age); // 26
```

#### for 循环中的 let 声明

在 let 出现之前，for 循环定义的迭代变量会渗透到循环体外部

改成使用 let 之后，这个问题就消失了，因为迭代变量的作用域仅限于 for 循环块内部

```js
for (var i = 0; i < 5; ++i) { 
 // 循环逻辑 
} 
console.log(i); // 5
```

```js
for (let i = 0; i < 5; ++i) { 
 // 循环逻辑
} 
console.log(i); // ReferenceError: i 没有定义
```

```js
for (var i = 0; i < 5; ++i) { 
 setTimeout(() => console.log(i), 0) 
} 
// 你可能以为会输出 0、1、2、3、4 
// 实际上会输出 5、5、5、5、5
```

之所以会这样，是因为在退出循环时，迭代变量保存的是导致循环退出的值：5。**在之后执行超时逻辑时，所有的 i 都是同一个变量**，因而输出的都是同一个最终值

而在使用 let 声明迭代变量时，**JavaScript 引擎在后台会为每个迭代循环声明一个新的迭代变量**。 **每个 setTimeout 引用的都是不同的变量实例**，所以 console.log 输出的是我们期望的值，也就是循环执行过程中每个迭代变量的值

这种每次迭代声明一个独立变量实例的行为适用于所有风格的 for 循环，包括 for-in 和 for-of 循环。

```js
for (let i = 0; i < 5; ++i) { 
 setTimeout(() => console.log(i), 0) 
} 
// 会输出 0、1、2、3、4
```

### const 声明

const 的行为与 let 基本相同，唯一一个重要的区别是用它声明变量时必须同时初始化变量，且 尝试修改 const 声明的变量会导致运行时错误。

```js
// const 也不允许重复声明
const name = 'Matt'; 
const name = 'Nicholas'; // SyntaxError 

// const 声明的作用域也是块
const name = 'Matt'; 
if (true) { 
 const name = 'Nicholas'; 
} 
console.log(name); // Matt 
```

**const 声明的限制只适用于它指向的变量的引用**。换句话说，如果 const 变量引用的是一个对象， 那么修改这个对象内部的属性并不违反 const 的限制。如果想让整个对象都不能修改，可以使用 **Object.freeze()**，这样再给属性赋值时虽然不会报错， 但会静默失败（无效化）

JavaScript 引擎会为 for 循环中的 let 声明分别创建独立的变量实例，虽然 const 变量跟 let 变 量很相似，但是**不能用 const 来声明迭代变量（因为迭代变量会自增）**（不太懂，当使用let和const时，for的实现不一样？）

由于 const 声明暗示变量的值是单一类型且不可修改，**JavaScript 运行时编译器可以将其所有实例 都替换成实际的值，而不会通过查询表进行变量查找**。谷歌的 V8 引擎就执行这种优化