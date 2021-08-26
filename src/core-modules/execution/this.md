---
nav:
  title: 核心知识
  order: 3
group:
  title: 执行阶段
  order: 3
title: this
order: 4
---
# this

## 调用位置

在理解`this`的绑定过程之前，首先理解`this`的调用位置：调用位置就是函数在代码中被**调用**的位置（而不是声明的位置）。

理解`this`的调用位置，我们要分析**调用栈**，同时，我们得分析是谁调用了当前执行的函数。

```js
function baz() {
  // 当前调用栈是：baz
  // 因此，当前调用位置是全局作用域
  console.log('baz');
  bar(); // <-- bar 的调用的位置
}

function bar() {
  // 当前调用栈是 baz -> bar
  // 因此，当前调用调用位置在 baz 中
  console.log('bar');
  foo(); // <-- foo 的调用位置
}

function foo() {
  // 当前调用栈是 baz -> bar -> foo
  // 因此，当前调用位置在 bar 中
  console.log('foo');
}

baz(); // <-- baz 的调用位置
```

注意我们是如何从调用栈中分析出真正的调用位置，因为它决定了 `this` 的绑定。

## 绑定规则

函数执行过程中调用位置决定`this`的绑定对象。

你必须找到调用位置，然后判断需要应用下面四条规则中的哪一条。我们首先会分别解释这四条规则，然后解释多条规则都可用时它们的优先级如何排列。

```js
调用栈 => 调用位置 => 绑定规则 => 规则优先级;
```

### 默认绑定

首先最常见的是：**独立函数调用**。可以看成是无应用其他规则时的默认规则。

tip: 谁调用，`this`指向谁，独立函数调用，理解用全局对象调用该函数。

```js
function foo() {
  console.log(this.a);
}

var a = 2;
// 相当于在全局声明了a = 2

// 调用的时候，相当于是全局对象调用的foo，所以this会指向全局对象。
// 而我们上部分中全局的a
foo(); //2
```

如果使用严格模式的话

```js
function foo() {
	"use strict";
  console.log(this.a);
}

var a = 2;
foo();	// TypeError: this is undefined
```

这里有一个微妙但是非常重要的细节，虽然 `this` 的绑定规则完全取决于调用位置，但是只有 `foo()` 运行在非严格模式下时，默认绑定才能绑定到全局对象；在严格模式下调用 `foo` 则不受默认绑定影响。

```js
function foo() {  console.log(this.a);}
var a = 2;
(function foo() {
  'use strict';
  foo(); // 2
})();
```

⚠️ **注意**：通常来说你不应该在代码中混合使用严格模式和非严格模式。整个程序要么严格要么非严格。然而，有时候你可能会用到第三方库，其严格程度和你代码有所不同，因此一定要注意这类兼容性细节。

### 隐式绑定

另一条需要考虑的规则是调用位置是否有**上下文对象**，或者说是否**被某个对象拥有或者包含**，不过这种说法可能会造成一些误导。

tip: 谁调用，`this`指向谁，对象调用，理解为该对象调用该函数。

**举个例子：**

```js
function foo() {  console.log(this.a);}
const container = {  a: 2,  foo: foo,};
container.foo(); // 2
```

在这里我们要注意，函数是存放在堆中的，变量只是存储了函数的引用。

然而，调用位置会使用 `container` 上下文来引用函数，因此你可以说函数被调用时 `container` 对象 **拥有** 或者 **包含** 它。

当函数引用有上下文时，隐式绑定规则会把函数调用中的 `this` 绑定到这个上下文对象。因为调用 `foo` 时 `this` 被绑定到 `container` 上，因此 `this.a` 和 `container.a` 是一样的。（**此时的this指向的是调用函数的对象**）。

注意，如果是对象上的对象调用的函数，则函数中的this指向的是直接调用该函数的对象

```js
function foo() {
  console.log(this.a);
}

var obj2 = {
  a: 42,
  foo: foo
}

var obj1 = {
  a: 2,
  obj2: obj2
}

obj1.obj2.foo()	// 42
```

#### 隐式丢失

一个最常见的 `this` 绑定问题就是**被隐式绑定的函数会丢失绑定对象**，也就是说它会应用默认绑定，从而把 `this` 绑定到全局对象或者 `undefined` 上（这取决于是否是严格模式）。

tip: 谁调用this就指向谁，这里很明显，b相当于是被全局对象所调用的，而即使这个`b=container.foo`,但也是只b指向了foo这个函数的引用。

**举个例子😊**

```js
function foo() {
  console.log(this.a)
}

const container = {
  a: 2,
  foo: foo
}

const bar = container.foo

var a = "test";

b();	//test 
```

虽然 `bar` 是 `container.foo` 的一个引用，但是实际上，它引用的是 `foo` 函数本身，因此此时的 `bar` 其实是一个不带任何修饰的函数调用，因此应用了默认绑定。

一种更微妙、更常见并且更出乎意料的情况发生在传入回调函数时。

**举个例子😊**

```js
function foo() {
  console.log(this.a);
}

function bar(fn) {
  // fn 其实引用的是 foo
  fn(); // <--调用位置
}

var container = {
  a: 2,
  foo: foo,
};

// a 是全局对象的属性
var a = 'Hello world!';

bar(container.foo);	// Hello world!
```

参数传递其实是一种**隐式赋值**，因此我们传入函数时也会被隐式赋值，所以结果和上个示例一样。

如果把函数传入语言内置的函数而不是传入你自己声明的函数，结果是一样的，没有区别。

传入的其实是引用

```js
function foo() {
  console.log(this.a);
}

const container = {
  a: 2,
  foo: foo,
};

// a 是全局对象的属性
var a = 'Hello world!';

setTimeout(container.foo, 100);
// 'Hello world!'
```

回调函数丢失 `this` 绑定是非常常见的。

除此之外，还有一种情况 `this` 的行为会出乎我们意料：调用回调函数的函数可能会修改 `this`。在一些流行的 JavaScript 库中事件处理器会把回调函数的 `this` 强制绑定到触发事件的 DOM 元素上。这在一些情况下可能很有用，但是有时它可能会让你感到非常郁闷。遗憾的是，这些工具通常无法选择是否启用这个行为。

无论是哪种情况，`this` 的改变都是意想不到的，实际上你无法控制回调函数的执行方式，因此就没有办法控制调用位置以得到期望的绑定。之后我们会介绍如何通过固定 `this` 来修复这个问题。

### 显式绑定

就像我们刚才看到的那样，在分析隐式绑定时，我们必须在一个对象内部包含一个指向函数的属性，并通过这个属性间接引用函数，从而把 `this` 隐式绑定到该对象上。

JavaScript 提供了 `apply`、`call` 和 `bind` 方法，为创建的所有函数绑定宿主环境。通过这些方法绑定函数的 `this` 指向称为**显式绑定**。

#### 硬绑定

硬绑定可以解决之前提出的丢失绑定的问题。

**标准例子😊**

```js
function foo() {  console.log(ths.a);}
const container = {  a: 2,};
var bar = function() {  foo.call(container);};
bar();// 2
setTimeout(bar, 100);// 2
// 硬绑定的 bar 不可能再修改它的 thisbar.call(window);// 2
```

这些函数实际上就是通过 `call` 或者 `apply` 实现了显式绑定，这样代码会更加优雅。

#### 内置函数

第三方库的许多函数，以及 JavaScript 语言和宿主环境中许多新的内置函数，都提供了一个可选的参数，通常被称为 **上下文（context）**，其作用和 `bind` 一样，确保你的回调函数使用指定的 `this` 。

```js
function foo(item) {  console.log(this.title, item);}
const columns = {  title: 'No:',}[  
  // 调用 foo 时把 this 绑定到 columns  
  (1, 2, 3)].forEach(foo, columns);// No:1 No:2 No:3
```

这些函数实际上就是通过 `call` 或者 `apply` 实现了显式绑定，这样代码会更加优雅。

### new 绑定

在 JavaScript 中，构造函数只是使用 `new` 操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类，实际上，它们甚至都不能说是一种特殊的函数类型，它们只是被 `new` 操作符调用的普通函数而已。

new的时候会产生一个新的对象。

所以，包括内置对象函数在内的所有函数都可以用 `new` 来调用，这种函数调用被称为**构造函数调用**。这里有一个重要但是非常细微的区别：实际上并不存在所谓的构造函数，只是对于函数的 **构造调用**。

🎉 使用 `new` 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

1. 创建全新的对象
2. 新的对象的\_\_proto\_\_会关联到构造函数的prototype
3. 将函数的this绑定到该对象上，并执行函数
4. 如果函数的值是引用值得的，返回这个引用值，否则返回该对象。

```js
function _new(Fn, ...args) {
  let instance = {};
  instance.__proto__ === Fn.prototype;
  let res = Fn.call(instance, ...args);
  return typeof res === "object" && res !== null ? res : instance;
}
```

解剖内部操作后，我们能得出结论 `new` 操作符是为了实现该过程的一种**语法糖**。

## 优先级

上文介绍了函数调用中 `this` 绑定的四条规则，你需要做的就是找到函数的调用位置并判断应用哪条规则。但是，如果某个调用位置应用多条规则，则必须给这些规则设定优先级。

毫无疑问，默认绑定的优先级是四条规则中最低的，所以我们先不考虑它。

```js
显式绑定 > new 绑定() > 隐式绑定;
```

### 隐式绑定和显式绑定

```js
function foo() {  console.log(this.a);}
const container1 = {  a: 1,  foo: foo,};
const container2 = {  a: 2,  foo: foo,};
container1.foo();// 1
container2.foo();// 2
container1.foo.call(container2);// 2
container2.foo.call(container1);// 1
```

显示绑定优先级更高。

### new 绑定和隐式绑定

```js
function foo(something) {
  this.a = something;
}

const container1 = {
  foo: foo,
};

const container2 = {};

container1.foo(2);
console.log(container1.a);
// 2

container1.foo.call(container2, 3);
console.log(container2.a);
// 3

var bar = new container1.foo(4);
console.log(container1.a);
// 2
console.log(bar.a);
// 4
```

可以看到 `new` 绑定比隐式绑定优先级高。但是 `new` 绑定和显式绑定谁的优先级更高呢？

`new` 和 `call/apply` 无法一起使用，因此无法通过 `new foo.call(obj1)` 来直接进行测试。但是我们可以使用硬绑定来测试他俩的优先级。

在看代码之前先回忆一下硬绑定是如何工作的。`Function.prototype.bind` 会创建一个新的包装函数，这个函数会忽略它当前的 `this` 绑定（无论绑定的对象是什么），并把我们提供的对象绑定到 `this` 上。

```js
function foo(something) {
  this.a = something;
}

var container1 = {};

var bar = foo.bind(container1);
bar(2);
console.log(container1.a);
// 2

var baz = new bar(3);
console.log(container1.a);
// 2
console.log(baz.a);
```

在上方我们已经bar的指向，但当我们使用的时候，发现`new`的比显示的高。

## 绑定例外

### 忽略指向

如果将 `null` 或 `undefined` 作为 `this` 的绑定对象传入 `call`、`apply` 或 `bind`，这些值在调用时会被忽略，实际应用的是默认绑定规则。

```js
function foo() {  console.log(this.a);}
var a = 2;
foo.call(null);// 2
```

此类写法常用于 `apply` 来展开数组，并当作参数传入一个函数，类似地，`bind` 可以对参数进行柯里化（预先设置一些参数）。

```js
function foo(a, b) {  console.log('a:' + a + ',b:' + b);}
// 把数组展开成参数
foo.apply(null, [2, 3]);// a:2, b:3
// 使用 bind 进行柯里化
var bar = foo.bind(null, 2);bar(3);// a:2, b:3
```

这两种方法都需要传入一个参数当作 `this` 的绑定对象。如果函数并不关心 `this` 的话，你 仍然需要传入一个占位值，这时 `null` 可能是一个不错的选择。

### 软绑定

硬绑定这种方式可以把 `this` 强制绑定到指定的对象（除了使用 `new` 时），防止函数调用应用默认绑定规则。问题在于，硬绑定会大大降低函数的灵活性，使用硬绑定之后就无法使用隐式绑定或者显式绑定来修改 `this`。

### 指向变更

如下列出四种方法可以在编码中改变 `this` 指向。

- 使用 ES6 的箭头函数
- 在函数内部使用 `_this = this`
- 使用 `apply`、`call` 和 `bind`
- `new` 实例化一个对象

## 箭头函数

箭头函数并不是使用 `function` 关键字定义的，而是使用被称为胖箭头的操作符 `=>` 定义的。箭头函数不使用 `this` 的四种标准规则，而是根据外层（函数或者全局）作用域来决定 `this`。并且，箭头函数拥有静态的上下文，即一次绑定之后，便不可再修改。

箭头函数中没有`this`指针。

**箭头函数的this是由定义该箭头函数的上下文的this所决定的。**

`this` 指向的固定化，并不是因为箭头函数内部有绑定 `this` 的机制，实际原因是箭头函数根本没有自己的 `this`，导致内部的 `this` 就是外层代码块的 `this`。正是因为它没有 `this`，所以也就不能用作构造函数。

```js
function foo() {
  // 返回一个箭头函数
  return a => {
    // this 继承自 foo()
    console.log(this.a);
  };
}
const container1 = { a: 1 };

const container2 = { a: 2 };

const bar = foo.call(container1);
bar.call(container2);	// 1
```

`foo`中的`a`函数中的this会使用foo的`this`,所以这个时候只需要看`foo`的`this`指向，在这里`foo`函数被显示调用被赋值给`bar`，故此时foo中this指向的是container1,无法进行更改了。

箭头函数可以像 `bind` 一样确保函数的 `this` 被绑定到指定对象，此外，其重要性还体现在它用更常见的词法作用域取代了传统的 `this` 机制。实际上，在 ES6 之前我们就已经在使用一种几乎和箭头函数完全一样的模式。

虽然 `self = this` 和箭头函数看起来都可以取代 `bind`，但是从本质上来说，它们想替代的是 `this` 机制。

如果你经常编写 `this` 风格的代码，但是绝大部分时候都会使用 `self = this` 或者箭头函数来否定 `this` 机制，那你或许应当:

- 只使用词法作用域并完全抛弃错误 `this` 风格的代码
- 完全采用 `this` 风格，在必要时使用 `bind`，尽量避免使用 `self = this` 和箭头函数

## 应用场景总结

1. 函数的普通调用
2. 函数作为对象方法调用
3. 函数作为构造函数调用
4. 函数通过 call、apply、bind 间接调用
5. 箭头函数的调用

---

**参考资料**：

- [📚 你不知道的 JavaScript（上卷）](https://github.com/getify/You-Dont-Know-JS/blob/master/this %26 object prototypes/README)
- [📝 this](https://tsejx.github.io/javascript-guidebook/core-modules/executable-code-and-execution-contexts/execution/this)