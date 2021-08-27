---
nav:
  title: 计算机网络
  order: 4
group:
  title: 运输层协议
  order: 3
title: UDP
order: 3
---
# UDP 用户数据报协议

用户数据报协议(User Datagram Protocol, UDP),是一种简单的面向数据报的传输层协议。

在TCP/IP模型中，UDP为网络层和应用层提供了简单的接口。UDP只提供数据的不可靠传递，一旦应用进程发给网络层的数据发送出去，就不保留数据备份（UDP也被认为是不可靠的数据报协议）。UDP 在 IP 数据报的头部仅仅加入了复用和数据校验（字段）。

## 特点

- UDP无需建立连接（减少延迟）
- UDP尽努力提供交付，即不保证可靠交付。
- UDP首部开销小，8字节。
- UDP没有拥塞控制，因此网络出现拥塞并不会使得源主机发送速率降低。
- UDP支持一对一，一对多，多对多的交互通信。

## 应用

- 域名系统(DNS)
- 简单网络管理协议（SNMP）
- 动态主机配置协议(DHCP)
- 路由信息协议（RIP）
- 自举协议（BOOTP）
- 简单文件传输协议（TFTP）

## 结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190411122017924.png)

- **源端口号**：`占用16位`，报文来自哪个端口
- **目的端口号**：占用16位，报文发送到哪个端口
- **UDP长度**：占用16位，UDP报文的字节长度（包括首部和数据）
- **检验和**：占用16位，检验UDP首部和数据部分的正确性。

---

**参考资料**

- [UDP](https://tsejx.github.io/javascript-guidebook/computer-networks/computer-network-architecture/transport-layer-protocol#udp)