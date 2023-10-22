### 描述

命令模式（Command Pattern）是一种行为设计模式，用于将请求封装为一个对象，从而使用户可以使用不同的请求参数化其他对象，并支持请求的排队、记录和撤销操作。

### 作用

![image.png](https://raw.githubusercontent.com/hua-bang/assert-store/master/20231022215139.png)

解开耦合，请求者和接受者。

### **优点**

1. 解耦调用者和接收者。
2. 增加了代码的灵活性。
3. 支持撤销和重做操作。

### **缺点**

1. 可能导致系统有过多的具体命令类。

### **场景**

1. GUI 按钮和菜单项：每个按钮都是一个命令。
2. 撤销和重做功能：通过命令对象来实现。
3. 宏操作：一系列命令可以组合成一个复杂的命令。

### 实现

```jsx
// Command interface
interface Command {
  execute(): void;
  undo(): void;
}

// ConcreteCommand for Light
class LightOnCommand implements Command {
  constructor(private light: Light) {}

  execute() {
    this.light.turnOn();
  }

  undo() {
    this.light.turnOff();
  }
}

class LightOffCommand implements Command {
  constructor(private light: Light) {}

  execute() {
    this.light.turnOff();
  }

  undo() {
    this.light.turnOn();
  }
}

// Receiver
class Light {
  turnOn() {
    console.log("Light is ON");
  }

  turnOff() {
    console.log("Light is OFF");
  }
}

// Invoker with Undo capability
class RemoteControl {
  private commandHistory: Command[] = [];

  setCommand(command: Command) {
    this.commandHistory.push(command);
    command.execute();
  }

  undoCommand() {
    if (this.commandHistory.length > 0) {
      const lastCommand = this.commandHistory.pop();
      if (lastCommand) {
        lastCommand.undo();
      }
    }
  }
}

// Client code
const light = new Light();
const lightOn = new LightOnCommand(light);
const lightOff = new LightOffCommand(light);

const remote = new RemoteControl();

remote.setCommand(lightOn);  // Output: Light is ON
remote.setCommand(lightOff); // Output: Light is OFF

remote.undoCommand();  // Output: Light is ON
remote.undoCommand();  // Output: Light is OFF
```

### 参考链接

- [https://chat.openai.com/share/898d26b6-36ac-43a0-a2b6-c68fa4faf3c0](https://chat.openai.com/share/898d26b6-36ac-43a0-a2b6-c68fa4faf3c0)
