JavaScript 对象简谱（JSON，JavaScript Object Notation）

理解 JSON 最关键的一点是要**把它当成一种数据格式**，而不是编程语言。JSON 不属于 JavaScript， 它们只是拥有相同的语法而已。JSON 也不是只能在 JavaScript 中使用，它是一种通用数据格式。很多语 言都有解析和序列化 JSON 的内置能力

JSON 没有变量、函数或对象实例的概念。JSON 的所有记号都只为表示结构化数据

JavaScript 字符串与 JSON 字符串的主要区别是，JSON 字符串必须使用双引号

### 语法

JSON 语法支持表示 3 种类型的值

- 简单值：字符串、数值、布尔值和 null 可以在 JSON 中出现，就像在 JavaScript 中一样。特殊 值 undefined 不可以
- 对象：第一种复杂数据类型，对象表示有序键/值对。每个值可以是简单值，也可以是复杂类型
- 数组：第二种复杂数据类型，数组表示可以通过数值索引访问的值的有序列表。数组的值可以 是任意类型，包括简单值、对象，甚至其他数组

### 解析与序列化

#### JSON 对象

JSON 对象有两个方法：**stringify()**和 **parse()**。在简单的情况下，这两个方法分别可以将 JavaScript 序列化为 JSON 字符串，以及将 JSON 解析为原生 JavaScript 值

在序列化 JavaScript 对象时，**所有函数和原型成员都会有意地在结果中省略。此外，值为 undefined 的任何属性也会被跳过。**最终得到的就是所有实例属性均为有效 JSON 数据类型的表示

#### 序列化选项

实际上，JSON.stringify()方法除了要序列化的对象，还可以接收两个参数。这两个参数可以用 于指定其他序列化 JavaScript 对象的方式。第一个参数是过滤器，可以是数组或函数；第二个参数是用于缩进结果 JSON 字符串的选项。单独或组合使用这些参数可以更好地控制 JSON 序列化

##### 过滤结果

如果第二个参数是一个数组，那么 JSON.stringify()返回的结果只会包含该数组中列出的对象 属性

```js
let book = { 
 title: "Professional JavaScript", 
 authors: [ 
 "Nicholas C. Zakas", 
 "Matt Frisbie" 
 ], 
 edition: 4, 
 year: 2017 
}; 
let jsonText = JSON.stringify(book, ["title", "edition"]);
// {"title":"Professional JavaScript","edition":4} 
```

##### 字符串缩进

JSON.stringify()方法的第三个参数控制缩进和空格。在这个参数是数值时，表示每一级缩进的 空格数。

##### toJSON()方法

有时候，对象需要在 JSON.stringify()之上自定义 JSON 序列化。此时，可以在要序列化的对象 中添加 toJSON()方法，序列化时会基于这个方法返回适当的 JSON 表示。

```js

let book = { 
 title: "Professional JavaScript", 
 authors: [ 
 "Nicholas C. Zakas", 
 "Matt Frisbie" 
 ], 
 edition: 4, 
 year: 2017, 
 toJSON: function() { 
 return this.title; 
 }
}; 
let jsonText = JSON.stringify(book); 
// 这个对象会被序列化为简单字符串而非对象
```

toJSON()优先级高于第二个参数

#### 解析选项

JSON.parse()方法也可以接收一个额外的参数，这个函数会针对每个键/值对都调用一次。为区别 于传给 JSON.stringify()的起过滤作用的替代函数（replacer），这个函数被称为还原函数（reviver）。 实际上它们的格式完全一样，即还原函数也接收两个参数，属性名（key）和属性值（value），另外也 需要返回值。

**如果还原函数返回 undefined，则结果中就会删除相应的键。如果返回了其他任何值，则该值就 会成为相应键的值插入到结果中。**还原函数经常被用于把日期字符串转换为 Date 对象

```js
let book = { 
 title: "Professional JavaScript", 
 authors: [ 
 "Nicholas C. Zakas", 
 "Matt Frisbie" 
 ], 
 edition: 4, 
 year: 2017, 
 releaseDate: new Date(2017, 11, 1) 
}; 
let jsonText = JSON.stringify(book); 
let bookCopy = JSON.parse(jsonText, 
 (key, value) => key == "releaseDate" ? new Date(value) : value); 
alert(bookCopy.releaseDate.getFullYear());
```

