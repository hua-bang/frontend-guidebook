---
nav:
  title: Babel
  order: 7
group:
  title: 概况
  order: 1
title: 介绍
order: 1
---

# Babel的介绍

## 前言

`babel`最开始叫6to5，意思是es6转es5，但后面随着es标准的推进，改名为`babel`。

`babel`是巴别塔的意思。

## babel的用途

### 转移esnext、typescript、flow的等到目标环境的js

这个是比较常见的功能，用来将esnext、typescript、flow的语法转成基于目标环境支持的语法的实现。并且可以支持api的`ployfill`。

### 一些特定用途的代码转换

`babel`是一个转移器，暴露了很多`api`。可以用这些`api`完成代码`parse`成`AST`,`AST`的代码的转换，以及目标代码的生成。

我们可以利用这些来完成一些特定用途的转换：
- 函数插桩
- 自动国际化
- default import 和 named imported

### 代码的静态分析

由于`babel`是可以将代码进行`parse`形成`AST`,可以用过`AST`的结构去了解代码，除了能转成新的目标代码，同时也可以用于分析代码。

- linter工具分析AST结构，对代码进行规范。
- api文档自动生成工具。
- type checker会根据从AST提取的信息，来检查类型的错误
- 压缩混淆工具。分析代码结构，删除死代码，变量名混淆等编译优化。
- js解释器，除了对AST进行各种信息的提取和检查之外，还可以直接执行AST。
  
## 总结
babel的名字来自巴别塔的典故，是一个js转译器。
- 可转化esnext、typescript等代码。
- 可做特定用途转换
- 可做静态分析

## 参考
- [Babel的介绍](https://juejin.cn/book/6946117847848321055/section/6956174385904353288)