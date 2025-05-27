---
文章标题: "[[10_※HTTP状态]]" 
文章作者: Dakkk
文章概要: |
  文章详细阐述了HTTP协议无状态的特性，并依次介绍了解决这一问题的Cookie、Session和Token（特别是JWT）机制。它解释了三者各自的工作原理、在客户端/服务端的存储方式、属性、应用场景及在分布式环境下的优缺点，强调了它们在Web状态保持和身份认证中的关键作用。
tags:
- "HTTP"
- "Cookie"
- "Session"
- "Token"
- "JWT"
- "身份认证"
- "无状态"
- "分布式"
相关文章:
- "[[6_📕Spring Security 整合 JWT ：实现身份认证]]"
- "[[8_会话_过滤器_监听器]]"
- "[[9_Cookie&Session]]"
- "[[0_建议]]"
- "[[11_存储 Token 到 Cookie 中]]"
文章分类: "📐 计算机基础"
文章路径: "07-📐 计算机基础/04-🌐 计算机网络/2_应用层/1_HTTP/10_※HTTP状态.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-28 00:44:02
---

HTTP 是“无状态”的，这既是优点也是缺点。优点是服务器没有状态差异，可以很容易地组成集群，而缺点就是无法支持需要记录状态的事务操作。

好在 HTTP 协议是可扩展的，后来发明的 Cookie 技术、Session和Token技术，给 HTTP 增加了“记忆能力”。

## 1 什么是 Cookie？

不知道你有没有看过克里斯托弗·诺兰导演的一部经典电影《记忆碎片》（Memento），里面的主角患有短期失忆症，记不住最近发生的事情。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fff84921d1e7bf37033e854d3088f405.png)

比如，电影里有个场景，某人刚跟主角说完话，大闹了一通，过了几分钟再回来，主角却是一脸茫然，完全不记得这个人是谁，刚才又做了什么，只能任人摆布。

这种情况就很像 HTTP 里“无状态”的 Web 服务器，只不过服务器的“失忆症”比他还要严重，连一分钟的记忆也保存不了，请求处理完立刻就忘得一干二净。即使这个请求会让服务器发生 500 的严重错误，下次来也会依旧“热情招待”。

如果 Web 服务器只是用来管理静态文件还好说，对方是谁并不重要，把文件从磁盘读出来发走就可以了。但`随着 HTTP 应用领域的不断扩大，对“记忆能力”的需求也越来越强烈`。比如网上论坛、电商购物，都需要“看客下菜”，只有记住用户的身份才能执行发帖子、下订单等一系列会话事务。

那该怎么样让原本无“记忆能力”的服务器拥有“记忆能力”呢？

看看电影里的主角是怎么做的吧。他通过纹身、贴纸条、立拍得等手段，在外界留下了各种记录，一旦失忆，只要看到这些提示信息，就能够在头脑中快速重建起之前的记忆，从而把因失忆而耽误的事情继续做下去。

HTTP 的 Cookie 机制也是一样的道理，既然服务器记不住，那就在外部想办法记住。`相当于是服务器给每个客户端都贴上一张小纸条，上面写了一些只有服务器才能理解的数据，需要的时候客户端把这些信息发给服务器，服务器看到 Cookie，就能够认出对方是谁了`

Cookie总是保存在客户端中，按在客户端中的存储位置，可分为内存Cookie和硬盘Cookie。
- 内存Cookie由浏览器维护，保存在内存中，浏览器关闭后就消失了，其存在时间是短暂的。
- 硬盘Cookie保存在硬盘里，有一个过期时间，除非用户手工清理或到了过期时间，硬盘Cookie不会被删除，其存在时间是长期的。
- 所以，按存在时间，可分为非持久Cookie和持久Cookie。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a9599c864ee60acc7ab1175215dde8ee.png)

## 2 Cookie 的工作过程

那么，Cookie 这张小纸条是怎么传递的呢？

这要用到两个字段：响应头字段**Set-Cookie**和请求头字段**Cookie**。

当用户通过浏览器第一次访问服务器的时候，服务器肯定是不知道他的身份的。所以，就要创建一个独特的身份标识数据，格式是“**key=value**”，然后放进 Set-Cookie 字段里，随着响应报文一同发给浏览器。

浏览器收到响应报文，看到里面有 Set-Cookie，知道这是服务器给的身份标识，于是就保存起来，下次再请求的时候就自动把这个值放进 Cookie 字段里发给服务器。

因为第二次请求里面有了 Cookie 字段，服务器就知道这个用户不是新人，之前来过，就可以拿出 Cookie 里的值，识别出用户的身份，然后提供个性化的服务。

不过因为服务器的“记忆能力”实在是太差，一张小纸条经常不够用。所以，服务器有时会在响应头里添加多个 Set-Cookie，存储多个“key=value”。但浏览器这边发送时不需要用多个 Cookie 字段，只要在一行里用“;”隔开就行。

我画了一张图来描述这个过程，你看过就能理解了。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d3dd9f38549cb4f3e0caeed138e6ba05.png)

从这张图中我们也能够看到，`Cookie 是由浏览器负责存储的，而不是操作系统`。所以，它是“`浏览器绑定`”的，只能在本浏览器内生效。

如果你换个浏览器或者换台电脑，新的浏览器里没有服务器对应的 Cookie，就好像是脱掉了贴着纸条的衣服，“健忘”的服务器也就认不出来了，只能再走一遍 Set-Cookie 流程。

在实验环境里，你可以用 Chrome 访问 URI“/19-1”，实地看一下 Cookie 工作过程。

首次访问时服务器会设置两个 Cookie。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/25e41024c3e42bfdf15f33a3534684fd.png)

然后刷新这个页面，浏览器就会在请求头里自动送出 Cookie，服务器就能认出你了。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/7bd36677202199f1809a332870f341c5.png)

如果换成 Firefox 等其他浏览器，因为 Cookie 是存在 Chrome 里的，所以服务器就又“蒙圈”了，不知道你是谁，就会给 Firefox 再贴上小纸条。

## 3 Cookie 的属性

说到这里，你应该知道了，`Cookie 就是服务器委托浏览器存储在客户端里的一些数据，而这些数据通常都会记录用户的关键识别信息`。所以，就需要在“key=value”外再用一些手段来保护，防止外泄或窃取，这些手段就是 Cookie 的属性。

下面这个截图是实验环境“/19-2”的响应头，我来对着这个实际案例讲一下都有哪些常见的 Cookie 属性。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/03e65af776969c8f0945994add2fab8d.png)

### 3.1 Cookie 的生存周期

首先，我们应该**设置 Cookie 的生存周期**，也就是它的有效期，让它只能在一段时间内可用，就像是食品的“保鲜期”，一旦超过这个期限浏览器就认为是 Cookie 失效，在存储里删除，也不会发送给服务器。

`Cookie 的有效期可以使用 Expires 和 Max-Age 两个属性来设置`。

“**Expires**”俗称“过期时间”，用的是绝对时间点，可以理解为“截止日期”（deadline）。“**Max-Age**”用的是相对时间，单位是秒，浏览器用收到报文的时间点再加上 Max-Age，就可以得到失效的绝对时间。

Expires 和 Max-Age 可以同时出现，两者的失效时间可以一致，也可以不一致，但`浏览器会优先采用 Max-Age 计算失效期`。

比如在这个例子里，Expires 标记的过期时间是“GMT 2019 年 6 月 7 号 8 点 19 分”，而 Max-Age 则只有 10 秒，如果现在是 6 月 6 号零点，那么 Cookie 的实际有效期就是“6 月 6 号零点过 10 秒”。

### 3.2 Cookie 的作用域

其次，我们需要**设置 Cookie 的作用域**，让浏览器仅发送给特定的服务器和 URI，避免被其他网站盗用。

作用域的设置比较简单，“**Domain**”和“**Path**”指定了 Cookie 所属的域名和路径，浏览器在发送 Cookie 前会从 URI 中提取出 host 和 path 部分，对比 Cookie 的属性。如果不满足条件，就不会在请求头里发送 Cookie。

使用这两个属性可以为不同的域名和路径分别设置各自的 Cookie，比如“/19-1”用一个 Cookie，“/19-2”再用另外一个 Cookie，两者互不干扰。不过现实中为了省事，通常 Path 就用一个“/”或者直接省略，表示域名下的任意路径都允许使用 Cookie，让服务器自己去挑。

### 3.3 Cookie 的安全性

最后要考虑的就是**Cookie 的安全性**了，尽量不要让服务器以外的人看到。

写过前端的同学一定知道，在 JS 脚本里可以用 document.cookie 来读写 Cookie 数据，这就带来了安全隐患，有可能会导致“跨站脚本”（XSS）攻击窃取数据。

属性“**HttpOnly**”会告诉浏览器，此 Cookie 只能通过浏览器 HTTP 协议传输，禁止其他方式访问，浏览器的 JS 引擎就会禁用 document.cookie 等一切相关的 API，脚本攻击也就无从谈起了。

另一个属性“**SameSite**”可以防范“跨站请求伪造”（XSRF）攻击，设置成“SameSite=Strict”可以严格限定 Cookie 不能随着跳转链接跨站发送，而“SameSite=Lax”则略宽松一点，允许 GET/HEAD 等安全方法，但禁止 POST 跨站发送。

还有一个属性叫“**Secure**”，表示这个 Cookie 仅能用 HTTPS 协议加密传输，明文的 HTTP 协议会禁止发送。但 Cookie 本身不是加密的，浏览器里还是以明文的形式存在。

Chrome 开发者工具是查看 Cookie 的有力工具，在“Network-Cookies”里可以看到单个页面 Cookie 的各种属性，另一个“Application”面板里则能够方便地看到全站的所有 Cookie。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9bba174f731537d168f64fdcf24620d2.png)

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a97cff8baa6c0aed74feac898cfd06f4.png)

## 4 Cookie 的应用

现在回到我们最开始的话题，有了 Cookie，服务器就有了“记忆能力”，能够保存“状态”，那么应该如何使用 Cookie 呢？

Cookie 最基本的一个用途就是**身份识别**，保存用户的登录信息，实现会话事务。

比如，你用账号和密码登录某电商，登录成功后网站服务器就会发给浏览器一个 Cookie，内容大概是“name=yourid”，这样就成功地把身份标签贴在了你身上。

之后你在网站里随便访问哪件商品的页面，浏览器都会自动把身份 Cookie 发给服务器，所以服务器总会知道你的身份，一方面免去了重复登录的麻烦，另一方面也能够自动记录你的浏览记录和购物下单（在后台数据库或者也用 Cookie），实现了“状态保持”。

Cookie 的另一个常见用途是**广告跟踪**。

你上网的时候肯定看过很多的广告图片，这些图片背后都是广告商网站（例如 Google），它会“偷偷地”给你贴上 Cookie 小纸条，这样你上其他的网站，别的广告就能用 Cookie 读出你的身份，然后做行为分析，再推给你广告。

这种 Cookie 不是由访问的主站存储的，所以又叫“第三方 Cookie”（third-party cookie）。如果广告商势力很大，广告到处都是，那么就比较“恐怖”了，无论你走到哪里它都会通过 Cookie 认出你来，实现广告“精准打击”。

为了防止滥用 Cookie 搜集用户隐私，互联网组织相继提出了 DNT（Do Not Track）和 P3P（Platform for Privacy Preferences Project），但实际作用不大。

## 5 Session

### 5.1 Session机制的概念

如果说Cookie是客户端行为，那么Session就是服务端行为。
![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/fea9f29846d6cd61fac6b05f71e61b0f.png)

Cookie机制在最初和服务端完成交互后，保持状态所需的信息都将存储在客户端，后续直接读取发送给服务端进行交互。

Session代表服务器与浏览器的一次会话过程，并且完全有服务端掌控，实现分配ID、会话信息存储、会话检索等功能。

Session机制将用户的所有活动信息、上下文信息、登录信息等都存储在服务端，只是生成一个唯一标识ID发送给客户端，后续的交互将没有重复的用户信息传输，取而代之的是唯一标识ID，暂且称之为Session-ID吧。

### 5.2 简单的交互流程

- 当客户端第一次请求session对象时候，服务器会为客户端创建一个session，并将通过特殊算法算出一个session的ID，用来标识该session对象。

- 当浏览器下次请求别的资源的时候，浏览器会将sessionID放置到请求头中，服务器接收到请求后解析得到sessionID，服务器找到该id的session来确定请求方的身份和一些上下文信息。

### 5.3 Session的实现方式

首先明确一点，Session和Cookie没有直接的关系，可以认为Cookie只是实现Session机制的一种方法途径而已，没有Cookie还可以用别的方法。

> Session和Cookie的关系就像加班和加班费的关系，看似关系很密切，实际上没啥关系。

session的实现主要两种方式：cookie与url重写，而cookie是首选方式，因为各种现代浏览器都默认开通cookie功能，但是每种浏览器也都有允许cookie失效的设置，因此对于Session机制来说还需要一个备胎。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8db666e98c32ef0fb9175f6968351577.png)

将会话标识号以参数形式附加在超链接的URL地址后面的技术称为URL重写。


```
原始的URL： http://taobao.com/getitem?name=baymax&action=buy 重写后的URL: http://taobao.com/getitem?sessionid=1wui87htentg&?name=baymax&action=buy`
```

### 5.4 存在的问题

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/b1051bb7dfa7fe39efa18457f0750244.png)

由于Session信息是存储在服务端的，因此如果用户量很大的场景，Session信息占用的空间就不容忽视。

对于大型网站必然是集群化&分布式的服务器配置，如果Session信息是存储在本地的，那么由于负载均衡的作用，原来请求机器A并且存储了Session信息，下一次请求可能到了机器B，此时机器B上并没有Session信息。

这种情况下要么在B机器重复创建造成浪费，要么引入高可用的Session集群方案，引入Session代理实现信息共享，要么实现定制化哈希到集群A，这样做其实就有些复杂了。

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/02d7b5191f7fc1afb499a34028b55e08.png)

<hr>

`Session-Cookie 在单节点的单体服务环境中是最合适的方案`，但当需要水平扩展服务能力，要部署集群时就开始面临麻烦了，由于 Session 存储在服务器的内存中，当`服务器水平拓展成多节点`时，设计者必须在以下三种方案中选择其一：

- `牺牲集群的一致性（Consistency）`，让均衡器采用亲和式的负载均衡算法，譬如根据用户 IP 或者 Session 来分配节点，==每一个特定用户发出的所有请求都一直被分配到其中某一个节点来提供服务==，每个节点都不重复地保存着一部分用户的状态，如果这个节点崩溃了，里面的用户状态便完全丢失。
- `牺牲集群的可用性（Availability）`，让==各个节点之间采用复制式的 Session，每一个节点中的 Session 变动都会发送到组播地址的其他服务器上==，这样某个节点崩溃了，不会中断都某个用户的服务，但 Session 之间组播复制的同步代价高昂，节点越多时，同步成本越高。
- `牺牲集群的分区容忍性（Partition Tolerance）`，让普通的服务节点中不再保留状态，将上下文集中放在一个所有服务节点都能访问到的数据节点中进行存储。此时的矛盾是数据节点就成为了单点，一旦数据节点损坏或出现网络分区，整个集群都不再能提供服务。

通过前面章节的内容，我们已经知道`只要在分布式系统中共享信息，CAP 就不可兼得，所以分布式环境中的状态管理一定会受到 CAP 的局限，无论怎样都不可能完美`。但如果只是解决分布式下的认证授权问题，并顺带解决少量状态的问题，就不一定只能依靠共享信息去实现。这句话的言外之意是提醒读者，接下来的 JWT 令牌与 Cookie-Session 并不是完全对等的解决方案，它只用来处理认证授权问题，充其量能携带少量非敏感的信息，只是 Cookie-Session 在认证授权问题上的替代品，而不能说 JWT 要比 Cookie-Session 更加先进，更不可能全面取代 Cookie-Session 机制。
## 6 Token

Token是令牌的意思，由服务端生成并发放给客户端，具有时效性的一种验证身份的手段。

Token避免了Session机制带来的海量信息存储问题，也避免了Cookie机制的一些安全性问题，属于典型的时间换空间的思路。在现代移动互联网场景、跨域访问等场景有广泛的用途。

### 6.1 简单的交互流程

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/a521753628ac0a3c410ac241eff94740.png)

- 客户端将用户的账号和密码提交给服务器
- 服务器对其进行校验，通过则生成一个token值，将其保存在数据库，同时也返回给客户端，作为后续的请求交互身份令牌
- 客户端拿到服务端返回的token值后，可将其保存在本地，以后每次请求服务器时都携带该token，提交给服务器进行身份校验
- 服务器接收到请求后，解析出其中的Token，再根据相同的加密算法和参数生成Token与客户端的Token进行对比，一致则通过，否则拒绝服务
- Token验证通过，服务端就可以根据该Token中的uid获取对应的用户信息，进行业务请求的响应

### 6.2 Token的设计思想

以JSON Web Token（JWT）为例，Token主要由3部分组成：

- Header头部信息  
    记录了使用的加密算法信息
    
- Payload 净荷信息  
    记录了用户信息和过期时间等
    
- Signature 签名信息  
    根据header中的加密算法和payload中的用户信息以及密钥key来生成，是服务端验证服务端的重要依据
    

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/612e346bd9ceb8247461659cebd15e05.png)

`header和payload的信息不做加密，只做一般的base64编码`，服务端收到token后剥离出header和payload获取算法、用户、过期时间等信息，然后根据自己的加密密钥来生成signature，并与客户端的sign进行一致性验证。

这样就实现了用CPU加解密的时间换取存储空间，干净利落，同时服务端密钥的重要性就显而易见，一旦泄露整个机制就崩塌了，这个时候就需要考虑HTTPS了。

### 6.3 Token方案的特点

- Token可以跨站共享，实现单点登录
- Token机制无需太多存储空间，Token包含了用户的信息，只需在客户端存储状态信息即可，对于服务端的扩展性很好
- Token机制的安全性依赖于服务端加密算法和密钥的安全性
- Token机制也不是万金油

## 7 JWT

JSON Web Token（缩写 JWT）是目前最流行的跨域认证解决方案。

`当服务器存在多个，客户端只有一个时，把状态信息存储在客户端，每次随着请求发回服务器去`。这样做的缺点是无法携带大量信息，而且有泄漏和篡改的安全风险。信息量受限的问题并没有太好的解决办法，但是要确保信息不被中间人篡改则还是可以实现的，JWT 便是这个问题的标准答案。
### 7.1 跨域认证的问题

互联网服务离不开用户认证。一般流程是下面这样。
> 1、用户向服务器发送用户名和密码。
> 
> 2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。
> 
> 3、服务器向用户返回一个 session_id，写入用户的 Cookie。
> 
> 4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。
> 
> 5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。

这种模式的问题在于，扩展性（scaling）不好。单机当然没有问题，如果是服务器集群，或者是跨域的服务导向架构，就要求 session 数据共享，每台服务器都能够读取 session。

举例来说，A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录，请问怎么实现？

一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。

另一种方案是服务器索性不保存 session 数据了，所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表。

### 7.2 JWT 的原理

JWT 的原理是，服务器认证以后，生成一个 JSON 对象，发回给用户，就像下面这样。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/90b95d8edb17e100a76a42a9e5cfedfd.png)
以后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名（详见后文）。

服务器就不保存任何 session 数据了，也就是说，服务器变成无状态了，从而比较容易实现扩展。

### 7.3 JWT 的数据结构

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e783acbc4b3c72dbb979d03c38a1d551.png)

右边的 JSON 结构是 JWT 令牌中携带的信息，左边的字符串呈现了 JWT 令牌的本体。它最常见的使用方式是附在名为 Authorization 的 Header 发送给服务端，前缀在[RFC 6750](https://tools.ietf.org/html/rfc6750)中被规定为 Bearer。

如下代码展示了一次采用 JWT 令牌的 HTTP 实际请求：
```http
GET /restful/products/1 HTTP/1.1
Host: icyfenix.cn
Connection: keep-alive
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJpY3lmZW5peCIsInNjb3BlIjpbIkFMTCJdLCJleHAiOjE1ODQ5NDg5NDcsImF1dGhvcml0aWVzIjpbIlJPTEVfVVNFUiIsIlJPTEVfQURNSU4iXSwianRpIjoiOWQ3NzU4NmEtM2Y0Zi00Y2JiLTk5MjQtZmUyZjc3ZGZhMzNkIiwiY2xpZW50X2lkIjoiYm9va3N0b3JlX2Zyb250ZW5kIiwidXNlcm5hbWUiOiJpY3lmZW5peCJ9.539WMzbjv63wBtx4ytYYw_Fo1ECG_9vsgAn8bheflL8
```

右边的状态信息是对令牌使用 Base64URL 转码后得到的明文，请特别注意是明文，JWT 只解决防篡改的问题，并不解决防泄漏的问题，因此令牌默认是不加密的。尽管你自己要加密也并不难做到，接收时自行解密即可，但这样做其实没有太大意义，

从明文中可以看到 JWT 令牌是以 JSON 结构（毕竟名字就叫 JSON Web Token）存储的，结构总体上可划分为三个部分，中间用点（`.`）分隔成三个部分。注意，JWT 内部是没有换行的，这里只是为了便于展示，将它写成了几行。

JWT 的三个部分依次如下。
- Header（头部）
- Payload（负载）
- Signature（签名）

写成一行，就是下面的样子。
```javascript
Header.Payload.Signature
```

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/5b42ceb9fbae329ef087c659988fee28.png)

#### 7.3.1 Header

`Header 部分是一个 JSON 对象`，描述 JWT 的元数据，通常是下面的样子。

```javascript 
{   
  "alg": "HS256",
  "typ": "JWT"
}
```

上面代码中
- `alg`属性表示签名的算法（algorithm），默认是 HMAC SHA256算法（写成 HS256）,其他各种系统支持的签名算法可以参考https://jwt.io/网站所列。![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/635e42a03ebb0e7f4c266140ec53e751.png)
- `typ`属性表示这个令牌（token）的类型（type），JWT 令牌统一写为`JWT`。

最后，将上面的 JSON 对象使用 Base64URL 算法（详见后文）转成字符串。

#### 7.3.2 Playload

`Payload 部分也是一个 JSON 对象`，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。
- iss (issuer)：签发人
- exp (expiration time)：过期时间
- sub (subject)：主题
- aud (audience)：受众
- nbf (Not Before)：生效时间
- iat (Issued At)：签发时间
- jti (JWT ID)：编号

除了官方字段，你还可以在这个部分定义私有字段，下面就是一个例子。
```javascript
{
  "username": "icyfenix",
  "authorities": [
    "ROLE_USER",
    "ROLE_ADMIN"
  ],
  "scope": [
    "ALL"
  ],
  "exp": 1584948947,
  "jti": "9d77586a-3f4f-4cbb-9924-fe2f77dfa33d",
  "client_id": "bookstore_frontend"
}
```

注意，`JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分`。

这个 JSON 对象也要使用 Base64URL 算法转成字符串。

#### 7.3.3 Signature

Signature 部分是对前两部分的签名，防止数据篡改。

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。

```javascript
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

签名的意义在于确保负载中的信息是可信的、没有被篡改的，也没有在传输过程中丢失任何信息。因为被签名的内容哪怕发生了一个字节的变动，也会导致整个签名发生显著变化。此外，由于签名这件事情只能由认证授权服务器完成（只有它知道 Secret），任何人都无法在篡改后重新计算出合法的签名值，所以服务端才能够完全信任客户端传上来的 JWT 中的负载信息。

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，就可以返回给用户。

#### 7.3.4 Base64URL

前面提到，Header 和 Payload 串型化的算法是 Base64URL。这个算法跟 Base64 算法基本类似，但有一些小的不同。

JWT 作为一个令牌（token），有些场合可能会放到 URL（`比如 api.example.com/?token=xxx`）。Base64 有三个字符`+`、`/`和`=`，在 URL 里面有特殊含义，所以要被替换掉：`=`被省略、`+`替换成`-`，`/`替换成`_` 。这就是 Base64URL 算法。

### 7.4 JWT的使用方式

客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage。

此后，客户端每次与服务器通信，都要带上这个 JWT。你可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息`Authorization`字段里面。

```javascript
Authorization: Bearer <token>
```

另一种做法是，跨域的时候，JWT 就放在 POST 请求的数据体里面。

### 7.5 JWT的几个特点

（1）JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。

（2）JWT 不加密的情况下，不能将秘密数据写入 JWT。

（3）JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。

（4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

（5）JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。

（6）为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。

## 8 小结

今天我们学习了 HTTP 里的 Cookie 知识。虽然现在已经出现了多种 Local Web Storage 技术，能够比 Cookie 存储更多的数据，但 Cookie 仍然是最通用、兼容性最强的客户端数据存储手段。

Cookie侧重于信息的存储，主要是客户端行为，Session和Token侧重于身份验证，主要是服务端行为。

简单小结一下今天的内容：

1. Cookie 是服务器委托浏览器存储的一些数据，让服务器有了“记忆能力”；
2. 响应报文使用 Set-Cookie 字段发送“key=value”形式的 Cookie 值；
3. 请求报文里用 Cookie 字段发送多个 Cookie 值；
4. 为了保护 Cookie，还要给它设置有效期、作用域等属性，常用的有 Max-Age、Expires、Domain、HttpOnly 等；
5. Cookie 最基本的用途是身份识别，实现有状态的会话事务。

`注意`：因为 Cookie 并不属于 HTTP 标准（RFC6265，而不是 RFC2616/7230），所以语法上与其他字段不太一致，使用的分隔符是“;”，与 Accept 等字段的“,”不同，小心不要弄错了。

## 9 课下作业

1. 如果 Cookie 的 Max-Age 属性设置为 0，会有什么效果呢？
2. Cookie 的好处已经很清楚了，你觉得它有什么缺点呢？


![unpreview|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/894bb7ca55f01e3cc70a4812011bec36.png)