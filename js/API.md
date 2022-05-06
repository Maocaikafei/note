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

列出所有实例属性名称，无论是否可以枚举（不包括原型）

`['foo', 'baz', 'qux']`

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

#### Object.prototype.toString()

如果此方法在自定义对象中未被覆盖，`toString()` 返回 "[object *type*]"，其中 `type` 是对象的类型

toString() 调用 null 返回[object Null]，undefined 返回 [object Undefined]

# Array

#### Array构造器

Array(8)会被优化为new Array(8)

对于 new Array(arg1, arg2)

- 当参数的长度为0或大于等于2时，参数将作为数组的项
- 当参数长度为1时，称其为len
  - 若len不是数值，同上
  - 若len为数值，作为数组长度
    - 如果希望传递单数值参数，且希望将其作为数组的项，可使用Array.of

注意，new Array(7)生成的数组，会有7个空位，而不是7个undefined

#### Array.of

`Array.of()` 方法创建一个具有可变数量参数的新数组实例，而不考虑参数的数量或类型。

`Array.of()` 和 `Array` 构造函数之间的区别在于处理整数参数：`Array.of(7)` 创建一个具有单个元素 **7** 的数组，而 **`Array(7)`** 创建一个长度为7的空数组（**注意：**这是指一个有7个空位(empty)的数组，而不是由7个`undefined`组成的数组）

#### Array.from

`Array.from()` 方法对一个类似数组或可迭代对象创建一个新的，浅拷贝的数组实例。

第二个参数，`mapFn` 可选。如果指定了该参数，新数组中的每个元素会执行该回调函数

```
console.log(Array.from('foo'));
// expected output: Array ["f", "o", "o"]

console.log(Array.from([1, 2, 3], x => x + x));
// expected output: Array [2, 4, 6]
```

#### Array.isArray()

Array.isArray() 用于确定传递的值是否是一个Array

## Array.prototype

#### fill()

`fill()` 方法用一个固定值填充一个数组中从起始索引到终止索引内的全部元素。（会改变原数组）

#### slice()

`slice()` 方法返回一个新的数组对象，这一对象是一个由 `begin` 和 `end` 决定的原数组的**浅拷贝**（包括 `begin`，不包括`end`）。原始数组不会被改变

若参数为负数，则表示取原数组中的倒数第几个元素

begin默认0，如果 `begin` 超出原数组的索引范围，则会返回空数组。

如果 `end` 被省略，则 `slice` 会一直提取到原数组末尾。如果 `end` 大于数组的长度，`slice` 也会一直提取到原数组末尾

#### concat()

`concat()` 方法用于合并两个或多个数组。此方法不会更改现有数组，而是返回一个新数组（浅拷贝）

#### splice()

**`splice()`** 方法通过删除或替换现有元素或者原地添加新的元素来修改数组,并以数组形式返回被修改的内容。**此方法会改变原数组**

arg1：指定修改的开始位置（从0计数）

arg2：表示要移除的数组元素的个数。如果 `deleteCount` 被省略了，那么`start`之后数组的所有元素都会被删除，如果 `deleteCount` 是 0 或者负数，则不移除元素。

arg3,arg4...：要添加进数组的元素,从`start` 位置开始。如果不指定，则 `splice()` 将只删除数组元素

#### filter()

`filter()` 方法创建一个新数组, 其包含通过所提供函数实现的测试的所有元素。 

```
const words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];

const result = words.filter(word => word.length > 6);

console.log(result);
// expected output: Array ["exuberant", "destruction", "present"]
```

#### find()

`find()` 方法返回数组中满足提供的测试函数的**第一个**元素的值。否则返回 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)。

#### findIndex()

`findIndex()`方法返回数组中满足提供的测试函数的第一个元素的**索引**。若没有找到对应元素则返回-1。

#### flat()

`flat(depth)` 方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回。`flat()` 方法会移除数组中的空项

depth：指定要提取嵌套数组的结构深度，默认值为 1

```js
var arr1 = [1, 2, [3, 4]];
arr1.flat();
// [1, 2, 3, 4]

var arr2 = [1, 2, [3, 4, [5, 6]]];
arr2.flat();
// [1, 2, 3, 4, [5, 6]]

var arr3 = [1, 2, [3, 4, [5, 6]]];
arr3.flat(2);
// [1, 2, 3, 4, 5, 6]

//使用 Infinity，可展开任意深度的嵌套数组
var arr4 = [1, 2, [3, 4, [5, 6, [7, 8, [9, 10]]]]];
arr4.flat(Infinity);
// [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

#### includes()

`includes()` 方法用来判断一个数组是否包含一个指定的值，根据情况，如果包含则返回 `true`，否则返回 `false`

```
const array1 = [1, 2, 3];

console.log(array1.includes(2));
// expected output: true

const pets = ['cat', 'dog', 'bat'];

console.log(pets.includes('cat'));
// expected output: true

console.log(pets.includes('at'));
// expected output: false
```

#### indexOf()

`indexOf()`方法返回在数组中可以找到一个给定元素的第一个索引，如果不存在，则返回-1。

#### lastIndexOf()

`lastIndexOf()` 方法返回指定元素在数组中的最后一个的索引，如果不存在则返回 -1。从数组的后面向前查找

#### join()

`join()` 方法将一个数组（或一个[类数组对象](https://developer.mozilla.org/zh-CN_docs/Web/JavaScript/Guide/Indexed_collections#working_with_array-like_objects)，例如arguments）的所有元素连接成一个字符串并返回这个字符串。如果数组只有一个项目，那么将返回该项目而不使用分隔符

参数separator：可选，指定一个字符串来分隔数组的每个元素，默认为`,`

#### pop()

`pop()` 方法从数组中删除最后一个元素，并返回该元素的值。此方法会更改数组的长度

#### push(item1, item2, ..., itemX)

`push()` 方法将一个或多个元素添加到数组的末尾，并**返回该数组的新长度**

#### shift()

`shift()` 方法从数组中删除**第一个**元素，并返回该元素的值。此方法更改数组的长度

#### unshift(item1, item2, ..., itemX)

`unshift()` 方法将一个或多个元素添加到数组的**开头**，并返回该数组的**新长度(该**方法修改原有数组**)**

#### reverse()

`reverse()` 方法将数组中元素的位置颠倒，并返回该数组。数组的第一个元素会变成最后一个，数组的最后一个元素变成第一个。该方法会改变原数组。

#### reduce()

`reduce()` 方法对数组中的每个元素按序执行一个由您提供的 **reducer** 函数，每一次运行 **reducer** 会将先前元素的计算结果作为参数传入，最后将其结果汇总为单个返回值

#### map()

`map()` 方法创建一个新数组，这个新数组由原数组中的每个元素都调用一次提供的函数后的返回值组成。（不改变原数组）

#### forEach()

`forEach()` 方法对数组的每个元素执行一次给定的函数

返回值为undefined

除了抛出异常以外，没有办法中止或跳出 `forEach()` 循环

# String

## String.prototype

#### charAt(index)

从一个字符串中返回指定的字符

#### concat(str2, [, ...strN])

将一个或多个字符串与原字符串连接合并，形成一个新的字符串并返回

建议使用[赋值操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Assignment_Operators)（`+`, `+=`）代替 `concat` 方法

#### startsWith(searchString[, position])

判断当前字符串是否以另外一个给定的子字符串开头，并根据判断结果返回 `true` 或 `false`。

#### endsWith(searchString[, length])

用来判断当前字符串是否是以另外一个给定的子字符串“结尾”的，根据判断结果返回 `true` 或 `false`。

#### includes(searchString[, position])

用于判断一个字符串是否包含在另一个字符串中，根据情况返回 true 或 false

#### indexOf(searchValue [, fromIndex])

返回调用它的 [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) 对象中第一次出现的指定值的索引，从 `fromIndex` 处进行搜索。如果未找到该值，则返回 -1

searchValue可以是多字符的字符串

#### lastIndexOf(searchValue[, fromIndex])

#### substring(indexStart[, indexEnd])

返回一个字符串在开始索引到结束索引之间的一个子集, 或从开始索引直到字符串的末尾的一个子集。

#### match(regexp)

检索返回一个字符串匹配正则表达式的结果

#### repeat(count)

构造并返回一个新字符串，该字符串包含被连接在一起的指定数量的字符串的副本

#### replace(regexp|substr, newSubStr|function)

返回一个由替换值（`replacement`）替换部分或所有的模式（`pattern`）匹配项后的新字符串。模式可以是一个字符串或者一个[正则表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp)，替换值可以是一个字符串或者一个每次匹配都要调用的回调函数。**如果`pattern`是字符串，则仅替换第一个匹配项。**

#### replaceAll(regexp|substr, newSubstr|function)

#### slice(beginIndex[, endIndex])

提取某个字符串的一部分，并返回一个新的字符串，且不会改动原字符串

#### split([separator[, limit]])

使用指定的分隔符字符串将一个[`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)对象分割成子字符串数组，以一个指定的分割字串来决定每个拆分的位置

#### toLowerCase()

将调用该方法的字符串值转为小写形式，并返回

#### toUpperCase()

#### trim()

从一个字符串的两端删除空白字符



