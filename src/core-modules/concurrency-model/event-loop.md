---
nav:
  title: 核心知识
  order: 3
group:
  title: 并发模型
  order: 4
title: 事件循环
order: 2
---
# 事件循环

为了协调事件、用户交互、脚本、UI 渲染、网络请求，用户代理必须使用 **事件循环机制（Event Loop）**。

这种事件循环机制是由 JavaScript 的宿主环境来实现的，在浏览器运行环境中由浏览器内核引擎实现，而在 NodeJS 中则由 [libuv](https://github.com/libuv/libuv) 引擎实现。

主线程运行时候，产生堆（Heap）和栈（Stack），栈中的代码调用各种外部 API，它们在任务队列中加入各种事件（Click、Load 和 Done）。只要栈中的代码执行完毕，主线程就会通过事件循环机制读取任务队列，依次执行那些事件所对应的回调函数。

## 介绍

单线程编程会因为I/O阻塞而使得硬件资源得不到更好的使用。

多线程编程会因为死锁、状态同步等问题加大开发的难度。

但游览器和Node环境则在两者之间给出了解决方案：利用单线程，避免多线程死锁，状态同步的问题；利用异步，让单线程原理阻塞，更好使用CPU。

## 浏览器环境

JavaScript的异步任务根据事件类型分成两种：**宏任务(MacroTask)**和**微任务(MicroTask)**

- **宏任务**：main script、setTimeout、setInterval、setImmediate（Node.js）、I/O（Mouse Events、Keyboard Events、Network Events）、UI Rendering（HTML Parsing）、MessageChannel、
- **微任务**：Promise.then（非 new Promise）、process.nextTick（Node.js）、MutationObserver

宏任务与微任务的区别在于队列中事件的执行优先级。进入整体代码（宏任务）后，开启首次事件循环，当执行上下文栈执行过程中，遇到同步任务的话，会直接继续执行，遇到异步任务，则异步任务会被分配给特定的线程处理（等待线程处理完，异步任务被推入任务队列中），执行上下文会继续执行，当执行完毕时，事件循环机制会优先检测微任务队列中的事件并推至主线程执行，当微任务队列清空之后，又去检测宏任务队列中的事件，再将事件推到主线程执行，而当上下文栈又清空时，事件循环机制会在检测微任务队列，如此反复循环。

**宏任务与微任务的优先级**

- 宏任务的优先级低于微任务
- 每个宏任务执行完毕后都必须将当前的微任务队列清空
- 第一个 `<script>` 标签的代码是第一个宏任务
- `process.nextTick` 优先级高于 `Promise.then`

![事件循环机制中宏任务和微任务图解](https://tsejx.github.io/javascript-guidebook/static/workflow.7125d86b.jpg)

```js
console.log(1);

setTimeout(() => {
  console.log(2);
}, 0);

let promise = new Promise(res => {
  console.log(3);
  resolve();
})
  .then(res => {
    console.log(4);
  })
  .then(res => {
    console.log(5);
  });

console.log(6);

// 1 3 6 4 5 2
```

## Node 环境

在 Node 中，事件循环表现出的状态与浏览器中大致相同。不同的是 Node 中有一套自己的模型。Node 中事件循环的实现是依靠的 [libuv](https://github.com/libuv/libuv) 引擎。我们知道 Node 选择 Chrome V8 引擎作为 JavaScript 解释器，V8 引擎将 JavaScript 代码分析后去调用对应的 Node API，而这些 API 最后则由 libuv 引擎驱动，执行对应的任务，并把不同的事件放在不同的队列中等待主线程执行。 因此实际上 Node 中的事件循环存在于 libuv 引擎中。

实际上，node在应用层面属于单线程，但底层其实通过libuv维护了一个阻塞I/O调用的线程池。

### 异步I/O

在Node中，JS是单线程没有错，但内部完成的I/O工作另有线程池，使用一个主进程和多个I/O线程模拟异步I/O

当主线程发起I/O调用时，I/O操作会被放在I/O线程来执行，主线程继续执行下面的任务，在I/O线程完成操作后会带着数据通知主线程发起回调。

### 事件循环

**事件循环**是Node的执行模型，正是这种模型使得回调函数非常普遍。

在进程启动时，Node便会创建一个类似while(true)的循环，执行每次循环的过程就是判断有没有待处理的事件，如果有，就取出事件及其相关的回调并执行他们，然后进入下一个循环。如果不再有事件处理，就退出进程。

![clipboard.png](https://segmentfault.com/img/bV3CVO?w=410&h=447)

Event loop是一种程序结构，是实现异步的一种机制。Event loop可以简单理解为：

1. 所有的任务都在主线程上执行，主线程会有一个执行栈(execution context stack)
2. 主线程之外，还有一个“任务队列”。系统会把异步任务放在这个“任务队列”之中，然后继续执行主线程上的任务。
3. 一旦“执行栈”上的任务执行完并且清空了，系统就会读取任务队列，将异步任务推入执行栈中。
4. 主线程不断重复上面的步骤。

**[Node中事件循环](https://link.segmentfault.com/?url=https%3A%2F%2Fnodejs.org%2Fen%2Fdocs%2Fguides%2Fevent-loop-timers-and-nexttick%2F)阶段解析:**

```js
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<──connections───     │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

每个阶段都有一个FIFO的回调队列要执行，而且每个阶段有自己的特殊之处。即当event loop进入某个阶段后，会执行该阶段特定的操作，然后才会执行这个阶段的队列里的回调。当队列被执行完，或者执行的回调数量达到上限后，event loop会进入下个阶段。

**Phases Overview 阶段总览**

- **timers**: 该阶段执行`setTimeout`和`setInterval`的回调
- **I/O callbacks**: 执行几乎所有的回调，除了`close callbacks`、`setTimeout()`、`setInterval()`、`setImmediate()`的回调。
- **idle, prepare:** 仅内部使用。
- **poll:** 获取新的I/O事件；node会在适当条件下阻塞在这里。

- **check:** 执行`setImmediate()`设定的回调。
- **close callbacks:** 执行比如socket.on('close', ...)的回调。

#### 1. timers

一个`timer`指定一个下限时间而不是准确时间，定时器`setTimeout()`和`setInterval()`在达到这个下限时间后执行回调。在指定的时间过后，timers会尽早的执行回调，但是系统调度或者其他回调的执行可能会延迟它们。
从技术上来说，`poll`阶段控制timers什么时候执行，而执行的具体位置在timers。
下限的时间有一个范围：`[1, 2147483647]`，如果设定的时间不在这个范围，将被设置为1。

#### 2. I/O callbacks

执行除了`close callbacks`、`setTimeout()`、`setInterval()`、`setImmediate()`回调之外几乎所有回调，比如说TCP连接发生错误。

#### 3. idle, prepare

系统内部的一些调用

#### 4. poll

这是最复杂的一个阶段。`poll`会检索新的`I/O events`,会在合适的事件阻塞，等待回调被加入。

poll阶段有两个主要的功能：一是执行下限时间已经达到的timers的回调，一是处理poll队列里的事件。
注：Node很多API都是基于事件订阅完成的，这些API的回调应该都在poll阶段完成。

当事件循环进入poll阶段：

- poll队列不为空时，事件循环肯定是先遍历队列并同步执行回调，直到队列清空或执行回调数达到系统上限。
- poll队列为空的时候
  - 如果代码已经被`setImmediate()`设定了回调，那么事件循环直接结束`poll`阶段进入`check`阶段来执行`check`队列里的回调。
  - 如果代码没有被`setImmediate`设定回调
    - 如果有被设定的timers，那么此时事件循环会检查timers，如果有一个或多个timers下限时间已经到达，那么事件循环将绕回timers阶段，并执行timers的有效回调队列。
    - 如果没有被设定timers，这个时候事件循环是**阻塞**在poll阶段等待事件回调被加入poll队列。

Node的很多API都是基于事件订阅完成的，比如fs.readFile，这些回调应该都在`poll`阶段完成。

#### 5. check

`setImmediate()`在这个阶段执行。

这个阶段允许在`poll`阶段结束后立即执行回调。如果`poll`阶段空闲，并且有被`setImmediate()`设定的回调，那么事件循环直接跳到`check`执行而不是阻塞在poll阶段等待poll 事件们 (poll events)被加入。

**注意**： 如果进到了poll阶段，setImmediate()具有最高优先级，只要`poll`队列为空且注册了setImmediate()，无论是否有`timers`达到下限时间，setImmediate()的代码都先执行。

#### 6. close callback

如果一个socket或handle被突然关掉（比如`socket.destroy()`），`close`事件将在这个阶段被触发，否则将通过process.nextTick()触发。

### 请求对象

对于Node中的异步I/O调用而言，回调函数不由开发者来调用，从JS发起调用到I/O操作完成，存在一个中间产物，叫**请求对象**。

在JS发起调用后，JS调用Node的核心模块，核心模块调用C++内建模块，內建模块通过libuv判断平台并进行系统调用。在进行系统调用时，从JS层传入的方法和参数都被封装在一个请求对象中，请求对象被放在线程池中等待执行。JS立即返回继续后续操作。

### 执行回调

在线程可用时，线程会取出请求对象来执行I/O操作，执行完后将结果放在请求对象中，并归还线程。
在事件循环中，I/O观察者会不断的找到线程池中已经完成的请求对象，从中取出回调函数和数据并执行。

![clipboard.png](https://segmentfault.com/img/bV3CWQ?w=767&h=717)

跑完当前执行环境下能跑完的代码。每一个事件消息都被运行直到完成为止，在此之前，任何其他事件都不会被处理。这和C等一些语言不通，它们可能在一个线程里面，函数跑着跑着突然停下来，然后其他线程又跑起来了。JS这种机制的一个典型的坏处，就是当某个事件处理耗时过长时，后面的事件处理都会被延后，直到这个事件处理结束，在浏览器环境中运行时，可能会出现某个脚本运行时间过长，页面无响应的提示。Node环境则可能出现大量用户请求被挂起，不能及时响应的情况。

## 非I/O的异步API

Node中除了异步I/O之外，还有一些与I/O无关的异步API，分别是：`setTimeout()`、`setInterval()`、`process.nextTick()`、`setImmediate()`，他们并不是像普通I/O操作那样真的需要等待事件异步处理结束再进行回调，而是出于定时或延迟处理的原因才设计的。

### `setTimeout()`与`setInterval()`

这两个方法实现原理与异步I/O相似，只不过不用I/O线程池的参与。
使用它们创建的定时器会被放入`timers`队列的一个红黑树中，每次事件循环执行时会从相应队列中取出并判断是否超过定时时间，超过就形成一个事件，回调立即执行。
所以，和浏览器中一样，这个并不精确，会被长时间的同步事件阻塞。

![clipboard.png](https://segmentfault.com/img/bV3CWW?w=581&h=427)

### `setImmediate()`

setImmediate()是放在`check`阶段执行的，实际上是一个特殊的timer，跑在event loop中一个独立的阶段。它使用`libuv`的API来设定在 `poll` 阶段结束后立即执行回调。

来看看这个例子：

```javascript
setTimeout(function() {
  console.log('setTimeout')
}, 0)
setImmediate(function() {
  console.log('setImmediate')
})                                // 输出不稳定
```

setTimeout与setImmediate先后入队之后，首先进入的是`timers`阶段，如果我们的机器性能一般或者加入了一个同步长耗时操作，那么进入`timers`阶段，1ms已经过去了，那么setTimeout的回调会首先执行。
如果没有到1ms，那么在`timers`阶段的时候，超时时间没到，setTimeout回调不执行，事件循环来到了`poll`阶段，这个时候队列为空，此时有代码被setImmediate()，于是先执行了setImmediate()的回调函数，之后在下一个事件循环再执行setTimemout的回调函数。

```javascript
setTimeout(function() {
  console.log('set timeout')
}, 0)
setImmediate(function() {
  console.log('set Immediate')
})
for (let i = 0; i < 100000; i++) {}           // 可以保证执行时间超过1ms
// 稳定输出： setTimeout    setImmediate
```

**再一个栗子：**

```javascript
const fs = require('fs')
fs.readFile('./filePath.js', (err, data) => {
  setTimeout(() => console.log('setTimeout') , 0)
  setImmediate(() => console.log('setImmediate'))
  console.log('开始了')
  for (let i = 0; i < 100000; i++) {}        
})                                         // 输出 开始了 setImmediate setTimeout
```

这里我们就会发现，setImmediate永远先于setTimeout执行。
fs.readFile的回调是在`poll`阶段执行的，当其回调执行完毕之后，setTimeout与setImmediate先后入了`timers`与`check`的队列，继续到`poll`，`poll`队列为空，此时发现有setImmediate，于是事件循环先进入`check`阶段执行回调，之后在下一个事件循环再在`timers`阶段中执行setTimeout回调，虽然这个setTimeout已经到了超时时间。

**再来个栗子：**
同样的，这段代码也是一样的道理：

```javascript
setTimeout(() => {
    setImmediate(() => console.log('setImmediate') );
    setTimeout(() => console.log('setTimeout') , 0);
}, 0);
```

以上的代码在`timers`阶段执行外部的setTimeout回调后，内层的setTimeout和setImmediate入队，之后事件循环继续往后面的阶段走，走到`poll`阶段的时候发现队列为空，此时有代码被setImmedate()，所以直接进入`check`阶段执行响应回调（注意这里没有去检测`timers`队列中是否有成员到达超时事件，因为setImmediate()优先）。之后在下一个事件循环的`timers`阶段中再去执行相应的回调。

### `process.nextTick()`与`Promise`

对于这两个，我们可以把它们理解成一个微任务。也就是说，它们其实不属于事件循环的一部分。

有时我们想要立即异步执行一个任务，可能会使用延时为0的定时器，但是这样开销很大。我们可以换而使用`process.nextTick()`，它会将传入的回调放入`nextTickQueue`队列中，~~下一轮Tick之后取出执行~~，不管事件循环进行到什么地步，都在当前**执行栈**的操作结束的时候调用，参见[Nodejs官网](https://link.segmentfault.com/?url=https%3A%2F%2Fnodejs.org%2Fen%2Fdocs%2Fguides%2Fevent-loop-timers-and-nexttick%2F%23process-nexttick)。

process.nextTick方法指定的回调函数，总是在**当前执行队列**的尾部触发，多个process.nextTick语句总是一次执行完（不管它们是否嵌套），递归调用process.nextTick，将会没完没了，主线程根本不会去读取事件队列，导致**阻塞后续调用**，直至达到最大调用限制。

相比于在定时器中采用红黑树的操作时间复杂度为0(lg(n))，而`process.nextTick()`的时间复杂度为0(1)，相比之下更高效

来举一个复杂的栗子，这个栗子搞懂基本上就全部理解了：

```javascript
setTimeout(() => {
  process.nextTick(() => console.log('nextTick1'))
  
  setTimeout(() => {
    console.log('setTimout1')
    process.nextTick(() => {
      console.log('nextTick2')
      setImmediate(() => console.log('setImmediate1'))
      process.nextTick(() => console.log('nextTick3'))
    })
    setImmediate(() => console.log('setImmediate2'))
    process.nextTick(() => console.log('nextTick4'))
    console.log('sync2')
    setTimeout(() => console.log('setTimout2'), 0)
  }, 0)
  
  console.log('sync1')
}, 0) 
// 输出： sync1 nextTick1 setTimout1 sync2 nextTick2 nextTick4 nextTick3 setImmediate2 setImmediate1 setTimout2
```

### 结论

1. `process.nextTick()`，效率最高，消费资源小，但会阻塞CPU的后续调用；
2. `setTimeout()`，精确度不高，可能有延迟执行的情况发生，且因为动用了红黑树，所以消耗资源大；
3. `setImmediate()`，消耗的资源小，也不会造成阻塞，但效率也是最低的。

### 阶段

- 外部输入数据
- 轮询阶段（Poll）：等待新的 I/O 事件，Node 在一些特殊情况下会阻塞在这里
- 检查阶段（Check）：`setImmediate` 的回调会在这个阶段执行
- 关闭事件回调阶段（Close Callback）
- 定时器检测阶段（Timer）：这个阶段执行定时器队列中的回调
- I/O 事件回调阶段（I/O Callbacks）：这个阶段执行几乎所有的回调，但是不包括 `close` 事件、定时器和 `setImmediate()` 的回调
- 闲置阶段（Idle Prepare）：仅在内部使用，不必理会

## 缺陷

当一个消息需要太长时间才能处理完毕时，Web 应用就无法处理用户的交互，例如点击或滚动。浏览器用程序需要过长时间运行的对话框来缓解这个问题。一个很好的做法是缩短消息处理，并在可能的情况下将一个消息裁剪成多个消息。