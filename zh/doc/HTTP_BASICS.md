# HTTP 基础知识

返回 **[目录](https://github.com/trimstray/nginx-admins-handbook#table-of-contents)** 或 **[下一步](https://github.com/trimstray/nginx-admins-handbook#whats-next)** 部分。

- **[≡ HTTP 基础知识](#http-basics)**
  * [介绍](#introduction)
  * [特性与架构](#features-and-architecture)
  * [HTTP/2](#http2)
    * [如何调试 HTTP/2？](#how-to-debug-http2)
  * [HTTP/3](#http3)
  * [URI vs URL](#uri-vs-url)
  * [连接 vs 请求](#connection-vs-request)
  * [HTTP 头](#http-headers)
    * [头部压缩](#header-compression)
  * [HTTP 方法](#http-methods)
  * [请求](#request)
    * [请求行](#request-line)
      * [方法](#methods)
      * [请求 URI](#request-uri)
      * [HTTP 版本](#http-version)
    * [请求头字段](#request-header-fields)
    * [消息体](#message-body)
    * [生成请求](#generate-requests)
  * [响应](#response)
    * [状态行](#status-line)
      * [HTTP 版本](#http-version-1)
      * [状态码与原因短语](#status-codes-and-reason-phrase)
    * [响应头字段](#response-header-fields)
    * [消息体](#message-body-1)
  * [HTTP 客户端](#http-client)
    * [IP 地址简写](#ip-address-shortcuts)
  * [后端 Web 架构](#back-end-web-architecture)
  * [有用的视频资源](#useful-video-resources)

<a id="introduction"></a>
#### 介绍

简而言之，HTTP 代表超文本传输协议（Hypertext Transfer Protocol），用于在互联网上传输数据（例如网页）。

关于 HTTP 的一些重要信息：

- 所有请求都源自客户端（例如浏览器）
- 服务器响应请求
- 请求和响应是明文可读的
- 请求之间相互独立，服务器无需跟踪请求

我不会详细描述 HTTP 协议，所以请将其视为一个入门介绍。我只会讨论最重要的事情，因为我们有一些非常优秀的文档已经详细描述了该协议：

- [RFC 2616 - HTTP/1.1](https://tools.ietf.org/html/rfc2616) <sup>[IETF]</sup>
- [RFC 7230 - HTTP/1.1: Message Syntax and Routing](https://tools.ietf.org/html/rfc7230) <sup>[IETF]</sup>
- [HTTP Made Really Easy](https://www.jmarshall.com/easy/http/)
- [MDN web docs - An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [LWP in Action - Chapter 2. Web Basics](http://lwp.interglacial.com/ch02_01.htm)
- [HTTP and everything you need to know about it](https://medium.com/faun/http-and-everything-you-need-to-know-about-it-8273bc224491)

我也推荐阅读：

- [Mini HTTP guide for developers](https://charemza.name/blog/posts/abstractions/http/http-guide-for-developers/)
- [10 Great Web Performance Blogs](https://www.aaronpeters.nl/blog/10-great-web-performance-blogs/)

我们有一些有趣的书籍：

- [HTTP: The Definitive Guide](https://www.amazon.com/HTTP-Definitive-Guide-Guides-ebook/dp/B0043D2EKO)
- [High Performance Browser Networking](https://hpbn.co/)

也可以查看本章的[有用的视频资源](#useful-video-resources)部分。最后，请查看[这个](https://github.com/bigcompany/know-your-http)关于 HTTP 协议的 A1 尺寸海报系列。

<a id="features-and-architecture"></a>
#### 特性与架构

HTTP（1.0/1.1 = h1）协议是一种基于客户端/服务器架构的请求/响应协议，其中 Web 浏览器、爬虫和搜索引擎等充当 HTTP 客户端，Web 服务器充当服务器。这就是 HTTP 基于消息的模型。每次 HTTP 交互都包含一个请求和一个响应。

HTTP 本质上是无状态的。无状态意味着所有请求之间相互独立。因此，浏览器发出的每个请求本身必须包含足够的信息，以便服务器能够处理该请求。

以下是简要说明：

- HTTP 通信最常使用 TCP 协议

- 默认端口是 TCP 80，但也可以使用其他端口

- HTTP 允许通过单个（持久）连接传输多个请求和响应（[RFC 7230 - Persistence](https://tools.ietf.org/html/rfc7230#section-6.3) <sup>[IETF]</sup>）

- HTTP 协议是无状态的（所有请求相互独立）

- HTTP 基于消息模型的每个事务都是独立处理的

- HTTP 客户端（即浏览器）发起 HTTP 请求，发出请求后，客户端等待响应

- HTTP 服务器处理来自客户端的请求（同时继续监听并接受其他请求），之后向客户端发送响应

- 只要客户端和服务器都知道如何处理数据内容，HTTP 可以发送任何类型的数据

- 服务器和客户端仅在当前请求期间感知对方的存在。之后，它们互相遗忘

HTTP 协议允许客户端和服务器进行通信。客户端使用 HTTP 方法发送请求，服务器在主机和端口上监听请求。以下是一个对比：

- **客户端** - HTTP 客户端通过 TCP/IP 连接向服务器发送请求，包含请求方法、URI 和协议版本，后跟类似 MIME 的消息，包含请求修饰符、客户端信息以及可能的正文内容

- **服务器** - HTTP 服务器返回响应，包含状态行（含消息的协议版本和成功或错误代码），后跟类似 MIME 的消息，包含服务器信息、实体元信息以及可能的实体正文内容

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/http/HTTP_steps.png" alt="HTTP_steps">
</p>

<sup><i>该信息图来自 [www.ntu.edu.sg - HTTP (HyperText Transfer Protocol)](https://www.ntu.edu.sg/home/ehchua/programming/webprogramming/HTTP_Basics.html)。</i></sup>

<a id="http2"></a>
#### HTTP/2

  > **:bookmark: [使用 HTTP/2 - 性能 - P1](RULES.md#beginner-use-http2)**

HTTP/2（h2）是 HTTP 网络协议的一个主要修订版本，旨在作为 HTTP/1.1 的高性能替代方案。它引入了若干新特性，同时保持语义兼容。HTTP/2 使支持它的浏览器和服务器之间的通信更加高效。所有主流浏览器的最新版本都支持 HTTP/2，包括 Chrome、Edge、Firefox、Opera 和 Safari。

请看下面的对比：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/http/http_comparison.png" alt="http_comparison">
</p>

我不会描述 HTTP/2，因为已有出色的研究资料：

- [RFC 7540 - HTTP/2](https://tools.ietf.org/html/rfc7540) <sup>[IETF]</sup>
- [HTTP/2 finalized - a quick overview](https://evertpot.com/http-2-finalized/)
- [Introduction to HTTP/2](https://developers.google.com/web/fundamentals/performance/http2)
- [HTTP2 Explained](https://daniel.haxx.se/http2/)
- [HTTP/2 in Action](https://www.manning.com/books/http2-in-action)
- [HTTP/2 Frequently Asked Questions](https://http2.github.io/faq/)
- [How Does HTTP/2 Work?](https://sookocheff.com/post/networking/how-does-http-2-work/)
- [HTTP/2: the difference between HTTP/1.1, benefits and how to use it](https://medium.com/@factoryhr/http-2-the-difference-between-http-1-1-benefits-and-how-to-use-it-38094fa0e95b)
- [HTTP2 Vs. HTTP1 – Let's Understand The Two Protocols](https://cheapsslsecurity.com/p/http2-vs-http1/)
- [HTTP 2 protocol – it is faster, but is it also safer?](https://research.securitum.com/http-2-protocol-it-is-faster-but-is-it-also-safer/)

然而，你应该知道，不支持 HTTP/2 的客户端永远不会向服务器请求升级到 HTTP/2 通信：它们之间的通信将完全使用 HTTP/1.1。支持 HTTP/2 的客户端默认不会发起 HTTP/2 请求，而是会使用 HTTP/1.1 并携带 `Upgrade: HTTP/2.0` 头部向服务器请求升级到 HTTP/2：

  - 如果服务器支持 HTTP/2，则服务器会相应通知客户端：它们之间的通信将切换到 HTTP/2
  - 如果服务器不支持 HTTP/2，则服务器会忽略升级请求并以 HTTP/1.1 响应：它们之间的通信将保持为 HTTP/1.1

另请参阅 [NGINX Updates Mitigate the August 2019 HTTP/2 Vulnerabilities](https://www.nginx.com/blog/nginx-updates-mitigate-august-2019-http-2-vulnerabilities/)。

<a id="how-to-debug-http2"></a>
##### 如何调试 HTTP/2？

这里有一篇关于[用于调试、测试和使用 HTTP/2 的工具](https://blog.cloudflare.com/tools-for-debugging-testing-and-using-http-2/)的优秀解释。另请参阅[用于 HTTP/2 调试的有用工具](https://community.akamai.com/customers/s/article/Useful-tools-for-HTTP-2-debugging?language=en_US)。

<a id="http3"></a>
#### HTTP/3

- [A QUIC look at HTTP/3](https://lwn.net/SubscriberLink/814522/ab3bfaa8f75c60ce/)
- [HTTP/3 explained](https://daniel.haxx.se/http3-explained/)
- [HTTP/3: the past, the present, and the future](https://blog.cloudflare.com/http3-the-past-present-and-future/)
- [What Is HTTP/3 – Lowdown on the Fast New UDP-Based Protocol](https://kinsta.com/blog/http3/)

<a id="uri-vs-url"></a>
#### URI vs URL

统一资源标识符（URI）是一个字符串，用于标识互联网上的名称或资源。URI 通过位置、名称或两者来标识资源。URI 有两种特化形式，称为 URL 和 URN。

我认为最好的解释在这里：[The Difference Between URLs, URIs, and URNs](https://danielmiessler.com/study/url-uri/)，作者 [Daniel Miessler](https://danielmiessler.com/about/)。

对我来说，这个简短明了的评论也很有趣：

  > URI **标识**，URL **标识**且**定位**；然而，**定位符也是标识符**，所以每个 URL 也是一个 URI，但有些 URI 不是 URL。

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/http/url_urn_uri.png" alt="url_urn_uri">
</p>

请看下面的例子来消除困惑并简单理解：

| <b>类型</b> | <b>描述</b> |
| :---:         | :---         |
| URL | `https://www.google.com/folder/page.html` |
| URL | `http://example.com/resource?foo=bar#fragment` |
| URL | `ftp://example.com/download.zip` |
| URL | `mailto:user@example.com` |
| URL | `file:///home/user/file.txt` |
| URL | `/other/link.html`（相对 URL） |
| URN | `www.pierobon.org/iis/review1.htm#one` |
| URN | `urn:ietf:rfc:2648` |
| URN | `urn:isbn:0451450523` |
| URI | `http://www.pierobon.org/iis/review1.htm.html#one` |

下图解释了 URL 格式：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/http/url_format.png" alt="url_format">
</p>

如果仍然不清楚，我建议你看看以下文章：

- [What is the difference between a URI, a URL and a URN?](https://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-and-a-urn/1984225)
- [The History of the URL: Path, Fragment, Query, and Auth](https://eager.io/blog/the-history-of-the-url-path-fragment-query-auth/)

<a id="connection-vs-request"></a>
#### 连接 vs 请求

基本上，建立连接是为了使用它来发起请求。因此，一个连接上可以有多个请求。当然不是在同时，但渲染一个网页是一个相当耗时的过程。

连接是两个端点之间基于 TCP 的可靠管道。每个连接需要跟踪两个端点的地址/端口、序列号以及哪些数据包未被确认或被假定丢失。

请求是针对给定资源使用特定方法的 HTTP 请求。每个会话可能有零个或多个请求。（是的，Web 浏览器可以并且确实会在不发送请求的情况下打开连接。）

大多数现代浏览器会同时打开多个连接，并同时下载不同的文件（图片、css、js）以加快页面加载速度。但是，正如我所说，每个连接也可以处理多个请求（下载）。

总结一下：

- HTTP 连接 - 客户端和服务器互相介绍；与服务器建立连接涉及 TCP 握手，基本上就是与服务器创建一个 socket 连接

- HTTP 请求 - 客户端向服务器请求某些内容；要发起 HTTP 请求，你应当已经与服务器建立了连接。如果你与服务器建立了连接，你可以使用同一个连接发起多个请求（HTTP/1.0 默认每个连接一个请求，HTTP/1.1 默认保持连接 keep-alive）

还有一个很好的对比：

- 依次打开 25 个连接，每个连接下载 1 个文件（最慢）
- 打开 1 个连接，通过它下载 25 个文件（慢）
- 打开 5 个连接，每个连接下载 5 个文件（快）
- 打开 25 个连接，每个连接下载 1 个文件（浪费资源）

因此，如果你过多地限制连接数或请求数，就会降低页面加载速度。

<a id="http-headers"></a>
#### HTTP 头

当客户端向服务器请求资源时，它使用 HTTP。这个请求包含一组键值对，提供诸如浏览器版本或它理解的文件格式等信息。这些键值对称为请求头。

服务器用请求的资源进行响应，同时也会发送响应头，提供关于资源或服务器本身的信息。

请参阅这些关于 HTTP 头的文章：

- [HTTP headers](https://developer.mozilla.org/pl/docs/Web/HTTP/Headers)
- [The HTTP Request Headers List](https://flaviocopes.com/http-request-headers/)
- [The HTTP Response Headers List](https://flaviocopes.com/http-response-headers/)
- [Exotic HTTP Headers](https://peteris.rocks/blog/exotic-http-headers/)
- [Secure your web application with these HTTP headers](https://odino.org/secure-your-web-application-with-these-http-headers/)
- [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)

HTTP 头允许客户端和服务器在请求或响应中传递附加信息。一个 HTTP 头由不区分大小写的名称、后跟冒号 `:`，再跟其值（不含换行符）组成。

<a id="header-compression"></a>
###### 头部压缩

头部压缩（HTTP/2）的作用：

  > _头部压缩使请求头的大小减少了约 88%，响应头的大小减少了约 85%。在较低带宽的 DSL 链路上（上传链路仅为 375 Kbps），请求头部压缩尤其显著地缩短了某些网站的页面加载时间（即那些发出大量资源请求的网站）。我们发现仅因头部压缩，页面加载时间就减少了 45 到 1142 毫秒。_

- HTTP/2 支持一种名为 HPACK 的新专用头部压缩算法
- HPACK 能够抵御 CRIME 攻击
- HPACK 使用三种压缩方法：静态字典、动态字典、霍夫曼编码

另请参阅：

- [Designing Headers for HTTP Compression](https://www.mnot.net/blog/2018/11/27/header_compression)
- [HPACK: Header Compression for HTTP/2](https://http2.github.io/http2-spec/compression.html)
- [HPACK: the silent killer (feature) of HTTP/2](https://blog.cloudflare.com/hpack-the-silent-killer-feature-of-http-2/)

<a id="http-methods"></a>
#### HTTP 方法

HTTP 协议包含一组方法，用于指示要对资源执行何种操作。最常见的方法是 `GET` 和 `POST`。但还有一些其他的：

- `GET` - 用于从服务器指定资源检索数据

- `POST` - 用于创建资源或将资源追加到现有资源

- `PUT` - 用于创建或更新（替换）现有资源

- `HEAD` - 与 `GET` 几乎相同，只是没有响应正文

- `TRACE` - 用于诊断/调试目的，将输入内容回显给用户

- `OPTIONS` - 用于描述目标资源可用的通信选项（HTTP 方法）

- `PATCH` - 用于对资源进行部分修改

- `DELETE` - 用于删除指定的资源

<a id="request"></a>
#### 请求

一个请求由以下部分组成：`(1) 命令或请求 + (2) 可选头部 + (4) 可选正文内容`：

```
                       HTTP 请求的字段         RFC 2616 中的章节
---------------------------------------------------------------------
  Request       = (1) : Request-line                 Section 5.1
                  (2) : *(( general-header           Section 4.5
                          | request-header           Section 5.3
                          | entity-header ) CRLF)    Section 7.1
                  (3) : CRLF
                  (4) : [ message-body ]             Section 4.3
```

从运行在 `localhost:8000` 的 Web 服务器获取 `/alerts/status` 页面的 HTTP 请求示例：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/http/http_request.png" alt="http_request">
</p>

关于请求的附加信息：

- 路由到端点
- 重写路径/查询
- 拒绝访问（ACL）
- 头部修改

<a id="request-line"></a>
##### 请求行

请求行以方法开头，后跟请求 URI 和协议版本，以 CRLF（`\r\n` 或十六进制 `0d0a`）结束。元素之间由空格 SP 字符分隔：

```
Request-Line = Method SP Request-URI SP HTTP-Version CRLF
```

<a id="methods"></a>
###### 方法

| <b>方法</b> | <b>描述</b> |
| :---:         | :---         |
| `GET` | 用于从服务器指定资源检索数据（[RFC 2616 - GET](https://tools.ietf.org/html/rfc2616#section-9.3)）<sup>[IETF]</sup> |

例如，假设你有一个带有 `/api/v2/users` 端点的 API。对该端点发出 `GET` 请求应返回所有可用用户的列表。

  > 使用 `GET` 方法的请求不会更改任何数据（资源状态），且绝不应用于向服务器提交新信息。此方法与 `HEAD` 几乎相同。如果 `GET /users` 返回用户列表，那么 `HEAD /users` 将发出相同的请求，但不会返回用户列表。

在基础层面上，应验证以下内容：

- 检查有效的 `GET` 请求是否返回 `200` 状态码
- 确保对特定资源的 `GET` 请求返回正确的数据

执行 `GET` 请求时，你向服务器请求一个或一组实体。为了允许客户端过滤结果，可以使用所谓的 URL 的"查询字符串"（`?`）。

| <b>方法</b> | <b>描述</b> |
| :---:         | :---         |
| `POST` | 用于向服务器发送数据以修改和更新资源（[RFC 2616 - POST](https://tools.ietf.org/html/rfc2616#section-9.5)）<sup>[IETF]</sup> |

请看 `POST` 的定义：

  > _`POST` 方法用于请求源服务器接受请求中封装的实体，作为由请求 URI 标识的资源的新下级 [...] 提交的实体从属于该 URI，就像文件从属于包含它的目录、新闻文章从属于其发布的新闻组、或者记录从属于数据库一样。_

执行 `POST` 请求时，客户端实际上是向远程主机提交一个新文档，但服务器决定将其存储在用户集合中的哪个位置。

  > 使用 `POST` 方法的请求通过修改或更新资源来更改后端服务器上的数据（资源状态）。

如果你不知道实际资源的位置，例如，当你添加一篇新文章但不知道存储在哪里时，你可以将其 `POST` 到一个 URL，让服务器决定实际的 URL。

以下是测试 `POST` 请求的一些提示：

- 使用 `POST` 请求创建一个资源（服务器可能响应 `201 Created`）
- 接下来，发出 `GET` 请求确保返回 `200 OK` 状态码（数据已正确保存）
- 添加测试以确保 `POST` 请求在使用不正确或格式错误的数据时失败

修改和更新资源：

```
POST /items/<existing_item> HTTP/1.1
Host: example.com
```

以下是一个错误用法：

```
POST /items/<new_item> HTTP/1.1
Host: example.com
```

| <b>方法</b> | <b>描述</b> |
| :---:         | :---         |
| `PUT` | 用于向服务器发送数据以创建或覆盖资源（[RFC 2616 - PUT](https://tools.ietf.org/html/rfc2616#section-9.6)）<sup>[IETF]</sup> |

请看 `PUT` 的定义：

  > _`PUT` 方法请求将封装的实体存储在提供的请求 URI 下。如果请求 URI 引用了一个已存在的资源，则封装实体应被视为源服务器上该资源的修改版本。如果请求 URI 不指向现有资源 [...] 则源服务器可以使用该 URI 创建资源。_

`PUT` 规范要求你已经知道要创建或更新的资源的 URL。在创建时，如果客户端为资源选择了标识符，则 `PUT` 请求将在指定的 URL 创建新资源。

如果你知道某个 URL 上已存在资源，你可以对该 URL 发出 `PUT` 请求来替换服务器上该资源的状态。

测试 `PUT` 请求时要检查这些内容：

- 重复调用 `PUT` 请求始终返回相同的结果（幂等性）
- 如果修改了现有资源，应发送 `200 OK` 或 `204 No Content` 响应码以指示请求成功完成
- 如果在请求中提供了无效数据，`PUT` 请求应失败 - 不应更新任何内容

对于新资源：

```
PUT /items/<new_item> HTTP/1.1
Host: example.com
```

覆盖现有资源：

```
PUT /items/<existing_item> HTTP/1.1
Host: example.com
```

对我来说，`POST` 和 `PUT` 的区别有时不太清楚。我认为两者都可以用于创建，最大的区别在于 `POST` 请求应该让服务器决定如何（以及是否）创建新资源。

在我看来，还有一个很好的解释：

  > `POST` 方法用于将用户生成的数据发送到 Web 服务器。例如，当用户在论坛上发表评论或上传个人资料图片时，使用 `POST` 方法。如果你不知道新创建的资源应该放在哪个具体的 URL，也应该使用 `POST` 方法。

  > `PUT` 方法完全替换目标 URL 上当前存在的任何内容。使用此方法，只要你确切知道请求 URI，就可以创建新资源或覆盖现有资源。

另请查看 [Brian R. Bondy](https://stackoverflow.com/users/3153/brian-r-bondy) 在 [Stack Overflow 上](https://stackoverflow.com/a/630475)的精彩回答。

<a id="request-uri"></a>
###### 请求 URI

请求 URI 是一个统一资源标识符，用于标识要应用请求的资源。互联网请求所标识的确切资源由请求 URI 和 Host 头字段共同确定。

请求 URI 最常见的形式是用于标识源服务器或网关上的资源。例如，希望直接从源服务器检索资源的客户端将创建一个到主机 `example.com` 端口 80 的 TCP 连接，并发送以下行：

```
GET /pub/index.html HTTP/1.1
Host: example.com
```

当请求发送到代理时，需要使用绝对 URI 形式：

```
GET http://example.com/pub/index.html HTTP/1.1
```

  > 注意，绝对路径不能为空；如果原始 URI 中不存在路径，则必须使用 `/`（服务器根目录）。当 HTTP 请求不适用于特定资源而适用于服务器本身时，使用星号 `*`，并且仅当所使用的方法不一定适用于资源时才允许。

<a id="http-version"></a>
###### HTTP 版本

请求的最后一部分，表示客户端支持的 HTTP 版本。HTTP 有四个版本——HTTP/0.9、HTTP/1.0、HTTP/1.1 和 HTTP/2.0。当前常用的版本是 HTTP/1.1 和 HTTP/2.0。

确定 HTTP 协议的适当版本非常重要，因为它允许你设置特定的 HTTP 方法或必需的头部（例如 HTTP/1.1 的 `cache-control`）。

这里有一个很好的解释：[How does a HTTP 1.1 server respond to a HTTP 1.0 request?](https://stackoverflow.com/questions/35850518/how-does-a-http-1-1-server-respond-to-a-http-1-0-request)

<a id="request-header-fields"></a>
##### 请求头字段

请求有三种类型的 HTTP 消息头：

- **通用头（General-header）** - 适用于请求和响应，但与最终在正文中传输的数据无关

- **请求头（Request-header）** - 包含关于要获取的资源或客户端本身的更多信息

- **实体头（Entity-header）** - 包含关于实体正文的更多信息，如其内容长度或 MIME 类型

请求头字段允许客户端向服务器传递关于请求本身以及客户端的附加信息。

HTTP 头部允许的最大大小是多少？阅读[这个](https://stackoverflow.com/questions/686217/maximum-on-http-header-values)优秀答案。HTTP 没有定义任何限制。然而，大多数 Web 服务器确实限制它们接受的头部大小。如果头部大小超过该限制，服务器将返回 `413 Entity Too Large` 错误。

<a id="message-body"></a>
##### 消息体

请求（消息）正文是 HTTP 请求中可以向服务器发送附加内容的部分。

它是可选的。大多数 HTTP 请求是没有正文的 `GET` 请求。然而，模拟带有正文的请求对于正确测试代理代码以及测试处理此类请求的各种钩子非常重要。大多数带有正文的 HTTP 请求使用 `POST` 或 `PUT` 请求方法。

<a id="generate-requests"></a>
##### 生成请求

如何生成请求？

- `curl`

  ```bash
  curl -Iks -v -X GET -H "Connection: keep-alive" -H "User-Agent: X-AGENT" https://example.com
  ```

- `httpie`

  ```bash
  http -p Hh GET https://example.com User-Agent:X-AGENT --follow
  ```

- `telnet`

  ```bash
  telnet example.com 80
  GET /index.html HTTP/1.1
  Host: example.com
  ```

- `openssl`

  ```bash
  openssl s_client -servername example.com -connect example.com:443
  ...
  ---
  GET /index.html HTTP/1.1
  Host: example.com
  ```

更多示例，请参阅[测试](HELPERS.md#testing)章节。

<a id="response"></a>
#### 响应

接收并解释请求消息后，服务器以 HTTP 响应消息进行响应：

```
                       HTTP 响应的字段         RFC 2616 中的章节
---------------------------------------------------------------------
  Response      = (1) : Status-line                  Section 6.1
                  (2) : *(( general-header           Section 4.5
                          | response-header          Section 6.2
                          | entity-header ) CRLF)    Section 7.1
                  (3) : CRLF
                  (4) : [ message-body ]             Section 7.2
```

对从运行在 `localhost:8000` 的 Web 服务器获取 `/alerts/status` 页面的请求的 HTTP 响应示例：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/http/http_response.png" alt="http_request">
</p>

关于响应的附加信息：

- 缓存对象（浏览器/代理）
- 头部修改
- 正文修改

<a id="status-line"></a>
##### 状态行

状态行由协议版本、后跟数字状态码及其关联的文本短语组成。

```
Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
```

<a id="http-version-1"></a>
###### HTTP 版本

  > 当 HTTP/1.1 消息发送给 HTTP/1.0 接收方或版本未知的接收方时，HTTP/1.1 消息的构造应使其在忽略所有较新特性的情况下，可以被解释为有效的 HTTP/1.0 消息。

<a id="status-codes-and-reason-phrase"></a>
###### 状态码与原因短语

- `1xx: Informational（信息性）` - 请求已接收，继续处理
- `2xx: Success（成功）` - 操作已成功接收、理解和接受
- `3xx: Redirection（重定向）` - 需要进一步操作才能完成请求
- `4xx: Client Error（客户端错误）` - 请求包含错误的语法或无法完成
- `5xx: Server Error（服务器错误）` - 服务器未能完成一个看似有效的请求

更多信息请参阅：

- [HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [HTTP Status Codes](https://httpstatuses.com/)
- [RFC 2616 - Status Code Definitions](https://tools.ietf.org/html/rfc2616#section-10)

一定要看看这个（这是天才之作！）：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/http/http_decision_diagram.png" alt="http_decision_diagram">
</p>

<sup><i>此图来自 [for-GET/http-decision-diagram](https://github.com/for-GET/http-decision-diagram) 仓库。</i></sup>

该图遵循 [RFC7230](https://tools.ietf.org/html/rfc7230)、[RFC7231](https://tools.ietf.org/html/rfc7231)、[RFC7232](https://tools.ietf.org/html/rfc7232)、[RFC7233](https://tools.ietf.org/html/rfc7233)、[RFC7234](https://tools.ietf.org/html/rfc7234)、[RFC7235](https://tools.ietf.org/html/rfc7235) 中的指示，并在必要时填补空白。此图在任何情况下都不能取代 HTTP 规范。

<a id="response-header-fields"></a>
##### 响应头字段

响应有三种类型的 HTTP 消息头：

- **通用头（General-header）** - 适用于请求和响应，但与最终在正文中传输的数据无关

- **响应头（Response-header）** - 这些头字段提供关于服务器以及进一步访问由请求 URI 标识的资源的信息

- **实体头（Entity-header）** - 包含关于实体正文的更多信息，如其内容长度或 MIME 类型

响应头字段允许服务器传递关于响应的附加信息。

<a id="message-body-1"></a>
##### 消息体

包含客户端请求的资源数据。

<a id="http-client"></a>
#### HTTP 客户端

HTTP 客户端是能够以 HTTP 格式向服务器发送请求并从服务器接收响应的客户端。客户端还负责发起连接、传递任何必要的身份验证令牌以及发送对特定数据的请求。

  > 客户端是任何向后端发送请求的东西。

<a id="ip-address-shortcuts"></a>
##### IP 地址简写

IP 地址可以通过删除零来缩短：

```
http://1.0.0.1 → http://1.1
http://127.0.0.1 → http://127.1
http://192.168.0.1 → http://192.168.1

http://0xC0A80001 or http://3232235521 → 192.168.0.1
http://192.168.257 → 192.168.1.1
http://192.168.516 → 192.168.2.4
```

  > 这可以绕过 WAF 对 SSRF、open-redirect 等的过滤器，其中任何 IP 作为输入都会被列入黑名单。

更多信息请参阅 [How to Obscure Any URL](http://www.pc-help.org/obscure.htm) 和 [Magic IP Address Shortcuts](https://stuff-things.net/2014/09/25/magic-ip-address-shortcuts/)。

<a id="back-end-web-architecture"></a>
#### 后端 Web 架构

  > 我建议阅读这些优秀的仓库：
  >
  >   - [Learn how to design large-scale systems](https://github.com/donnemartin/system-design-primer)<br>
  >   - [Web Architecture 101](https://engineering.videoblocks.com/web-architecture-101-a3224e126947)

后端，或称为"服务器端"，是处理传入请求、生成并发送响应到客户端所需的全部技术。这通常包括三个主要部分：

- **服务器（server）** - 这是接收请求的计算机（后端/源服务器）
- **应用（app）** - 这是在服务器上运行的应用程序，负责监听请求、从数据库检索信息以及发送响应
- **数据库（databases）** - 用于组织和持久化数据

<a id="useful-video-resources"></a>
#### 有用的视频资源

- [HTTP Crash Course & Exploration](https://youtu.be/iYM2zFP3Zn0)
- [CS50 2017 - Lecture 6 - HTTP](https://youtu.be/PUPDGbnpSjw)
- [HTTP/2 101 (Chrome Dev Summit 2015)](https://youtu.be/r5oT_2ndjms)
- [Headers for Hackers: Wrangling HTTP Like a Pro](https://youtu.be/TNlcoYLIGFk)
- [Hacking with Andrew and Brad: an HTTP/2 client](https://youtu.be/yG-UaBJXZ80)
- [Advanced HTTP Protocol Hacking](https://youtu.be/up8Vz5PTX3M)
- [Hacking HTTP/2 - New Attacks on the Internet's Next Generation Foundation](https://youtu.be/kM8cc0kB21s)
