---
nav:
  title: BOM
  order: 6
group:
  title: 游览器工作原理
  order: 1
title: 渲染机制
order: 1
---
# 渲染机制

游览器内核是支持游览器运行的最核心的程序。分为两个部分，一个渲染引擎，一个JS引擎。游览器引擎在不同的游览器中也不是都相同的。比如在 Firefox 中叫做 Gecko，在 Chrome 和 Safari 中都是基于 WebKit 开发的。本文我们主要介绍关于 WebKit 的这部分渲染引擎内容以及几个相关的问题。

![img](https://pic4.zhimg.com/80/v2-5d8ee9d00bcc898318d1c9a8bf6684b7_720w.jpg)

游览器工作大体流程图

![浏览器工作大致流程](https://tsejx.github.io/javascript-guidebook/static/browser-working-process.1ee79c8b.jpg)

## 工作流程

1. **游览器解析过程**
   - HTML / SVG / XHTML：渲染引擎通过三个 C++ 的类对应这三类文档，解析这三类文，件并构建 **DOM 树**（DOM Tree）
   - CSS：渲染引擎会解析外部的CSS以及内联的CSS样式，并构建CSS规则树
   - JavaScript：Js通过DOM API 和CSSOM Api操作DOM Tree和CSS Rule Tree
2. **构建渲染树（Rendering Tree）**
   - 解析完成之后，游览器会根据DOM树和CSS树规则生成**渲染树**
   - 渲染树并不等同于 DOM 树，因为一些像 `<header>` 或 `display: none` 的东西就没必要放到渲染树中
   - CSS 的 Rule Tree 主要是为了完成匹配并把 CSS Rule 附加至渲染树上的每个 Element 上。然后，计算每个渲染对象的位置，这通常是 布局（Layout） 和 重排（Reflow） 过程中发生。
   - 一旦渲染树构建完成，浏览器会把树里面的内容绘制在屏幕上
3. **布局（Layout）**：渲染树构建好之后，将会执行布局过程，它将确定每个节点在屏幕上的确切坐标
4. **绘制（Paint）**：再下一步就是绘制，即遍历渲染树，并使用渲染引擎绘制每个节点
5. **渲染层合并（Composite）**：页面中 DOM 元素的绘制是在多个层上进行的，在每个层上完成绘制过程之后，浏览器会将所有层按照合理的顺序合并成一个图层，然后在屏幕上呈现
6. 最后通过调用操作系统 NativeGUI API 进行绘制

值得注意的是，这个过程是逐步完成的。为了更好的加载以及体验，渲染引擎会尽可能早地将内容呈现在屏幕上，并不会等到所有HTML解析完成后再去构建和布局渲染树。它是解析完一部分内容就显示一部分内容，同时，可能还在通过网络下载其余内容。

## 问题

### 渲染过程遇到JS文件如何处理

首先，我们得看一下这个脚本是如果引入地，如果是通过async，或者defer的话，另作讨论，这里只讨论默认的引入。

在构建DOM渲染过程中，HTML如果遇到了JavaScript，则会暂停DOM的渲染，将控制权交给JavaScript引擎。等这个脚本执行完后，再继续构建DOM。

也就是说，如果你想首屏渲染的越快，就越不应该在首屏就加载 JS 文件，这也是都建议将 script 标签放在 body 标签底部的原因。当然在当下，并不是说 script 标签必须放在底部，因为你可以给 script 标签添加 defer 或者 async 属性（下文会介绍这两者的区别）。

JS文件不只是阻塞DOM的构建，它会导致CSSOM也阻塞DOM的构建。

原本的CSSOM和DOM是并行的，不互相影响的，但一旦引入JavaScript，CSSOM也开始阻塞DOM的构建，只有CSSOM构建完毕后，DOM再恢复DOM构建。

这是什么情况？

因为JavaScript不只是改变DOM，还可以更改样式，也就是可以操作CSSOM。但JavaScript中想访问CSSOM并更改它，那么在执行JavaScript时，必须要能拿到完整的CSSOM。所以就导致了一个现象，如果浏览器尚未完成CSSOM的下载和构建，而我们却想在此时运行脚本，那么浏览器将延迟脚本执行和DOM构建，直至其完成CSSOM的下载和构建。也就是说，**在这种情况下，浏览器会先下载和构建CSSOM，然后再执行JavaScript，最后在继续构建DOM**。

### 回流和重绘

![Webkit主流程实现](https://tsejx.github.io/javascript-guidebook/static/webkit-workflow.b713c1bf.png)

我们知道，当网页生成的时候，至少会渲染一次。在用户访问的过程中，还会不断重新渲染。重新渲染会重复上图中的第四步(回流)+第五步(重绘)或者只有第五个步(重绘)。

- **重绘**：当我们的文档的一些元素需要更新属性，但这些属性并不影响布局，更多的是元素的外观，风格，我们称为重绘。
- **回流**：当我们的文档中的元素，发生了元素的规模尺寸，布局等改变，这个时候引起元素的重新构建，我们称为回流。

**回流一定发生重绘，重绘不一定引发回流。**重绘和回流会在我们设置节点样式时频繁出现，同时也会很大程度上影响性能。回流所需的成本比重绘高的多，改变父节点里的子节点很可能会导致父节点的一系列回流。

#### 常见的回流属性

任何会改变元素几何信息(元素的位置和尺寸大小)的操作，都会触发回流，

- 添加或者删除可见的DOM元素；
- 元素尺寸改变——边距、填充、边框、宽度和高度
- 内容变化，比如用户在input框中输入文字
- 浏览器窗口尺寸改变——resize事件发生时
- 计算 offsetWidth 和 offsetHeight 属性
- 设置 style 属性的值

#### 常见引起重绘属性和方法

![img](https://pic4.zhimg.com/80/v2-be336593a1da8b4f65858af8aa2d761f_720w.jpg)

#### 如何减少回流、重绘

- 使用 transform 替代 top
- 使用 visibility 替换 display: none ，因为前者只会引起重绘，后者会引发回流（改变了布局）
- 不要把节点的属性值放在一个循环里当成循环里的变量。
- 不要使用 table 布局，可能很小的一个小改动会造成整个 table 的重新布局
- 动画实现的速度的选择，动画速度越快，回流次数越多，也可以选择使用 requestAnimationFrame
- CSS 选择符从右往左匹配查找，避免节点层级过多
- 将频繁重绘或者回流的节点设置为图层，图层能够阻止该节点的渲染行为影响别的节点。比如对于 video 标签来说，浏览器会自动将该节点变为图层。

### async和defer的作用是什么？有什么区别?

![图片描述](https://segmentfault.com/img/bVCBBR)

其中蓝色线代表JavaScript加载；红色线代表JavaScript执行；绿色线代表 HTML 解析。

#### 情况1`<script src="script.js"></script>`

没有defer和async，游览器立即加载并执行指定的脚本，并且停止DOM的渲染，也就是说不等待后续载入的文档元素，读到就加载并执行。

#### 情况2`<script async src="script.js"></script>` (**异步下载**)

async 属性表示异步执行引入的 JavaScript，与 defer 的区别在于，如果已经加载好，就会开始执行——无论此刻是 HTML 解析阶段还是 DOMContentLoaded 触发之后。需要注意的是，这种方式加载的 JavaScript 依然会阻塞 load 事件。换句话说，async-script 可能在 DOMContentLoaded 触发之前或之后执行，但一定在 load 触发之前执行。

#### 情况3 `<script defer src="script.js"></script>`(**延迟执行**)

defer 属性表示延迟执行引入的 JavaScript，即这段 JavaScript 加载时 HTML 并未停止解析，这两个过程是并行的。整个 document 解析完毕且 defer-script 也加载完成之后（这两件事情的顺序无关），会执行所有由 defer-script 加载的 JavaScript 代码，然后触发 DOMContentLoaded 事件。

defer 与相比普通 script，有两点区别：**载入 JavaScript 文件时不阻塞 HTML 的解析，执行阶段被放到 HTML 标签解析完成之后。

在加载多个JS脚本的时候，async是无顺序的加载，而defer是有顺序的加载。

### 为什么操作 DOM 慢

因为 DOM 是属于渲染引擎中的东西，而 JS 又是 JS 引擎中的东西。当我们通过 JS 操作 DOM 的时候，其实这个操作涉及到了两个线程之间的通信，那么势必会带来一些性能上的损耗。操作 DOM 次数一多，也就等同于一直在进行线程之间的通信，并且操作 DOM 可能还会带来重绘回流的情况，所以也就导致了性能上的问题。

## 渲染页面时常见哪些不良现象？

**由于浏览器的渲染机制不同，在渲染页面时会出现两种常见的不良现象—-白屏问题和FOUS（无样式内容闪烁）**

FOUC：由于浏览器渲染机制（比如firefox），再CSS加载之前，先呈现了HTML，就会导致展示出无样式内容，然后样式突然呈现的现象；

白屏：有些浏览器渲染机制（比如chrome）要先构建DOM树和CSSOM树，构建完成后再进行渲染，如果CSS部分放在HTML尾部，由于CSS未加载完成，浏览器迟迟未渲染，从而导致白屏；也可能是把js文件放在头部，脚本会阻塞后面内容的呈现，脚本会阻塞其后组件的下载，出现白屏问题。

## 总结

- 浏览器工作流程：构建DOM -> 构建CSSOM -> 构建渲染树 -> 布局 -> 绘制。
- CSSOM会阻塞渲染，只有当CSSOM构建完毕后才会进入下一个阶段构建渲染树。
- 通常情况下DOM和CSSOM是并行构建的，但是当浏览器遇到一个script标签时，DOM构建将暂停，直至脚本完成执行。但由于JavaScript可以修改CSSOM，所以需要等CSSOM构建完毕后再执行JS。
- 如果你想首屏渲染的越快，就越不应该在首屏就加载 JS 文件，建议将 script 标签放在 body 标签底部。

---

**参考资料：**

- [深入浅出浏览器渲染原理](https://zhuanlan.zhihu.com/p/53913989)

- [渲染进程的内部机制](https://tsejx.github.io/javascript-guidebook/browser-object-model/browser-working-principle/workflow)