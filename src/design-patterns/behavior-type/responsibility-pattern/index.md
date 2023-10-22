### 描述

![image.png](https://raw.githubusercontent.com/hua-bang/assert-store/master/20231022215021.png)

职责链模式（Chain of Responsibility Pattern）是一种行为设计模式，用于将请求沿一个处理对象链进行传递。每个处理对象都有机会处理该请求，或者将其传递给链中的下一个对象。

1. **Handler**: 抽象处理者，定义了一个处理请求的接口。
2. **ConcreteHandler**: 具体处理者，实现抽象处理者定义的接口。
3. **Client**: 客户端，发送请求。

### 作用

这样可以将请求的发送者和接收者解耦。

### **优点**

1. 解耦请求和处理。
2. 动态添加或删除责任：如果需要添加新的处理逻辑，只需添加一个新的具体处理者，并将其添加到现有的链中，而无需修改客户端或其他处理者的代码。
3. 易于维护和扩展：每个处理者只关注自己的处理逻辑，不关心其他处理者或整体流程，这使得系统更易于理解和维护。

### **缺点**

1. 性能问题：如果链太长，可能会影响性能。
2. 不保证请求一定会被处理。

### **场景**

1. **日志记录**: 职责链模式可以用于处理不同级别的日志消息，例如，DEBUG、INFO、ERROR 等。
2. **权限检查**: 在 Web 应用中，可以用职责链模式来按顺序执行一系列权限验证步骤，如身份验证、角色检查等。
3. **数据校验**: 在表单提交或 API 调用中，可以用职责链模式来进行多层次的数据验证，例如，基础验证、业务逻辑验证等。

### 实现

```jsx
// 定义处理者接口
interface Handler {
  setSuccessor(successor: Handler): void;
  handleRequest(request: string): void;
}

// 具体处理者1
class ConcreteHandler1 implements Handler {
  private successor: Handler | null = null;

  setSuccessor(successor: Handler) {
    this.successor = successor;
  }

  handleRequest(request: string) {
    if (request === 'R1') {
      console.log('Request R1 handled by Handler 1');
    } else if (this.successor) {
      this.successor.handleRequest(request);
    }
  }
}

// 具体处理者2
class ConcreteHandler2 implements Handler {
  private successor: Handler | null = null;

  setSuccessor(successor: Handler) {
    this.successor = successor;
  }

  handleRequest(request: string) {
    if (request === 'R2') {
      console.log('Request R2 handled by Handler 2');
    } else if (this.successor) {
      this.successor.handleRequest(request);
    }
  }
}

// 客户端代码
const handler1 = new ConcreteHandler1();
const handler2 = new ConcreteHandler2();

handler1.setSuccessor(handler2);

handler1.handleRequest('R1');  // Output: Request R1 handled by Handler 1
handler1.handleRequest('R2');  // Output: Request R2 handled by Handler 2
handler1.handleRequest('R3');  // Output: (No output, request not handled)
```

### 参考链接

- [https://chat.openai.com/share/898d26b6-36ac-43a0-a2b6-c68fa4faf3c0](https://chat.openai.com/share/898d26b6-36ac-43a0-a2b6-c68fa4faf3c0)
- [Wikipedia - Chain of Responsibility Pattern](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)
- [Refactoring Guru - Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility)
