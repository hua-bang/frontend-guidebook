---
nav:
  title: 读源码
  order: 8
group:
  title: 源码系列
  order: 1
title: 源码分析，Vue2中data和methods直接通过this访问的原理
order: 6
---

# Vue2中data和methods直接通过this访问的原理

## 前言
> 文章中的`Vue`均指的是`Vue2`版本。（笔者使用的`Vue`版本是[`2.7.0-alpha.4`](https://github.com/vuejs/vue/tree/v2.7.0-alpha.4)）
在`Vue`中, 我们经常用下面的写法定义组件的`methods`以及`data`, 从而定义一个`Vue`组件。
```js
const app = new Vue({
    el: '#app',
    data() {
        return {
            count: 0
        }
    },
    methods: {
        add() {
            this.count++;
        }
    }
});
```
这对于我们来说是在熟悉不过了。不过，为什么我们能在`Vue`内部定义的函数中通过`this`直接访问该`Vue`实例上的`data`和`methods`中定义的属性。

要回答这个问题，我们可以深入`Vue`的[源码](https://github.com/vuejs/vue), 再次声明，这里使用的版本是[`v2.7.0-alpha.4`](https://github.com/vuejs/vue/tree/v2.7.0-alpha.4)来看看。
(tips: 不同的版本可能会有所差异，`v2.7.0-alpha.4`目前应该已经使用了`ts`的写法)。

## 搭建调试环境

### 克隆代码
```shell
git clone https://github.com/vuejs/vue.git
```
下载到源码后，我们来首先看看`Readme.md`,接着我们看看`.github/CONTRIBUTING.md`, 阅读文档，来开始我们的开发环境。

### 安装依赖
`Vue`项目使用了`pnpm`, 进行开发，可以了解一下[pnpm](https://pnpm.io/)。
```shell
pnpm i
```
安装后，我们根据脚本`npm run dev`, 实质上是运行下方的代码
```shell
rollup -w -c scripts/config.js --environment TARGET:full-dev
```
这个时候，每当你改变了`src`下的源代码，`rollup`会进行检测，并重新打包，以保证最新产物。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0beb4c970cb544109d73ac23204227d4~tplv-k3u1fbpfcp-watermark.image?)

### 运行产物

>上方我们已经得到了产物，那我们怎么使用呢？

在这里，我们使用了[`http-server`](https://www.npmjs.com/package/http-server)去手动启动一个服务器，然后在对应的代码文件中引用即可。

#### 新建测试文件

我们在`examples`下新建文件`index.html`,编写如下代码
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <div id="app">
    The count is {{ count }}
    <button @click="count++">add</button>
  </div>
</body>
<script src="../dist/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data() {
      return {
        count: 1
      }
    },
    methods: {
      log() {
        console.log('create');
      }
    },
    mounted() {
      console.log('mounted');
      this.log();
      console.log(this.count);
    }
  })
</script>
</html>
```

#### `http-server`运行

接者，我们来到项目目录， 使用`http-server`
```shell
http-server -p 8082 .  -c-1
```

访问`http://127.0.0.1:8082/examples/`, 得到如下页面。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63bbb2df5beb424da25db4dff6029bad~tplv-k3u1fbpfcp-watermark.image?)

同时，我们去`src/core/index.ts`中修改源码。

```diff
import Vue from './instance/index'
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'

+ console.log(Vue);

initGlobalAPI(Vue)
```
看到下图，则说明我们可以边修改边调试源码了。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49b4b947c52f40eebd3887e803a46065~tplv-k3u1fbpfcp-watermark.image?)


但上面的准备，足够了吗？

### 生成`sourcemap`

实际上，上面，我们调试的仅仅时产物，其实不是特别方便，这里都是`js`的打包产物，对其进行`debug`，其实并没能回溯到我们的源代码。

所以，我们需要开启打包时生成`sourcemap`，追述到我们的源代码, 修改`package.json`中`dev`命令。

```json
{
    // ...
    “script”: {
        "dev": "rollup -w -c scripts/config.js --environment TARGET:full-dev --sourcemap",
        // ....
    }
    // ...
}
```

重新跑`npm run dev`, 得到下方页面。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1602a24a2b6c478aa6ee57ecc566c1a4~tplv-k3u1fbpfcp-watermark.image?)

并且，我们打上断点，如果进入调试模式，那么我们的调试环境基本完成了。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea4fb480c9734c77a289535346896f3e~tplv-k3u1fbpfcp-watermark.image?)

那么，接下来到了读源码的时候了。

## 分析源码

由于笔者对于`vue`有过一段了解，对于`data`和`methods`的话，我们猜测一下，应该是在`init`的阶段。
我们来看`vue`中的`__init`, 这里`__init`的函数时在`initMixin`时候定义的。

跟着`debug`, 我们来到了`initState(vm)`函数中，这里是初始化了`data`和`methods`。

![initState.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db9f65fac62f403b98423d9dfd3770b7~tplv-k3u1fbpfcp-watermark.image?)

这里，其实是初始化了`props`,`data`, `methods`等。这里我们主要看`initMethods`和`initData`,看看这两个过程做了啥？

### `initMethods`

下面就是`initMethods`,即`Vue`中初始过程中对`methods`属性的操作。

![initMethods.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd7969f365f6431f9718480e3d80054c~tplv-k3u1fbpfcp-watermark.image?)

这里我们看主要逻辑(`__DEV__`中的逻辑，主要是避免`methods`中的`key`和`props`,`保留字`冲突，报警告)。

```ts
function initMethods(vm: Component, methods: Object) {
  const props = vm.$options.props
  for (const key in methods) {
    // ... code  __DEV__ 环境下 处理逻辑
    
    // noop 为空函数 bind做了一层`polyfill`, 做兼容
    vm[key] = typeof methods[key] !== 'function' ? noop : bind(methods[key], vm)
  }
}
```
这里，我们就能在`Vue`实例中的函数通过`this`访问`options`中定义的`methods`中的属性了。

接着，我们看看`initData`.

### `initData`

下方便是`initData`的实现。

![initData.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ddbb073170545b687ef3b1f82d2b5e4~tplv-k3u1fbpfcp-watermark.image?)

首先，由于`options`中的`data`可以为函数/对象。在这里，我们要拿到实际的`data`对象，所以就有了

```ts
let data: any = vm.$options.data
data = vm._data = isFunction(data) ? getData(data, vm) : data || {}
```

接着，我们看看核心代码
```ts
function initData(vm: Component) {
  // get data
  // ...code
  // proxy data on instance
  // 得到`data`所有键
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  // 遍历代码，如果`data`中的`key`和`methods`/`props`中的键是否冲突，只有不冲突，并不为保留关键字的才会，使用执行proxy(vm, `_data`, key)
  while (i--) {
    const key = keys[i]
    if (__DEV__) {
      if (methods && hasOwn(methods, key)) {
        warn(`Method "${key}" has already been defined as a data property.`, vm)
      }
    }
    if (props && hasOwn(props, key)) {
      __DEV__ &&
        warn(
          `The data property "${key}" is already declared as a prop. ` +
            `Use prop default value instead.`,
          vm
        )
    } else if (!isReserved(key)) {
      proxy(vm, `_data`, key)
    }
  }
  // observe data
  // ... code
}
```
整个流程如下：

1. `get Data`, 得到传入的`data`, 获取`data`对象，并赋值到`vm._data`上。
2. 对`data`对象进行遍历，判断`key`是否和`methods`和`props`冲突。
3. 如果不冲突，再执行`proxy data`。

接着，我们看看`proxy`函数。

![proxy.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9c54cfa9db142bf92380a840432dd90~tplv-k3u1fbpfcp-watermark.image?)

实质上，是使用了[defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty), 拦截了`get`, `set`, 从而实现代理。

这样子，当我们访问对应代理的`key`时候，会访问`vm._data[key]`, 这样子我们就能再组件实例中的函数访问直接访问对应的变量了。

## 总结

阅读源码能让我们学习一些不错的设计思想，以及部分编程规范。这里我们通过阅读`vue`的源码，了解了以下的知识点：
1. `Vue2`的源码调试。
2. `Vue`中`init`阶段中的`initMethods`以及`initData`的过程。

`学而知不足`，上方的知识也是`Vue`的一小部分，也能让我们学习到部分知识点。通过阅读源码，学习别人编码思想，也是一种益处。

## 参考
- [Vue源码](https://github.com/vuejs/vue/tree/v2.7.0-alpha.4)
- [为什么 Vue2 this 能够直接获取到 data 和 methods ? 源码揭秘！](https://juejin.cn/post/7010920884789575711)