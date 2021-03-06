---
nav:
  title: 计算机网络
  order: 4
group:
  title: HTTP
  order: 4
title: HTTP3
order: 7
---
# HTTP3

## 发展历程

- HTTP/1.0 - 1996
- HTTP/1.1 - 1997 缓存、条件式请求、鉴权、FIFO Pipelining
- SPDY - 2012 强制压缩、多路复用、Pipeling、双向通信、优先级调用
- HTTP/2 - 2015 头部压缩、多路复用、Pipelining、Server push（解决 HTTP 队首阻塞）
- HTTP/3 - 2018 快速握手、可靠传输、有序交付（解决 TCP 队首阻塞）

| 应用层            | 传输层      | 网络层 |
| ----------------- | ----------- | ------ |
| HTTP              | TCP         | IP     |
| HTTP/2 + TLS/1.2+ | TCP         | IP     |
| HTTP/3 + TLS/1.3  | QUIC（UDP） | IP     |

🗑 TLS1.1- 由于安全因素，已经或将要废弃

TLS 1.2 当今的主流版本

TLS 1.3 简化了握手流程，增强了安全性

## 发展前景

- Pros
  - 在经常丢包的网络中表现更好
  - 更快的握手
  - 更少的延迟
- Cons
  - UDP 易受放大攻击
  - 内核处理 UDP 很慢
  - QUIC 太吃 CPU

## 性能优化如何转型

- HTTP/2 占比过三成
- HTTP/3 问世
- TLS 1.3 问世
- TLS 1.1/1.0 将废弃

性能优化的变化

- 协议层：更少的 TTL 和更细的资源粒度（更有利于缓存的命中）
- 浏览器层：更丰富的数据统计（浏览器提供的新 API）
- 在线优化：更强的缓存控制能力（QuickLink 技术实现 prefetch）
- 离线优化：走向 Zero Config

## 基于QUIC

QUIC 基于 UDP，但是UDP本身存在不稳定性等诸多问题，所以QUIC在UDP的基础上新增了很多功能，比如多路复用、0-RTT、使用 TLS1.3 加密、流量控制、有序交付、重传等等功能。优点诸多，参考[这里](https://link.juejin.cn?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fbb3eeb36b479)：

- **避免包阻塞：** 多个流的数据包在TCP连接上传输时，若一个流中的数据包传输出现问题，TCP需要等待该包重传后，才能继续传输其它流的数据包。但在基于UDP的QUIC协议中，不同的流之间的数据传输真正实现了相互独立互不干扰，某个流的数据包在出问题需要重传时，并不会对其他流的数据包传输产生影响。
- **快速重启会话：** 普通基于tcp的连接，是基于两端的ip和端口和协议来建立的。在网络切换场景，例如手机端切换了无线网，使用4G网络，会改变本身的ip，这就导致tcp连接必须重新创建。而QUIC协议使用特有的UUID来标记每一次连接，在网络环境发生变化的时候，只要UUID不变，就能不需要握手，继续传输数据。