---
nav:
  title: 基本语法
  order: 1
group:
  title: 数据类型和值
  order: 3
title: 类型检测
order: 2
---

# 类型检测

类型检测的方法

1. `typeof`
2. `instanceof`
3. `Object.prototype.toString`
4. `constructor`

## typeof

typeof返回字符串，表示就该操作数返回的类型。

```js
typeof undefined; // "undefined"
typeof null;  // "object"
typeof 100; // "number"
typeof NaN; // "number"
typeof true; // "boolean"
typeof "foo"; // "string"
typeof function(){} // "object"
typeof [1,2] // "object"
typeof new Oject();// "object"
```

`typeof` 操作符适合对 **基本类型**（除 `null` 之外）及 `function` 的检测使用，而对引用数据类型（如 Array）等不适合使用。

## instanceof

`instanceof` 操作符用于检查一个对象**原型链**上是否有**构造函数**的`prototype`属性。

**使用方法**

```js
obj instanceof Object;	// true
```

```js
function A() {}
function B() {}
B.prototype = new A();
B.prototype.constructor = B;

let b = new B();
b instanceof B;	// true
b instanceof A;	// true
```

任何一个函数都有`prototype`属性，这是普通对象所不具备的。

这个对象属性将用作 `new` 实例化对象的原型对象。

```js
let a = {};
a.__proto__ = Object.prototype;
```

## Object.prototype.toString

可以通过 `toString()` 来获取每个对象的类型。

为了**每个对象**都能通过 `Object.prototype.toString` 来检测，需要以 `Function.prototype.call` 或者 `Function.prototype.apply` 的形式来调用，传递要检查的对象作为第一个参数。

```js
Obejct.prototype.toString.call(undefined)；
//  "[object Undefined]"
```

💡 使用 `Object.prototype.toString` 方法能精准地判断出值的数据类型。

💡 `Object.prototype.toString` 属于 `Object` 的原型方法，而 `Array` ， `Function` 等类型作为 `Object` 的实例，都重写了 `toString` 方法。因此，不同对象类型调用 `toString` 方法时，调用的是重写后的 `toString` 方法，而非 `Object` 上原型 `toString` 方法，所以采用 `xxx.toString()` 不能得到其对象类型，只能将 `xxx` 转换成字符串类型。

## constructor

任何对象都有 `constructor` 属性，继承自原型对象，`constructor` 会指向构造这个对象的构造器或构造函数。

```js
Student.prototype.constructor === Student
```

## 数组检测

ECMAScript5 将 `Array.isArray()` 正式引入 JavaScript，该方法能准确检测一个变量是否为数组类型。

```js
Array.isArray(arr);
```

## NaN检测

```js
Number.isNaN(NaN)
```

