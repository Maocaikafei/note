# Object

#### Object.create(obj1, [propertiesObject])

接收两个参数：作为新对象原型的对象，以及给新对象定义额外属性的对象（可选，注意格式，需要传入一个对象，该对象的属性类型参照[`Object.defineProperties()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties)的第二个参数)

（如果第一个参数是`null`，那新对象就彻彻底底是个空对象，没有继承`Object.prototype`上的任何属性和方法，如`hasOwnProperty()、toString()`等）

#### Object.keys(obj)

返回包含该对象所有可枚举属性名称的字符串数组（不包括原型）

```
console.log(Object.keys(o)); // ['foo', 'baz', 'qux']
```

#### Object.values(obj)

返回对象值的数组

```
const o = { 
 foo: 'bar', 
 baz: 1, 
 qux: {} 
}; 
console.log(Object.values(o)); // [bar, 1, {}]
```

#### Object.entries(obj)

返回键/值对的数组

```
console.log(Object.entries(o)); // [["foo", "bar"], ["baz", 1], ["qux", {}]]
```

#### Object.fromEntries(array）

通过`键/值对数组的集合`构建对象，执行与 Object.entries()方法相反的操作

```
Object.fromEntries([["foo", "bar"], ["baz", 1], ["qux", {}]])
// { 
 foo: 'bar', 
 baz: 1, 
 qux: {} 
}; 
```

#### Object.assign()

这个方法接收一个目标对象和一个 或多个源对象作为参数，然后将每个源对象中可枚举（Object.propertyIsEnumerable()返回 true） 和自有（Object.hasOwnProperty()返回 true）属性复制到目标对象。

Object.assign()实际上对每个源对象执行的是浅复制。如果多个源对象都有相同的属性，则使 用最后一个复制的值。

#### Object.freeze()

- 冻结一个对象，被冻结后的对象的属性不能修改，不能添加新属性，不能删除已有属性。
- 不能修改对象已有属性的可枚举性、可配置性、可写性
- 不能修改已有的属性值（浅冻结。只管一层，第二层以后的对象还是可以修改）
- 冻结的对象的原型的指向也不能修改

#### Object.isFrozen()

判断一个对象是否已经被冻结

#### Object.getOwnPropertyNames()

列出所有实例属性，无论是否可以枚举（不包括原型）

#### Object.defineProperties()

直接在一个对象上定义新的属性或修改现有属性，并返回该对象（会修改原对象）

```
Object.defineProperties(obj, props)
```

```
var obj = {};
Object.defineProperties(obj, {
  'property1': {
    value: true,
    writable: true
  },
  'property2': {
    value: 'Hello',
    writable: false
  }
});
```

配置属性：

- writable，是否可修改，默认false
- configurable，是否可修改和删除，默认false
- enumerable，是否可枚举，默认false
- value，与属性关联的值，默认undefined
- get，作为该属性的 getter 函数，函数返回值将被用作属性的值，默认为undefined
- set，作为属性的 setter 函数，函数将仅接受参数赋值给该属性的新值，默认为undefined

#### Object.defineProperty(obj, prop, descriptor)

直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

```
Object.defineProperty(obj, "key", {
  enumerable: false,
  configurable: false,
  writable: false,
  value: "static"
});
```

#### Object.getPrototypeOf()

返回参数的内部特性 [[Prototype]]的值，即原型对象

```
console.log(Object.getPrototypeOf(person1) == Person.prototype); // true
```

## Object.prototype

Object原型上的方法

#### Object.prototype.isPrototypeOf()

用来判断该对象是否为**参数对象**的原型。

```
var o1 = {};
var o2 = Object.create(o1);
var o3 = Object.create(o2);

o2.isPrototypeOf(o3)       // true
o1.isPrototypeOf(o3)       // true
Object.prototype.isPrototypeOf({})  // true
```

由于`Object.prototype`处于原型链的最顶端，所以对各种实例都返回`true`，只有直接继承自`null`的对象除外。

#### Object.prototype.hasOwnProperty(key)

返回一个布尔值，指示对象是否具有指定的属性作为自身属性（会忽略掉那些从原型链上继承到的属性）（可查找到不可枚举属性）

配合for..in，可在遍历对象所有属性，同时忽略继承属性。