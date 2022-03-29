1. **等于操作符与全等操作符**：

   1. `==`：会先进行类型转换（通常称为强制类型转换）再确定操作数是否相等
      1. 如果两个对象使用==比较，且他们指向同一地址， 则返回 true。（`===`同样适用）
   2. `===`：在比较相等时不转换操作数

2. **基本数据类型**：`null`、`undefined`、`boolean`、`string`、`number`。数据是直接按值存放的，他们的值在内存中占据着固定大小的空间，并被保存在栈内存中，可以直接访问，其数据类型的值是不可变的

   1. 基本类型的比较是值的比较，只要它们的值相等就认为他们是相等的

3. **引用数据类型**包括对象和数组，其存储在堆当中，而变量实际上是一个存放在栈内存的指针，这个指针指向堆内存中的地址。 当我们访问的时候，实际上是访问指针，然后指针去寻找对象或数组，其数据类型的值是可变的。

   1. 引用类型的比较是引用的比较，看引用是否指向同一个对象

4. **浅拷贝**：只复制指向某个对象的指针，新旧对象共享一块内存。（**也有一种说法是只拷贝了数据对象的第一层，第一层数据不会相互影响，但更深层次会影响**）（注：浅拷贝、深拷贝都是对引用数据类型来说的）

   1. 可用`Object.assign`, `扩展运算符，如：{...obj}`,  数组的`slice`, `concat`实现

5. **深拷贝**：**无限层级拷贝**，深拷贝后的原对象不会和拷贝对象互相影响。

   简单的说，深拷贝就是先新建一个空对象，内存中**新开辟一块地址**，把被复制对象的所有**可枚举的**属性方法一一复制过来，注意要用**递归**来复制子对象里面的所有属性和方法，直到子属性为基本数据类型。

6. 