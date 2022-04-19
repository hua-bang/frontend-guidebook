---
nav:
  title: 计算机网络
  order: 4
group:
  title: 应用层协议
  order: 5
title: DNS 域名解析系统
order: 1
---
# DNS 域名解析系统

**DNS域名解析系统**(Domain Name System)是用于将域名解析称IP的计算机网络服务。DNS维护着一个包含[域名](https://developer.mozilla.org/zh-CN/docs/Glossary/Domain_name)与对应资源例如[IP地址](https://developer.mozilla.org/zh-CN/docs/Glossary/IP_Address)的列表。

## Why DNS?

使用ip地址而非使用域名通信的原因

- IP地址是固定长度的，IPV432位，IPV6128位。而域名是变长的，不便于计算机处理
- IP地址对于用户来说不方便记忆，但域名便于用户使用。

IP面向主机，域名面向用户。

DNS实质上就是将域名解析称对应的IP地址。

## 域名的分层结构

由于因特网的用户数量较多，所以因特网在命名时采用的是层次树状结构的命名方法。任何一个连接在因特网上的主机或路由器，都有一个唯一的层次结构的名字，即域名（domain name）。这里，**域**（domain）是名字空间中一个可被管理的划分。从语法上讲，每一个域名都是有标号（label）序列组成，而各标号之间用点（`.`）隔开。域名可以划分为各个子域，子域还可以继续划分为子域的子域，这样就形成了顶级域、主域名、子域名等。

**域名系统必须要保持唯一性。**

特点：

1. 每个域名都是一个标号序列，用字母（大小写等价）、数字（0-9）和连接符（-）组成
2. 标号序列总长度不能超过 255 个字符，它由各标点之间用 `.` 分割成一个个的标号
3. 每个标号应该在 63 个字符之内，每个标号都可以堪称一个层次的域名
4. 级别最低的域名写在左边，级别最高的域名写在右边

域名解析系统是基于`UDP`协议的，端口号为53.

以 `www.example.google.com` 为例：

- `.com`：是顶级域名
- `google.com`：是主域名（也可称托管一级域名），主要指企业名
- `example.google.com`：是子域名（也可称为托管二级域名）
- `www.example.google.com`：是子域名的子域（也可称为托管三级域名）

域名可以划分多个子域，子域也可以继续划分自己的子域，形成顶级域，二级域，三级域等。

| 顶级域名     | 标识                                                         |
| ------------ | ------------------------------------------------------------ |
| 国家顶级域名 | 中国 `cn`；美国 `us`；英国 `uk`                              |
| 通用顶级域名 | 公司企业 `com`；教育机构 `edu`；政府部门 `gov`；国际组织 `int`；军事部门 `mil`；网络 `net`；非盈利组织 `org` |
| 反向域名     | arpa 用于 PTR 查询（IP 地址转换为域名）                      |

## DNS 的分层结构

域名是分层结构，域名 DNS 服务器也是对应的层级结构。有了域名结构，还需要有域名 DNS 服务器去解析域名，且是需要由遍及全世界的域名 DNS 服务器去解析，域名 DNS 服务器实际上就是装有域名系统的主机。域名解析过程涉及 4 个 DNS 服务器，分别如下：

| 分类           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| 根 DNS 服务器  | Root nameserver。本地域名服务器在本地查询不到解析结果时，则第一步会向它进行查询，并获取顶级域名服务器的 IP 地址。 |
| 顶级域名服务器 | TLD（Top-level） nameserver。负责管理在该顶级域名服务器下注册的二级域名，例如 `www.example.com`、`.com` 则是顶级域名服务器，在向它查询时，可以返回二级域名 `example.com` 所在的权威域名服务器地址 |
| 权威域名服务器 | authoritative nameserver。在特定区域内具有唯一性，负责维护该区域内的域名与 IP 地址之间的对应关系，例如云解析 DNS。 |
| 本地域名服务器 | DNS resolver 或 Local DNS。本地域名服务器是响应来自客户端的递归请求，并最终跟踪直到获取到解析结果的 DNS 服务器。例如用户本机自动分配的 DNS、运营商 ISP 分配的 DNS、谷歌/114 公共 DNS 等 |

> 每个层的域名上都有自己的域名服务器，最顶层的是根域名服务器
>
> 每一级域名服务器都知道下级域名服务器的 IP 地址，以便于一级一级向下查询

## DNS 的记录类型

在 DNS 系统中，最常见的资源记录方式是 Internet 类记录，该记录由包含 4 个字段的数据构成：`Name`、`Value`、`Type`、`TTL`。其中 `Name` 和 `Value` 可以理解为一对键值对，但是其具体含义取决于 `Type` 的类型，`TTL` 记录了该条记录应当从缓存中删除的时间。在资源记录的类型中中，最为常见且重要的类型 `Type` 主要有：

- **A**：将域名指向一个IPv4的地址，`A` 记录用于描述目标域名到 IP 地址的映射关系，将目标域名与 `A` 记录的 `Name` 字段进行匹配，将成功匹配的记录的 Value 字段的内容（IP 地址）输出到 DNS 回应报文中。

- **CNAME**：将域名指向另一个域名，`CNAME` 记录用于描述目的域名和别名的对应关系，如果说 `A` 记录可以将目标域名转换为对应主机的 IP 地址，那么 `CNAME` 记录则可以将一个域名（别名）转换为另一个域名，如果多条 `CNAME` 记录指向同一个域名，则可以将多个不同的域名的请求指向同一台服务器主机。并且，`CNAME` 记录通常还对应了一条 `A` 记录，用于提供被转换的域名的 IP 地址。

- **NS**：将子域名指向另一个 DNS 服务器解析，`NS` 记录用于描述目标域名到负责解析该域名的 DNS 的映射关系，根据目标域名对 `NS` 记录的 `Name` 字段进行匹配，将成功匹配的记录的 `Value` 字段（负责解析目标域名的 DNS 的 IP 地址）输出到 DNS 回应报文中。
- **AAAA**：将域名指向一个 IPv6 地址

- **MX**：将域名指向邮件服务器地址
- **SRV**：记录提供特定的服务的服务器
- **TXT**：文本长度限制 512，通常做 SDF 记录（反垃圾邮件）
- **CAA**：CA 证书颁发机构授权校验
- **显示URL**：将域名重定向至另一个地址
- **隐形 URL**：与显性 URL 类似，但是会隐藏真实的目标地址

## DNS解析过程

DNS 是一种使用 **UDP** 协议进行域名查询的协议，其最主要的目标就是**将域名转换为 IP 地址**。

DNS 查询的结果通常会在本地域名服务器中进行缓存，如果本地域名服务器中有缓存的情况下，则会跳过如下 DNS 查询步骤，很快返回解析结果。下面的示例则概述了本地域名服务器没有缓存的情况下，DNS 查询所需的 8 个步骤：

1. 用户在WEB游览默输入`www.taobao.com`，则有本地域名服务器开始递归查询。
2. 本地域名服务器采用迭代查询的方法，向跟服务器进行查询
3. 根服务器告诉本地服务器，下一个应该查询的顶级域`.com`的IP地址
4. 主机拿到之后向该顶级域的ip发送请求查询。
5. `.com`服务器告诉本地域名服务器，下一步查询`www.taobao.com`的**权威服务器**的IP地址
6. 本地服务器向`www.taobao.com`权威域名服务器发送查询。
7. 该权威域名服务器告诉本地服务器域名服务器所查询的主机IP地址
8. 本地域名服务器把ip地址告诉web服务器，游览器就能开始发送请求了
9. 游览器建立TCP连接后，发送HTTP请求
10. 该 IP 处的 web 服务器返回要在浏览器中呈现的网页

![DNS Workflow](https://tsejx.github.io/javascript-guidebook/static/workflow.bc8acb30.jpeg)

**详细解析：**

以查询 `www.taobao.com` 对应的 IP 地址为例，操作系统首先会在本地尝试解析，比如使用众所周知的 `hosts` 文件，同时如果有解析缓存的话，操作系统也会去查询。如果是在浏览器中进行查询，浏览器自己有时也会有解析缓存。

- 用户设备
  - 浏览器可能会缓存域名解析
  - 用户系统中可以有自己的域名映射表
- 公共域名服务器
  - 通常由 ISP 提供
  - 缓存上一级域名服务器的结果

在查询没有结果时，设备最终会开始向域名服务器发起查询请求。公共域名服务器一般就是用户的 ISP 提供的。这种公共域名服务器通常会缓存查询结果，因此如果缓存命中，查询就可以到此结束。当然缓存本身是有时效的，这个时效就被称为 TTL。对于超过时效的查询结果，域名服务器有义务重新发起查询请求。但查询本身是非常消耗流量的事情，因此也有一些公共服务器不严格遵守 TTL，超时缓存。

未命名缓存的查询，公共服务器会向顶级域名服务器进行查询。以上述例子来说，因为公共域名服务器不知道 `taobao.com` 的解析权归谁，因此它会向顶级域名服务器——`com` 域名服务器发起请求，寻找 `taobao.com` 对应的域名服务器。顶级域名服务器一般是由域名经营机构来维护的，有些甚至归属国家机关管理，例如国别域名。理论上来说，在顶级域名服务器之上还有一个根域名服务器，不过在平时很难意识到它的存在。

- 公共域名服务器
  - DNS 级联的特性决定了中途可以有更多域名服务器
- 顶级域名服务器
  - 由顶级域名经营机构维护
  - 可细分为与国家、通用

在查找到 `taobao.com` 的域名服务器之后，就可以向域名服务器查询 `www.taobao.com` 的 IP 了。这个过程是由上到下指定下来的，所以这种域名服务器可以被称为权威域名服务器。对于开发者来说，我们自己平时在域名服务器那里购买到域名之后，录入自己域名对应的 IP，其实就是在向权威域名服务器录入信息。一些大型企业会自己维护权威域名服务器，这样既可以抵御一些针对性的攻击，同时也可以更好地优化解析的速度。

- 公共域名服务器
- 权威域名服务器
  - 通常由专业的域名服务机构提供
  - 购买域名时一般会提供

## DNS 排查与优化

### 常见问题

- DNS 服务器本身有问题，响应慢并且不稳定
- 或者是，客户端到 DNS 服务器的网络延迟比较大
- 再或者，DNS 请求或者响应包，在某些情况下被链路中的网络设备弄丢了

### 故障排查顺序

1. 检查本地 `hosts`：`cat /etc/hosts`
2. 检查 `resolv.conf` 文件：`cat /etc/resolv.conf`。在 `Redhat7 / Centos7` 上修改 `resolv.conf` 里的 DNS 地址后，重启启网络服务发现 DNS 地址消失了，那么检查下网卡配置文件。
3. 检查网卡配置文件：`cat /etc/sysconfig/network-scripts/ifcfg-<网卡名称>`，看下里头有没 DNS 配置信息，没有的话补上去。

### 常见优化技术

1. HttpDNS：客户端基于 HTTP 协议，向 CDN 服务商指定的 DNS 服务器发送域名解析请求，从而避免 LocalDNS 造成的域名劫持和跨网访问
2. Http 302 跳转：CDN 厂商维护 CDN 域名 IP 库，根据用户访问终端的 IP 和 CDN 边缘节点的状态，选择最合适的 CDN 节点，发出 HTTP 的 302 返回码，将用户的请求跳转到合适的 CDN 边缘节点。

### 常见优化方法

- 对 DNS 解析的结果进行缓存。缓存是最有效的方法，但要注意，一旦缓存过期，还是要去 DNS 服务器重新获取新记录。不过，这对大部分应用程序来说都是可接受的。
- 对DNS解析进行预请求。这是浏览器等 Web 应用中最常用的方法，也就是说，不等用户点击页面上的超链接，浏览器就会在后台自动解析域名，并把结果缓存起来。
- 使用 HTTPDNS 取代常规的 DNS 解析。这是很多移动应用会选择的方法，特别是如今域名劫持普遍存在，使用 HTTP 协议绕过链路中的 DNS 服务器，就可以避免域名劫持的问题。
- 基于 DNS 的全局负载均衡（GSLB）。这不仅为服务提供了负载均衡和高可用的功能，还可以根据用户的位置，返回距离最近的 IP 地址。

- 对于移动客户端，在 APP 启动时对需要解析的域名做预先解析，然后把解析的结果缓存到本地的一个 LRU 缓存里面。这样当我们要使用这个域名的时候，只需要从缓存中直接拿到所需要的 IP 地址就好了，如果缓存中不存在才会走整个 DNS 查询的过程。同时为了避免 DNS 解析结果的变更造成缓存内数据失效，我们可以启动一个定时器定期地更新缓存中的数据。

### DNS 污染解决方案

一般是考虑尽可能自主控制 DNS 解析，比如使用专用 DNS 服务器，HTTPDNS，甚至是直接使用 IP 地址跳过解析

## DNS 术语

**递归查询**

递归查询是一种 DNS 服务器的查询模式，在该模式下 DNS 服务器接收到客户机请求，必须使用一个准确的查询结果回复客户机。如果 DNS 服务器本地没有存储查询 DNS 信息，那么该服务器会询问其他服务器，并将返回的查询结果提交给客户机。所以，一般情况下服务器跟内网 DNS 或直接 DNS 之间都采用递归查询。

**迭代查询**

DNS 服务器另外一种查询方式为迭代查询，DNS 服务器会向客户机提供其他能够解析查询请求的 DNS 服务器地址，当客户机发送查询请求时，DNS 服务器并不直接回复查询结果，而是告诉客户机另一台 DNS 服务器地址，客户机再向这台 DNS 服务器提交请求，依次循环直到返回查询的结果。所以一般内网 dns 和外网 dns 之间的都采用迭代查询。

**DNS 缓存**

DNS 缓存是将解析数据存储在靠近发起请求的客户端的位置，也可以说 DNS 数据是可以缓存在任意位置，最终目的是以此减少递归查询过程，可以更快的让用户获得请求结果。

**TTL**

英文全称 Time To Live ，这个值是告诉本地域名服务器，域名解析结果可缓存的最长时间，缓存时间到期后本地域名服务器则会删除该解析记录的数据，删除之后，如有用户请求域名，则会重新进行递归查询/迭代查询的过程。

**DNS Query Flood Attack**

指域名查询攻击，攻击方法是通过操纵大量傀儡机器，发送海量的域名查询请求，当每秒域名查询请求次数超过 DNS 服务器可承载的能力时，则会造成解析域名超时从而直接影响业务的可用性。

**URL 转发**

英文 Url Forwarding，也可称地址转向，它是通过服务器的特殊设置，将一个域名指向到另外一个已存在的站点

**DNSSEC**

域名系统安全扩展（DNS Security Extensions），简称 DNSSEC。它是通过数字签名来保证 DNS 应答报文的真实性和完整性，可有效防止 DNS 欺骗和缓存污染等攻击，能够保护用户不被重定向到非预期地址，从而提高用户对互联网的信任。

## 常用 DNS

- 114 DNS：`114.114.114.114` 或 `114.114.115.115`
- 阿里 DNS：`223.5.5.5` 或 `223.6.6.6`
- 百度 DNS：`180.76.76.76`
- DNS 派：
  - 电信 `101.226.4.6`
  - 联通 `123.125.81.6`
  - 移动 `101.226.4.6`
  - 铁通 `101.226.4.6`
- OneDNS
  - 南方 `112.124.47.27`
  - 北方 `114.215.126.16`
  - 共用 `42.236.82.22`
- Google DNS：`8.8.8.8` 或 `8.8.4.4`
- OpenDNS：`208.67.222.222` 或 `208.67.220.220`
- 360 DNS：`101.226.4.6` 或 `123.125.81.6`

---

**参考资料：**

- [DNS](https://tsejx.github.io/javascript-guidebook/computer-networks/computer-network-architecture/dns)