---
nav:
  title: 基本语法
  order: 1
group:
  title: 数据类型和值
  order: 3
title: 数据类型
order: 1
---

# 数据类型

JavaScript是一门**弱类型语言**（动态语言）。

这意味着变量本身是没有类型的，值，即**数据才有类型**。

在JS中有8中**基本数据类型**（7种**原始类型**还有1种**引用类型**）.

我们可以将任何类型的值存入变量中

**举个例子**😀

```js
let a = 1;
a = "1";
```

💡 这意味着变量不会统一成某一数据类型。

JavaScript中规定了**原始数据类型**和**引用数据类型**

- **原始数据类型**：数据直接存放在栈中，变量可以直接按值访问，直接操作保存在变量中实际的值。
  - **空值**（null）
  - **未定义**(undefined)
  - **布尔值**(boolean)
  - **数字**(number)
  - **字符串**(string)
  - **符号**(Symbol)
  - **Bigint**(并非所有游览器都兼容)
- **引用数据类型**：存储在堆中，变量指向的只是堆的地址，按引用访问。
  - **对象**(Object)

## 原始数据类型

### 空值

特殊的`null`值不属于上述如何一种类型, 同时他也是**字面量**。

💡 但使用typeof 检测null时候会返回object

**举个例子**😀

```js
let a = null;
typeof null; // "object"
```

### 未定义值

`undefined`的含义就是`未被赋值`

当一个变量已经被声明，但没有赋值，则为`undefined`

```js
let a;
console.log(a) // undefined
```

### 布尔值

布尔值表示逻辑上的真/假 `true`和 `false`

### 数字

number表示所有整数和浮点数

**操作**：+ 、- 、 * 、 / 等

**特殊值**：`Infinity`、`-Infinity` 和 `NaN`。

**进制数**:

- 十进制：JavaScript 中默认的进制数
- 八进制：第一位必须是 0，然后是 0-7 的数字组成
- 十六进制：前两位必须是 `0x`，然后是 0-9 及 A-F（字母不区分大小写）

```js
// 十进制
let num = 20;

// 八进制
let num = 010;

// 十进制
let num = 019;

// 十六进制
let num = ox2d
```

⚠️ **注意：** 八进制在严格模式下 `"use strict"` 是无效的，会导致 JavaScript 报错，避免使用。

**浮点数**：

```js
0.1 + 0.2 === 0.3	//false
```

在 JavaScript 中不论小数还是整数只有一种数据类型表示，这就是 Number 类型，其遵循 IEEE 754 标准，使用双精度浮点数（double）64 位（8 字节）来存储一个浮点数（所以在 **JS 中 1 === 1.0**）。**其中能够真正决定数字精度的是尾部，即 $2^{53-1}$**

**范围问题**：

- `Number.MIN_VALUE` 表示 JavaScript 中的最小值
- `Number.MAX_VALUE` 表示 JavaScript 中的最大值
- `Infinity`表示无穷大
- `-Infinity`表示无穷小

**NaN**:

`NaN` （Not a number）的含义是本该返回数值的操作未返回数值，返回了 `NaN` 就不会抛出异常影响语句流畅性。

⚠️**注意**：NaN 不等于自身。

```js
NaN === NaN //false
Number.isNaN(NaN) // true
```

### 字符串

JavaScript 中的字符串必须被括在引号里。

```js
let str = "Hello";
let str2 = 'Single quotes are ok too';
let phrase = `can embed another ${str}`;
```

**三种括号**：

- 双引号
- 单引号
- 反引号

⚠️**注意**：JS中没有char类型。

### 符号

`symbol` 类型用于创建对象的唯一标识符。

该类型的性质在于这个类型的值可以用来创建匿名的对象属性。

该数据类型通常被用作一个对象属性的键值，当这个属性是用于类或对象类型的内部使用的时候。

```js
var myPrivateMethod = Symbol();

this[myPrivateMethod] = function() {
  // ...
};

Obj[Symbol.iterator]
```

### BigInt类型

在 JavaScript 中，“number” 类型无法表示大于 `(253-1)`（即 `9007199254740991`），或小于 `-(253-1)` 的整数。这是其内部表示形式导致的技术限制。

在大多数情况下，这个范围就足够了，但有时我们需要很大的数字，例如用于加密或微秒精度的时间戳。

`BigInt` 类型是最近被添加到 JavaScript 语言中的，用于表示任意长度的整数。

可以通过将 `n` 附加到整数字段的末尾来创建 `BigInt` 值。

## 引用数据类型

引用类型通常叫做类（Class），也就是说，遇到引用值，所处理的就是对象。

`object` 类型是一个特殊的类型。

其他所有的数据类型都被称为“原始类型”，因为它们的值只包含一个单独的内容（字符串、数字或者其他）。相反，`object` 则用于储存数据集合和更复杂的实体。

```js
let a = {};
```

⚠️**注意**：对象存放在堆中，变量指向的是该对象的引用地址。

---

**参考资料**：

- [数据类型](https://zh.javascript.info/types)

- [数据类型](https://tsejx.github.io/javascript-guidebook/basic-concept/data-types/data-types/#%E5%AD%97%E7%AC%A6%E4%B8%B2)
- [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_VALUE)

- [JavaScript 浮点数之迷：0.1 + 0.2 为什么不等于 0.3？](https://www.javascriptc.com/books/nodejs-roadmap/javascript/floating-point-number-0.1-0.2.html)