
## 1 HTTP的发展

**HTTP**（HyperText Transfer Protocol）是万维网（World Wide Web）的基础协议。自 Tim Berners-Lee 博士和他的团队在 1989-1991 年间创造出它以来，HTTP 已经发生了太多的变化，在保持协议简单性的同时，不断扩展其灵活性。如今，HTTP 已经从一个只在实验室之间交换文件的早期协议进化到了可以传输图片，高分辨率视频和 3D 效果的现代复杂互联网协议。

### 1.1 万维网的发明

1989 年，当时在 CERN 工作的 Tim Berners-Lee 博士写了一份关于建立一个通过网络传输超文本系统的报告。这个系统起初被命名为 _Mesh_，在随后的 1990 年项目实施期间被更名为 _万维网_（World Wide Web）。它在现有的 TCP 和 IP 协议基础之上建立，由四个部分组成：

- 一个用来表示超文本文档的文本格式，_超文本标记语言（HTML）_
- 一个用来交换超文本文档的简单协议，超文本传输协议（HTTP）
- 一个显示（以及编辑）超文本文档的客户端，即网络浏览器。第一个网络浏览器被称为 _WorldWideWeb。_
- 一个服务器用于提供可访问的文档，即 _httpd_ 的前身。

这四个部分完成于 1990 年底，且第一批服务器已经在 1991 年初在 CERN 以外的地方运行了。1991 年 8 月 16 日，Tim Berners-Lee 在公开的超文本新闻组上发表的文章被视为是万维网公共项目的开始。

HTTP 在应用的早期阶段非常简单，后来被称为 HTTP/0.9，有时也叫做单行（one-line）协议。

### 1.2 HTTP/0.9——单行协议

最初版本的 HTTP 协议并没有版本号，后来它的版本号被定位在 0.9 以区分后来的版本。HTTP/0.9 极其简单：请求由单行指令构成，以唯一可用方法 `GET`开头，其后跟目标资源的路径（一旦连接到服务器，协议、服务器、端口号这些都不是必须的）。

```http
GET /mypage.html
```

响应也极其简单的：只包含响应文档本身。

```html
<html>
  这是一个非常简单的 HTML 页面
</html>
```

跟后来的版本不同，HTTP/0.9 的响应内容并不包含 HTTP 头。这意味着只有 HTML 文件可以传送，无法传输其他类型的文件。也没有状态码或错误代码。一旦出现问题，一个特殊的包含问题描述信息的 HTML 文件将被发回，供人们查看。

### 1.3 HTTP/1.0——构建可扩展性

由于 HTTP/0.9 协议的应用十分有限，浏览器和服务器迅速扩展内容使其用途更广：

- 协议版本信息现在会随着每个请求发送（`HTTP/1.0` 被追加到了 `GET` 行）。
- 状态码会在响应开始时发送，使浏览器能了解请求执行成功或失败，并相应调整行为（如更新或使用本地缓存）。
- 引入了 HTTP 标头的概念，无论是对于请求还是响应，允许传输元数据，使协议变得非常灵活，更具扩展性。
- 在新 HTTP 标头的帮助下，具备了传输除纯文本 HTML 文件以外其他类型文档的能力（凭借 `Content-Type`标头）。

一个典型的请求看起来就像这样：

```http
GET /mypage.html HTTP/1.0
User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)

200 OK
Date: Tue, 15 Nov 1994 08:12:31 GMT
Server: CERN/3.0 libwww/2.17
Content-Type: text/html
<HTML>
一个包含图片的页面
  <IMG SRC="/myimage.gif">
</HTML>
```

接下来是第二个连接，请求获取图片（并具有相同的响应）：

```http
GET /myimage.gif HTTP/1.0
User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)

200 OK
Date: Tue, 15 Nov 1994 08:12:32 GMT
Server: CERN/3.0 libwww/2.17
Content-Type: text/gif
(这里是图片内容)
```

在 1991-1995 年，这些新扩展并没有被引入到标准中以促进协助工作，而仅仅作为一种尝试。服务器和浏览器添加这些新扩展功能，但出现了大量的互操作问题。直到 1996 年 11 月，为了解决这些问题，一份新文档（RFC 1945）被发表出来，用以描述如何操作实践这些新扩展功能。文档 [RFC 1945](https://datatracker.ietf.org/doc/html/rfc1945) 定义了 HTTP/1.0，但它是狭义的，并不是官方标准。

### 1.4 HTTP/1.1——标准化的协议

HTTP/1.0 多种不同的实现方式在实际运用中显得有些混乱。自 1995 年开始，即 HTTP/1.0 文档发布的下一年，就开始修订 HTTP 的第一个标准化版本。在 1997 年初，HTTP1.1 标准发布，就在 HTTP/1.0 发布的几个月后。

HTTP/1.1 消除了大量歧义内容并引入了多项改进：

- 连接可以复用，节省了多次打开 TCP 连接加载网页文档资源的时间。
- 增加管线化技术，允许在第一个应答被完全发送之前就发送第二个请求，以降低通信延迟。
- 支持响应分块。
- 引入额外的缓存控制机制。
- 引入内容协商机制，包括语言、编码、类型等。并允许客户端和服务器之间约定以最合适的内容进行交换。
- 凭借 `Host`标头，能够使不同域名配置在同一个 IP 地址的服务器上。

一个典型的请求流程，所有请求都通过一个连接实现，看起来就像这样：

```http
GET /zh-CN/docs/Glossary/Simple_header HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/zh-CN/docs/Glossary/Simple_header

200 OK
Connection: Keep-Alive
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
Date: Wed, 20 Jul 2016 10:55:30 GMT
Etag: "547fa7e369ef56031dd3bff2ace9fc0832eb251a"
Keep-Alive: timeout=5, max=1000
Last-Modified: Tue, 19 Jul 2016 00:59:33 GMT
Server: Apache
Transfer-Encoding: chunked
Vary: Cookie, Accept-Encoding

(content)


GET /static/img/header-background.png HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/zh-CN/docs/Glossary/Simple_header

200 OK
Age: 9578461
Cache-Control: public, max-age=315360000
Connection: keep-alive
Content-Length: 3077
Content-Type: image/png
Date: Thu, 31 Mar 2016 13:34:46 GMT
Last-Modified: Wed, 21 Oct 2015 18:27:50 GMT
Server: Apache

(image content of 3077 bytes)
```

HTTP/1.1 在 1997 年 1 月以 [RFC 2068](https://datatracker.ietf.org/doc/html/rfc2068) 文件发布。

### 1.5 超过 15 年的扩展

由于 HTTP 协议的可扩展性使得创建新的头部和方法是很容易的。即使 HTTP/1.1 协议进行过两次修订，[RFC 2616](https://datatracker.ietf.org/doc/html/rfc2616) 发布于 1999 年 6 月，而另外两个文档 [RFC 7230](https://datatracker.ietf.org/doc/html/rfc7230)-[RFC 7235](https://datatracker.ietf.org/doc/html/rfc7235) 发布于 2014 年 6 月（在 HTTP/2 发布之前）。HTTP/1.1 协议已经稳定使用超过 15 年了。

#### 1.5.1 HTTP 用于安全传输

HTTP 最大的变化发生在 1994 年底。HTTP 在基本的 TCP/IP 协议栈上发送信息，网景公司（Netscape Communication）在此基础上创建了一个额外的加密传输层：SSL。SSL 1.0 没有在公司以外发布过，但 SSL 2.0 及其后继者 SSL 3.0 允许通过加密来保证服务器和客户端之间交换消息的真实性，来创建电子商务网站。SSL 在标准化道路上最终成为了 TLS。

与此同时，人们对一个加密传输层的需求也愈发高涨：因为 Web 最早几乎是一个学术网络，相对信任度很高，但如今不得不面对一个险恶的丛林：广告客户、随机的个人或者犯罪分子争相劫取个人信息，将信息占为己有，甚至改动将要被传输的数据。随着通过 HTTP 构建的应用程序变得越来越强大，可以访问越来越多的私人信息，如地址簿、电子邮件或用户的地理位置，即使在电子商务使用之外，对 TLS 的需求也变得普遍。

#### 1.5.2 HTTP 用于复杂应用

Tim Berners-Lee 对于 Web 的最初设想不是一个只读媒体。他设想一个 Web 是可以远程添加或移动文档，是一种分布式文件系统。大约 1996 年，HTTP 被扩展到允许创作，并且创建了一个名为 WebDAV 的标准。它进一步扩展了某些特定的应用程序，如 CardDAV 用来处理地址簿条目，CalDAV 用来处理日历。但所有这些 *DAV 扩展有一个缺陷：它们必须由要使用的服务器来实现，这是非常复杂的。并且他们在网络领域的使用必须保密。

在 2000 年，一种新的使用 HTTP 的模式被设计出来：[具象状态传输（representational state transfer）](https://developer.mozilla.org/zh-CN/docs/Glossary/REST) (或者说 REST)。由 API 发起的操作不再通过新的 HTTP 方法传达，而只能通过使用基本的 HTTP / 1.1 方法访问特定的 URI。这允许任何 Web 应用程序通过提供 API 以允许查看和修改其数据，而无需更新浏览器或服务器。所有需要的内容都被嵌入到由网站通过标准 HTTP/1.1 提供的文件中。REST 模型的缺点在于每个网站都定义了自己的非标准 RESTful API，并对其进行了全面的控制。不同于 *DAV 扩展，客户端和服务器是可互操作的。RESTful API 在 2010 年变得非常流行。

自 2005 年以来，可用于 Web 页面的 API 大大增加，其中几个 API 为特定目的扩展了 HTTP 协议，大部分是新的特定 HTTP 头：
- [Server-sent events](https://developer.mozilla.org/zh-CN/docs/Web/API/Server-sent_events)，服务器可以偶尔推送消息到浏览器。
- [WebSocket](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSockets_API)，一个新协议，可以通过升级现有 HTTP 协议来建立。
#### 1.5.3 放松安全措施——基于当前的 Web 模型

HTTP 和 Web 安全模型——[同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)是互不相关的。事实上，当前的 Web 安全模型是在 HTTP 被创造出来后才被发展的！这些年来，已经证实了它如果能通过在特定的约束下移除一些这个策略的限制来管的宽松些的话，将会更有用。这些策略导致大量的成本和时间被花费在通过转交到服务端来添加一些新的 HTTP 头来发送。这些被定义在了[跨源资源共享](https://developer.mozilla.org/zh-CN/docs/Glossary/CORS)（CORS）和[内容安全策略](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)（CSP）规范里。

不只是这大量的扩展，很多的其他的头也被加了进来，有些只是实验性的。比较著名的有 [`DNT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/DNT)（Do Not Track）来控制隐私，[`X-Frame-Options`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/X-Frame-Options)，还有很多。

### 1.6 HTTP/2——为了更优异的表现

这些年来，网页愈渐变得的复杂，甚至演变成了独有的应用，可见媒体的播放量，增进交互的脚本大小也增加了许多：更多的数据通过 HTTP 请求被传输。HTTP/1.1 链接需要请求以正确的顺序发送，理论上可以用一些并行的链接（尤其是 5 到 8 个），带来的成本和复杂性堪忧。比如，HTTP 管线化（pipelining）就成为了 Web 开发的负担。为此，在 2010 年早期，谷歌通过实践了一个实验性的 SPDY 协议。这种在客户端和服务器端交换数据的替代方案引起了在浏览器和服务器上工作的开发人员的兴趣。明确了响应数量的增加和解决复杂的数据传输，SPDY 成为了 HTTP/2 协议的基础。

HTTP/2 在 HTTP/1.1 有几处基本的不同：

- HTTP/2 是二进制协议而不是文本协议。不再可读，也不可无障碍的手动创建，改善的优化技术现在可被实施。
- 这是一个多路复用协议。并行的请求能在同一个链接中处理，移除了 HTTP/1.x 中顺序和阻塞的约束。
- 压缩了标头。因为标头在一系列请求中常常是相似的，其移除了重复和传输重复数据的成本。
- 其允许服务器在客户端缓存中填充数据，通过一个叫服务器推送的机制来提前请求。

在 2015 年 5 月正式标准化后，HTTP/2 取得了极大的成功，在 2022 年 1 月达到峰值，占所有网站的 46.9%（见[这些统计数据](https://w3techs.com/technologies/details/ce-http2)）。高流量的站点最迅速的普及，在数据传输上节省了可观的成本和支出。

这种迅速的普及率很可能是因为 HTTP2 不需要站点和应用做出改变：使用 HTTP/1.1 和 HTTP/2 对他们来说是透明的。拥有一个最新的服务器和新点的浏览器进行交互就足够了。只有一小部分群体需要做出改变，而且随着陈旧的浏览器和服务器的更新，而不需 Web 开发者做什么，用的人自然就增加了。

### 1.7 后 HTTP/2 进化

随着 HTTP/2.的发布，就像先前的 HTTP/1.x 一样，HTTP 没有停止进化，HTTP 的扩展性依然被用来添加新的功能。特别的，我们能列举出 2016 年里 HTTP 的新扩展：

- 对 Alt-Svc 的支持允许了给定资源的位置和资源鉴定，允许了更智能的 CDN 缓冲机制。
- [客户端提示（client hint）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Client_hints) 的引入允许浏览器或者客户端来主动交流它的需求，或者是硬件约束的信息给服务端。
- 在 Cookie 头中引入安全相关的的前缀，现在帮助保证一个安全的 Cookie 没被更改过。

### 1.8 HTTP/3——基于 QUIC 的 HTTP

HTTP 的下一个主要版本，HTTP/3 有着与 HTTP 早期版本的相同语义，但在传输层部分使用 [QUIC (en-US)](https://developer.mozilla.org/en-US/docs/Glossary/QUIC "Currently only available in English (US)") 而不是 [TCP](https://developer.mozilla.org/zh-CN/docs/Glossary/TCP)。到 2022 年 10 月，[26% 的网站正在使用 HTTP/3](https://w3techs.com/technologies/details/ce-http3)。

QUIC 旨在为 HTTP 连接设计更低的延迟。类似于 HTTP/2，它是一个多路复用协议，但是 HTTP/2 通过单个 TCP 连接运行，所以在 TCP 层处理的数据包丢失检测和重传可以阻止所有流。QUIC 通过 [UDP](https://developer.mozilla.org/zh-CN/docs/Glossary/UDP) 运行多个流，并为每个流独立实现数据包丢失检测和重传，因此如果发生错误，只有该数据包中包含数据的流才会被阻止。

[RFC 9114](https://datatracker.ietf.org/doc/html/rfc9114) 定义的 HTTP/3 被[大多数主流浏览器所支持](https://caniuse.com/http3)，包括 Chromium（及其他的变体，例如 Chrome 和 Edge）和 Firefox。

## 2 HTTP/2 牛逼在哪？

![Pasted image 20231104133029.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4c475b66b29a48f6864964992112c0eb.png)


### 2.1 HTTP/1.1 协议的性能问题

我们得先要了解下 HTTP/1.1 协议存在的性能问题，因为 HTTP/2 协议就是把这些性能问题逐个攻破了。

现在的站点相比以前变化太多了，比如：
- _消息的大小变大了_，从几 KB 大小的消息，到几 MB 大小的消息；
- _页面资源变多了_，从每个页面不到 10 个的资源，到每页超 100 多个资源；
- _内容形式变多样了_，从单纯到文本内容，到图片、视频、音频等内容；
- _实时性要求变高了_，对页面的实时性要求的应用越来越多；

这些变化带来的最大性能问题就是 **HTTP/1.1 的高延迟**，延迟高必然影响的就是用户体验。主要原因如下几个：

- _延迟难以下降_，虽然现在网络的「带宽」相比以前变多了，但是延迟降到一定幅度后，就很难再下降了，说白了就是到达了延迟的下限；
- _并发连接有限_，谷歌浏览器最大并发连接数是 6 个，而且每一个连接都要经过 TCP 和 TLS 握手耗时，以及 TCP 慢启动过程给流量带来的影响；
- _队头阻塞问题_，同一连接只能在完成一个 HTTP 事务（请求和响应）后，才能处理下一个事务；
- _HTTP 头部巨大且重复_，由于 HTTP 协议是无状态的，每一个请求都得携带 HTTP 头部，特别是对于有携带 Cookie 的头部，而 Cookie 的大小通常很大；
- _不支持服务器推送消息_，因此当客户端需要获取通知时，只能通过定时器不断地拉取消息，这无疑浪费大量了带宽和服务器资源。

这里举例几个常见的优化手段：
- 将多张小图合并成一张大图供浏览器 JavaScript 来切割使用，这样可以将多个请求合并成一个请求，但是带来了新的问题，当某张小图片更新了，那么需要重新请求大图片，浪费了大量的网络带宽；
- 将图片的二进制数据通过 Base64 编码后，把编码数据嵌入到 HTML 或 CSS 文件中，以此来减少网络请求次数；
- 将多个体积较小的 JavaScript 文件使用 Webpack 等工具打包成一个体积更大的 JavaScript 文件，以一个请求替代了很多个请求，但是带来的问题，当某个 js 文件变化了，需要重新请求同一个包里的所有 js 文件；
- 将同一个页面的资源分散到不同域名，提升并发连接上限，因为浏览器通常对同一域名的 HTTP 连接最大只能是 6 个；

尽管对 HTTP/1.1 协议的优化手段如此之多，但是效果还是不尽人意，因为这些手段都是对 HTTP/1.1 协议的“外部”做优化，**而一些关键的地方是没办法优化的，比如请求-响应模型、头部巨大且重复、并发连接耗时、服务器不能主动推送等，要改变这些必须重新设计 HTTP 协议，于是 HTTP/2 就出来了！**

---

### 2.2 兼容 HTTP/1.1

HTTP/2 出来的目的是为了改善 HTTP 的性能。协议升级有一个很重要的地方，就是要**兼容**老版本的协议，否则新协议推广起来就相当困难，所幸 HTTP/2 做到了兼容 HTTP/1.1。

那么，HTTP/2 是怎么做的呢？

第一点，HTTP/2 没有在 URI 里引入新的协议名，仍然用「http://」表示明文协议，用「https://」表示加密协议，于是只需要浏览器和服务器在背后自动升级协议，这样可以让用户意识不到协议的升级，很好的实现了协议的平滑升级。

第二点，只在应用层做了改变，还是基于 TCP 协议传输，应用层方面为了保持功能上的兼容，HTTP/2 把 HTTP 分解成了「语义」和「语法」两个部分，「语义」层不做改动，与 HTTP/1.1 完全一致，比如请求方法、状态码、头字段等规则保留不变。

但是，HTTP/2 在「语法」层面做了很多改造，基本改变了 HTTP 报文的传输格式。

### 2.3 头部压缩

HTTP 协议的报文是由「Header + Body」构成的，对于 Body 部分，HTTP/1.1 协议可以使用头字段 「Content-Encoding」指定 Body 的压缩方式，比如用 gzip 压缩，这样可以节约带宽，但报文中的另外一部分 Header，是没有针对它的优化手段。

HTTP/1.1 报文中 Header 部分存在的问题：

- 含很多固定的字段，比如 Cookie、User Agent、Accept 等，这些字段加起来也高达几百字节甚至上千字节，所以有必要**压缩**；
- 大量的请求和响应的报文里有很多字段值都是重复的，这样会使得大量带宽被这些冗余的数据占用了，所以有必须要**避免重复性**；
- 字段是 ASCII 编码的，虽然易于人类观察，但效率低，所以有必要改成**二进制编码**；

HTTP/2 对 Header 部分做了大改造，把以上的问题都解决了。

HTTP/2 没使用常见的 gzip 压缩方式来压缩头部，而是开发了 **HPACK** 算法，HPACK 算法主要包含三个组成部分：

- 静态字典；
- 动态字典；
- Huffman 编码（压缩算法）；

客户端和服务器两端都会建立和维护「**字典**」，用长度较小的索引号表示重复的字符串，再用 Huffman 编码压缩数据，**可达到 50%~90% 的高压缩率**。

#### 2.3.1 静态表编码

HTTP/2 为高频出现在头部的字符串和字段建立了一张**静态表**，它是写入到 HTTP/2 框架里的，不会变化的，静态表里共有 `61` 组，如下图：

![Pasted image 20231104133213.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f39b4b3bb8215d46d6b397b3ff82abbf.png)


表中的 `Index` 表示索引（Key），`Header Value` 表示索引对应的 Value，`Header Name` 表示字段的名字，比如 Index 为 2 代表 GET，Index 为 8 代表状态码 200。

你可能注意到，表中有的 Index 没有对应的 Header Value，这是因为这些 Value 并不是固定的而是变化的，这些 Value 都会经过 Huffman 编码后，才会发送出去。

这么说有点抽象，我们来看个具体的例子，下面这个 `server` 头部字段，在 HTTP/1.1 的形式如下：

```http
server: nghttpx\r\n
```

算上冒号空格和末尾的`\r\n`，共占用了 17 字节，**而使用了静态表和 Huffman 编码，可以将它压缩成 8 字节，压缩率大概 47%**。

我抓了个 HTTP/2 协议的网络包，你可以从下图看到，高亮部分就是 `server` 头部字段，只用了 8 个字节来表示 `server` 头部数据。

![Pasted image 20231104133238.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6e32f8a16333a05c8f94590292888818.png)


根据 RFC7541 规范，如果头部字段属于静态表范围，并且 Value 是变化，那么它的 HTTP/2 头部前 2 位固定为 `01`，所以整个头部格式如下图：

![Pasted image 20231104133256.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/791446eef1e20fd090a1e75b554f6ab7.png)


HTTP/2 头部由于基于**二进制编码**，就不需要冒号空格和末尾的\r\n作为分隔符，于是改用表示字符串长度（Value Length）来分割 Index 和 Value。

接下来，根据这个头部格式来分析上面抓包的 `server` 头部的二进制数据。

首先，从静态表中能查到 `server` 头部字段的 Index 为 54，二进制为 110110，再加上固定 01，头部格式第 1 个字节就是 `01110110`，这正是上面抓包标注的红色部分的二进制数据。

然后，第二个字节的首个比特位表示 Value 是否经过 Huffman 编码，剩余的 7 位表示 Value 的长度，比如这次例子的第二个字节为 `10000110`，首位比特位为 1 就代表 Value 字符串是经过 Huffman 编码的，经过 Huffman 编码的 Value 长度为 6。

最后，字符串 `nghttpx` 经过 Huffman 编码后压缩成了 6 个字节，Huffman 编码的原理是将高频出现的信息用「较短」的编码表示，从而缩减字符串长度。

于是，在统计大量的 HTTP 头部后，HTTP/2 根据出现频率将 ASCII 码编码为了 Huffman 编码表，可以在 RFC7541 文档找到这张**静态 Huffman 表**，我就不把表的全部内容列出来了，我只列出字符串 `nghttpx` 中每个字符对应的 Huffman 编码，如下图：

![Pasted image 20231104133303.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/53b52554e87896db06a16363500f3e4e.png)


通过查表后，字符串 `nghttpx` 的 Huffman 编码在下图看到，共 6 个字节，每一个字符的 Huffman 编码，我用相同的颜色将他们对应起来了，最后的 7 位是补位的。
![Pasted image 20231104133309.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1f4c3469fd21a4f118eae2984519bba7.png)


最终，`server` 头部的二进制数据对应的静态头部格式如下：

![Pasted image 20231104133325.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/eb142e4f10470386c5c59389cec12891.png)


#### 2.3.2 动态表编码

静态表只包含了 61 种高频出现在头部的字符串，不在静态表范围内的头部字符串就要自行构建**动态表**，它的 Index 从 `62` 起步，会在编码解码的时候随时更新。

比如，第一次发送时头部中的「`User-Agent` 」字段数据有上百个字节，经过 Huffman 编码发送出去后，客户端和服务器双方都会更新自己的动态表，添加一个新的 Index 号 62。**那么在下一次发送的时候，就不用重复发这个字段的数据了，只用发 1 个字节的 Index 号就好了，因为双方都可以根据自己的动态表获取到字段的数据**。

所以，使得动态表生效有一个前提：**必须同一个连接上，重复传输完全相同的 HTTP 头部**。如果消息字段在 1 个连接上只发送了 1 次，或者重复传输时，字段总是略有变化，动态表就无法被充分利用了。

因此，随着在同一 HTTP/2 连接上发送的报文越来越多，客户端和服务器双方的「字典」积累的越来越多，理论上最终每个头部字段都会变成 1 个字节的 Index，这样便避免了大量的冗余数据的传输，大大节约了带宽。

理想很美好，现实很骨感。动态表越大，占用的内存也就越大，如果占用了太多内存，是会影响服务器性能的，因此 Web 服务器都会提供类似 `http2_max_requests` 的配置，用于限制一个连接上能够传输的请求数量，避免动态表无限增大，请求数量到达上限后，就会关闭 HTTP/2 连接来释放内存。

综上，HTTP/2 头部的编码通过「静态表、动态表、Huffman 编码」共同完成的。

![Pasted image 20231104133345.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/14de1d6c5c2ebefa398f4af81c6e5995.png)


---

### 2.4 二进制帧

HTTP/2 厉害的地方在于将 HTTP/1 的文本格式改成二进制格式传输数据，极大提高了 HTTP 传输效率，而且二进制数据使用位运算能高效解析。

你可以从下图看到，HTTP/1.1 的响应和 HTTP/2 的区别：

![Pasted image 20231104133400.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/12a400fa1b16b1e4b32fbab60691d000.png)


HTTP/2 把响应报文划分成了两类**帧（*Frame*）**，图中的 HEADERS（首部）和 DATA（消息负载） 是帧的类型，也就是说一条 HTTP 响应，划分成了两类帧来传输，并且采用二进制来编码。

比如状态码 200 ，在 HTTP/1.1 是用 '2''0''0' 三个字符来表示（二进制：00110010 00110000 00110000），共用了 3 个字节，如下图

![Pasted image 20231104133412.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f782bd8fe88252012fcfdc408237aad5.png)


在 HTTP/2 对于状态码 200 的二进制编码是 10001000，只用了 1 字节就能表示，相比于 HTTP/1.1 节省了 2 个字节，如下图：

![Pasted image 20231104133429.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/175c46a9ab1567f378fca558f0285197.png)


Header: :status: 200 OK 的编码内容为：1000 1000，那么表达的含义是什么呢？

![Pasted image 20231104133436.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0afb10ed58e4d9f0779938ba7a9c8d2f.png)


1. 最前面的 1 标识该 Header 是静态表中已经存在的 KV。
2. 我们再回顾一下之前的静态表内容，“:status: 200 OK”其静态表编码是 8，即 1000。

因此，整体加起来就是 1000 1000。

HTTP/2 **二进制帧**的结构如下图：


![Pasted image 20231104133443.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e9fe4686b4de4f4a7a2b2ba876bbc454.png)


帧头（Frame Header）很小，只有 9 个字节，帧开头的前 3 个字节表示帧数据（Frame Playload）的**长度**。

帧长度后面的一个字节是表示**帧的类型**，HTTP/2 总共定义了 10 种类型的帧，一般分为**数据帧**和**控制帧**两类，如下表格：
![Pasted image 20231104133451.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/217f65414e76949d4bd3266e765dfcb1.png)


帧类型后面的一个字节是**标志位**，可以保存 8 个标志位，用于携带简单的控制信息，比如：

- **END_HEADERS** 表示头数据结束标志，相当于 HTTP/1 里头后的空行（“`\r\n`”）；
- **END_Stream** 表示单方向数据发送结束，后续不会再有数据帧。
- **PRIORITY** 表示流的优先级；

帧头的最后 4 个字节是**流标识符**（Stream ID），但最高位被保留不用，只有 31 位可以使用，因此流标识符的最大值是 2^31，大约是 21 亿，它的作用是用来标识该 Frame 属于哪个 Stream，接收方可以根据这个信息从乱序的帧里找到相同 Stream ID 的帧，从而有序组装信息。

最后面就是**帧数据**了，它存放的是通过 **HPACK 算法**压缩过的 HTTP 头部和包体。

---

### 2.5 并发传输

知道了 HTTP/2 的帧结构后，我们再来看看它是如何实现**并发传输**的。

我们都知道 HTTP/1.1 的实现是基于请求-响应模型的。同一个连接中，HTTP 完成一个事务（请求与响应），才能处理下一个事务，也就是说在发出请求等待响应的过程中，是没办法做其他事情的，如果响应迟迟不来，那么后续的请求是无法发送的，也造成了**队头阻塞**的问题。

而 HTTP/2 就很牛逼了，通过 Stream 这个设计，**多个 Stream 复用一条 TCP 连接，达到并发的效果**，解决了 HTTP/1.1 队头阻塞的问题，提高了 HTTP 传输的吞吐量。

为了理解 HTTP/2 的并发是怎样实现的，我们先来理解 HTTP/2 中的 Stream、Message、Frame 这 3 个概念。

![Pasted image 20231104133523.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e8a00d916c2e9112aaa99258980a776e.png)


你可以从上图中看到：

- 1 个 TCP 连接包含一个或者多个 Stream，Stream 是 HTTP/2 并发的关键技术；
- Stream 里可以包含 1 个或多个 Message，Message 对应 HTTP/1 中的请求或响应，由 HTTP 头部和包体构成；
- Message 里包含一条或者多个 Frame，Frame 是 HTTP/2 最小单位，以二进制压缩格式存放 HTTP/1 中的内容（头部和包体）；

因此，我们可以得出个结论：多个 Stream 跑在一条 TCP 连接，同一个 HTTP 请求与响应是跑在同一个 Stream 中，HTTP 消息可以由多个 Frame 构成， 一个 Frame 可以由多个 TCP 报文构成。

![Pasted image 20231104133534.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5c25309d0f6dedceb0d2b528e47c4247.png)


在 HTTP/2 连接上，**不同 Stream 的帧是可以乱序发送的（因此可以并发不同的 Stream ）**，因为每个帧的头部会携带 Stream ID 信息，所以接收端可以通过 Stream ID 有序组装成 HTTP 消息，而**同一 Stream 内部的帧必须是严格有序的**。

比如下图，服务端**并行交错地**发送了两个响应： Stream 1 和 Stream 3，这两个 Stream 都是跑在一个 TCP 连接上，客户端收到后，会根据相同的 Stream ID 有序组装成 HTTP 消息。

![Pasted image 20231104133541.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/59de279689b1413ccbcf54d0acb405e1.png)


客户端和服务器**双方都可以建立 Stream**，因为服务端可以主动推送资源给客户端， 客户端建立的 Stream 必须是奇数号，而服务器建立的 Stream 必须是偶数号。

比如下图，Stream 1 是客户端向服务端请求的资源，属于客户端建立的 Stream，所以该 Stream 的 ID 是奇数（数字 1）；Stream 2 和 4 都是服务端主动向客户端推送的资源，属于服务端建立的 Stream，所以这两个 Stream 的 ID 是偶数（数字 2 和 4）。

![Pasted image 20231104133547.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7798e25775f660f9343cfef073d213dd.png)


同一个连接中的 Stream ID 是不能复用的，只能顺序递增，所以当 Stream ID 耗尽时，需要发一个控制帧 `GOAWAY`，用来关闭 TCP 连接。

在 Nginx 中，可以通过 `http2_max_concurrent_Streams` 配置来设置 Stream 的上限，默认是 128 个。

HTTP/2 通过 Stream 实现的并发，比 HTTP/1.1 通过 TCP 连接实现并发要牛逼的多，**因为当 HTTP/2 实现 100 个并发 Stream 时，只需要建立一次 TCP 连接，而 HTTP/1.1 需要建立 100 个 TCP 连接，每个 TCP 连接都要经过 TCP 握手、慢启动以及 TLS 握手过程，这些都是很耗时的。**

HTTP/2 还可以对每个 Stream 设置不同**优先级**，帧头中的「标志位」可以设置优先级，比如客户端访问 HTML/CSS 和图片资源时，希望服务器先传递 HTML/CSS，再传图片，那么就可以通过设置 Stream 的优先级来实现，以此提高用户体验。

### 2.6 服务器主动推送资源

HTTP/1.1 不支持服务器主动推送资源给客户端，都是由客户端向服务器发起请求后，才能获取到服务器响应的资源。

比如，客户端通过 HTTP/1.1 请求从服务器那获取到了 HTML 文件，而 HTML 可能还需要依赖 CSS 来渲染页面，这时客户端还要再发起获取 CSS 文件的请求，需要两次消息往返，如下图左边部分：

![Pasted image 20231104133603.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2be052951e47da174ec89d2aa22af8ce.png)


如上图右边部分，在 HTTP/2 中，客户端在访问 HTML 时，服务器可以直接主动推送 CSS 文件，减少了消息传递的次数。

在 Nginx 中，如果你希望客户端访问 /test.html 时，服务器直接推送 /test.css，那么可以这么配置：

```
location /test.html { 
  http2_push /test.css; 
}
```

那 HTTP/2 的推送是怎么实现的？

客户端发起的请求，必须使用的是奇数号 Stream，服务器主动的推送，使用的是偶数号 Stream。服务器在推送资源时，会通过 `PUSH_PROMISE` 帧传输 HTTP 头部，并通过帧中的 `Promised Stream ID` 字段告知客户端，接下来会在哪个偶数号 Stream 中发送包体。

![Pasted image 20231104133612.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d70740bd382b6333a438cdd8fa199321.png)


如上图，在 Stream 1 中通知客户端 CSS 资源即将到来，然后在 Stream 2 中发送 CSS 资源，注意 Stream 1 和 2 是可以**并发**的。

---

### 2.7 总结

HTTP/2 协议其实还有很多内容，比如流控制、流状态、依赖关系等等。

这次主要介绍了关于 HTTP/2 是如何提升性能的几个方向，它相比 HTTP/1 大大提高了传输效率、吞吐能力。

第一点，对于常见的 HTTP 头部通过**静态表和 Huffman 编码**的方式，将体积压缩了近一半，而且针对后续的请求头部，还可以建立**动态表**，将体积压缩近 90%，大大提高了编码效率，同时节约了带宽资源。

不过，动态表并非可以无限增大， 因为动态表是会占用内存的，动态表越大，内存也越大，容易影响服务器总体的并发能力，因此服务器需要限制 HTTP/2 连接时长或者请求次数。

第二点，**HTTP/2 实现了 Stream 并发**，多个 Stream 只需复用 1 个 TCP 连接，节约了 TCP 和 TLS 握手时间，以及减少了 TCP 慢启动阶段对流量的影响。不同的 Stream ID 可以并发，即使乱序发送帧也没问题，比如发送 A 请求帧 1 -> B 请求帧 1 -> A 请求帧 2 -> B 请求帧2，但是同一个 Stream 里的帧必须严格有序。

另外，可以根据资源的渲染顺序来设置 Stream 的**优先级**，从而提高用户体验。

第三点，**服务器支持主动推送资源**，大大提升了消息的传输性能，服务器推送资源时，会先发送 PUSH_PROMISE 帧，告诉客户端接下来在哪个 Stream 发送资源，然后用偶数号 Stream 发送资源给客户端。

HTTP/2 通过 Stream 的并发能力，解决了 HTTP/1 队头阻塞的问题，看似很完美了，但是 HTTP/2 还是存在“队头阻塞”的问题，只不过问题不是在 HTTP 这一层面，而是在 TCP 这一层。

**HTTP/2 是基于 TCP 协议来传输数据的，TCP 是字节流协议，TCP 层必须保证收到的字节数据是完整且连续的，这样内核才会将缓冲区里的数据返回给 HTTP 应用，那么当「前 1 个字节数据」没有到达时，后收到的字节数据只能存放在内核缓冲区里，只有等到这 1 个字节数据到达时，HTTP/2 应用层才能从内核中拿到数据，这就是 HTTP/2 队头阻塞问题。**

有没有什么解决方案呢？既然是 TCP 协议自身的问题，那干脆放弃 TCP 协议，转而使用 UDP 协议作为传输层协议，这个大胆的决定，HTTP/3 协议做了！

![Pasted image 20231104133628.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f1073ce65d5327427c9741b168abfcf3.png)

## 3 HTTP/3 强势来袭

HTTP/3 现在（2022 年 5 月）还没正式推出，不过自 2017 年起，HTTP/3 已经更新到 34 个草案了，基本的特性已经确定下来了，对于包格式可能后续会有变化。

所以，这次 HTTP/3 介绍不会涉及到包格式，只说它的特性。

![Pasted image 20231104133652.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a54e5db400c166351e84338444aaaa1.png)


### 3.1 美中不足的 HTTP/2

HTTP/2 通过头部压缩、二进制编码、多路复用、服务器推送等新特性大幅度提升了 HTTP/1.1 的性能，而美中不足的是 HTTP/2 协议是基于 TCP 实现的，于是存在的缺陷有三个。

- 队头阻塞；
- TCP 与 TLS 的握手时延迟；
- 网络迁移需要重新连接；

#### 3.1.1 队头阻塞

HTTP/2 多个请求是跑在一个 TCP 连接中的，那么当 TCP 丢包时，整个 TCP 都要等待重传，那么就会阻塞该 TCP 连接中的所有请求。

比如下图中，Stream 2 有一个 TCP 报文丢失了，那么即使收到了 Stream 3 和 Stream 4 的 TCP 报文，应用层也是无法读取读取的，相当于阻塞了 Stream 3 和 Stream 4 请求。

![Pasted image 20231104133709.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/58b0ad1712185a9140914e67451724bd.png)


因为 TCP 是字节流协议，TCP 层必须保证收到的字节数据是完整且有序的，如果序列号较低的 TCP 段在网络传输中丢失了，即使序列号较高的 TCP 段已经被接收了，应用层也无法从内核中读取到这部分数据，从 HTTP 视角看，就是请求被阻塞了。

举个例子，如下图：

![Pasted image 20231104133714.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/74c82cace37a7a4633b2b32b5dafa101.png)


图中发送方发送了很多个 Packet，每个 Packet 都有自己的序号，你可以认为是 TCP 的序列号，其中 Packet 3 在网络中丢失了，即使 Packet 4-6 被接收方收到后，由于内核中的 TCP 数据不是连续的，于是接收方的应用层就无法从内核中读取到，只有等到 Packet 3 重传后，接收方的应用层才可以从内核中读取到数据，这就是 HTTP/2 的队头阻塞问题，是在 TCP 层面发生的。

#### 3.1.2 TCP 与 TLS 的握手时延迟

发起 HTTP 请求时，需要经过 TCP 三次握手和 TLS 四次握手（TLS 1.2）的过程，因此共需要 3 个 RTT 的时延才能发出请求数据。

![Pasted image 20231104133728.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7c57a6be2256022cee2617eb4f448f22.png)


另外，TCP 由于具有「拥塞控制」的特性，所以刚建立连接的 TCP 会有个「慢启动」的过程，它会对 TCP 连接产生“减速”效果。

#### 3.1.3 网络迁移需要重新连接

一个 TCP 连接是由四元组（源 IP 地址，源端口，目标 IP 地址，目标端口）确定的，这意味着如果 IP 地址或者端口变动了，就会导致需要 TCP 与 TLS 重新握手，这不利于移动设备切换网络的场景，比如 4G 网络环境切换成 WiFi。

这些问题都是 TCP 协议固有的问题，无论应用层的 HTTP/2 在怎么设计都无法逃脱。要解决这个问题，就必须把**传输层协议替换成 UDP**，这个大胆的决定，HTTP/3 做了！

![Pasted image 20231104133739.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f1073ce65d5327427c9741b168abfcf3.png)


### 3.2 QUIC 协议的特点

我们深知，UDP 是一个简单、不可靠的传输协议，而且是 UDP 包之间是无序的，也没有依赖关系。

而且，UDP 是不需要连接的，也就不需要握手和挥手的过程，所以天然的就比 TCP 快。

当然，HTTP/3 不仅仅只是简单将传输协议替换成了 UDP，还基于 UDP 协议在「应用层」实现了 **QUIC 协议**，它具有类似 TCP 的连接管理、拥塞窗口、流量控制的网络特性，相当于将不可靠传输的 UDP 协议变成“可靠”的了，所以不用担心数据包丢失的问题。

QUIC 协议的优点有很多，这里举例几个，比如：

- 无队头阻塞；
- 更快的连接建立；
- 连接迁移；

#### 3.2.1 无队头阻塞

QUIC 协议也有类似 HTTP/2 Stream 与多路复用的概念，也是可以在同一条连接上并发传输多个 Stream，Stream 可以认为就是一条 HTTP 请求。

由于 QUIC 使用的传输协议是 UDP，UDP 不关心数据包的顺序，如果数据包丢失，UDP 也不关心。

不过 QUIC 协议会保证数据包的可靠性，每个数据包都有一个序号唯一标识。当某个流中的一个数据包丢失了，即使该流的其他数据包到达了，数据也无法被 HTTP/3 读取，直到 QUIC 重传丢失的报文，数据才会交给 HTTP/3。

而其他流的数据报文只要被完整接收，HTTP/3 就可以读取到数据。这与 HTTP/2 不同，HTTP/2 只要某个流中的数据包丢失了，其他流也会因此受影响。

所以，QUIC 连接上的多个 Stream 之间并没有依赖，都是独立的，某个流发生丢包了，只会影响该流，其他流不受影响。

![Pasted image 20231104133754.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ea210dba758c4ae57f6ac637d8ed10f5.png)


#### 3.2.2 更快的连接建立

对于 HTTP/1 和 HTTP/2 协议，TCP 和 TLS 是分层的，分别属于内核实现的传输层、OpenSSL 库实现的表示层，因此它们难以合并在一起，需要分批次来握手，先 TCP 握手，再 TLS 握手。

HTTP/3 在传输数据前虽然需要 QUIC 协议握手，这个握手过程只需要 1 RTT，握手的目的是为确认双方的「连接 ID」，连接迁移就是基于连接 ID 实现的。

但是 HTTP/3 的 QUIC 协议并不是与 TLS 分层，而是 **QUIC 内部包含了 TLS，它在自己的帧会携带 TLS 里的“记录”，再加上 QUIC 使用的是 TLS 1.3，因此仅需 1 个 RTT 就可以「同时」完成建立连接与密钥协商，甚至在第二次连接的时候，应用数据包可以和 QUIC 握手信息（连接信息 + TLS 信息）一起发送，达到 0-RTT 的效果**。

如下图右边部分，HTTP/3 当会话恢复时，有效负载数据与第一个数据包一起发送，可以做到 0-RTT：


#### 3.2.3 连接迁移

在前面我们提到，基于 TCP 传输协议的 HTTP 协议，由于是通过四元组（源 IP、源端口、目的 IP、目的端口）确定一条 TCP 连接。

![Pasted image 20231104133837.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d61014f3772aa1b3aa2171c951a880f5.png)


那么当移动设备的网络从 4G 切换到 WiFi 时，意味着 IP 地址变化了，那么就必须要断开连接，然后重新建立连接，而建立连接的过程包含 TCP 三次握手和 TLS 四次握手的时延，以及 TCP 慢启动的减速过程，给用户的感觉就是网络突然卡顿了一下，因此连接的迁移成本是很高的。

而 QUIC 协议没有用四元组的方式来“绑定”连接，而是通过**连接 ID** 来标记通信的两个端点，客户端和服务器可以各自选择一组 ID 来标记自己，因此即使移动设备的网络变化后，导致 IP 地址变化了，只要仍保有上下文信息（比如连接 ID、TLS 密钥等），就可以“无缝”地复用原连接，消除重连的成本，没有丝毫卡顿感，达到了**连接迁移**的功能。

### 3.3 HTTP/3 协议

了解完 QUIC 协议的特点后，我们再来看看 HTTP/3 协议在 HTTP 这一层做了什么变化。

HTTP/3 同 HTTP/2 一样采用二进制帧的结构，不同的地方在于 HTTP/2 的二进制帧里需要定义 Stream，而 HTTP/3 自身不需要再定义 Stream，直接使用 QUIC 里的 Stream，于是 HTTP/3 的帧的结构也变简单了。

![Pasted image 20231104133848.png|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/29922a9eecf20891e5f6998008962c15.png)


从上图可以看到，HTTP/3 帧头只有两个字段：类型和长度。

根据帧类型的不同，大体上分为数据帧和控制帧两大类，Headers 帧（HTTP 头部）和 DATA 帧（HTTP 包体）属于数据帧。

HTTP/3 在头部压缩算法这一方面也做了升级，升级成了 **QPACK**。与 HTTP/2 中的 HPACK 编码方式相似，HTTP/3 中的 QPACK 也采用了静态表、动态表及 Huffman 编码。

对于静态表的变化，HTTP/2 中的 HPACK 的静态表只有 61 项，而 HTTP/3 中的 QPACK 的静态表扩大到 91 项。

HTTP/2 和 HTTP/3 的 Huffman 编码并没有多大不同，但是动态表编解码方式不同。

所谓的动态表，在首次请求-响应后，双方会将未包含在静态表中的 Header 项更新各自的动态表，接着后续传输时仅用 1 个数字表示，然后对方可以根据这 1 个数字从动态表查到对应的数据，就不必每次都传输长长的数据，大大提升了编码效率。

可以看到，**动态表是具有时序性的，如果首次出现的请求发生了丢包，后续的收到请求，对方就无法解码出 HPACK 头部，因为对方还没建立好动态表，因此后续的请求解码会阻塞到首次请求中丢失的数据包重传过来**。

HTTP/3 的 QPACK 解决了这一问题，那它是如何解决的呢？

QUIC 会有两个特殊的单向流，所谓的单向流只有一端可以发送消息，双向则指两端都可以发送消息，传输 HTTP 消息时用的是双向流，这两个单向流的用法：

- 一个叫 QPACK Encoder Stream，用于将一个字典（Key-Value）传递给对方，比如面对不属于静态表的 HTTP 请求头部，客户端可以通过这个 Stream 发送字典；
- 一个叫 QPACK Decoder Stream，用于响应对方，告诉它刚发的字典已经更新到自己的本地动态表了，后续就可以使用这个字典来编码了。

这两个特殊的单向流是用来**同步双方的动态表**，编码方收到解码方更新确认的通知后，才使用动态表编码 HTTP 头部。

### 3.4 总结

HTTP/2 虽然具有多个流并发传输的能力，但是传输层是 TCP 协议，于是存在以下缺陷：

- **队头阻塞**，HTTP/2 多个请求跑在一个 TCP 连接中，如果序列号较低的 TCP 段在网络传输中丢失了，即使序列号较高的 TCP 段已经被接收了，应用层也无法从内核中读取到这部分数据，从 HTTP 视角看，就是多个请求被阻塞了；
- **TCP 和 TLS 握手时延**，TCP 三次握手和 TLS 四次握手，共有 3-RTT 的时延；
- **连接迁移需要重新连接**，移动设备从 4G 网络环境切换到 WiFi 时，由于 TCP 是基于四元组来确认一条 TCP 连接的，那么网络环境变化后，就会导致 IP 地址或端口变化，于是 TCP 只能断开连接，然后再重新建立连接，切换网络环境的成本高；

HTTP/3 就将传输层从 TCP 替换成了 UDP，并在 UDP 协议上开发了 QUIC 协议，来保证数据的可靠传输。

QUIC 协议的特点：

- **无队头阻塞**，QUIC 连接上的多个 Stream 之间并没有依赖，都是独立的，也不会有底层协议限制，某个流发生丢包了，只会影响该流，其他流不受影响；
- **建立连接速度快**，因为 QUIC 内部包含 TLS 1.3，因此仅需 1 个 RTT 就可以「同时」完成建立连接与 TLS 密钥协商，甚至在第二次连接的时候，应用数据包可以和 QUIC 握手信息（连接信息 + TLS 信息）一起发送，达到 0-RTT 的效果。
- **连接迁移**，QUIC 协议没有用四元组的方式来“绑定”连接，而是通过「连接 ID 」来标记通信的两个端点，客户端和服务器可以各自选择一组 ID 来标记自己，因此即使移动设备的网络变化后，导致 IP 地址变化了，只要仍保有上下文信息（比如连接 ID、TLS 密钥等），就可以“无缝”地复用原连接，消除重连的成本；

另外 HTTP/3 的 QPACK 通过两个特殊的单向流来同步双方的动态表，解决了 HTTP/2 的 HPACK 队头阻塞问题。