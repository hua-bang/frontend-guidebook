---
nav:
  title: 计算机网络
  order: 4
group:
  title: WebSocket
  order: 6
title: WebSocket-api
order: 3
---

# WebSocket API

## 概述

市面大部分的游览器都支持`ws`了，那我们如何在我们的游览器使用`ws`,这时候，我们来参考一下[`mdn`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket#%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)的文档吧。

> WebSocket 对象提供了用于创建和管理 WebSocket 连接，以及可以通过该连接发送和接收数据的 API。
>
## Constructor

```js
const ws = new WebSocket(url [, protocols]);
```

我们可以通过`WebSocket`的构造函数去得到一个`WebSocket`的对象。

### Syntax

```js
new WebSocket(url);
new WebSocket(url, protocols);
```

### Parameters

#### `url`
The URL to which to connect.

### `protocols` 可选
一个协议字符串或者一个包含协议字符串的数组。这些字符串用于指定子协议，这样单个服务器可以实现多个WebSocket子协议


## Events

`WebSocket`会支持我们去配置事件触发函数，我们可以使用`addEventListener`或者`onEvent`去配置触发函数，来监听下面的事件。

### [open](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/open_event)
当`ws`连接成功时进行触发。

### [close](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/close_event)
当`ws`断开连接时候从触发。

### [error](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/error_event)
当`ws`抛出异常的时候触发，我们在回调中能够获取相关的参数（报错原因）

### [message](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/API/WebSocket/message_event)
当`ws`接收到信息时候触发，我们能在回调中拿到对应的参数（信息）。

`例子`:

```tsx
import React, { useState, useEffect } from 'react';

function initWSConnect(fn?: (data: any) => void) {
  const url = 'ws://82.157.123.54:9010/ajaxchattest';

  // Connect to ws
  const socket = new WebSocket(url);

  // Connection opened
  socket.addEventListener('open', function (event) {
    socket.send('Hello Server!');
  });

  // Listen for messages
  socket.addEventListener('message', function (event) {
    console.log('Message from server ', event.data);
    fn && fn(event.data);
  });

  return socket;

  socket.addEventListener('error', (err) => {
    console.log(err)
  });

  socket.addEventListener('close', () => {
    console.log('close')
  });
}

const Demo = () => {

  const [value, setValue] = useState('');  
  const [messageList, setMessageList] = useState<string[]>([]);
  const [socket, setSocket] = useState<any>();

  const logFn = (data: any) => {
    setMessageList(prev => [...prev, data]);
  }

  const handleInputChange = (e) => {
    const { value: val } = e.target;
    setValue(val);
  }

  const send = () => {
    socket && socket.send(value);
    setValue('');
  }

  const connect = () => {
    const ws = initWSConnect(logFn);
    setSocket(ws);
  };

  return (
    <div style={{ width: '100%', display: 'flex' }}>
      <div style={{ flex: 1 }}>
        <h3>
          Client 
          { 
            socket ? 
              'connected' : (
                <>
                  no connect, please click 
                  <button style={{ marginLeft: '10px' }} onClick={connect}>connect</button>
                </>
              )
          }
        </h3>
        <div>
          <input value={value} onChange={handleInputChange} />
          <button onClick={send}>send</button>
        </div>
      </div>
      <div style={{ flex: 1 }}>
        <h3>Server</h3>
        <div>
          { messageList.map(item => (<div key={item}>{item}</div>)) }
        </div>
      </div>
    </div>
  );
}

export default Demo;
```

## Methods

`WebSocket API` 支持`send()` and `close()`

### [`send`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/send)

即发送信息，支持格式`字符串`和`二进制`。

```js
socket.send('hello, world');
```

### [`close`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/close)

即关闭当前的连接。

```js
socket.close();
```

## [常量](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket#常量)

| **Constant**           | **Value** |
| ---------------------- | --------- |
| `WebSocket.CONNECTING` | `0`       |
| `WebSocket.OPEN`       | `1`       |
| `WebSocket.CLOSING`    | `2`       |
| `WebSocket.CLOSED`     | `3`       |

## [属性](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket#属性)

- [`WebSocket.binaryType`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/binaryType)

  使用二进制的数据类型连接。

- [`WebSocket.bufferedAmount`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/bufferedAmount) 只读

  未发送至服务器的字节数。

- [`WebSocket.extensions`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/extensions) 只读

  服务器选择的扩展。

- [`WebSocket.onclose`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/close_event)

  用于指定连接关闭后的回调函数。

- [`WebSocket.onerror`](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/API/WebSocket/error_event)

  用于指定连接失败后的回调函数。

- [`WebSocket.onmessage`](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/API/WebSocket/message_event)

  用于指定当从服务器接受到信息时的回调函数。

- [`WebSocket.onopen`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/open_event)

  用于指定连接成功后的回调函数。

- [`WebSocket.protocol`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/protocol) 只读

  服务器选择的下属协议。

- [`WebSocket.readyState`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/readyState) 只读

  当前的链接状态。

- [`WebSocket.url`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/url) 只读

  WebSocket 的绝对路径。


## 参考
- [WebSocket](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket)