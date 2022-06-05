## 一、继承基础

虽然类继承使用的是新语法，但背后依旧使用的是原型链

ES6 类支持单继承。使用 extends 关键字，就可以继承任何拥有[[Construct]]和原型的对象。 很大程度上，这意味着不仅可以继承一个类，也可以继承普通的构造函数（保持向后兼容）

```js
class Vehicle {} 
// 继承类
class Bus extends Vehicle {} 
let b = new Bus(); 
console.log(b instanceof Bus); // true 
console.log(b instanceof Vehicle); // true 
function Person() {} 
// 继承普通构造函数
class Engineer extends Person {} 
let e = new Engineer(); 
console.log(e instanceof Engineer); // true 
console.log(e instanceof Person); // true 
```

派生类都会通过原型链访问到类和原型上定义的方法。this 的值会反映调用相应方法的实例或者类

```Js
class Vehicle { 
 identifyPrototype(id) { 
 console.log(id, this); 
 } 
 
 static identifyClass(id) { 
 console.log(id, this); 
 } 
} 
class Bus extends Vehicle {} 
let v = new Vehicle(); 
let b = new Bus(); 
b.identifyPrototype('bus'); // bus, Bus {} 
v.identifyPrototype('vehicle'); // vehicle, Vehicle {} 
Bus.identifyClass('bus'); // bus, class Bus {} 
Vehicle.identifyClass('vehicle'); // vehicle, class Vehicle {} 
```

extends 关键字也可以在类表达式中使用，因此 `let Bar = class extends Foo {}` 是有效的语法

## 二、构造函数、HomeObject 和 super()

派生类的方法可以通过 super 关键字引用它们的原型。这个关键字只能在派生类中使用，而且**仅限**于类构造函数和静态方法内部。

1、在类构造函数中使用 super 可以调用父类构造函数。

```js
class Vehicle { 
 constructor() { 
 this.hasEngine = true; 
 } 
} 
class Bus extends Vehicle { 
 constructor() { 
 // 不要在调用 super()之前引用 this，否则会抛出 ReferenceError 
 
 super(); // 相当于 super.constructor()  可以传参
 console.log(this instanceof Vehicle); // true 
 console.log(this); // Bus { hasEngine: true } 
 } 
} 
```

2、在静态方法中可以通过 super 调用继承的类上定义的静态方法

```js
class Vehicle { 
 static identify() { 
 console.log('vehicle'); 
 } 
} 
class Bus extends Vehicle { 
 static identify() { 
 super.identify(); 
 } 
} 

Bus.identify(); // vehicle
```

super 只能在**派生类**构造函数和静态方法中使用

不能单独引用 super 关键字，要么用它调用构造函数，要么用它引用静态方法

```js
class Vehicle {} 
class Bus extends Vehicle { 
 constructor() { 
 console.log(super); 
 // SyntaxError: 'super' keyword unexpected here 
 } 
}
```

调用 super()会调用父类构造函数，并将返回的实例赋值给 this。

如果没有定义类构造函数，在实例化派生类时会调用 super()，而且会传入所有传给派生类的 参数。

**在类构造函数中，不能在调用 super()之前引用 this**

```js
class Vehicle {} 
class Bus extends Vehicle { 
 constructor() { 
 console.log(this); 
 } 
} 
new Bus(); 
// ReferenceError: Must call super constructor in derived class 
// before accessing 'this' or returning from derived constructor 
```

