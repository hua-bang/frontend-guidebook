---
nav:
  title: 设计模式
  order: 5
group:
  title: 行为型
  order: 3
title: 观察者模式
order: 1
---
# 观察者模式

**观察者模式（Observer Pattern）**：定义对象间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。

所谓观察者模式，其实就是为了实现 **松耦合（Loosely Coupled）**。

举个例子，当数据有更新，如 `changed` 方法被调用，用于更新 `state` 数据，比如温度、气压等。

这些的问题是，如果向更新更多的信息，比如说湿度，那就要去修改 `changed` 方法的代码，这就是紧耦合的坏处。

对于观察者模式，我们仅仅维护一个可观察对象即可，即一个 Observable 实例，当有数据变更时，它只需维护一套观察者（Observer）的集合，这些 Observer 实现相同的接口，Subject 只需要知道，通知 Observer 时，需要调用哪个同一方法就好了。

## 概述

**解决问题**：一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作

**应用时机**：一个对象（目标对象）的状态发生改变，会对所有的依赖对象（观察者）发送通知。

**如何解决**：使用面向对象技术，可以将这种依赖关系弱化

**类比实例**：

- 拍卖的时候，拍卖师观察最高标价，然后通知给其他竞价者竞价。
- 面试的时候，结束之后每个面试官都会对我说：“请留下你的联系方式， 有消息我们会通知你”。 在这里“我”是订阅者， 面试官是发布者。所以我不用每天或者每小时都去询问面试结果， 通讯的主动权掌握在了面试官手上。而我只需要提供一个联系方式。

**优点**：

- 观察者和被观察者是 **抽象耦合** 的
- 建立一套触发机制

**缺点**：

- 如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间
- 如果观察者和观察目标存在循环依赖的话，那可能产生死循环。
- 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化

## 结构

观察者模式包含如下角色：

- Subject（目标）：知道它的通知对象，事件发生后会通知所有它知道的对象，提供添加删除观察者的接口。
- ConcreteSubject（具体目标）：被观察者具体的实例，存储观察者感兴趣的状态。
- Observer（观察者）：提供通知后的更新事件。
- ConcreteObserver（具体观察者）：被观察者具体的实例，存储观察者感兴趣的状态。

## 代码实现

```js
class Subject {
  constructor() {
    this.state = "";
    this.observers = [];
  }
  
  attack(observer) {
    this.observers.push(observer);
  }
  
  updateState(nextState) {
    this.state = nextState
    this.nofity(nextState);
  }
  
  notify(...args) {
    this.observers.forEach(observer => {
      observer.update(...args);
    });
  }
}

class Observer {
  constructor(updater) {
    this.update = updater;
  }
}
```

注：观察者模型和发布-订阅模型是有所区别的，不要混淆。

---

**参考资料**

- [观察者模式](https://tsejx.github.io/javascript-guidebook/design-patterns/behavioral/observer)