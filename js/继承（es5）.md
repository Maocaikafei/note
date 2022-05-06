## 一、简介

面向对象语言通常支持两种继承：接口继承和实现继承。 前者只继承方法签名，后者继承实际的方法。

接口继承在 ECMAScript 中是不可能的，因为函数没有签名。

实现继承是 ECMAScript 唯一支持的继承方式，而这主要是通过原型链实现的。

## 二、原型链

### 基本思想

通过原型继承多个引用类型的属性和方法。

### 构造函数、原型和实例的关系

每个构造函数都有一个原型对象，原型有一个属性指回构造函数，而实例有一个内部指针指向原型。

### 如果原型是另一个类型的实例

这意味着这个原型本身有一个内部指针指向另一个原型，相应地另一个原型也有一个指针指向另一个构造函数。这样就在实例和原型之间构造了一条原型链。这就是原型链的基本构想。

```js
function SuperType() { 
 this.property = true; 
} 
SuperType.prototype.getSuperValue = function() { 
 return this.property; 
}; 
function SubType() { 
 this.subproperty = false; 
} 
// 继承 SuperType 
SubType.prototype = new SuperType(); 
SubType.prototype.getSubValue = function () { // 这个新方法可以理解为添加在SuperType的实例上
 return this.subproperty; 
}; 
let instance = new SubType(); 
console.log(instance.getSuperValue()); // true 
```

**SubType 通过创建 SuperType 的实例并将其赋值给自己的原型 SubTtype.prototype 实现了对 SuperType 的继承**。这个赋值**重写了 SubType 最初的原型，将其替换为 SuperType 的实例**。这意味着 SuperType 实例可以访问的所有属性和方法也会存在于 SubType.prototype。

这个例子实现继承的关键，是 SubType 没有使用默认原型，而是将其替换成了一个新的对象。这个新的对象恰好是 SuperType 的实例。这样一来，SubType 的实例不仅能从 SuperType 的实例中继承属性和方法，而且**还与 SuperType 的原型挂上了钩**。

最后的instance（通过内部的[[Prototype]]）指向 SubType.prototype，而 SubType.prototype（作为 SuperType 的实例又通过内部的[[Prototype]]） 指向 SuperType.prototype。

### 原型搜索机制

在读取实例上的属性时，首先会在实例上搜索 这个属性。如果没找到，则会继续搜索实例的原型。在通过原型链实现继承之后，搜索就可以继续向上， **搜索原型的原型**。对前面的例子而言，调用 instance.getSuperValue()经过了 3 步搜索：instance、 SubType.prototype 和 SuperType.prototype，最后一步才找到这个方法。

### 默认原型

默认情况下，所有引用类型都继承自 Object，这也是通过原型链实现的。任何函数的默认原型都是一个 Object 的实例，这意味着这个实例有一个内部指针指向 Object.prototype。这也是为什么自定义类型能够继承包括 toString()、valueOf()在内的所有默 认方法的原因。

在上述例子中，SubType 继承 SuperType，而 SuperType 继承 Object。

### 原型与实例的关系

原型与实例的关系可以通过两种方式来确定

1. instanceof 操作符，如果一个实 例的原型链中出现过相应的构造函数，则 instanceof 返回 true

   ```js
   console.log(instance instanceof Object); // true 
   console.log(instance instanceof SuperType); // true 
   console.log(instance instanceof SubType); // true
   ```

2. isPrototypeOf()方法。原型链中的每个原型都可以调用这个 方法，如下例所示，只要原型链中包含这个原型，这个方法就返回 true

   ```js
   console.log(Object.prototype.isPrototypeOf(instance)); // true 
   console.log(SuperType.prototype.isPrototypeOf(instance)); // true 
   console.log(SubType.prototype.isPrototypeOf(instance)); // true
   ```

### 原型链的问题

1、主要问题就是原型中包含的引用值，会在所有实例间共享。这也是为什么属性通常会在构造函数中定义而不会定义在原型上的原因。

但是，就算在构造函数中定义熟悉，当使用原型实现继承时，原型实际上**成了另一个类型的实例**。这意味着原先的**实例属性摇身一变成为了原型属性**

（所有实例共享一个原型，因为引用值可以修改，基本值无法修改，只能覆盖，因此只有引用值会有这种问题）

```js
function SuperType() { 
 this.colors = ["red", "blue", "green"]; 
} 
function SubType() {} 
// 继承 SuperType 
SubType.prototype = new SuperType(); 
let instance1 = new SubType(); 
instance1.colors.push("black"); 
console.log(instance1.colors); // "red,blue,green,black" 
let instance2 = new SubType(); 
console.log(instance2.colors); // "red,blue,green,black"

```

2、原型链的第二个问题是，子类型在实例化时不能给父类型的构造函数传参。（会影响到其他对象实例）事实上，我们无法在**不影响所有对象实例**的情  况下把参数传进父类的构造函数。（因为父类构造函数是以原型的方式与子类相联系的，要传参数到父类构造函数，就会修改到子类的原型对象，就会影响到所有子类的实例）再加上之前提到的原型中包含引用值的问题， 就导致原型链基本不会被单独使用

## 二、盗用构造函数

盗用构造函数（constructor stealing，也叫“对象伪装”或“经典继承”）可解决原型包含引用值导致的继承问题

### 基本思路

在子类构造函数中调用父类构造函数。因为毕竟函数就是在特定上下文中执行代码的简单对象，所以可以使用 `apply()`和 `call()`方法**以新创建的对象为上下文**执行构造函数。

```js
function SuperType() { 
 this.colors = ["red", "blue", "green"]; 
} 
function SubType() { 
 // 继承 SuperType 
 SuperType.call(this); 
} 
let instance1 = new SubType(); 
instance1.colors.push("black"); 
console.log(instance1.colors); // "red,blue,green,black" 
let instance2 = new SubType(); 
console.log(instance2.colors); // "red,blue,green" 
```

通过使用 call()（或 apply()）方法，SuperType 构造函数在为 SubType 的实例创建的新对象的上下文中执行了。这**相当于新的 SubType 对象上运行了 SuperType()函数中的所有初始化代码**。结果就是每个实例都会有自己的 colors 属性。

### 优点：传递参数

相比于使用原型链，盗用构造函数的一个优点就是可以在子类构造函数中向父类构造函数传参

```js
function SuperType(name){ 
 this.name = name; 
} 
function SubType() { 
 // 继承 SuperType 并传参
 SuperType.call(this, "Nicholas"); 
 // 实例属性
 this.age = 29; 
} 
let instance = new SubType(); 
console.log(instance.name); // "Nicholas"; 
console.log(instance.age); // 29 
```

### 缺点

盗用构造函数的主要缺点，也是使用构造函数模式自定义类型（这里说成`自定义构造函数模式`更容易理解点）的问题：必须在构造函数中定义方法，因此函数不能重用。

此外，**子类也不能访问父类原型上定义的方法**，因此所有类型只能使用构造函数模式。

上面涉及到自定义构造函数模式的问题：构造函数定义的方法会在每个实例上都创建一遍。ECMAScript 中的函数是对象，因此每次定义函数时，都会初始化一个对象。

```js
function Person(name, age, job){ 
 this.name = name; 
 this.age = age; 
 this.job = job; 
 this.sayName = function() { 
 	console.log(this.name); 
 }; 
 // this.sayName = new Function("console.log(this.name)"); // 逻辑等价
} 
```

因为都是做一样的事，所以没必要定义两个不同的 Function 实例。（拥有不同的作用域链和标识符解析）况且，this 对象可以把函数 与对象的绑定推迟到运行时。要解决这个问题，可以把函数定义转移到构造函数外部。

## 三、组合继承

组合继承（Combination Inheritance，有时候也叫伪经典继承）综合了原型链和盗用构造函数，将两者的优点集中了起来。是 JavaScript 中最常用的继承模式。

### 基本思路

使用原型链继承原型上的属性和方法（个人理解主要是方法），而通过盗用构造函数继承实例属性。这样既可以**把方法定义在原型上以实现重用**，又可以**让每个实例都有自己的属性**。

```js
function SuperType(name){ 
 this.name = name; 
 this.colors = ["red", "blue", "green"]; 
} 
SuperType.prototype.sayName = function() { 
 console.log(this.name); 
}; 
function SubType(name, age){ 
 // 继承属性
 SuperType.call(this, name); 
 this.age = age; 
} 
// 继承方法
SubType.prototype = new SuperType(); 
SubType.prototype.sayAge = function() { 
 console.log(this.age); 
}; 
let instance1 = new SubType("Nicholas", 29); 
instance1.colors.push("black"); 
console.log(instance1.colors); // "red,blue,green,black" 
instance1.sayName(); // "Nicholas"; 
instance1.sayAge(); // 29 
let instance2 = new SubType("Greg", 27); 
console.log(instance2.colors); // "red,blue,green" 
instance2.sayName(); // "Greg"; 
instance2.sayAge(); // 27 
```

## 四、原型式继承

Douglas Crockford提出一种方法，即使不自定义类型也可以通过原型实现对象之间的信息共享

```
function object(o) { 
 function F() {} 
 F.prototype = o; 
 return new F(); 
}
```

这个 object()函数会创建一个临时构造函数，将传入的对象赋值给这个构造函数的原型，然后返 回这个临时类型的一个实例。（返回的对象和这个临时函数没有直接关联，所以临时函数在返回后就销毁了）

本质上，object()是对传入的对象执行了一次浅复制。

```js
let person = { 
 name: "Nicholas", 
 friends: ["Shelby", "Court", "Van"] 
}; 
let anotherPerson = object(person); 
anotherPerson.name = "Greg"; 
anotherPerson.friends.push("Rob"); 
let yetAnotherPerson = object(person); 
yetAnotherPerson.name = "Linda"; 
yetAnotherPerson.friends.push("Barbie"); 
console.log(person.friends); // "Shelby,Court,Van,Rob,Barbie"
```

Crockford 推荐的原型式继承**适用于这种情况：你有一个对象，想在它的基础上再创建一个新对象**。 你需要把这个对象先传给 object()，然后再对返回的对象进行适当修改。

在这个例子中，把person传给 object()之后会返回一个新对象。这个新对象的原型是 person，意味着它的原型上既有原始值属性又有引用值属性。这意味着 person.friends 不仅是 person 的属性，也会跟 anotherPerson 和 yetAnotherPerson 共享(实际上就是原型继承中的引用值会在所有实例中共享的问题)

### Object.create()

ECMAScript 5 通过增加 Object.create()方法将原型式继承的概念规范化了。这个方法接收两个参数：作为新对象原型的对象，以及给新对象定义额外属性的对象（第二个可选）。在只有一个参数时， Object.create()与这里的 object()方法效果相同

Object.create()的第二个参数与 Object.defineProperties()的第二个参数一样：每个新增属性都通过各自的描述符来描述。**以这种方式添加的属性会遮蔽原型对象上的同名属性**。

```js
let person = { 
 name: "Nicholas", 
 friends: ["Shelby", "Court", "Van"] 
}; 
let anotherPerson = Object.create(person); 
anotherPerson.name = "Greg"; 
anotherPerson.friends.push("Rob"); 

let yetAnotherPerson = Object.create(person); 
yetAnotherPerson.name = "Linda"; 
yetAnotherPerson.friends.push("Barbie"); 
console.log(person.friends); // "Shelby,Court,Van,Rob,Barbie" 

let anotherPerson2 = Object.create(person, { 
 friends: { 
     value: []
 } 
}); 

console.log(anotherPerson2.friends); // []
anotherPerson2.friends.push("Rob2"); 
console.log(anotherPerson2.friends); // ['Rob2']
console.log(person.friends); // "Shelby,Court,Van,Rob,Barbie" 
```

原型式继承非常**适合不需要单独创建构造函数，但仍然需要在对象间共享信息的场合**。但要记住， 属性中包含的引用值始终会在相关对象间共享，跟使用原型模式是一样的

## 五、寄生式继承

与原型式继承比较接近的一种继承方式是寄生式继承

寄生式继承背后的思路类似于寄生构造函数和工厂模式：创建一个实现继承的函数，以某种 方式增强对象，然后返回这个对象。基本的寄生继承模式如下

```js
function createAnother(original){ 
 let clone = object(original); // 通过调用函数创建一个新对象
 clone.sayHi = function() { // 以某种方式增强这个对象
 console.log("hi"); 
 }; 
 return clone; // 返回这个对象
} 
```

寄生式继承实际上就是`调用某函数，根据要继承的对象，来创造出一个新对象（这里是调用object）`，然后给这个新对象增加新的属性或方法，最后返回这个新对象

**寄生式继承同样适合主要关注对象，而不在乎类型和构造函数的场景**。

**object()函数不是寄生式继承所必需的，任何返回新对象的函数都可以在这里使用**

`通过寄生式继承给对象添加函数会导致函数难以重用（即在每个对象上都会定义一个相同的函数），与构造函数模式类似。`

## 六、寄生式组合继承

组合继承其实也存在效率问题。**最主要的效率问题就是父类构造函数始终会被调用两次**：一次在是 创建子类原型时调用，另一次是在子类构造函数中调用。

```js
function SuperType(name) { 
 this.name = name; 
 this.colors = ["red", "blue", "green"]; 
} 
SuperType.prototype.sayName = function() { 
 console.log(this.name); 
}; 
function SubType(name, age){ 
 SuperType.call(this, name); // 第二次调用 SuperType() 
 this.age = age; 
} 
SubType.prototype = new SuperType(); // 第一次调用 SuperType() 
SubType.prototype.constructor = SubType; 
SubType.prototype.sayAge = function() { 
 console.log(this.age); 
}; 
```

在子类实例上，有两组 name 和 colors 属性：一组在实例上，另一组在 SubType 的原型上。这是 调用两次 SuperType 构造函数的结果。

寄生式组合继承通过盗用构造函数继承属性，但使用混合式原型链继承方法。基本思路是不通过调用父类构造函数给子类原型赋值，而是取得父类原型的一个副本。说到底就是**使用寄生式继承来继承父类原型，然后将返回的新对象赋值给子类原型**。

寄生式组合继承的基本模式如下所示

```js
function inheritPrototype(subType, superType) { 
 let prototype = object(superType.prototype); // 创建对象
 prototype.constructor = subType; // 增强对象 给返回的prototype 对象设置 constructor 属性，解决由于重写原型导致默认 constructor 丢失的问题。
 subType.prototype = prototype; // 赋值对象
}
```

完整的寄生式组合继承如下：

```js
function SuperType(name) { 
 this.name = name; 
 this.colors = ["red", "blue", "green"]; 
} 
SuperType.prototype.sayName = function() { 
 console.log(this.name); 
}; 
function SubType(name, age) { 
 SuperType.call(this, name); 
 this.age = age; 
} 
inheritPrototype(SubType, SuperType); 
SubType.prototype.sayAge = function() { 
 console.log(this.age); 
}; 
```

这里只调用了一次 SuperType 构造函数，避免了 SubType.prototype 上不必要也用不到的属性， 因此可以说这个例子的效率更高。而且，原型链仍然保持不变，因此 instanceof 操作符和 isPrototypeOf()方法正常有效。寄生式组合继承可以算是引用类型继承的最佳模式。
