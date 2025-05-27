---
文章标题: "[[1_※认识HTTP]]" 
文章作者: Dakkk
文章概要: |
  本文全面阐述了HTTP协议，从其诞生背景、Web运作机制、TCP/IP协议基础，到请求响应报文结构、HTTP方法、状态码详解。文章还深入探讨了持久连接、Cookie状态管理、Web服务器协作方式（代理、网关、隧道）及缓存机制，并详细介绍了各类HTTP首部字段及其作用。
tags:
- "HTTP协议"
- "Web基础"
- "TCP/IP"
- "HTTP方法"
- "状态码"
- "报文首部"
- "缓存"
- "Cookie"
相关文章:
- "[[5_XML和HTTP]]"
- "[[5_※HTTP状态码]]"
- "[[1_缓存基础面试与分析]]"
- "[[1_Redis攻略]]"
- "[[1_Redis快速入门]]"
文章分类: "📐 计算机基础"
文章路径: "07-📐 计算机基础/04-🌐 计算机网络/2_应用层/1_HTTP/1_※认识HTTP.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:44:02
---

## 1 了解Web及网络基础

本章概述了 Web 是建立在何种技术之上，以及 HTTP 协议是如何诞 生并发展的。我们从其背景着手，来深入了解这部分内容。
### 1.1 使用HTTP协议访问Web

你知道当我们在网页浏览器（Web browser）的地址栏中输入 URL 时，Web 页面是如何呈现的吗？
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dfab5c0a4a0aba62e63f57d38cb91a75.png)

Web 页面当然不能凭空显示出来。根据 Web 浏览器地址栏中指定的 URL，Web 浏览器从 Web 服务器端获取文件资源（resource）等信 息，从而显示出 Web 页面。 像这种通过发送请求获取服务器资源的 Web 浏览器等，都可称为客 户端（client）。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/447d6f0234ca64d785f7831cd93b3c85.png)

Web 使用一种名为 HTTP（HyperText Transfer Protocol，超文本传输协议 ）的协议`作为规范`，`完成从客户端到服务器端等一系列运作流程`。而协议是指规则的约定。可以说，Web 是建立在 HTTP 协议上通 信的。
### 1.2 HTTP的诞生

在深入学习 HTTP 之前，我们先来介绍一下 HTTP 诞生的背景。了解 背景的同时也能了解当初制定 HTTP 的初衷，这样有助于我们更好地 理解。 

#### 1.2.1 为知识共享而规划 Web 

1989 年 3 月，互联网还只属于少数人。在这一互联网的黎明期， HTTP 诞生了。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7a5266ede4c470788ca8783c8c9c2723.png)

CERN（欧洲核子研究组织）的蒂姆 • 伯纳斯 - 李（Tim BernersLee） 博士提出了一种能让远隔两地的研究者们共享知识的设想。 

最初设想的基本理念是：借助多文档之间相互关联形成的超文本 （HyperText），连成可相互参阅的 WWW（World Wide Web，万维 网）。 

现在已提出了 3 项 WWW 构建技术，分别是：把 SGML（Standard Generalized Markup Language，标准通用标记语言）作为页面的文本标 记语言的 HTML（HyperText Markup Language，超文本标记语言）； 作为文档传递协议的 HTTP ；指定文档所在地址的 URL（Uniform Resource Locator，统一资源定位符）。 

==WWW 这一名称，是 Web 浏览器当年用来浏览超文本的客户端应用 程序时的名称。现在则用来表示这一系列的集合，也可简称为 Web==

#### 1.2.2 Web 成长时代 

1990 年 11 月，CERN 成功研发了世界上第一台 Web 服务器和 Web 浏览器。两年后的 1992 年 9 月，日本第一个网站的主页上线了。 

- 日本第一个主页 http://www.ibarakiken.gr.jp/www/ 

1990 年，大家针对 HTML1.0 草案进行了讨论，因 HTML1.0 中存在 多处模糊不清的部分，草案被直接废弃了。 

- HTML1.0 http://www.w3.org/MarkUp/draft-ietf-iiir-html-01.txt 

1993 年 1 月，现代浏览器的祖先 NCSA（National Center for Supercomputer Applications，美国国家超级计算机应用中心）研发的 `Mosaic `问世了。它以 in-line（内联）等形式显示 HTML的图像，在 图像方面出色的表现使它迅速在世界范围内流行开来。 

同年秋天，Mosaic 的 Windows 版和 Macintosh 版面世。使用 CGI 技 术的 NCSA Web 服务器、NCSA HTTPd 1.0 也差不多是在这个时期出 现的。 

- NCSA Mosaic bounce page http://archive.ncsa.illinois.edu/mosaic.html 
- The NCSA HTTPd Home Page（存档） http://web.archive.org/web/20090426182129/http://hoohoo.ncsa.illinois.edu/ （原址已失效） 

1994 年 的 12 月，网景通信公司发布了 Netscape Navigator 1.0，1995 年微软公司发布 Internet Explorer 1.0 和 2.0。 

紧随其后的是现在已然成为 Web 服务器标准之一的 Apache，当时它 以 Apache 0.2 的姿态出现在世人眼前。而 HTML也发布了 2.0 版本。 那一年，Web 技术的发展突飞猛进。 

时光流转，从 1995 年左右起，微软公司与网景通信公司之间爆发的 浏览器大战愈演愈烈。两家公司都各自对 HTML做了扩展，于是导 致在写 HTML页面时，必须考虑兼容他们两家公司的浏览器。时至 今日，这个问题仍令那些写前端页面的工程师感到棘手。 

在这场==浏览器供应商之间的竞争==中，他们不仅对当时发展中的各种 Web 标准化视而不见，还屡次出现新增功能没有对应说明文档的情 况。 

2000 年前后，这场浏览器战争随着网景通信公司的衰落而暂告一段 落。但就在 2004 年，Mozilla 基金会发布了 Firefox 浏览器，第二次 浏览器大战随即爆发。 

Internet Explorer 浏览器的版本从 6 升到 7 前后花费了 5 年时间。之后 接连不断地发布了 8、9、10 版本。另外，Chrome、Opera、Safari 等浏览器也纷纷抢占市场份额。

#### 1.2.3 驻足不前的 HTTP 

`HTTP/0.9` 
- HTTP 于 1990 年问世。那时的 HTTP 并没有作为正式的标准被建立。 现在的 HTTP 其实含有 HTTP1.0 之前版本的意思，因此被称为 HTTP/0.9。 

`HTTP/1.0 `
- HTTP `正式作为标准`被公布是在 1996 年的 5 月，版本被命名为 HTTP/1.0，并记载于 RFC1945。虽说是初期标准，但该协议标准至今 仍被广泛使用在服务器端。 
	- RFC1945 - Hypertext Transfer Protocol -- HTTP/1.0 http://www.ietf.org/rfc/rfc1945.txt 

`HTTP/1.1`
- 1997 年 1 月公布的 HTTP/1.1 是`目前主流的 HTTP 协议版本`。当初的 标准是 RFC2068，之后发布的修订版 RFC2616 就是当前的最新版本。 
	- RFC2616 - Hypertext Transfer Protocol -- HTTP/1.1 http://www.ietf.org/rfc/rfc2616.txt 

可见，作为 Web 文档传输协议的 HTTP，它的版本几乎没有更新。新 一代 HTTP/2.0 正在制订中，但要达到较高的使用覆盖率，仍需假以时日。 

当年 HTTP 协议的出现`主要是为了解决文本传输的难题`。由于协议本身非常简单，于是在此基础上设想了很多应用方法并投入了实际使 用。现在 HTTP 协议已经超出了 Web 这个框架的局限，被运用到了 各种场景里。
### 1.3 网络基础TCP/IP

为了理解 HTTP，我们有必要事先了解一下 TCP/IP 协议族。 

通常使用的网络（包括互联网）是在 TCP/IP 协议族的基础上运作 的。而 HTTP 属于它内部的一个子集。 

接下来，我们仅介绍理解 HTTP 所需掌握的 TCP/IP 协议族的概要。 若想进一步学习有关 TCP/IP 的知识，请参考其他讲解 TCP/IP 的专业 书籍。

#### 1.3.1 TCP/IP 协议族 

计算机与网络设备要相互通信，`双方就必须基于相同的方法`。比如， 如何探测到通信目标、由哪一边先发起通信、使用哪种语言进行通 信、怎样结束通信等规则都需要事先确定。不同的硬件、操作系统之 间的通信，所有的这一切都需要一种规则。而我们就把这种规则称为 协议（protocol）。

![](assets/Pasted%20image%2020231026012805.png )
"图：TCP/IP 是互联网相关的各类协议族的总称"

协议中存在各式各样的内容。从电缆的规格到 IP 地址的选定方法、 寻找异地用户的方法、双方建立通信的顺序，以及 Web 页面显示需 要处理的步骤，等等。 

像这样把与互联网相关联的协议集合起来总称为 TCP/IP。也有说法 认为，TCP/IP 是指 TCP 和 IP 这两种协议。还有一种说法认为，TCP/ IP 是在 IP 协议的通信过程中，使用到的协议族的统称。

#### 1.3.2 TCP/IP 的分层管理 

TCP/IP 协议族里重要的一点就是`分层`。TCP/IP 协议族按层次分别分 为以下 4 层：应用层、传输层、网络层和数据链路层。 

把 TCP/IP 层次化是有好处的。比如，如果互联网只由一个协议统 筹，某个地方需要改变设计时，就必须把所有部分整体替换掉。而分 层之后只需把变动的层替换掉即可。把各层之间的接口部分规划好之 后，每个层次内部的设计就能够自由改动了。 

值得一提的是，层次化之后，设计也变得相对简单了。处于应用层上 的应用可以只考虑分派给自己的任务，而不需要弄清对方在地球上哪 个地方、对方的传输路线是怎样的、是否能确保传输送达等问题。 TCP/IP 协议族各层的作用如下。 

`应用层`
- 应用层决定了向用户提供应用服务时通信的活动。 
- TCP/IP 协议族内预存了各类通用的应用服务。比如，FTP（File Transfer Protocol，文件传输协议）和 DNS（Domain Name System，域 名系统）服务就是其中两类。 
- HTTP 协议也处于该层。

`传输层` 
- 传输层对上层应用层，提供处于网络连接中的两台计算机之间的数据传输。 
- 在传输层有两个性质不同的协议：TCP（Transmission Control Protocol，传输控制协议）和 UDP（User Data Protocol，用户数据报协议）。 

`网络层（又名网络互连层）` 
- 网络层用来处理在网络上流动的数据包。数据包是网络传输的最小数据单位。该层规定了通过怎样的路径（所谓的传输路线）到达对方计 算机，并把数据包传送给对方。 
- 与对方计算机之间通过多台计算机或网络设备进行传输时，网络层所起的作用就是在众多的选项内选择一条传输路线。 

`链路层（又名数据链路层，网络接口层）` 
- 用来处理连接网络的硬件部分。包括控制操作系统、硬件的设备驱 动、NIC（Network Interface Card，网络适配器，即网卡），及光纤等 物理可见部分（还包括连接器等一切传输媒介）。硬件上的范畴均在 链路层的作用范围之内。

#### 1.3.3 TCP/IP 通信传输流

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a73bbf4e332d79345e3c4058766988a4.png)

利用 TCP/IP 协议族进行网络通信时，会通过分层顺序与对方进行通 信。发送端从应用层往下走，接收端则往应用层往上走。

我们用 HTTP 举例来说明，首先作为发送端的客户端在应用层 （HTTP 协议）发出一个想看某个 Web 页面的 HTTP 请求。 

接着，为了传输方便，在传输层（TCP 协议）把从应用层处收到的数 据（HTTP 请求报文）进行`分割`，并在各个报文上`打上标记序号`及`端口号`后转发给网络层。 

在网络层（IP 协议），增加作为通信目的地的 MAC 地址后转发给链 路层。这样一来，发往网络的通信请求就准备齐全了。 

接收端的服务器在链路层接收到数据，按序往上层发送，一直到应用 层。当传输到应用层，才能算真正接收到由客户端发送过来的 HTTP 请求。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a07834d2e05d9d842c773340defd6498.png)

发送端在层与层之间传输数据时，每经过一层时必定会被打上一个该 层所属的首部信息。反之，接收端在层与层传输数据时，每经过一层 时会把对应的首部消去。 

这种把数据信息包装起来的做法称为封装（encapsulate）。
### 1.4 与HTTP关系密切的协议：IP、TCP和DNS

下面我们分别针对在 TCP/IP 协议族中与 HTTP 密不可分的 3 个协议 （IP、TCP 和 DNS）进行说明。

#### 1.4.1 负责传输的 IP 协议

按层次分，IP（Internet Protocol）网际协议位于网络层。Internet Protocol 这个名称可能听起来有点夸张，但事实正是如此，因为几乎 所有使用网络的系统都会用到 IP 协议。TCP/IP 协议族中的 IP 指的就 是网际协议，协议名称中占据了一半位置，其重要性可见一斑。可能 有人会把“IP”和“IP 地址”搞混，“IP”其实是一种协议的名称。

`IP 协议的作用是把各种数据包传送给对方`。而要保证确实传送到对方 那里，则需要满足各类条件。其中两个重要的条件是 `IP 地址`和 `MAC 地址`（Media Access Control Address）。 

IP 地址指明了节点被分配到的地址，MAC 地址是指网卡所属的固定 地址。IP 地址可以和 MAC 地址进行配对。`IP 地址可变换，但 MAC 地址基本上不会更改`。 

使用 ARP 协议凭借 MAC 地址进行通信 

`IP 间的通信依赖 MAC 地址`。在网络上，通信的双方在同一局域网 （LAN）内的情况是很少的，通常是经过多台计算机和网络设备中转 才能连接到对方。而在进行中转时，会`利用下一站中转设备的 MAC 地址来搜索下一个中转目标`。这时，会采用 ARP 协议（Address Resolution Protocol）。`ARP 是一种用以解析地址的协议，根据通信方 的 IP 地址就可以反查出对应的 MAC 地址`。 

没有人能够全面掌握互联网中的传输状况 

在到达通信目标前的中转过程中，那些计算机和路由器等网络设备只 能获悉很粗略的传输路线。 

这种机制称为路由选择（routing），有点像快递公司的送货过程。想要寄快递的人，只要将自己的货物送到集散中心，就可以知道快递公司是否肯收件发货，该快递公司的集散中心检查货物的送达地址，明确下站该送往哪个区域的集散中心。接着，那个区域的集散中心自会判断是否能送到对方的家中。 

我们是想通过这个比喻说明，无论哪台计算机、哪台网络设备，它们 都无法全面掌握互联网中的细节。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2f754a099905593201efd37bc72cfec4.png)

#### 1.4.2 确保可靠性的 TCP 协议 

按层次分，TCP 位于传输层，`提供可靠的字节流服务`。 

所谓的字节流服务（Byte Stream Service）是指，为了方便传输，将大 块数据分割成以报文段（segment）为单位的数据包进行管理。而可靠的传输服务是指，能够把数据准确可靠地传给对方。一言以蔽之， TCP 协议为了更容易传送大数据才把数据分割，而且 TCP 协议能够确认数据最终是否送达到对方。 

确保数据能到达目标 

为了准确无误地将数据送达目标处，`TCP 协议采用了三次握手 （three-way handshaking）策略`。用 TCP 协议把数据包送出去后，TCP 不会对传送后的情况置之不理，它一定会向对方确认是否成功送达。握手过程中使用了 TCP 的标志（flag） —— SYN（synchronize） 和 ACK（acknowledgement）。 

发送端首先发送一个带 SYN 标志的数据包给对方。接收端收到后， 回传一个带有 SYN/ACK 标志的数据包以示传达确认信息。最后，发送端再回传一个带 ACK 标志的数据包，代表“握手”结束。 

若在握手过程中某个阶段莫名中断，TCP 协议会再次以相同的顺序发送相同的数据包。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/488051d5f9cb64eed1e2095ce2ec156e.png)

除了上述三次握手，TCP 协议还有其他各种手段来保证通信的可靠 性。
### 1.5 负责域名解析的DNS服务

DNS（Domain Name System）服务是和 HTTP 协议一样位于应用层的协议。它提供`域名到 IP 地址之间的解析服务`。 

计算机既可以被`赋予 IP 地址`，也可以被`赋予主机名和域名`。比如 www.hackr.jp。 

用户`通常使用主机名或域名来访问`对方的计算机，而不是直接通过 IP 地址访问。因为与 IP 地址的一组纯数字相比，用字母配合数字的表示形式来指定计算机名更符合人类的记忆习惯。 

但要让计算机去理解名称，相对而言就变得困难了。因为计算机更擅 长处理一长串数字。 

为了解决上述的问题，DNS 服务应运而生。`DNS 协议提供通过域名 查找 IP 地址，或逆向从 IP 地址反查域名的服务`。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/67206299d59aa4c4a1ca73986da0f015.png)

### 1.6 各种协议与HTTP协议的关系

学习了和 HTTP 协议密不可分的 TCP/IP 协议族中的各种协议后，我 们再通过这张图来了解下 IP 协议、TCP 协议和 DNS 服务在使用 HTTP 协议的通信过程中各自发挥了哪些作用。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ce14082f99b511df859b464ed5bf8ff6.png)

### 1.7 URI和URL

与 URI（统一资源标识符）相比，我们更熟悉 URL（Uniform Resource Locator，统一资源定位符）。URL正是使用 Web 浏览器等访问 Web 页面时需要输入的网页地址。比如，下图的 http://hackr.jp/ 就是 URL。

![|200|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/93e7b90cbdcfb4cb4db28c443a4bfd9b.png)

#### 1.7.1 统一资源标识符 URI

URI 是 Uniform Resource Identifier 的缩写。RFC2396 分别对这 3 个单 词进行了如下定义。 

`Uniform `
- 规定统一的格式可方便处理多种不同类型的资源，而不用根据上下文 环境来识别资源指定的访问方式。另外，加入新增的协议方案（如 http: 或 ftp:）也更容易。 

`Resource `
- 资源的定义是“可标识的任何东西”。除了文档文件、图像或服务（例 如当天的天气预报）等能够区别于其他类型的，全都可作为资源。另 外，资源不仅可以是单一的，也可以是多数的集合体。 

`Identifier` 
- 表示可标识的对象。也称为标识符。 综上所述，URI 就是由某个协议方案表示的资源的定位标识符。协议 方案是指访问资源所使用的协议类型名称。 `采用 HTTP 协议时，协议方案就是 http`。除此之外，还有 ftp、mailto、telnet、file 等。标准的 URI 协议方案有 30 种左右，由隶属于 国际互联网资源管理的非营利社团 ICANN（Internet Corporation for Assigned Names and Numbers，互联网名称与数字地址分配机构）的 IANA（Internet Assigned Numbers Authority，互联网号码分配局）管理 颁布。
	- IANA - Uniform Resource Identifier (URI) SCHEMES（统一资源 标识符方案） http://www.iana.org/assignments/uri-schemes

`URI 用字符串标识某一互联网资源`，而 `URL表示资源的地点（互联网上所处的位置）`。可见 `URL是 URI 的子集`。

“RFC3986：统一资源标识符（URI）通用语法”中列举了几种 URI 例 子，如下所示。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e01f9c0f5764e7c1b50c51e8cfc48178.png)
本书接下来的章节中会频繁出现 URI 这个术语，在充分理解的基础 上，也可用 URL替换 URI。

#### 1.7.2 URI 格式 

表示指定的 URI，要使用涵盖全部必要信息的绝对 URI、绝对 URL以及相对 URL。`相对 URL，是指从浏览器中基本 URI 处指定的 URL`， 形如 /image/logo.gif。 让我们先来了解一下绝对 URI 的格式。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0ffec25a3a3f20a2efdee3ed99a86fb9.png)
使用 http: 或 https: 等协议方案名获取访问资源时要指定协议类型。不区分字母大小写，最后附一个冒号（:）。 

也可使用 data: 或 javascript: 这类指定数据或脚本程序的方案名。

`登录信息（认证） `
- 指定用户名和密码作为从服务器端获取资源时必要的登录信息（身份 认证）。此项是可选项。

`服务器地址 `
- 使用绝对 URI 必须指定待访问的服务器地址。地址可以是类似 hackr.jp 这种 DNS 可解析的名称，或是 192.168.1.1 这类 IPv4 地址名，还可以是 [0:0:0:0:0:0:0:1] 这样用方括号括起来的 IPv6 地址名。 

`服务器端口号 `
- 指定服务器连接的网络端口号。此项也是可选项，若用户省略则自动使用默认端口号。 

`带层次的文件路径 `
- 指定服务器上的文件路径来定位特指的资源。这与 UNIX 系统的文件 目录结构相似。 

`查询字符串 `
- 针对已指定的文件路径内的资源，可以使用查询字符串传入任意参 数。此项可选。 

`片段标识符 `
- 使用片段标识符通常可标记出已获取资源中的子资源（文档内的某个位置）。但在 RFC 中并没有明确规定其使用方法。该项也为可选项。

>并不是所有的应用程序都符合 **RFC** 
>
>有一些用来制定 HTTP 协议技术标准的文档，它们被称为 RFC（Request for Comments，`征求修正意见书`）。 
>
>通常，应用程序会遵照由 RFC 确定的标准实现。可以说，RFC 是 互联网的设计文档，要是不按照 RFC 标准执行，就有可能导致无 法通信的状况。比如，有一台 Web 服务器内的应用服务没有遵照 RFC 的标准实现，那 Web 浏览器就很可能无法访问这台服务器 了。 
>
>由于不遵照 RFC 标准实现就无法进行 HTTP 协议通信，所以基本 上客户端和服务器端都会以 RFC 为标准来实现 HTTP 协议。但也 存在某些应用程序因客户端或服务器端的不同，而未遵照 RFC 标 准，反而将自成一套的“标准”扩展的情况。 
>
>不按 RFC 标准来实现，当然也不必劳心费力让自己的“标准”符合 其他所有的客户端和服务器端。但设想一下，如果这款应用程序的 使用者非常多，那会发生什么情况？不难想象，其他的客户端或服 务器端必然都不得不去配合它。 
>
>实际在互联网上，已经实现了 HTTP 协议的一些服务器端和客户端里就存在上述情况。说不定它们会与本书介绍的 HTTP 协议的实现情况不一样。 
>
>本书接下来要介绍的 HTTP 协议内容，除去部分例外，基本上都以 RFC 的标准为准。
## 2 简单的HTTP协议

本章将针对 HTTP 协议结构进行讲解，主要使用HTTP/1.1版本。学完 这章，想必大家就能理解 HTTP 协议的基础了。
### 2.1 HTTP协议用于客户端和服务器端之间的通信

HTTP 协议和 TCP/IP 协议族内的其他众多的协议相同，用于客户端和服务器之间的通信。 

请求访问文本或图像等资源的一端称为客户端，而提供资源响应的一 端称为服务器端。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cf6e380fdafcb20305d4f458af412ca7.png)

在两台计算机之间使用 HTTP 协议通信时，在一条通信线路上`必定有 一端是客户端，另一端则是服务器端`。 

有时候，按实际情况，两台计算机作为客户端和服务器端的角色`有可 能会互换`。但就仅从一条通信路线来说，服务器端和客户端的角色是确定的，而用 `HTTP 协议能够明确区分哪端是客户端，哪端是服务器端`。
### 2.2 通过请求和响应的交换达成通信

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/4ef1a1b5f981cf4a2ced0e8a8f06e172.png)

HTTP 协议规定，请求从客户端发出，最后服务器端响应该请求并返 回。换句话说，肯定是先从客户端开始建立通信的，服务器端在没有 接收到请求之前不会发送响应。 

下面，我们来看一个具体的示例。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/06be8fb8f2e85f5c3d2f7a1e136f5277.png)

下面则是从客户端发送给某个 HTTP 服务器端的请求报文中的内容。 ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f1d64d4f918a68d84204bcad7d1e91a7.png)

起始行开头的GET表示请求访问服务器的类型，称为方法 （method）。随后的字符串 /index.htm 指明了请求访问的资源对象， 也叫做请求 URI（request-URI）。最后的 HTTP/1.1，即 HTTP 的版本号，用来提示客户端使用的 HTTP 协议功能。 

综合来看，这段请求内容的意思是：请求访问某台 HTTP 服务器上的 /index.htm 页面资源。 `请求报文`是由==请求方法==、==请求 URI==、==协议版本==、==可选的请求首部字段== 和==内容实体==构成的。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a7893e00456ba3a4d9d88e2e49bf156b.png)

请求首部字段及内容实体稍后会作详细说明。接下来，我们继续讲 解。接收到请求的服务器，会将请求内容的处理结果以响应的形式返 回。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/da5c6ef2bca824d0e454669b4b2183d7.png)
在起始行开头的 HTTP/1.1 表示服务器对应的 HTTP 版本。

紧挨着的 200 OK 表示请求的处理结果的状态码（status code）和原因短语（reason-phrase）。下一行显示了创建响应的日期时间，是首部字段（header field）内的一个属性。 

接着以一空行分隔，之后的内容称为资源实体的主体（entity body）。 

`响应报文`基本上由==协议版本==、==状态码==（表示请求成功或失败的数字代 码）、用以==解释状态码的原因短语==、==可选的响应首部字段==以及==实体主体==构成。稍后我们会对这些内容进行详细说明。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/37b1eaf3cd647ea225e838abaf681cbb.png)

### 2.3 HTTP是不保存状态的协议

HTTP 是一种不保存状态，即无状态（stateless）协议。HTTP 协议自 身不对请求和响应之间的通信状态进行保存。也就是说在 HTTP 这个级别，协议对于发送过的请求或响应都不做持久化处理。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dc344fb1e7f5091d679e98b5e4a356ad.png)

使用 HTTP 协议，每当有新的请求发送时，就会有对应的新响应产 生。协议本身并不保留之前一切的请求或响应报文的信息。这是为了 更快地处理大量事务，确保协议的可伸缩性，而特意把 HTTP 协议设 计成如此简单的。 

可是，随着 Web 的不断发展，因无状态而导致业务处理变得棘手的 情况增多了。比如，用户登录到一家购物网站，即使他跳转到该站的其他页面后，也需要能继续保持登录状态。针对这个实例，网站为了 能够掌握是谁送出的请求，需要保存用户的状态。

HTTP/1.1 虽然是无状态协议，但为了实现期望的保持状态功能，于 是引入了 Cookie 技术。有了 Cookie 再用 HTTP 协议通信，就可以管 理状态了。有关 Cookie 的详细内容稍后讲解。
### 2.4 请求URI定位资源

HTTP 协议使用 URI 定位互联网上的资源。正是因为 URI 的特定功 能，在互联网上任意位置的资源都能访问到。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f4bf45ee7fe62358a952dde6a8a35ba1.png)

当客户端请求访问资源而发送请求时，URI 需要将作为请求报文中的 请求 URI 包含在内。指定请求 URI 的方式有很多。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/42452f8c1aa73ba029dbde29fcfa8ac1.png)

除此之外，如果不是访问特定资源而是对服务器本身发起请求，可以 用一个 * 来代替请求 URI。下面这个例子是查询 HTTP 服务器端支持的 HTTP 方法种类。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a6e07a2951a32aaf98762c7b8940218b.png)
### 2.5 告知服务器意图的HTTP方法

下面，我们介绍 HTTP/1.1 中可使用的方法。

`GET ：获取资源 GET `
- 方法用来请求访问已被 URI 识别的资源。指定的资源经服务器 端解析后返回响应内容。也就是说，如果请求的资源是文本，那就保 持原样返回；如果是像 CGI（Common Gateway Interface，通用网关接口）那样的程序，则返回经过执行后的输出结果。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cffd3ae40a2f49c3c5f76b3d7b596170.png)
- 使用 GET 方法的请求·响应的例子![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3e58f05727f3bd234ad6069a833b183b.png)
`POST：传输实体主体 `
- POST 方法用来传输实体的主体。 虽然用 GET 方法也可以传输实体的主体，但一般不用 GET 方法进行 传输，而是用 POST 方法。虽说 POST 的功能与 GET 很相似，但 POST 的主要目的并不是获取响应的主体内容。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/53594c823c471bf68c78e58f2835c7c1.png)
- 使用 POST 方法的请求·响应的例子![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7ce83d23b893f3f69e75705a27e7f17b.png)
`PUT：传输文件 PUT`
- 方法用来传输文件。就像 FTP 协议的文件上传一样，要求在请 求报文的主体中包含文件内容，然后保存到请求 URI 指定的位置。
- 但是，鉴于 HTTP/1.1 的 PUT 方法自身不带验证机制，任何人都可以 上传文件 , 存在安全性问题，因此一般的 Web 网站不使用该方法。若 配合 Web 应用程序的验证机制，或架构设计采用 REST（REpresentational State Transfer，表征状态转移）标准的同类 Web 网站，就可能会开放使用 PUT 方法。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/44f60bb581d92ae5b8a4ffdb036c6369.png)
- 使用 PUT 方法的请求·响应的例子![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c07afa7fd0ca5031545297259482b44d.png)
`HEAD：获得报文首部 `
- HEAD 方法和 GET 方法一样，只是不返回报文主体部分。用于确认 URI 的有效性及资源更新的日期时间等。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/708dcc5ef747a2054cbb7b0aa23c67f2.png)
- 使用 HEAD 方法的请求·响应的例子![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9c2c29d2faefa4c39958e96f13d2124d.png)
`DELETE：删除文件` 
- DELETE 方法用来删除文件，是与 PUT 相反的方法。DELETE 方法按 请求 URI 删除指定的资源。 
- 但是，HTTP/1.1 的 DELETE 方法本身和 PUT 方法一样不带验证机制，所以一般的 Web 网站也不使用 DELETE 方法。当配合 Web 应用程序的验证机制，或遵守 REST 标准时还是有可能会开放使用的。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/02520528bc834bc68bbd3d8d4a7ab702.png)
- 使用 DELETE 方法的请求·响应的例子![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/faba3494b029d418d23bbacbf02e6b41.png)
`OPTIONS：询问支持的方法 `
- OPTIONS 方法用来查询针对请求 URI 指定的资源支持的方法。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ffcb31122a874d2ab2d364f6266f5249.png)
- 使用 OPTIONS 方法的请求·响应的例子![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0e1189ddc85f05f31c5bf33dc04ae17f.png)
`TRACE：追踪路径 TRACE` 
- 方法是让 Web 服务器端将之前的请求通信环回给客户端的方法。
- 发送请求时，在 Max-Forwards 首部字段中填入数值，每经过一个服务器端就将该数字减 1，当数值刚好减到 0 时，就停止继续传输，最后接收到请求的服务器端则返回状态码 200 OK 的响应。
- 客户端通过 TRACE 方法可以查询发送出去的请求是怎样被加工修改 / 篡改的。这是因为，请求想要连接到源目标服务器可能会通过代理中转，`TRACE 方法就是用来确认连接过程中发生的一系列操作`。 
- 但是，TRACE 方法本来就不怎么常用，再加上它容易引发 XST（Cross-Site Tracing，跨站追踪）攻击，通常就更不会用到了![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5ecc027c54bcee37a6496d64338b2bcf.png)
- 使用 TRACE 方法的请求·响应的例子![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/20bfd47368cbfc6bc80b6fb490cf9750.png)
`CONNECT：要求用隧道协议连接代理 `
- CONNECT 方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行 TCP 通信。主要使用 SSL（Secure Sockets Layer，安全套接层）和 TLS（Transport Layer Security，传输层安全）协议把通信内容 加密后经网络隧道传输。 
- CONNECT 方法的格式如下所示。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ed2c0fe8d1af3c1dd4f1fc3b49726143.png)
- 使用 CONNECT 方法的请求·响应的例子![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f50df0e057c397da1713411c658cdb7c.png)
### 2.6 使用方法下达命令

向请求 URI 指定的资源发送请求报文时，采用称为方法的命令。 方法的作用在于，可以指定请求的资源按期望产生某种行为。方法中 有 GET、POST 和 HEAD 等。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2f8f3ea567dd0eefcd9df88b85465123.png)
下表列出了 HTTP/1.0 和 HTTP/1.1 支持的方法。另外，方法名区分大 小写，注意要用大写字母。 

表 2-1：HTTP/1.0 和 HTTP/1.1 支持的方法![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6fabaeddbed568065dc57dbf95acda2d.png)
### 2.7 持久连接节省通信量

HTTP 协议的初始版本中，每进行一次 HTTP 通信就要断开一次 TCP 连接。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8306bf88e91da44b566000bfafd4bc6.png)
以当年的通信情况来说，因为都是些容量很小的文本传输，所以即使 这样也没有多大问题。可随着 HTTP 的普及，文档中包含大量图片的 情况多了起来。 

比如，使用浏览器浏览一个包含多张图片的 HTML页面时，在发送 请求访问 HTML页面资源的同时，也会请求该 HTML页面里包含的 其他资源。因此，每次的请求都会造成无谓的 TCP 连接建立和断 开，增加通信量的开销。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6896587e2153a4075c481bbace427e7a.png)

#### 2.7.1 持久连接 

为解决上述 TCP 连接的问题，HTTP/1.1 和一部分的 HTTP/1.0 想出了 `持久连接（HTTP Persistent Connections，也称为 HTTP keep-alive 或 HTTP connection reuse）`的方法。持久连接的特点是，只要任意一端 没有明确提出断开连接，则保持 TCP 连接状态。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9a929e406391e13dcd2c42002c26ebf9.png)

持久连接的好处在于`减少了 TCP 连接的重复建立和断开所造成的额 外开销，减轻了服务器端的负载`。另外，减少开销的那部分时间，使 HTTP 请求和响应能够更早地结束，这样 Web 页面的显示速度也就相 应提高了。 

在 HTTP/1.1 中，所有的连接默认都是持久连接，但在 HTTP/1.0 内并未标准化。虽然有一部分服务器通过非标准的手段实现了持久连接， 但服务器端不一定能够支持持久连接。毫无疑问，除了服务器端，`客户端也需要支持持久连接`。

#### 2.7.2 管线化 

持久连接使得多数请求以管线化（pipelining）方式发送成为可能。从前发送请求后需等待并收到响应，才能发送下一个请求。管线化技术出现后，`不用等待响应亦可直接发送下一个请求`。

这样就能够做到同时并行发送多个请求，而不需要一个接一个地等待 响应了。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/550b9c57d7ef230206fc52bde7865162.png)
比如，当请求一个包含 10 张图片的 HTMLWeb 页面，与挨个连接相 比，用持久连接可以让请求更快结束。而管线化技术则比持久连接还 要快。请求数越多，时间差就越明显。
### 2.8 使用Cookie的状态管理

HTTP 是无状态协议，它不对之前发生过的请求和响应的状态进行管 理。也就是说，无法根据之前的状态进行本次的请求处理。 

假设要求登录认证的 Web 页面本身无法进行状态的管理（不记录已 登录的状态），那么每次跳转新页面不是要再次登录，就是`要在每次 请求报文中附加参数来管理登录状态`。 

不可否认，无状态协议当然也有它的优点。由于不必保存状态，自然 可减少服务器的 CPU 及内存资源的消耗。从另一侧面来说，也正是 因为 HTTP 协议本身是非常简单的，所以才会被应用在各种场景里。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bc9e17972ffd587ef83ae2c58c8bc9e0.png)

保留无状态协议这个特征的同时又要解决类似的矛盾问题，于是引入 了 Cookie 技术。`Cookie 技术通过在请求和响应报文中写入 Cookie 信 息来控制客户端的状态`。

Cookie 会根据从服务器端发送的响应报文内的一个叫做 Set-Cookie 的 首部字段信息，通知客户端保存 Cookie。当下次客户端再往该服务器 发送请求时，客户端会自动在请求报文中加入 Cookie 值后发送出去。 

服务器端发现客户端发送过来的 Cookie 后，会去检查究竟是从哪一 个客户端发来的连接请求，然后对比服务器上的记录，最后得到之前 的状态信息。

- 没有 Cookie 信息状态下的请求![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9339a306fb5f73621c7167b1a1e09037.png)
- 第 2 次以后（存有 Cookie 信息状态）的请求![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/92cd9996090367ac208f543f89d59eae.png)
上图展示了发生 Cookie 交互的情景，HTTP 请求报文和响应报文的内容如下。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c9992e889319952b27c196f7580ebacd.png)

## 3 HTTP报文内的HTTP信息

HTTP 通信过程包括从客户端发往服务器端的`请求`及从服务器端返回 客户端的`响应`。本章就让我们来了解一下请求和响应是怎样运作的。
### 3.1 HTTP报文

`用于 HTTP 协议交互的信息被称为 HTTP 报文`。请求端（客户端）的 HTTP 报文叫做`请求报文`，响应端（服务器端）的叫做`响应报文`。 HTTP 报文本身是由多行（用 CR+LF 作换行符）数据构成的字符串文 本。

HTTP 报文大致可分为`报文首部`和`报文主体`两块。两者由最初出现的 空行（CR+LF）来划分。通常，并不一定要有报文主体。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8feb4df0e2c25034a92ef9ac008cf632.png)

### 3.2 请求报文及响应报文的结构

我们来看一下请求报文和响应报文的结构。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7997d10374b4070b6cd0110b2429d0a6.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5930f2663612f3abd9667ee91bdb8461.png)

请求报文和响应报文的首部内容由以下数据组成。现在出现的各种首 部字段及状态码稍后会进行阐述。 

`请求行 `
- 包含用于请求的方法，请求 URI 和 HTTP 版本。 

`状态行 `
- 包含表明响应结果的状态码，原因短语和 HTTP 版本。 

`首部字段 `
- 包含表示请求和响应的各种条件和属性的各类首部。
- 一般有 4 种首部，分别是：通用首部、请求首部、响应首部和实体首 部。 

`其他 `
- 可能包含 HTTP 的 RFC 里未定义的首部（Cookie 等）。

### 3.3 编码提升传输速率

HTTP 在传输数据时可以按照数据原貌直接传输，但也可以`在传输过 程中通过编码提升传输速率`。通过在传输时编码，能有效地处理大量 的访问请求。但是，编码的操作需要计算机来完成，因此会消耗更多 的 CPU 等资源。

#### 3.3.1 报文主体和实体主体的差异

`报文（message）`
- 是 HTTP 通信中的基本单位，由 8 位组字节流（octet sequence， 其中 octet 为 8 个比特）组成，通过 HTTP 通信传输。 

`实体（entity） `
- 作为请求或响应的有效载荷数据（补充项）被传输，其内容由实体首部和实体主体组成。 

`HTTP 报文的主体用于传输请求或响应的实体主体`。 

==通常，报文主体等于实体主体==。只有当传输中进行编码操作时，实体 主体的内容发生变化，才导致它和报文主体产生差异。 

报文和实体这两个术语在之后会经常出现，请事先理解两者的差异。

#### 3.3.2 压缩传输的内容编码

向待发送邮件内增加附件时，为了使邮件容量变小，我们会先用 ZIP 压缩文件之后再添加附件发送。`HTTP 协议中有一种被称为内容编码 的功能也能进行类似的操作`。 

内容编码指明应用在实体内容上的编码格式，并保持实体信息原样压缩。内容编码后的实体由客户端接收并负责解码。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7fbe4de83577792e027aa75451551963.png)

常用的内容编码有以下几种。 
- gzip（GNU zip） 
- compress（UNIX 系统的标准压缩） 
- deflate（zlib） 
- identity（不进行编码）
#### 3.3.3 分割发送的分块传输编码

在 HTTP 通信过程中，请求的编码实体资源尚未全部传输完成之前， 浏览器无法显示请求页面。在传输大容量数据时，通过把数据分割成多块，能够让浏览器逐步显示页面。 

这种`把实体主体分块的功能称为分块传输编码（Chunked Transfer Coding）`。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5a5bc70fb64fde984bce851762ccd9bf.png)

`分块传输编码会将实体主体分成多个部分（块）`。每一块都会用十六 进制来标记块的大小，而实体主体的最后一块会使用“0(CR+LF)”来标 记。 

使用分块传输编码的实体主体会由接收的客户端负责解码，恢复到编 码前的实体主体。 

HTTP/1.1 中存在一种称为传输编码（Transfer Coding）的机制，它可 以在通信时按某种编码方式传输，但只定义作用于分块传输编码中。

### 3.4 发送多种数据的多部分对象集合

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/28bcf71d4238eecb0dbe2c9e9372b92e.png)

发送邮件时，我们可以在邮件里写入文字并添加多份附件。这是因为 采用了 MIME（Multipurpose Internet Mail Extensions，多用途因特网邮件扩展）机制，它允许邮件处理文本、图片、视频等多个不同类型的数据。例如，图片等二进制数据以 ASCII 码字符串编码的方式指明， 就是利用 MIME 来描述标记数据类型。而在 MIME 扩展中会使用一 种称为多部分对象集合（Multipart）的方法，来容纳多份不同类型的 数据。

相应地，`HTTP 协议中也采纳了多部分对象集合`，发送的一份报文主 体内可含有多类型实体。通常是在图片或文本文件等上传时使用。 多部分对象集合包含的对象如下。

`multipart/form-data `
- 在 Web 表单文件上传时使用。 ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9afc236bcaffcf2e46dc30facaa2786e.png)

`multipart/byteranges `
- 状态码 206（Partial Content，部分内容）响应报文包含了多个范 围的内容时使用。 ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f5bdc5355795e9f630f13b3225abd2e3.png)
在 HTTP 报文中使用多部分对象集合时，`需要在首部字段里加上 Content-type`。有关这个首部字段，我们稍后讲解。 

使用 boundary 字符串来划分多部分对象集合指明的各类实体。在 boundary 字符串`指定的各个实体的起始行之前插入“--”标记`（例如：- -AaB03x、--THIS_STRING_SEPARATES），而在多部分对象集合对 应的字符串的最后插入“--”标记（例如：--AaB03x--、-- THIS_STRING_SEPARATES--）作为结束。 

多部分对象集合的每个部分类型中，都可以含有首部字段。另外，可 以在某个部分中嵌套使用多部分对象集合。有关多部分对象集合更详 细的解释，请参考 RFC2046。

### 3.5 获取部分内容的范围请求

以前，用户不能使用现在这种高速的带宽访问互联网，当时，下载一 个尺寸稍大的图片或文件就已经很吃力了。如果下载过程中遇到网络 中断的情况，那就必须重头开始。为了解决上述问题，需要一种可恢 复的机制。所谓恢复是指能从之前下载中断处恢复下载。 

要实现该功能需要指定下载的实体范围。像这样，指定范围发送的请 求叫做范围请求（Range Request）。 对一份 10 000 字节大小的资源，如果使用范围请求，可以只请求 5001~10 000 字节内的资源。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b59e9eb7901e5b54f755468e4dcca213.png)

执行范围请求时，会用到首部字段 Range 来指定资源的 byte 范围。 

byte 范围的指定形式如下。 

`5001~10 000 字节 `
- Range: bytes=5001-10000 

`从 5001 字节之后全部的 `
- Range: bytes=5001- 

`从一开始到 3000 字节和 5000~7000 字节的多重范围 `
- Range: bytes=-3000, 5000-7000 

针对范围请求，响应会返回状态码为 206 Partial Content 的响应报 文。另外，对于多重范围的范围请求，响应会在首部字段 Content-Type 标明 multipart/byteranges 后返回响应报文。

如果服务器端无法响应范围请求，则会返回状态码 200 OK 和完整的 实体内容。

### 3.6 内容协商返回最合适的内容

同一个 Web 网站有可能存在着多份相同内容的页面。比如英语版和 中文版的 Web 页面，它们内容上虽相同，但使用的语言却不同。 

当浏览器的默认语言为英语或中文，访问相同 URI 的 Web 页面时， 则会显示对应的英语版或中文版的 Web 页面。这样的机制称为内容 协商（Content Negotiation）。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d43af5c41824617ef7fa0e2f05183685.png)

内容协商机制是指客户端和服务器端就响应的资源内容进行交涉，然 后提供给客户端最为适合的资源。内容协商会以响应资源的语言、字 符集、编码方式等作为判断的基准。 

包含在请求报文中的某些首部字段（如下）就是判断的基准。这些首 部字段的详细说明请参考下一章。
- **Accept**
- **Accept-Charset**
- **Accept-Encoding**
- **Accept-Language**
- **Content-Language**

内容协商技术有以下 3 种类型。

`服务器驱动协商（Server-driven Negotiation）` 
- 由服务器端进行内容协商。以请求的首部字段为参考，在服务器端自动处理。但对用户来说，以浏览器发送的信息作为判定的依据，并不 一定能筛选出最优内容。 

客户端驱动协商（Agent-driven Negotiation） 
- 由客户端进行内容协商的方式。用户从浏览器显示的可选项列表中手 动选择。还可以利用 JavaScript 脚本在 Web 页面上自动进行上述选 择。比如按 OS 的类型或浏览器类型，自行切换成 PC 版页面或手机 版页面。 

透明协商（Transparent Negotiation） 
- 是服务器驱动和客户端驱动的结合体，是由服务器端和客户端各自进 行内容协商的一种方法。

## 4 返回结果的HTTP状态码

HTTP 状态码负责表示客户端 HTTP 请求的返回结果、标记服务器端 的处理是否正常、通知出现的错误等工作。让我们通过本章的学习， 好好了解一下状态码的工作机制。

### 4.1 状态码告知从服务器端返回的请求结果

状态码的职责是当客户端向服务器端发送请求时，`描述返回的请求结果`。借助状态码，用户可以知道服务器端是正常处理了请求，还是出 现了错误。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dae527515d3b9817e65f3260b1918c3b.png)

状态码如 200 OK，以 3 位数字和原因短语组成。 

数字中的第一位指定了响应类别，后两位无分类。响应类别有以下 5 种。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6848c289cb9a63ff11ce54d2c03f7b81.png)
只要遵守状态码类别的定义，即使改变 RFC2616 中定义的状态码， 或服务器端自行创建状态码都没问题。 

仅记录在 RFC2616 上的 HTTP 状态码就达 40 种，若再加上 WebDAV（Web-based Distributed Authoring and Versioning，基于万维网的分布式创作和版本控制）（RFC4918、5842） 和附加 HTTP 状态码 （RFC6585）等扩展，数量就达 60 余种。别看种类繁多，`实际上经常使用的大概只有 14 种`。接下来，我们就介绍一下这些具有代表性 的 14 个状态码。

### 4.2 2XX 成功

2XX 的响应结果表明请求被正常处理了。

#### 4.2.1 200 OK

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0ce2fa69057072723b284957c8aba0fa.png)
表示从客户端发来的请求在服务器端被正常处理了。 

在响应报文内，随状态码一起返回的信息会因方法的不同而发生改 变。比如，使用 GET 方法时，对应请求资源的实体会作为响应返 回；而使用 HEAD 方法时，对应请求资源的实体首部不随报文主体 作为响应返回（即在响应中只返回首部，不会返回实体的主体部 分）。

#### 4.2.2 204 No Content

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9457b2e18311708b1f8358c2099157b9.png)

该状态码代表服务器接收的请求已成功处理，但在返回的响应报文中 不含实体的主体部分。另外，也不允许返回任何实体的主体。比如， 当从浏览器发出请求处理后，返回 204 响应，那么浏览器显示的页面 不发生更新。 

一般在只需要从客户端往服务器发送信息，而对客户端不需要发送新信息内容的情况下使用。
#### 4.2.3 206 Partial Content

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6089e3f7023c8895add1578e0e261063.png)
该状态码表示客户端进行了范围请求，而服务器成功执行了这部分的 GET 请求。响应报文中包含由 Content-Range 指定范围的实体内容。

### 4.3 3XX 重定向

#### 4.3.1 301 Moved Permanently

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a6643115b183a1bddef3cc581c6e8fe7.png)

`永久性重定向`。该状态码表示请求的资源已被分配了新的 URI，以后应使用资源现在所指的 URI。也就是说，如果已经把资源对应的 URI 保存为书签了，这时应该按 Location 首部字段提示的 URI 重新保存。 

像下方给出的请求 URI，当指定资源路径的最后忘记添加斜杠“/”，就 会产生 301 状态码。
http://example.com/sample
#### 4.3.2 302 Found

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fb40a9fa94085463c6ec6be2adb8c1b6.png)

`临时性重定向`。该状态码表示请求的资源已被分配了新的 URI，希望 用户（本次）能使用新的 URI 访问。 

和 301 Moved Permanently 状态码相似，但 302 状态码代表的资源不是被永久移动，只是临时性质的。换句话说，已移动的资源对应的 URI 将来还有可能发生改变。比如，用户把 URI 保存成书签，但不会像 301 状态码出现时那样去更新书签，而是仍旧保留返回 302 状态码 的页面对应的 URI。

#### 4.3.3 303 See Other

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/79bae2c707bc5250c91a9748593392fd.png)

该状态码表示由于请求对应的资源存在着另一个 URI，应使用 GET 方法定向获取请求的资源。 

303 状态码和 302 Found 状态码有着相同的功能，但 `303 状态码明确表示客户端应当采用 GET 方法获取资源`，这点与 302 状态码有区 别。

比如，当使用 POST 方法访问 CGI 程序，其执行后的处理结果是希望 客户端能以 GET 方法重定向到另一个 URI 上去时，返回 303 状态 码。虽然 302 Found 状态码也可以实现相同的功能，但这里使用 303 状态码是最理想的。

>当 301、302、303 响应状态码返回时，几乎所有的浏览器都会把 POST 改成 GET，并删除请求报文内的主体，之后请求会自动再次 发送。 
>
>301、302 标准是禁止将 POST 方法改变成 GET 方法的，但实际使用时大家都会这么做。

#### 4.3.4 304 Not Modified

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9eff6669f7369ec8bbd9a2ec2be6b5b2.png)

该状态码表示客户端发送附带条件的请求时，服务器端允许请求访问资源，但未满足条件的情况。304 状态码返回时，不包含任何响应 的主体部分。`304 虽然被划分在 3XX 类别中，但是和重定向没有关系`。

#### 4.3.5 307 Temporary Redirect 

`临时重定向`。该状态码与 302 Found 有着相同的含义。尽管 302 标准 64 禁止 POST 变换成 GET，但实际使用时大家并不遵守。 `307 会遵照浏览器标准`，不会从 POST 变成 GET。但是，对于处理响应时的行为，每种浏览器有可能出现不同的情况。

### 4.4 4XX 客户端错误

4XX 的响应结果表明客户端是发生错误的原因所在。

#### 4.4.1 4..1 400 Bad Request

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cf3be924dc4f3b0b4797c81d0417522c.png)

该状态码表示请求报文中存在语法错误。当错误发生时，需修改请求 的内容后再次发送请求。另外，浏览器会像 200 OK 一样对待该状态 码。

#### 4.4.2 401 Unauthorized

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f9681fec2eb59811ca0c4ab42034380a.png)

该状态码表示发送的请求需要有通过 HTTP 认证（BASIC 认证、 DIGEST 认证）的认证信息。另外若之前已进行过 1 次请求，则表示 用 户认证失败。 

返回含有 401 的响应必须包含一个适用于被请求资源的 WWW-Authenticate 首部用以质询（challenge）用户信息。当浏览器初次接收 到 401 响应，会弹出认证用的对话窗口。

#### 4.4.3 403 Forbidden

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a4e7a289f6a4f4f824f9e958de920236.png)

该状态码表明对请求资源的访问被服务器拒绝了。服务器端没有必要 给出拒绝的详细理由，但如果想作说明的话，可以在实体的主体部分对原因进行描述，这样就能让用户看到了。

未获得文件系统的访问授权，访问权限出现某些问题（从未授权的发 送源 IP 地址试图访问）等列举的情况都可能是发生 403 的原因。
#### 4.4.4 404 Not Found

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7e338ce2e35ef6289cdffc67bc6e6dd.png)

该状态码表明服务器上无法找到请求的资源。除此之外，也可以在服 务器端拒绝请求且不想说明理由时使用。

### 4.5  5XX 服务器错误

5XX 的响应结果表明服务器本身发生错误。

#### 4.5.1 500 Internal Server Error

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bd4f4f1bb4f5b7b02cc313f949b2b86a.png)

该状态码表明服务器端在执行请求时发生了错误。也有可能是 Web 应用存在的 bug 或某些临时的故障。
#### 4.5.2 503 Service Unavailable

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6cbaa68800811a359627311a3c7c3e3a.png)

该状态码表明服务器暂时处于超负载或正在进行停机维护，现在无法 处理请求。如果事先得知解除以上状况需要的时间，最好写入 RetryAfter 首部字段再返回给客户端。

>状态码和状况的不一致 
>
>不少返回的状态码响应都是错误的，但是用户可能察觉不到这点。 比如 Web 应用程序内部发生错误，状态码依然返回 200 OK，这种 情况也经常遇到。

## 5 与HTTP协作的Web服务器

一台 Web 服务器可搭建多个独立域名的 Web 网站，也可作为通信路 径上的中转服务器提升传输效率。
### 5.1 用单台虚拟主机实现多个域名

HTTP/1.1 规范允许一台 HTTP 服务器搭建多个 Web 站点。比如，提供 Web 托管服务（Web Hosting Service）的供应商，`可以用一台服务器为多位客户服务`，`也可以每位客户持有的域名运行各自不同的网站`。这是因为利用了虚拟主机（Virtual Host，又称虚拟服务器）的功能。

即使物理层面只有一台服务器，但`只要使用虚拟主机的功能，则可以 假想已具有多台服务器`。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c59fa2540b1477202b4d9ccff8133c13.png)

客户端使用 HTTP 协议访问服务器时，会经常采用类似 www.hackr.jp 这样的主机名和域名。 

在互联网上，域名通过 DNS 服务映射到 IP 地址（域名解析）之后访问目标网站。可见，当请求发送到服务器时，已经是以 IP 地址形式 访问了。 

所以，如果一台服务器内托管了 www.tricorder.jp 和 www.hackr.jp 这 两个域名，当收到请求时就需要弄清楚究竟要访问哪个域名。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/18a61836c8da41f584814581585dd50a.png)

在相同的 IP 地址下，由于虚拟主机可以寄存多个不同主机名和域名 的 Web 网站，因此在发送 HTTP 请求时，必须在 Host 首部内完整指 定主机名或域名的 URI。
### 5.2 通信数据转发程序：代理、网关、隧道

HTTP 通信时，除客户端和服务器以外，还有一些`用于通信数据转发的应用程序，例如代理、网关和隧道`。它们可以`配合服务器工作`

这些应用程序和服务器可以将请求转发给通信线路上的下一站服务 器，并且能接收从那台服务器发送的响应再转发给客户端。 

`代理` 
- `代理是一种有转发功能的应用程序`，它扮演了位于服务器和客户 端“中间人”的角色，接收由客户端发送的请求并转发给服务器，同时 也接收服务器返回的响应并转发给客户端。 

`网关` 
- `网关是转发其他服务器通信数据的服务器`，接收从客户端发送来的请求时，它就像自己拥有资源的源服务器一样对请求进行处理。有时客户端可能都不会察觉，自己的通信目标是一个网关。 

`隧道` 
- `隧道是在相隔甚远的客户端和服务器两者之间进行中转`，并保持双方通信连接的应用程序。

#### 5.2.1 代理

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7fe0a902091259342a33ecc2d60489d9.png)

代理服务器的基本行为就是接收客户端发送的请求后转发给其他服务 器。代理不改变请求 URI，会直接发送给前方持有资源的目标服务 器。 

持有资源实体的服务器被称为源服务器。从源服务器返回的响应经过 代理服务器后再传给客户端。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e8a3eed4892aadec4fad47f5f194b376.png)

在 HTTP 通信过程中，`可级联多台代理服务器`。请求和响应的转发会 经过数台类似锁链一样连接起来的代理服务器。转发时，需要`附加 Via 首部字段以标记出经过的主机信息`。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f3e26a2341d30bdae143132b16d6878d.png)

`使用代理服务器的理由`有：利用缓存技术（稍后讲解）`1.减少网络带宽 的流量`，`2.组织内部针对特定网站的访问控制`，以`3.获取访问日志为主要 目的`，等等。 

代理有多种使用方法，按两种基准分类。一种是是否使用缓存，另一 种是是否会修改报文。 

`缓存代理` 
- 代理转发响应时，缓存代理（Caching Proxy）会预先将资源的副本 （缓存）保存在代理服务器上。 当代理再次接收到对相同资源的请求时，就可以不从源服务器那里获 取资源，而是将之前缓存的资源作为响应返回。 

`透明代理` 
- 转发请求或响应时，不对报文做任何加工的代理类型被称为透明代理 （Transparent Proxy）。反之，对报文内容进行加工的代理被称为非透明代理。
#### 5.2.2 网关

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/28dcf15fbd2ef9fe53caee8d0785ca65.png)

网关的工作机制和代理十分相似。而`网关能使通信线路上的服务器提 供非 HTTP 协议服务`。 

`利用网关能提高通信的安全性`，因为可以在客户端与网关之间的通信线路上加密以确保连接的安全。比如，网关可以连接数据库，使用 SQL语句查询数据。另外，在 Web 购物网站上进行信用卡结算时， 网关可以和信用卡结算系统联动。
#### 5.2.3 隧道

隧道可按要求建立起一条与其他服务器的通信线路，届时使用 SSL等 加密手段进行通信。隧道的目的是确保客户端能与服务器进行安全的 通信。 

隧道本身不会去解析 HTTP 请求。也就是说，请求保持原样中转给之 后的服务器。隧道会在通信双方断开连接时结束。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3181b7e7939a7784575edbcfa4133c81.png)

### 5.3 3、保存资源的缓存

缓存是指代理服务器或客户端本地磁盘内保存的资源副本。利用缓存 可减少对源服务器的访问，因此也就节省了通信流量和通信时间。 

缓存服务器是代理服务器的一种，并归类在缓存代理类型中。换句话 说，当代理转发从服务器返回的响应时，代理服务器将会保存一份资 源的副本。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c9983e76678aef236efd5b1df9c48351.png)

缓存服务器的优势在于利用缓存可避免多次从源服务器转发资源。因 此`客户端可就近从缓存服务器上获取资源`，而`源服务器也不必多次处理相同的请求了`。

#### 5.3.1 缓存的有效期限

即便缓存服务器内有缓存，也不能保证每次都会返回对同资源的请 求。因为这关系到被缓存资源的有效性问题。 

当遇上源服务器上的资源更新时，如果还是使用不变的缓存，那就会 演变成返回更新前的“旧”资源了。 

即使存在缓存，也会因为客户端的要求、缓存的有效期等因素，向源 服务器确认资源的有效性。若判断缓存失效，缓存服务器将会再次从 源服务器上获取“新”资源。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/38ffb421380e235bb3a68b3408c8100a.png)

#### 5.3.2 客户端的缓存

缓存不仅可以存在于缓存服务器内，还可以存在客户端浏览器中。以 Internet Explorer 程序为例，把客户端缓存称为临时网络文件 （Temporary Internet File）。 

浏览器缓存如果有效，就不必再向服务器请求相同的资源了，可以直 接从本地磁盘内读取。 

另外，和缓存服务器相同的一点是，当判定缓存过期后，会向源服务 器确认资源的有效性。若判断浏览器缓存失效，浏览器会再次请求新 资源。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/94ed9eebf78a92821af0aa1de46180f2.png)

>在 HTTP 出现之前的协议 
>在 HTTP 普及之前，也就是从互联网的诞生期至今，曾出现过各式各样的协议。在 HTTP 规范确立之际，制定者们参考了那些协议的功能。也有某些协议现在已经彻底退出了人们的视线。接下来，我 们会简单介绍一下这些协议。 
>
>`FTP（File Transfer Protocol） 传输文件时使用的协议`。该协议历史久远，可追溯到 1973 年前 后，比 TCP/IP 协议族的出现还要早。虽然它在 1995 年被 HTTP 的 流量（Traffic）超越，但时至今日，仍被广泛沿用。 
>
>`NNTP（Network News Transfer Protocol） 用于 NetNews 电子会议室内传送消息的协议`。在 1986 年前后出 现，属于比较古老的一类协议。现在，利用 Web 交换信息已成主流，所以该协议已经不怎么使用了。 
>
>`Archie 搜索 anonymous FTP 公开的文件信息的协议`。1990 年前后出现，现 在已经不常使用。 
>
>`WAIS（Wide Area Information Servers） 以关键词检索多个数据库使用的协议`。1991 年前后出现。由于现 在已经被 HTTP 协议替代，也已经不怎么使用了。 
>
>`Gopher 查找与互联网连接的计算机内信息的协议`。1991 年前后出现，由 于现在已经被 HTTP 协议替代，也已经不怎么使用了。

## 6 六、HTTP首部

HTTP 协议的请求和响应报文中`必定包含 HTTP 首部`，只是我们平时 在使用 Web 的过程中感受不到它。本章我们一起来学习 HTTP 首部的结构，以及首部中各字段的用法。
### 6.1 1、HTTP报文首部

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/72d15b82d218c7fa4edf5e8263ab6afd.png)

HTTP 协议的请求和响应报文中必定包含 HTTP 首部。首部内容为客户端和服务器分别处理请求和响应提供所需要的信息。对于客户端用户来说，这些信息中的大部分内容都无须亲自查看。 报文首部由几个字段构成。

#### 6.1.1 HTTP 请求报文 

在请求中，HTTP 报文由`方法`、`URI`、`HTTP 版本`、`HTTP 首部字段`等部分构成。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a49176cda2f5b756872fd4f97e7972bc.png)
下面的示例是访问 http://hackr.jp 时，请求报文的首部信息。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6e8320160802ba8e0f417e1b13b3d392.png)
#### 6.1.2 HTTP 响应报文

在响应中，HTTP 报文由 `HTTP 版本`、`状态码`（数字和原因短语）、 `HTTP 首部字段` 3 部分构成。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ae387fb4b6ab38ad3dedbcb5565faff3.png)

以下示例是之前请求访问 http://hackr.jp/ 时，返回的响应报文的首部 信息。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/52e01f473c017b0956aa7dba72fbc49b.png)
在报文众多的字段当中，HTTP 首部字段包含的信息最为丰富。首部 字段同时存在于请求和响应报文内，并涵盖 HTTP 报文相关的内容信息。 

因 HTTP 版本或扩展规范的变化，首部字段可支持的字段内容略有不 同。本书主要涉及 HTTP/1.1 及常用的首部字段。

### 6.2 2、HTTP首部字段

#### 6.2.1 HTTP 首部字段传递重要信息

HTTP 首部字段是构成 HTTP 报文的要素之一。在客户端与服务器之 间以 HTTP 协议进行通信的过程中，无论是请求还是响应都会使用首部字段，

它能起到传递额外重要信息的作用。 使用首部字段是为了给浏览器和服务器提供报文主体大小、所使用的语言、认证信息等内容。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1c0d8be101e66763f5e5f7dc5524541e.png)
#### 6.2.2 HTTP 首部字段结构 HTTP 

首部字段是由`首部字段名`和`字段值`构成的，中间用冒号“:” 分 隔。 
- 首部字段名: 字段值 

例如，在 HTTP 首部中以 Content-Type 这个字段来表示报文主体的 对象类型。 
- Content-Type: text/html 

就以上述示例来看，首部字段名为 Content-Type，字符串 text/html 是字段值。 

另外，字段值对应单个 HTTP 首部字段可以有多个值，如下所示。
- Keep-Alive: timeout=15, max=100

>`若 HTTP 首部字段重复了会如何` 
>
>当 HTTP 报文首部中出现了两个或两个以上具有相同首部字段名时 会怎么样？这种情况在规范内尚未明确，根据浏览器内部处理逻辑的不同，结果可能并不一致。有些浏览器会优先处理第一次出现的首部字段，而有些则会优先处理最后出现的首部字段。

#### 6.2.3 4 种 HTTP 首部字段类型

HTTP 首部字段根据实际用途被分为以下 4 种类型。 

`通用首部字段（General Header Fields） `
- 请求报文和响应报文两方都会使用的首部。 

`请求首部字段（Request Header Fields） `
- 从客户端向服务器端发送请求报文时使用的首部。补充了请求的附加 内容、客户端信息、响应内容相关优先级等信息。 

`响应首部字段（Response Header Fields） `
- 从服务器端向客户端返回响应报文时使用的首部。补充了响应的附加 内容，也会要求客户端附加额外的内容信息。 

`实体首部字段（Entity Header Fields） `
- 针对请求报文和响应报文的实体部分使用的首部。补充了资源内容更 新时间等与实体有关的信息。

#### 6.2.4 HTTP/1.1 首部字段一览 

HTTP/1.1 规范定义了如下 47 种首部字段。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e2e8f304b67fc5e08356e5e08ab1f1e7.png)
#### 6.2.5 非 HTTP/1.1 首部字段 

在 HTTP 协议通信交互中使用到的首部字段，不限于 RFC2616 中定 义的 47 种首部字段。还有 Cookie、Set-Cookie 和 Content-Disposition 等在其他 RFC 中定义的首部字段，它们的使用频率也很高。 

这些非正式的首部字段统一归纳在 RFC4229 HTTP Header Field Registrations 中。

#### 6.2.6 End-to-end 首部和 Hop-by-hop 首部

HTTP 首部字段将定义成缓存代理和非缓存代理的行为，分成 2 种类型。 

`端到端首部（End-to-end Header） `
- 分在此类别中的首部会转发给请求 / 响应对应的最终接收目标，且必须保存在由缓存生成的响应中，另外规定它必须被转发。 

`逐跳首部（Hop-by-hop Header）` 
- 分在此类别中的首部只对单次转发有效，会因通过缓存或代理而不再转发。HTTP/1.1 和之后版本中，如果要使用 hop-by-hop 首部，需提 供 Connection 首部字段。

下面列举了 HTTP/1.1 中的逐跳首部字段。除这 8 个首部字段之外， 其他所有字段都属于端到端首部。 
- Connection Keep-Alive 
- Proxy-Authenticate 
- Proxy-Authorization 
- Trailer 
- TE 
- Transfer-Encoding 
- Upgrade
### 6.3 3、HTTP/1.1 通用首部字段

通用首部字段是指，请求报文和响应报文双方都会使用的首部。

#### 6.3.1 Cache-Control

通过指定首部字段 Cache-Control 的指令，就能操作缓存的工作机 制。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8596fb862640b02bbe7bba28ce965400.png)

指令的参数是可选的，多个指令之间通过“,”分隔。首部字段 Cache-Control 的指令可用于请求及响应时。 
- Cache-Control: private, max-age=0, no-cache 

Cache-Control 指令一览 可用的指令按请求和响应分类如下所示。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c88db90041c53b947e895fc83a0dd74e.png)

##### 6.3.1.1 表示是否能缓存的指令

`public 指令 `
- Cache-Control: public 当指定使用 public 指令时，则明确表明其他用户也可利用缓存。 

`private 指令`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/815433b9c4a4ffb65b2d555c4051d0c0.png)
- `当指定 private 指令后，响应只以特定的用户作为对象`，这与 public 指令的行为相反。 缓存服务器会对该特定用户提供资源缓存的服务，对于其他用户发送过来的请求，代理服务器则不会返回缓存。

`no-cache 指令`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6a4bda89aefab767c62062a37939e3da.png)
- Cache-Control: no-cache

- ==使用 no-cache 指令的目的是为了防止从缓存中返回过期的资源==。

- 客户端发送的请求中如果包含 no-cache 指令，则表示客户端将不会接收缓存过的响应。于是，“中间”的缓存服务器必须把客户端请求转发给源服务器。 

- 如果服务器返回的响应中包含 no-cache 指令，那么缓存服务器不能对资源进行缓存。源服务器以后也将不再对缓存服务器请求中提出的资源有效性进行确认，且禁止其对响应资源进行缓存操作。
- Cache-Control: no-cache=Location

- 由服务器返回的响应中，若报文首部字段 Cache-Control 中对 no-cache 字段名具体指定参数值，那么客户端在接收到这个被指定参数值的首部字段对应的响应报文后，就不能使用缓存。换言之，`无参数值的首部字段可以使用缓存。只能在响应指令中指定该参数`。

##### 6.3.1.2 控制可执行缓存的对象的指令

`no-store 指令 `
Cache-Control: no-store ==当使用 no-store 指令时，暗示请求（和对应的响应）或响应中包含 机密信息==。 

因此，该指令规定缓存不能在本地存储请求或响应的任一部分。 指定缓存期限和认证的指令 s-maxage 指令 Cache-Control: s-maxage=604800（单位 ：秒）

>从字面意思上很容易把 no-cache 误解成为不缓存，但`事实上 no-cache 代表不缓 存过期的资源`，缓存会向源服务器进行有效期确认后处理资源，也许称为 do-not-serve-from-cache-without-revalidation 更合适。no-store 才是真正地不进行缓存，请读者注意区别理解。

因此，该指令规定缓存不能在本地存储请求或响应的任一部分。

##### 6.3.1.3 指定缓存期限和认证的指令

`s-maxage 指令 `
- Cache-Control: s-maxage=604800（单位 ：秒）
- s-maxage 指令的功能和 max-age 指令的相同，它们的不同点是 s-maxage 指令只适用于供多位用户使用的公共缓存服务器。也就是 说，对于向同一用户重复返回响应的服务器来说，这个指令没有任何 作用。
- 另外，当使用 s-maxage 指令后，则直接忽略对 Expires 首部字段及 max-age 指令的处理。

`max-age 指令`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/24c84262c921f70a3c45e9a74e999931.png)
- Cache-Control: max-age=604800（单位：秒）
- 当客户端发送的请求中包含 max-age 指令时，如果判定缓存资源的缓存时间数值比指定时间的数值更小，那么客户端就接收缓存的资源。 另外，当指定 max-age 值为 0，那么缓存服务器通常需要将请求转发给源服务器。
- 当服务器返回的响应中包含 max-age 指令时，缓存服务器将不对资源 的有效性再作确认，而 max-age 数值代表资源保存为缓存的最长时间。
- 应用 HTTP/1.1 版本的缓存服务器遇到同时存在 Expires 首部字段的情 况时，会优先处理 max-age 指令，而忽略掉 Expires 首部字段。而 HTTP/1.0 版本的缓存服务器的情况却相反，max-age 指令会被忽略掉。

`min-fresh 指令`![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/aafa9b13f2dce727c24d2adcd06754b5.png)
- Cache-Control: min-fresh=60（单位：秒）
- min-fresh 指令要求缓存服务器返回至少还未过指定时间的缓存资源。
- 比如，当指定 min-fresh 为 60 秒后，过了 60 秒的资源都无法作为响 应返回了。

`max-stale 指令`
- Cache-Control: max-stale=3600（单位：秒）
- 使用 max-stale 可指示缓存资源，即使过期也照常接收。
- 如果指令未指定参数值，那么无论经过多久，客户端都会接收响应； 如果指令中指定了具体数值，那么即使过期，只要仍处于 max-stale 指定的时间内，仍旧会被客户端接收。

`only-if-cached 指令`
- Cache-Control: only-if-cached
- 使用 only-if-cached 指令表示客户端仅在缓存服务器本地缓存目标资 源的情况下才会要求其返回。换言之，该指令要求缓存服务器不重新 加载响应，也不会再次确认资源有效性。若发生请求缓存服务器的本 地缓存无响应，则返回状态码 504 Gateway Timeout。

`must-revalidate 指令`
- Cache-Control: must-revalidate
- 使用 must-revalidate 指令，代理会向源服务器再次验证即将返回的响 应缓存目前是否仍然有效。
- 若代理无法连通源服务器再次获取有效资源的话，缓存必须给客户端 一条 504（Gateway Timeout）状态码。
- 另外，使用 must-revalidate 指令会忽略请求的 max-stale 指令（即使已 经在首部使用了 max-stale，也不会再有效果）。

`proxy-revalidate 指令`
- Cache-Control: proxy-revalidate
- proxy-revalidate 指令要求所有的缓存服务器在接收到客户端带有该指 令的请求返回响应之前，必须再次验证缓存的有效性。

`no-transform 指令`
- Cache-Control: no-transform
- 使用 no-transform 指令规定无论是在请求还是响应中，缓存都不能改 变实体主体的媒体类型。
- 这样做可防止缓存或代理压缩图片等类似操作。

##### 6.3.1.4 Cache-Control 扩展

`cache-extension token`
- Cache-Control: private, community="UCI"
- 通过 cache-extension 标记（token），可以扩展 Cache-Control 首部字 段内的指令。
- 如上例，Cache-Control 首部字段本身没有 community 这个指令。借助 extension tokens 实现了该指令的添加。如果缓存服务器不能理解 community 这个新指令，就会直接忽略。因此，extension tokens 仅对 能理解它的缓存服务器来说是有意义的。
#### 6.3.2 Connection

Connection 首部字段具备如下两个作用。
- 控制不再转发给代理的首部字段
- 管理持久连接

控制不再转发给代理的首部字段![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/acfa83ec57d2e5965905022ee91f80f6.png)
- 在客户端发送请求和服务器返回响应内，使用 Connection 首部字 段，可控制不再转发给代理的首部字段（即 Hop-by-hop 首 部）。

管理持久连接![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c049d8fd0c1a69d6c73dc36c92d270d2.png)
- HTTP/1.1 版本的默认连接都是持久连接。为此，客户端会在持 久连接上连续发送请求。当服务器端想明确断开连接时，则指定 Connection 首部字段的值为 Close。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dcb511cf7eaadb699398e4a247f3d520.png)
- HTTP/1.1 之前的 HTTP 版本的默认连接都是非持久连接。为 此，如果想在旧版本的 HTTP 协议上维持持续连接，则需要指定 Connection 首部字段的值为 Keep-Alive。
- 如上图①所示，客户端发送请求给服务器时，服务器端会像上图 ②那样加上首部字段 Keep-Alive 及首部字段 Connection 后返回 响应。
#### 6.3.3 Date

首部字段 Date 表明创建 HTTP 报文的日期和时间。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1cb5831b8b20e266e4abbe76822855f7.png)
HTTP/1.1 协议使用在 RFC1123 中规定的日期时间的格式，如下 示 例。
- `Date: Tue, 03 Jul 2012 04:40:59 GMT`

之前的 HTTP 协议版本中使用在 RFC850 中定义的格式，如下所示。
- `Date: Tue, 03-Jul-12 04:40:59 GMT`

除此之外，还有一种格式。它与 C 标准库内的 asctime() 函数的输出 格式一致。
- `Date: Tue Jul 03 04:40:59 2012`
#### 6.3.4 Pragma

Pragma 是 HTTP/1.1 之前版本的历史遗留字段，仅作为与 HTTP/1.0 的向后兼容而定义。 

规范定义的形式唯一，如下所示。 
- Pragma: no-cache 
- 该首部字段属于通用首部字段，但==只用在客户端发送的请求==中。客户端会要求所有的中间服务器不返回缓存的资源。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0728ddb735970cf34f7628041a2cd35d.png)
- 所有的中间服务器如果都能以 HTTP/1.1 为基准，那直接采用 Cache-Control: no-cache 指定缓存的处理方式是最为理想的。但要整体掌握 全部中间服务器使用的 HTTP 协议版本却是不现实的。因此，发送的 请求会同时含有下面两个首部字段。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3545fbcf6584de280924dadc5144e6d8.png)
#### 6.3.5 Trailer

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/91e814ab4612aafb0d47395b67d92a0f.png)

首部字段 Trailer 会事先说明在报文主体后记录了哪些首部字段。该 首部字段可应用在 HTTP/1.1 版本分块传输编码时。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/976f663038acaa722ba06d91df26ad75.png)
以上用例中，指定首部字段 Trailer 的值为 Expires，在报文主体之后 （分块长度 0 之后）出现了首部字段 Expires。
#### 6.3.6 Transfer-Encoding

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0ea634936393e8c50f0f28ce69af2513.png)

首部字段 Transfer-Encoding 规定了传输报文主体时采用的编码方式。 

HTTP/1.1 的传输编码方式仅对分块传输编码有效。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1ee32c5bea0a41947b41db5b1af807ba.png)
以上用例中，正如在首部字段 Transfer-Encoding 中指定的那样，有效 使用分块传输编码，且分别被分成 3312 字节和 914 字节大小的分块 数据。

#### 6.3.7 Upgrade

首部字段 Upgrade 用于检测 HTTP 协议及其他协议是否可使用更高的 版本进行通信，其参数值可以用来指定一个完全不同的通信协议。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5ba182a8c259b9571849881257cc7527.png)
上图用例中，首部字段 Upgrade 指定的值为 TLS/1.0。请注意此处两 个字段首部字段的对应关系，Connection 的值被指定为 Upgrade。 Upgrade 首部字段产生作用的 Upgrade 对象仅限于客户端和邻接服务 器之间。因此，使用首部字段 Upgrade 时，还需要额外指定 Connection:Upgrade。

对于附有首部字段 Upgrade 的请求，服务器可用 101 Switching Protocols 状态码作为响应返回。

#### 6.3.8 Via

使用首部字段 Via 是为了追踪客户端与服务器之间的请求和响应报文的`传输路径`。 

报文经过代理或网关时，会先在首部字段 Via 中附加该服务器的信 息，然后再进行转发。这个做法和 traceroute 及电子邮件的 Received 首部的工作机制很类似。 

首部字段 Via 不仅用于追踪报文的转发，还可避免请求回环的发生。 所以`必须在经过代理时附加该首部字段内容`。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b9dfaaeb38f5e9a9dcad0caa4a26ed3f.png)

上图用例中，在经过代理服务器 A 时，Via 首部附加了“1.0 gw.hackr.jp (Squid/3.1)”这样的字符串值。行头的 1.0 是指接收请求的服务器上应用的 HTTP 协议版本。接下来经过代理服务器 B 时亦是如 此，在 Via 首部附加服务器信息，也可增加 1 个新的 Via 首部写入服 务器信息。

Via 首部是为了追踪传输路径，所以经常会和 TRACE 方法一起使 用。比如，代理服务器接收到由 TRACE 方法发送过来的请求（其中 Max-Forwards: 0）时，代理服务器就不能再转发该请求了。这种情况 下，代理服务器会将自身的信息附加到 Via 首部后，返回该请求的响 应。

#### 6.3.9 Warning

HTTP/1.1 的 Warning 首部是从 HTTP/1.0 的响应首部（Retry-After）演变过来的。该首部通常会告知用户一些与缓存相关的问题的警告。 
- Warning: 113 gw.hackr.jp:8080 "Heuristic expiration" Tue, 03 Jul 2012 05:09:44 GMT 

Warning 首部的格式如下。最后的日期时间部分可省略。 
- Warning: `[警告码][警告的主机:端口号]“[警告内容]”([日期时间])` 

HTTP/1.1 中定义了 7 种警告。警告码对应的警告内容仅推荐参考。 另外，警告码具备扩展性，今后有可能追加新的警告码。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ae89b7165c1a4e099c152b93d705dcfd.png)
### 6.4 4、请求首部字段

请求首部字段是从客户端往服务器端发送请求报文中所使用的字段， 用于补充请求的附加信息、客户端信息、对响应内容相关的优先级等 内容。

#### 6.4.1 Accept

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f27d1901134814fdfd402ab805207017.png)

`Accept 首部字段可通知服务器，用户代理能够处理的媒体类型及媒体类型的相对优先级`。可使用 type/subtype 这种形式，一次指定多种媒体类型。

下面我们试举几个媒体类型的例子。

- `文本文件` text/html, text/plain, text/css ... application/xhtml+xml, application/xml ... 
- `图片文件` image/jpeg, image/gif, image/png ... 
- `视频文件` video/mpeg, video/quicktime ... 
- `应用程序使用的二进制文件` application/octet-stream, application/zip ... 

比如，如果浏览器不支持 PNG 图片的显示，那 Accept 就不指定 image/png，而指定可处理的 image/gif 和 image/jpeg 等图片类型。 

若想要给显示的媒体类型增加优先级，则使用 q= 来额外表示权重值 ，用分号（;）进行分隔。权重值 q 的范围是 0~1（可精确到小数点 后 3 位），且 1 为最大值。不指定权重 q 值时，默认权重为 q=1.0。

当服务器提供多种内容时，将会首先返回权重值最高的媒体类型。
#### 6.4.2 Accept-Charset

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9ca48f2349db43428c56458146989997.png)

`Accept-Charset 首部字段可用来通知服务器用户代理支持的字符集及 字符集的相对优先顺序`。另外，可一次性指定多种字符集。与首部字 段 Accept 相同的是可用权重 q 值来表示相对优先级。 

该首部字段应用于内容协商机制的服务器驱动协商。

#### 6.4.3 Accept-Encoding

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cc9726deebac8b55aba115823615b55b.png)

`Accept-Encoding 首部字段用来告知服务器用户代理支持的内容编码及内容编码的优先级顺序`。可一次性指定多种内容编码。

下面试举出几个内容编码的例子。

- `gzip` 由文件压缩程序 gzip（GNU zip）生成的编码格式 （RFC1952），采用 Lempel-Ziv 算法（LZ77）及 32 位循环冗余 校验（Cyclic Redundancy Check，通称 CRC）。 
- `compress` 由 UNIX 文件压缩程序 compress 生成的编码格式，采用 Lempel-Ziv-Welch 算法（LZW）。 
- `deflate` 组合使用 zlib 格式（RFC1950）及由 deflate 压缩算法 （RFC1951）生成的编码格式。 
- `identity` 不执行压缩或不会变化的默认编码格式 

采用权重 q 值来表示相对优先级，这点与首部字段 Accept 相同。另 外，也可使用星号`（*）`作为通配符，指定任意的编码格式。
#### 6.4.4 Accept-Language

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0e3911699ea11c8357460c84e6b7c40b.png)

`首部字段 Accept-Language 用来告知服务器用户代理能够处理的自然语言集（指中文或英文等），以及自然语言集的相对优先级`。可一次 指定多种自然语言集。 

和 Accept 首部字段一样，按权重值 q 来表示相对优先级。在上述图 例中，客户端在服务器有中文版资源的情况下，会请求其返回中文版 对应的响应，没有中文版时，则请求返回英文版响应。
#### 6.4.5 Authorization

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b722b4a5779138294f2598a2dc1145e7.png)

`首部字段 Authorization 是用来告知服务器，用户代理的认证信息（证书值）`。通常，想要通过服务器认证的用户代理会在接收到返回的 401 状态码响应后，把首部字段 Authorization 加入请求中。共用缓存在接收到含有 Authorization 首部字段的请求时的操作处理会略有差异。 

有关 HTTP 访问认证及 Authorization 首部字段，稍后的章节还会详细 说明。另外，读者也可参阅 RFC2616。
#### 6.4.6 Expect

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9ff86c9ae3b6d8c3c40f7773d8467c4a.png)

`客户端使用首部字段 Expect 来告知服务器，期望出现的某种特定行 为`。因服务器无法理解客户端的期望作出回应而发生错误时，会返回 状态码 417 Expectation Failed。 

客户端可以利用该首部字段，写明所期望的扩展。虽然 HTTP/1.1 规 范只定义了 100-continue（状态码 100 Continue 之意）。 

等待状态码 100 响应的客户端在发生请求时，需要指定 Expect:100- continue。
#### 6.4.7 From

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f82b2c1a45bdf35dd7978dd6e789758d.png)

`首部字段 From 用来告知服务器使用用户代理的用户的电子邮件地址`。通常，其使用目的就是为了显示搜索引擎等用户代理的负责人的 电子邮件联系方式。使用代理时，应尽可能包含 From 首部字段（但 可能会因代理不同，将电子邮件地址记录在 User-Agent 首部字段 内）。
#### 6.4.8 Host

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6099231a2e24a60dd2fc2626f5d26965.png)


首部字段 Host 会告知服务器，请求的资源所处的互联网主机名和端 口号。`Host 首部字段在 HTTP/1.1 规范内是唯一一个必须被包含在请求内的首部字段`。 

首部字段 Host 和以单台服务器分配多个域名的虚拟主机的工作机制 有很密切的关联，这是首部字段 Host 必须存在的意义。 

请求被发送至服务器时，请求中的主机名会用 IP 地址直接替换解 决。但如果这时，相同的 IP 地址下部署运行着多个域名，那么服务 器就会无法理解究竟是哪个域名对应的请求。因此，就需要使用首部 字段 Host 来明确指出请求的主机名。若服务器未设定主机名，那直 接发送一个空值即可。如下所示。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7eb5c50abb9ce9b73da8c692df57856.png)
#### 6.4.9 If-Match

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ad9185ef897831e9b81ecc59c36821db.png)

形如 If-xxx 这种样式的请求首部字段，都可称为条件请求。服务器接收到附带条件的请求后，只有判断指定条件为真时，才会执行请求。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6191761d6703a054c747e6134718d520.png)
`If-Match: "123456"`

首部字段 If-Match，属附带条件之一，它会告知服务器匹配资源所用的实体标记（ETag）值。这时的服务器无法使用弱 ETag 值。（请参 照本章有关首部字段 ETag 的说明） 

服务器会比对 If-Match 的字段值和资源的 ETag 值，仅当两者一致 时，才会执行请求。反之，则返回状态码 412 Precondition Failed 的响应。 

还可以使用`星号（*）`指定 If-Match 的字段值。针对这种情况，服务 器将会忽略 ETag 的值，只要资源存在就处理请求。
#### 6.4.10 If-Modified-Since

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/31db1be536354cd8b95d5447b4ad1df2.png)

首部字段 If-Modified-Since，属附带条件之一，它会告知服务器若 If-Modified-Since 字段值早于资源的更新时间，则希望能处理该请求。 而在指定 If-Modified-Since 字段值的日期时间之后，如果请求的资源 都没有过更新，则返回状态码 304 Not Modified 的响应。 

If-Modified-Since 用于确认代理或客户端拥有的本地资源的有效性。 获取资源的更新日期时间，可通过确认首部字段 Last-Modified 来确定。
#### 6.4.11 If-None-Match

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2d292542b7e5f106f9d6e1d536e68a23.png)

首部字段 If-None-Match 属于附带条件之一。它和首部字段 If-Match 作用相反。用于指定 If-None-Match 字段值的实体标记（ETag）值与 请求资源的 ETag 不一致时，它就告知服务器处理该请求。 

在 GET 或 HEAD 方法中使用首部字段 If-None-Match 可获取最新的资 源。因此，这与使用首部字段 If-Modified-Since 时有些类似。

#### 6.4.12 If-Range

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e222f50ae2f10483612179c17799932e.png)

首部字段 If-Range 属于附带条件之一。它告知服务器若指定的 If-Range 字段值（ETag 值或者时间）和请求资源的 ETag 值或时间相一 致时，则作为范围请求处理。反之，则返回全体资源。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d01a50d9e4686116b782484de7285db1.png)

下面我们思考一下不使用首部字段 If-Range 发送请求的情况。服务器 端的资源如果更新，那客户端持有资源中的一部分也会随之无效，当然，范围请求作为前提是无效的。这时，服务器会暂且以状态码 412 Precondition Failed 作为响应返回，其目的是催促客户端再次发送请求。这样一来，与使用首部字段 If-Range 比起来，就需要花费两倍的功夫。
#### 6.4.13 If-Unmodified-Since

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bb2157b4d4b8cc06704e956820e580e3.png)

首部字段 If-Unmodified-Since 和首部字段 If-Modified-Since 的作用相反。它的作用的是告知服务器，指定的请求资源只有在字段值内指定的日期时间之后，未发生更新的情况下，才能处理请求。如果在指定日期时间后发生了更新，则以状态码 412 Precondition Failed 作为响应 返回。

#### 6.4.14 Max-Forwards

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7889be675b5bf5dad5221cbd291062e5.png)

通过 TRACE 方法或 OPTIONS 方法，发送包含首部字段 Max-Forwards 的请求时，该字段以十进制整数形式指定可经过的服务器最 大数目。服务器在往下一个服务器转发请求之前，Max-Forwards 的值减 1 后重新赋值。当服务器接收到 Max-Forwards 值为 0 的请求 时，则不再进行转发，而是直接返回响应。 

使用 HTTP 协议通信时，请求可能会经过代理等多台服务器。途中， 如果代理服务器由于某些原因导致请求转发失败，客户端也就等不到服务器返回的响应了。对此，我们无从可知。 

可以灵活使用首部字段 Max-Forwards，针对以上问题产生的原因展开调查。由于当 Max-Forwards 字段值为 0 时，服务器就会立即返回响应，由此我们至少可以对以那台服务器为终点的传输路径的通信状况有所把握。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bb07d83721ae3002b07a7dd492d4c170.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/608c143c7bff727da9e523fdbd5e65c6.png)
#### 6.4.15 Proxy-Authorization

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0a234a32cfe82184018a58997abde71b.png)

接收到从代理服务器发来的认证质询时，客户端会发送包含首部字段 Proxy-Authorization 的请求，以告知服务器认证所需要的信息。 

这个行为是与客户端和服务器之间的 HTTP 访问认证相类似的，不同之处在于，认证行为发生在客户端与代理之间。客户端与服务器之间 的认证，使用首部字段 Authorization 可起到相同作用。有关 HTTP 访问认证，后面的章节会作详尽阐述。
#### 6.4.16 Range

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7cfd4d2d6c999e41952679b6c86687ec.png)

对于只需获取部分资源的范围请求，包含首部字段 Range 即可告知服务器资源的指定范围。上面的示例表示请求获取从第 5001 字节至第 10000 字节的资源。 

接收到附带 Range 首部字段请求的服务器，会在处理请求之后返回状态码为 206 Partial Content 的响应。无法处理该范围请求时，则会返回状态码 200 OK 的响应及全部资源。
#### 6.4.17 Referer

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/68fbb600bc56e26c32a2f690c5a423da.png)

客户端一般都会发送 Referer 首部字段给服务器。但当直接在浏览器 的地址栏输入 URI，或出于安全性的考虑时，也可以不发送该首部字段。 

因为原始资源的 URI 中的查询字符串可能含有 ID 和密码等保密信 息，要是写进 Referer 转发给其他服务器，则有可能导致保密信息的泄露。 

另外，Referer 的正确的拼写应该是 Referrer，但不知为何，大家一直沿用这个错误的拼写。
#### 6.4.18 TE

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/570c20aa025b9b74a502a2c66ee7922c.png)

首部字段 TE 会告知服务器客户端能够处理响应的传输编码方式及相 对优先级。它和首部字段 Accept-Encoding 的功能很相像，但是用于传输编码。 

首部字段 TE 除指定传输编码之外，还可以指定伴随 trailer 字段的分块传输编码的方式。应用后者时，只需把 trailers 赋值给该字段值。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/54f8bc8f976fc4bd1742f353b6c0632a.png)
#### 6.4.19 User-Agent

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/981e973acc4f8cebeb3d0e3550ef7807.png)

首部字段 User-Agent 会将创建请求的浏览器和用户代理名称等信息传达给服务器。 

由网络爬虫发起请求时，有可能会在字段内添加爬虫作者的电子邮件 地址。此外，如果请求经过代理，那么中间也很可能被添加上代理服 务器的名称。

### 6.5 5、响应首部字段

响应首部字段是由服务器端向客户端返回响应报文中所使用的字段， 用于补充响应的附加信息、服务器信息，以及对客户端的附加要求等 信息。

#### 6.5.1 Accept-Ranges

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/dad708c312b2aa297108459fc4789ae7.png)

首部字段 Accept-Ranges 是用来告知客户端服务器`是否能处理范围请求`，以指定获取服务器端某个部分的资源。 

可指定的字段值有两种，可处理范围请求时指定其为 bytes，反之则 指定其为 none。
#### 6.5.2 Age

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/cdb59fe20c51c826cf58aeeecc2e7743.png)

首部字段 Age 能告知客户端，源服务器在多久前创建了响应。字段值的单位为秒。 

若创建该响应的服务器是缓存服务器，Age 值是指缓存后的响应再次发起认证到认证完成的时间值。代理创建响应时必须加上首部字段 Age。
#### 6.5.3 ETag

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/283ed17eccf862774aaee0d6932df88d.png)

首部字段 ETag 能告知客户端实体标识。它是一种可将资源以字符串 形式做唯一性标识的方式。`服务器会为每份资源分配对应的 ETag 值`。

另外，`当资源更新时，ETag 值也需要更新`。生成 ETag 值时，并没有统一的算法规则，而仅仅是由服务器来分配。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fb47cf190e37d4bd6d0dc5f774498f6f.png)

资源被缓存时，就会被分配唯一性标识。例如，当使用中文版的浏览 器访问 http://www.google.com/ 时，就会返回中文版对应的资源，而使用英文版的浏览器访问时，则会返回英文版对应的资源。两者的 URI 是相同的，所以仅凭 URI 指定缓存的资源是相当困难的。若在下载过程中出现连接中断、再连接的情况，都会依照 ETag 值来指定资 源。

`强 ETag 值和弱 Tag 值`
- 强 ETag 值，不论实体发生多么细微的变化都会改变其值。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f18f741aee998ce5fa1f6ea2ee8ddea8.png)
- 弱 ETag 值只用于提示资源是否相同。只有资源发生了根本改变，产 生差异时才会改变 ETag 值。这时，会在字段值最开始处附加 W/。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8e9ff31cdae902e9a10e3e4d0f77b303.png)
#### 6.5.4 Location

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/78b744a573977b7bd1b7e20855143998.png)

使用首部字段 Location 可以将响应接收方引导至某个与请求 URI 位置 不同的资源。 

基本上，该字段会配合 3xx ：Redirection 的响应，提供重定向的 URI。 

几乎所有的浏览器在接收到包含首部字段 Location 的响应后，都会强制性地尝试对已提示的重定向资源的访问。

#### 6.5.5 Proxy-Authenticate

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7df2b848e66a8a1a294611535e042714.png)

首部字段 Proxy-Authenticate 会把由`代理服务器所要求的认证信息发送给客户端`。 

它与客户端和服务器之间的 HTTP 访问认证的行为相似，不同之处在于其认证行为是在客户端与代理之间进行的。而客户端与服务器之间 进行认证时，首部字段 WWW-Authorization 有着相同的作用。有关 HTTP 访问认证，后面的章节会再进行详尽阐述。
#### 6.5.6 Retry-After

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1c71272cfdc44a640c3fe21523e34966.png)

首部字段 Retry-After 告知客户端应该在多久之后再次发送请求。主要 配合状态码 503 Service Unavailable 响应，或 3xx Redirect 响应一起使用。 

字段值可以指定为具体的日期时间（Wed, 04 Jul 2012 06：34：24 GMT 等格式），也可以是创建响应后的秒数。
#### 6.5.7 Server

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2109b1efbddaa3947af883d4aeb2e41b.png)

首部字段 Server 告知客户端当前服务器上安装的 HTTP 服务器应用程 序的信息。不单单会标出服务器上的软件应用名称，还有可能包括版 本号和安装时启用的可选项。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/3e99fbbff8509419f5aec4cac2cf4d87.png)
#### 6.5.8 Vary

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a4c554f35aafd8a30f9c96e98198af1f.png)

当代理服务器接收到带有 Vary 首部字段指定获取资源的请求 时，如果使用的 Accept-Language 字段的值相同，那么就直接从缓存返回响应。反之，则需要先从源服务器端获取资源后才能作为响应返回

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ea2698051d5cd2892d87820859ac2da1.png)
首部字段 Vary 可对缓存进行控制。源服务器会向代理服务器传达关 于本地缓存使用方法的命令。 

从代理服务器接收到源服务器返回包含 Vary 指定项的响应之后，若 再要进行缓存，仅对请求中含有相同 Vary 指定首部字段的请求返回 缓存。即使对相同资源发起请求，但由于 Vary 指定的首部字段不相 同，因此必须要从源服务器重新获取资源。
#### 6.5.9 WWW-Authenticate

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/534965e2dd753474eac445f188db8020.png)

首部字段 WWW-Authenticate 用于 HTTP 访问认证。它会告知客户端适用于访问请求 URI 所指定资源的认证方案（Basic 或是 Digest）和 带参数提示的质询（challenge）。状态码 401 Unauthorized 响应中， 肯定带有首部字段 WWW-Authenticate。 

上述示例中，realm 字段的字符串是为了辨别请求 URI 指定资源所受到的保护策略。有关该首部，请参阅本章之后的内容。

### 6.6 6、实体首部字段

实体首部字段是包含在请求报文和响应报文中的实体部分所使用的首 部，用于补充内容的更新时间等与实体相关的信息。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/444f656b8817e83f06e30693ae61ff0c.png)
#### 6.6.1 Allow

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b69a1267d53737411a8aaa2be4d203e7.png)

首部字段 Allow 用于通知客户端能够支持 Request-URI 指定资源的所 有 HTTP 方法。当服务器接收到不支持的 HTTP 方法时，会以状态码 405 Method Not Allowed 作为响应返回。与此同时，还会把所有能支 持的 HTTP 方法写入首部字段 Allow 后返回。
#### 6.6.2 Content-Encoding

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6b3a0c661498ddb83616b0d80406e94a.png)

首部字段 Content-Encoding 会告知客户端服务器对实体的主体部分选用的内容编码方式。内容编码是指在不丢失实体信息的前提下所进行 的压缩。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e42391efaae38cea5d6afd75ac1ebd0d.png)

主要采用以下 4 种内容编码的方式。（各方式的说明请参考 6.4.3 节 Accept-Encoding 首部字段）。
#### 6.6.3 Content-Language

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e6415baa26b00fd605a99bda206af110.png)

首部字段 Content-Language 会告知客户端，实体主体使用的自然语言 （指中文或英文等语言）。
#### 6.6.4 Content-Length

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b7bf0cf2f0cdb7c0a455b154e299d0cd.png)

首部字段 Content-Length 表明了实体主体部分的大小（单位是字 节）。对实体主体进行内容编码传输时，不能再使用 Content-Length 首部字段。由于实体主体大小的计算方法略微复杂，所以在此不再展 开。读者若想一探究竟，可参考 RFC2616 的 4.4。
#### 6.6.5 Content-Location

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/035af624081a18e4acc8195965ca0ac0.png)

首部字段 Content-Location 给出与报文主体部分相对应的 URI。和首部字段 Location 不同，Content-Location 表示的是报文主体返回资源对应的 URI。 

比如，对于使用首部字段 Accept-Language 的服务器驱动型请求，当 返回的页面内容与实际请求的对象不同时，首部字段 Content-Location 内会写明 URI。（访问 http://www.hackr.jp/ 返回的对象却是 http://www.hackr.jp/index-ja.html 等类似情况）
#### 6.6.6 Content-MD5

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/586f61fcd7253377f6f62d7f6db363d6.png)

首部字段 Content-MD5 是一串由 MD5 算法生成的值，其目的在于检查报文主体在传输过程中是否保持完整，以及确认传输到达。 

对报文主体执行 MD5 算法获得的 128 位二进制数，再通过 Base64 编 码后将结果写入 Content-MD5 字段值。由于 HTTP 首部无法记录二进制值，所以要通过 Base64 编码处理。为确保报文的有效性，作为接收方的客户端会对报文主体再执行一次相同的 MD5 算法。计算出的值与字段值作比较后，即可判断出报文主体的准确性。 

采用这种方法，对内容上的偶发性改变是无从查证的，也无法检测出 恶意篡改。其中一个原因在于，内容如果能够被篡改，那么同时意味 着 Content-MD5 也可重新计算然后被篡改。所以处在接收阶段的客户 端是无法意识到报文主体以及首部字段 Content-MD5 是已经被篡改过的。
#### 6.6.7 Content-Range

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e28d0e37b2a9f3acb842e3656ba97217.png)

针对范围请求，返回响应时使用的首部字段 Content-Range，能告知客 户端作为响应返回的实体的哪个部分符合范围请求。字段值以字节为 单位，表示当前发送部分及整个实体大小。
#### 6.6.8 Content-Type

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ecceaa2512715574ede9305c5d04eee5.png)

首部字段 Content-Type 说明了实体主体内对象的媒体类型。和首部字 段 Accept 一样，字段值用 type/subtype 形式赋值。 

参数 charset 使用 iso-8859-1 或 euc-jp 等字符集进行赋值。
#### 6.6.9 Expires

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/71f488cab35c29a3319790bda7359977.png)

首部字段 Expires 会将资源失效的日期告知客户端。缓存服务器在接收到含有首部字段 Expires 的响应后，会以缓存来应答请求，在 Expires 字段值指定的时间之前，响应的副本会一直被保存。当超过 指定的时间后，缓存服务器在请求发送过来时，会转向源服务器请求 资源。 

源服务器不希望缓存服务器对资源缓存时，最好在 Expires 字段内写入与首部字段 Date 相同的时间值。 

但是，当首部字段 Cache-Control 有指定 max-age 指令时，比起首部字段 Expires，会优先处理 max-age 指令。
#### 6.6.10 Last-Modified

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9753dc8603b1d9f00c3c7cc2ef392d31.png)

首部字段 Last-Modified 指明资源最终修改的时间。一般来说，这个值就是 Request-URI 指定资源被修改的时间。但类似使用 CGI 脚本进 行动态数据处理时，该值有可能会变成数据最终修改时的时间。
### 6.7 7、为Cookie服务的首部字段

管理服务器与客户端之间状态的 Cookie，虽然没有被编入标准化 HTTP/1.1 的 RFC2616 中，但在 Web 网站方面得到了广泛的应用。 

Cookie 的工作机制是用户识别及状态管理。Web 网站为了管理用户的状态会通过 Web 浏览器，把一些数据临时写入用户的计算机内。接着当用户访问该Web网站时，可通过通信方式取回之前发放的 Cookie。 

调用 Cookie 时，由于可校验 Cookie 的有效期，以及发送方的域、路径、协议等信息，所以正规发布的 Cookie 内的数据不会因来自其他 Web 站点和攻击者的攻击而泄露。 

至 2013 年 5 月，Cookie 的规格标准文档有以下 4 种。 

`由网景公司颁布的规格标准 `
- 网景通信公司设计并开发了 Cookie，并制定相关的规格标准。1994 年前后，Cookie 正式应用在网景浏览器中。目前最为普及的 Cookie 方式也是以此为基准的。 

`RFC2109 `
- 某企业尝试以独立技术对 Cookie 规格进行标准化统筹。原本的意图是想和网景公司制定的标准交互应用，可惜发生了微妙的差异。现在该标准已淡出了人们的视线。 

`RFC2965 `
- 为终结 Internet Explorer 浏览器与 Netscape Navigator 的标准差异而导致的浏览器战争，RFC2965 内定义了新的 HTTP 首部 Set-Cookie2 和 Cookie2。可事实上，它们几乎没怎么投入使用。 

`RFC6265 `
- 将网景公司制定的标准作为业界事实标准（De facto standard），重新定义 Cookie 标准后的产物。 
- 目前使用最广泛的 Cookie 标准却不是 RFC 中定义的任何一个。而是 在网景公司制定的标准上进行扩展后的产物。 

本节接下来就对目前使用最为广泛普及的标准进行说明。 下面的表格内列举了与 Cookie 有关的首部字段。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0811f8182526270ba7902fba1033a64d.png)

#### 6.7.1 Set-Cookie

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/2bdc1d5c7bf48b7b4d6bc0544f0016a5.png)

当服务器准备开始管理客户端的状态时，会事先告知各种信息。 

下面的表格列举了 Set-Cookie 的字段值。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f3782830489c412d93a0f4f9efb7fda3.png)

`expires 属性 `
- Cookie 的 expires 属性指定浏览器可发送 Cookie 的有效期。 
- 当省略 expires 属性时，其有效期仅限于维持浏览器会话（Session） 时间段内。这通常限于浏览器应用程序被关闭之前。
- 另外，一旦 Cookie 从服务器端发送至客户端，服务器端就不存在可以显式删除 Cookie 的方法。但可通过覆盖已过期的 Cookie，实现对 客户端 Cookie 的实质性删除操作。 

`path 属性 `
- Cookie 的 path 属性可用于限制指定 Cookie 的发送范围的文件目录。 不过另有办法可避开这项限制，看来对其作为安全机制的效果不能抱 有期待。 

`domain 属性 `
- 通过 Cookie 的 domain 属性指定的域名可做到与结尾匹配一致。比 如，当指定 example.com 后，除 example.com 以外，www.example.com 或 www2.example.com 等都可以发送 Cookie。 
- 因此，除了针对具体指定的多个域名发送 Cookie 之 外，不指定 domain 属性显得更安全。 

`secure 属性` 
- Cookie 的 secure 属性用于限制 Web 页面仅在 HTTPS 安全连接时，才可以发送 Cookie。 
- 发送 Cookie 时，指定 secure 属性的方法如下所示。 ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8232e2561e94ac2ff3355b0d55b7932.png)
- 以上例子仅当在 https://www.example.com/（HTTPS）安全连接的情况 下才会进行 Cookie 的回收。也就是说，即使域名相同， http://www.example.com/（HTTP）也不会发生 Cookie 回收行为。 
- 当省略 secure 属性时，不论 HTTP 还是 HTTPS，都会对 Cookie 进行 回收。 

`HttpOnly 属性 `
- Cookie 的 HttpOnly 属性是 Cookie 的扩展功能，它使 JavaScript 脚本无法获得 Cookie。其主要目的为防止跨站脚本攻击（Cross-site scripting，XSS）对 Cookie 的信息窃取。 
- 发送指定 HttpOnly 属性的 Cookie 的方法如下所示。 ![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5e98a7d17190c1e9d76e8eed1c64d7d8.png)
- 通过上述设置，通常从 Web 页面内还可以对 Cookie 进行读取操作。 但使用 JavaScript 的 document.cookie 就无法读取附加 HttpOnly 属性后 的 Cookie 的内容了。因此，也就无法在 XSS 中利用 JavaScript 劫持 Cookie 了。 
- 虽然是独立的扩展功能，但 Internet Explorer 6 SP1 以上版本等当下的主流浏览器都已经支持该扩展了。另外顺带一提，该扩展并非是为了防止 XSS 而开发的。
#### 6.7.2 Cookie

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/abd8cfa4f0e5dd219ece673b062dbc94.png)

首部字段 Cookie 会告知服务器，当客户端想获得 HTTP 状态管理支 持时，就会在请求中包含从服务器接收到的 Cookie。接收到多个 Cookie 时，同样可以以多个 Cookie 形式发送。

### 6.8 8、其他首部字段

HTTP 首部字段是可以自行扩展的。所以在 Web 服务器和浏览器的应 用上，会出现各种非标准的首部字段。 

接下来，我们就一些最为常用的首部字段进行说明。
- X-Frame-Options 
- X-XSS-Protection 
- DNT 
- P3P

#### 6.8.1 X-Frame-Options 

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ba62145bb823315c737a7b6d7040deb5.png)

`首部字段 X-Frame-Options 属于 HTTP 响应首部`，用于控制网站内容在其他 Web 网站的 Frame 标签内的显示问题。其主要目的是为了防止点击劫持（clickjacking）攻击。 

首部字段 X-Frame-Options 有以下两个可指定的字段值。 
- DENY ：拒绝 
- SAMEORIGIN ：仅同源域名下的页面（Top-level-browsing-context）匹配时许可。（比如，当指定 http://hackr.jp/sample.html 页面为 SAMEORIGIN 时，那么 hackr.jp 上所有页面的 frame 都被允许可加载该页面，而 example.com 等其他域名的页面就不行 了） 

支持该首部字段的浏览器有：Internet Explorer 8、Firefox 3.6.9+、 Chrome 4.1.249.1042+、Safari 4+ 和 Opera 10.50+ 等。现在主流的浏览 器都已经支持。 

能在所有的 Web 服务器端预先设定好 X-Frame-Options 字段值是最理想的状态。 

对 apache2.conf 的配置实例![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7ad0f9c7df80d9b68682e399ba4b74f9.png)
#### 6.8.2 X-XSS-Protection 

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/15497a530dc0267d928610ad4635faa1.png)

`首部字段 X-XSS-Protection 属于 HTTP 响应首部`，它是针对跨站脚本攻击（XSS）的一种对策，用于控制浏览器 XSS 防护机制的开关。 

首部字段 X-XSS-Protection 可指定的字段值如下。 
- 0 ：将 XSS 过滤设置成无效状态 
- 1 ：将 XSS 过滤设置成有效状态
#### 6.8.3 DNT 

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ffba972809a024448d9ee0b94333925c.png)

`首部字段 DNT 属于 HTTP 请求首部`，其中 DNT 是 Do Not Track 的简称，意为`拒绝个人信息被收集`，是表示拒绝被精准广告追踪的一种方法。 

首部字段 DNT 可指定的字段值如下。 
- 0 ：同意被追踪 
- 1 ：拒绝被追踪 

由于首部字段 DNT 的功能具备有效性，所以 Web 服务器需要对 DNT 做对应的支持。
#### 6.8.4 P3P

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/47254803df048f20c18a3ec7eb89aa77.png)

`首部字段 P3P 属于 HTTP 相应首部`，通过利用 P3P（The Platform for Privacy Preferences，在线隐私偏好平台）技术，可以让 Web 网站上的个人隐私变成一种仅供程序可理解的形式，以达到保护用户隐私的目的。 

要进行 P3P 的设定，需按以下操作步骤进行。 
- 步骤 1：创建 P3P 隐私 
- 步骤 2：创建 P3P 隐私对照文件后，保存命名在 /w3c/p3p.xml 
- 步骤 3：从 P3P 隐私中新建 Compact policies 后，输出到 HTTP 响应中 

有关 P3P 的详细规范标准请参看下方链接。
- The Platform for Privacy Preferences 1.0（P3P1.0）Specification http://www.w3.org/TR/P3P/

>协议中对 X- 前缀的废除 
>
>在 HTTP 等多种协议中，通过给非标准参数加上前缀 X-，来区别 于标准参数，并使那些非标准的参数作为扩展变成可能。但是这种 简单粗暴的做法有百害而无一益，因此在“RFC 6648 - Deprecating the "X-" Prefix and Similar Constructs in Application Protocols”中提议停止该做法。 
>
>然而，对已经在使用中的 X- 前缀来说，不应该要求其变更。