---
nav:
  title: Babel
  order: 7
group:
  title: 介绍
  order: 1
title: Generate
order: 7
---

# Generator & SourceMap 的奥秘

`AST`转换完之后就到了`generate`阶段。`generate`是怎么生成代码和`sourcemap`的呢? `sourcemap`有啥作用呢？

本节就来探索一下`generate`的奥秘。

## Generate

`generate`是把`AST`打印成字符串，是一个从根节点递归打印的过程。对不同的`AST`节点做不同的处理。在这个过程中把抽象语法树中省略的一些分隔符重新加回来。

## sourcemap

我们知道可以在`generate`的时候选择是否生成`sourcemap`，那为什么要生成`sourcemap`呢？

### sourcemap的作用

babel对源码进行了修改，生成的目标代码改动可能是比较大的。那我们怎么调试代码，所以需要的是一种关联源码的方式，就是`sourcemap`。

- 调试代码到定位到源代码
- 线上报错定位到源代码

## 总结

学完这一节之后，我们知道了 AST 是怎么生成目标代码和 sourcemap的，加上前两节的内容，把整个 babel 的编译流程串联了起来。