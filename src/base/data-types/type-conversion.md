---
nav:
  title: 基本语法
  order: 1
group:
  title: 数据类型和值
  order: 3
title: 类型转换
order: 3
---
# 类型转换

JavaScript是一门弱类型的语言，不像其他强类型语言规定好数据类型，Js中变量类型是不确定的，其中，我们在编码的过程中，程序也允许变量类型的**隐式类型转换**和**强制类型转换**。

## 基本规则

### String

> 此处所说的 **String** 是指其他类型的值转换为字符串类型的操作。

有以下的规则:

- null 转化成 "null"
- undefined转化成"undefined"
- Boolean类型:
  - true 转化成 "true"
  - false转化成 "false"
- Number类型:转化成数字的字符串类型
  - 如10转化成"10"
  - 1e21转化成"1e+21"
- Array:转化成字符串，各元素以小写逗号链接,类型`Arrar.prototype.join(",")`
  - 空数组转成空字符串`""`
  - 数组中的`null`和`undefined`当成**空字符串**处理
- 普通对象：相当于调用`Object.prototype.toString()`

### Number

- null: 转化成0
- undefined: NaN
- String：纯数字，则转化成对应数字
  - 空字符为0
  - 否则一律NaN
- Boolean类型
  - true为1
  - false为0
- Array类型：数组会转化成原始数据类型，也就是ToPrimitive，再比较。
- 对象：对象会转化成原始数据类型，也就是ToPrimitive，再比较。

### Boolean

JavaScript中假值只有`false`,`null`,`undefined`,`""`,`0`,`NaN`,引用类型的都为`true`

### Primitive

> ToPrimitive 方法用于将引用类型转换为原始数据类型的操作

值为引用数据类型时，会调用 JavaScript 内置的 `@@ToPrimitive(hint)` 方法来指定其目标类型。

- 如果传入值为 Number 类型，则调用对象的 `valueOf()` 方法，若返回值为原始数据类型，则结束 `@@ToPrimitive` 操作，如果返回的不是原始数据类型，则继续调用对象的 `toString()` 方法，若返回值为原始数据类型，则结束 `@@ToPrimitive` 操作，如果返回的还是引用数据类型，则抛出异常。
- 如果传入值为 String 类型，则先调用 `toString()` 方法，再调用 `valueOf()` 方法。

值得一提的是对于 **数值类型** 的 `valueOf()` 函数的调用结果仍为数组，因此数组类型的隐式类型转换结果是字符串。

而在 ES6 中引入 **Symbol** 类型之后，JavaScript 会优先调用对象的 `[Symbol.toPrimitive]` 方法来将该对象转化为原始类型，那么方法的调用顺序就变为了：

- 当 `obj[Symbol.toPrimitive](preferredType)` 方法存在时，优先调用该方法
- 如果 `preferredType` 参数为 String 类型，则依次尝试 `obj.toString()` 与 `obj.valueOf()`
- 如果 `preferredType` 参数为 Number 类型或者默认值，则依次尝试 `obj.valueOf()` 与 `obj.toString()`

## 显式类型转换

- 转换成数值类型
  - `Number(mix)`
  - `parseInt(string, radix)`
  - `parseFloat(string)`
- 转化成字符串类型
  - `toString(radix)`
  - `String(mix)`
- 转换成布尔类型
  - `Boolean(mix)`

## 隐式类型转换

JavaScript中，当运算符在运算时，如果**两边数据类型不统一**，就无法进行运算，这时我们编译器会自动将运算符两边的数据做一个数据类型转换，转成相同的数据类型再计算。

无需开发者手动转化，而是借助编译器自动转换的方式称为**隐式类型转化**。

JavaScript中的数据类型隐式转换分为三种情况

- 转换成**Boolean**类型
- 转换成**Number**类型
- 转换成**String**类型

值在 **逻辑判断** 和 **逻辑运算** 时会隐式转换为 Boolean 类型。

Boolean 类型转换规则表：

| 数据值              | 转换后的值 |
| ------------------- | ---------- |
| 数字 `0`            | false      |
| `NaN`               | false      |
| 空字符串 `""`       | false      |
| `null`              | false      |
| `undefined`         | false      |
| 非 `!0` 数字        | true       |
| 非空字符串 `!""`    | true       |
| 非 `!null` 对象类型 | true       |

**注意事项**：使用 `new` 运算符创建的对象隐式转换为 Boolean 类型的值都是 `true`。

连续非操作可以取得Boolean类型

```js
!!1
!!null
!!{}
```

### 运行环境

很多内置函数期望传入的参数的数据类型是固定的，如 `alert(value)`，它期望传入的 `value` 为 String 类型，但是如果我们传入的是 Number 类型或者 Object 类型等非 String 类型的数据的时候，就会发生数据类型的隐式转换。这就是环境运行环境对数据类型转换的影响。

类似的方法有：

- `alert()`
- `parseInt()`

### 运算符

#### 加号运算符

当加号运算符作为**一元运算符**运算值时，它会将该值转换为 Number 类型。

```js
+"123"	// 123
+"123string" //NaN
+ undefined // NaN
+ [] // 0
+ [1] //1
+ [1,2] //NaN
+ {} // NaN
+ function() {} // NaN
```

当加号运算符作为**二元运算符**操作值时，它会根据两边值类型进行数据类型隐式转换。

- 首先，如果操作数中存在引用对象，会涉及引用值转化成原始数据类型的操作
  - 如果是原始数据类型，无需转化
  - 如果是引用类型。则
    - 调用实例的toPrimitive()方法。
    - 调用实例的 `valueOf()` 方法，如果有返回的是基础类型，停止下面的过程；否则继续
    - 调用实例的 `toString()` 方法，如果有返回的是基础类型，停止下面的过程；否则继续
    - 都没返回原始类型，就会报错
- 如果运算符两边均为原始数据类型时，则按照以下规则解释：
  - 字符串连接符：两个操作数中只要存在一个操作数为String，则另一个操作对象会调用`String()`变成字符串后拼接。
  - 算术运算符：如果两个操作数都不是 String 类型，两个操作数会调用 `Number()` 方法隐式转换为 Number 类型（如果无法成功转换成数字，则变为 `NaN`，再往下操作），然后进行加法算术运算

值转换为 Number 类型和 String 类型都会遵循一个原则：如果该值为原始数据类型，则直接转换为 String 类型或 Number 类型。如果该值为引用数据类型，那么先通过固定的方法将复杂值转换为原始数据类型，再转为 String 类型或 Number 类型

**注意事项**：当 `{} + 任何值` 时， 前一个 `{}` 都会被 JavaScript 解释成空块并忽略他。

```js
"1" + 1             // "11"
"1" + "1"           // "11"
"1" + true          // "1true"
"1" + NaN           // "1NaN"
"1" + [1]            // "1"
"1" + {}            // "1[object Object]"
"1" + function(){}  // "1function(){}"
"1" + new Boolean() // "1false"

1 + NaN             // NaN
1 + "true"          // "1true"
1 + true            // 2
1 + undefined       // NaN
1 + null            // 1

1 + []              // "1"
1 + [1, 2]          // "11,2"
1 + {}              // "1[object Object]"
1 + function(){}    // "1function(){}"
1 + Number()        // 1
1 + String()        // "1"

[] + []             // ""
{} + {}             // "[object Object][object Object]"
{} + []				// 相当于+[]
{a: 0} + 1          // 1
[] + {}             // "[object Object]"
[] + !{}            // "false"
![] + []            // "false"
'' + {}             // "[object Object]"
{} + ''             // 相当于 +'' 0
[]["map"] + []      // "function map(){ [native code] }" 数组的内置函数
[]["a"] + []        // "undefined"
[][[]] + []         // "undefined"
+!![] + []			// "1"
+!![]               // 1
1-{}                // NaN
1-[]				// 1
true - 1            // 0
{} -1 				// -1
[] !== []           // true

(![]+[])[+[]]       // "f"
(![]+[])[+!![]]     // "a"
```

#### 相等运算符

相等运算符 `==` 会对操作值进行隐式转换后进行比较

- 如果其中一个操作值为布尔值，则在比较之前先将其转换为数值
- 如果其中一个操作值为字符串，另一个操作值为数值，则通过 `Number()` 函数将字符串转换为数值
- 如果其中一个操作值是对象，另一个不是，则调用对象的 `toPrimitive` 方法，得到的结果按照前面的规则进行比较.若没有，则调用`valueOf()`,则在使用`toString()`
- `null` 与`undefined` 是相等的
- 如果一个操作值为 `NaN`，则返回 `false`
- 如果两个操作值都是对象，则比较它们是不是指向同一个对象

```js
'1' == true; // true
'1' == 1; // true
'1' == {}; // 
false'1' == []; // false
undefined == undefined; // true
undefined == null; // true
null == null; // true
```

#### 关系运算符

关系运算符：会把其他数据类型转换成 Number 之后再比较关系（除了 Date 类型对象）

- 如果两个操作值都是数值，则进行 **数值** 比较

- 如果两个操作值都是字符串，则比较字符串对应的

   

  ASCII 字符编码值

  - 多个字符则从左往右依次比较

- 如果只有一个操作值是数值，则将另一个操作值转换为数值，进行 **数值** 比较

- 如果一个操作数是对象，则调用 `valueOf()` 方法（如果对象没有 `valueOf()` 方法则调用 `toString()` 方法），得到的结果按照前面的规则执行比较

- 如果一个操作值是布尔值，则将其转换为 **数值**，再进行比较

📍 `NaN` 是非常特殊的值，它不和任何类型的值相等，包括它自己，同时它与任何类型的值比较大小时都返回 `false`。

```js
5 > 10;
// false

'2' > 10;
// false

'2' > '10';
// true

'abc' > 'b';
// false

'abc' > 'aad';
// true
```

**JavaScript 原始类型转换表**

| 原始值             | 转换为数字类型 | 转换为字符串类型  | 转换为布尔类型 |
| ------------------ | -------------- | ----------------- | -------------- |
| false              | 0              | "false"           | false          |
| true               | 1              | "true"            | true           |
| 0                  | 0              | "0"               | false          |
| 1                  | 1              | "1"               | true           |
| "0"                | 0              | "0"               | true           |
| "000"              | 0              | "000"             | true           |
| "1"                | 1              | "1"               | true           |
| NaN                | NaN            | "NaN"             | false          |
| Infinity           | Infinity       | "Infinity"        | true           |
| -Infinity          | -Infinity      | "-Inifinity"      | true           |
| ""                 | 0              | ""                | false          |
| " "                | 0              | " "               | true           |
| "20"               | 20             | "20"              | true           |
| "Hello"            | NaN            | "Hello"           | true           |
| []                 | 0              | ""                | true           |
| [20]               | 20             | "20"              | true           |
| [10, 20]           | NaN            | "10,20"           | true           |
| ["Hello"]          | NaN            | "Hello"           | true           |
| ["Hello", "World"] | NaN            | "Hello,World"     | true           |
| function(){}       | NaN            | "function(){}"    | true           |
| {}                 | NaN            | "[object Object]" | true           |
| null               | 0              | "null"            | false          |
| undefined          | NaN            | "undefined"       | false          |