---
nav:
  title: babel
  order: 7
group:
  title: 概况
  order: 1
title: babel的api
order: 4
---

# Babel的API

学习完babel的编译流程和AST后，我们来学习下babel的api，通过这些api来操作AST。

## Babel的API有哪些

我们知道babel的编译流程分为三步，`parse`,`transform`,`generate`,每一步都会暴露对应的接口给我们。

- `parse`阶段有 `@babel/parse`,功能是将源码转成AST。
- `transform`阶段有`@babel/traverse`,可以遍历AST，并调用visitor函数修改AST，自然包括增删改，这时候就需要`@babel/types`,当批量创建AST的时候，可以使用`@babel/template`来简化`AST`的创建逻辑。
- `generate`阶段是把AST打印成代码字符串，并生成`sourceMap`,需要`@babel/generator`包.
- 中途遇到错误想打印代码位置的时候，使用 `@babel/code-frame` 包
- babel 的整体功能通过 `@babel/core` 提供，基于上面的包完成 babel 整体的编译流程，并实现插件功能。

## 总结

这一节我们了解了编译过程中各阶段的 api：

- `@babel/parser` 对源码进行 parse，可以通过 plugins、sourceType 等来指定 parse 语法
- `@babel/traverse` 通过 visitor 函数对遍历到的 ast 进行处理，分为 enter 和 exit 两个阶段，具体操作 AST 使用 path 的 api，还可以通过 state 来在遍历过程中传递一些数据
- `@babel/types` 用于创建、判断 AST 节点，提供了 xxx、isXxx、assertXxx 的 api
- `@babel/template` 用于批量创建节点
- `@babel/code-frame` 可以创建友好的报错信息
- `@babel/generator` 打印 AST 成目标代码字符串，支持 comments、minified、sourceMaps 等选项。
- `@babel/core` 基于上面的包来完成 babel 的编译流程，可以从源码字符串、源码文件、AST 开始。


