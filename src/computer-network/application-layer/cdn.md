---
nav:
  title: 计算机网络
  order: 4
group:
  title: 应用层协议
  order: 5
title: CDN 内容发布网络
order: 2
---
# CDN 内容发布网络

**内容发布网络**(Content Delivery Network,简称CDN)通过将源站内容分发到各个区域，当用户访问请求的时候，会选择离用户**距离最近，有相关请求内容**的结点放回给用户，从而降低核心系统负载(系统、网络)。使用户可就近取得所需内容，**提高用户访问的响应速度**。解决了因为分布、带宽、服务器性能带来的延迟问题。适用于图片小文件、大文件下载、音视频点播、全站加速和安全加速等场景。

## 工作原理

通过在网络里各处放置节点服务器所构成的在现在的互联网基础之上的一层智能虚拟网络，CDN能够实时根据**网络流量**和**各节点的连接、负载状况**以及用户的距离，响应时间来综合从而将用户的请求重新导向离用户最近的服务器。

利用公式简述 CDN 可表示为：

```js
CDN = 更智能的镜像 + 缓存 + 流量导流;
```

简单地说，CDN 是一个经策略性部署的整体系统，包括**分布式存储**、**负载均衡**、**网络请求的重定向** 和 **内容管理** 4 个要件，而内容管理和全局的网络流量管理（Traffic Management）是 CDN 的核心所在。

**主要功能**

- 分布式存储
- 负载均衡
- 网络请求的重定向
- 内容管理

## 工作流程

用户终端访问 CDN 的过程分为两个步骤，一是用户通过 DNS 找到最近的 CDN 边缘节点 IP，二是数据在网络中送达用户终端。

最简单的 CDN 网络由一个 DNS 服务器和几台缓存服务器组成，假设您的加速域名为 `www.taobao.com`，接入 CDN 网络，开始使用加速服务后，当终端用户（广州）发起 HTTP 请求时，处理流程如下

![CDN Workflow](https://tsejx.github.io/javascript-guidebook/static/cdn-workflow.c9080c30.jpg)

1. 当终端用户向`www.taobao.com`下的资源发送请求，首先向LDNS（本地服务器）发起域名解析请求。
2. LDNS中缓存如果有对应的IP地址记录的话，则直接将ip方会给终端用户，否则，则取请求授权DNS服务器
3. 当授权DNS解析`www.taobao.com`，返回域名对应的ip地址，
4. 域名解析后请求发送到DNS调度系统，请求最佳的节点的IP地址
5. 本地服务器获得IP地址
6. 用户向该IP地址发送请求。
   - 如果该 IP 地址对应的节点已缓存该资源，则会将数据直接返回给用户，例如，图中步骤 7 和 8，请求结束。
   - 如果该 IP 地址对应的节点未缓存该资源，则节点向它的上级缓存服务器请求内容，直至追溯到网站的源站发起对该资源的请求。获取资源后，结合用户自定义配置的缓存策略，将资源缓存至节点，例如，途中的杭州节点，并返回给用户，请求结束。

在步骤四中，DNS 调度系统可以实现负载均衡功能，负载均衡分为全局负载均衡和区域负载均衡，其内部逻辑大致如下：

1. CDN 全局负载均衡设备会根据用户 IP 地址，以及用户请求的内容 URL，选择一台用户所属区域的**区域负载均衡设备**，告诉用户向这台设备发起请求。
2. 区域负载均衡设备会为用户选择一台合适的**缓存服务器**提供服务，选择的依据包括：
   - 根据用户 IP 地址，判断哪一台服务器距用户最近；
   - 用户所处的运营商；
   - 根据用户所请求的 URL 中携带的内容名称，判断哪一台服务器上有用户所需内容；
   - 查询各个服务器当前的负载情况，判断哪一台服务器尚有服务能力。 基于以上这些条件的综合分析之后，区域负载均衡设备会向全局负载均衡设备返回一台缓存服务器的 IP 地址。
3. 全局负载均衡设备把服务器的 IP 地址返回给用户。

---

**参考资料**

- [CDN 内容分发网络](https://tsejx.github.io/javascript-guidebook/computer-networks/computer-network-architecture/cdn)