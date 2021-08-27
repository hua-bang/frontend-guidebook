---
nav:
  title: 设计模式
  order: 5
group:
  title: 创造型
  order: 2
title: 单例模式
order: 1
---
# 单例模式

定义: 保证一个类仅有一个实例，并提供一个访问它的全局访问点

特点： 唯一 & 全局访问

## 功能

- 定义私有命名空间
- 管理代码库模块
- 创建惰性实例
- 管理静态变量

产生一个类的唯一实例

### 优点

1. 提供唯一实例
2. 节省系统资源和性能
3. 免对共享资源的多重占用

### 缺点

1. 扩展性差
2. 指责过重

### 场景

消息弹窗，登录弹窗等

```js
function SingleTon(fn) {
  let result;
  return function(...args) {
    const context = this;
    return result || (result = fn.call(context, ...args));
  }
}
```

