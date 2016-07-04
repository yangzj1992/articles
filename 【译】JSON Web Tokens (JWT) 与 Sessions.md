title: 【译】JSON Web Tokens (JWT) 与 Sessions
date: 2016-07-04 23:21:10
categories: 身份认证
tags: [JWT,RESTful,session,web安全,token,JSON]
---
>** 原文链接 : [JSON Web Tokens (JWT) vs Sessions](https://float-middle.com/json-web-tokens-jwt-vs-sessions/)**
** 原文作者 : [Jacek Ciolek](https://float-middle.com/author/jacek-ciolek/)**
** 译文出自 : [众成翻译](http://zcfy.cc/)**
** 译者 : [yangzj1992](http://qcyoung.com)**
** 校对者: [lisa](https://www.zhihu.com/people/ha-ha-qiu-52)**
** 首发于: [众成翻译](http://zcfy.cc/article/json-web-tokens-jwt-vs-sessions-685.html)**

## 什么是 JWT?

> 本质上它是一段签名的 JSON 格式的数据。由于它是带有签名的，因此接收者便可以验证它的真实性。同时由于它是 JSON 格式的因此它的体积也很小。如果你想了解有关它的正式定义，可以在 [RFC 7519](https://tools.ietf.org/html/rfc7519) 中找到。

_这篇文章发布于[黑客新闻](https://news.ycombinator.com/item?id=11929267)上。在这里也可以看一下关于这篇文章的[案例分析](https://float-middle.com/i-got-featured-on-hacker-news-case-study/)，它主要包含了文章内容的公开分析、SEO 影响、性能影响以及更多其他的内容。_

数据签名已经不是什么新事物了 - 令人值得兴奋的是如何在不依靠 sessions 的情况下使用 JWT 创建真正的 RESTful 服务，目前这个想法已经被事实证明有一段时间了。下面是介绍它在现实中具体实现的工作原理 - 首先在这里我来做一个类比：

想象一下你刚从国外度完假回来，你在边境上说 - 你可以让我通过，我是这里的公民。这样的回答很好也没有问题，但是你要如何去支持你的说法呢？最有可能的方案是你携带了护照来证明你的身份。这里我们假设边境工作人员也都被要求去核实护照是真正由你的国家的护照办签发的。那么护照就会被核实，这样他们也才会放你回国。

现在，让我们从 JWT 的角度看一下这个故事，它们各自又都扮演着什么样的角色：

*   **护照办** - 发布 JWT 的身份验证服务。
  
*   **护照** - 你通过"护照办"获得的 JWT 签名。你的身份对于任何人都是可读的，但是只有它是真实的时候相关方才会对其核实。

*   **公民资格** - 在 JWT 中包含的你的声明(你的护照)。

*   **边境** - 你的应用程序的安全层，在被允许访问受保护的资源之前由它来核实你的 JWT 令牌身份，在这种情况下指的是 - 国家。 

*   **国家** - 你想要获取的资源(例如 API)。

## 看啊！没有 session!

简单来说，JWT 非常的酷，因为你不用再为了鉴别用户而在你的服务器上去保留你的 session 数据。这个工作流将会变得像下面这样：

*   用户调用身份验证服务，通常是发送了用户名及密码。

*   身份验证服务响应并返回了签名的 JWT，上面包含了用户是谁的内容。

*   用户向安全服务发送请求收到安全服务返回的令牌。

*   安全层检验令牌上的签名并且在签名为真实的时候授权予以通过。

让我们考虑一下这样做的结果。

### 没有会话存储

没有 sessions 意味着你没有会话存储。但除非您的应用程序需要横向扩展，否则这也不太重要，如果你的应用程序是运行在多个服务器上的，那么共享 session 数据将会成为一个负担。你需要一个专门的服务器来只存储会话数据或是共享磁盘空间或是在负载均衡上粘滞会话。当你不使用 sessions 时上面的这些也就自然不再需要了。

### 没有对 sessions 的垃圾收集

通常来讲 sessions 需要留意过期和垃圾收集的情况。JWT 可以在用户数据中包含自己的过期日期。因此安全层在检验 JWT 的授权时可以同时核对它的过期时间来拒绝访问。

### 真正的 RESTful 服务

只有在无 sessions 的情况下你可以创建真正的 RESTful 服务，因为它被认为是[无状态的](https://en.wikipedia.org/wiki/Representational_state_transfer#Stateless)。 JWT 很小所以它可以在每一个请求中被一起发出去，就像一个 session cookie一样。然而与 session cookie 不同的是，它并不指向服务器上的任何存储数据， JWT 本身包含了这些数据。

## 真实的 JWT 到底是什么样的?

在我们更深入讨论之前，有一件事需要了解。JWT 自身并不是一个东西。它是 [JSON 网络签名（JWS)](https://tools.ietf.org/html/rfc7515)或 [JSON 网络加密 (JWE)](https://tools.ietf.org/html/rfc7516)中的一种类型。它的定义如下：

> 一个 JWT 的声明内容会被编码为一个 JSON 对象，它被作为 JSON 网络签名结构的有效载荷或是作为 JSON 网络加密结构的明文信息。

前者给我们的只是一个签名并且它包含的数据(或是平时所称呼的 "claims" 的命名)是对任何人都可读的。后者则提供了加密的内容，所以只有拥有密钥的人可以解密它。JWS 在实现上更加容易并且基本用法上是不需要加密的 - 毕竟如果你在客户端上有密钥的话，你还不如把所有的东西不加密的好。因此 JWS 在大多数情况下都是适用的，也因此在之后我将主要关注 JWS。

### 那么 JWT/JWS 是由什么构成的？

*   **头部** - 关于签名算法的信息，以 JSON 格式的负载类型(JWT)等等。

*   **负载** - JSON 格式的实际的数据(或是声明)。

*   **签名** - 额... 就是签名。
   
我将在之后具体解释这些细节。现在让我们先来分析下基础要素。

上述所提到的每一部分(头部，负载和签名）是基于 base64url 编码的，然后他们用 '.' 作为分隔符粘连起来组成 JWT。 下面是这个实现方式可能看上去的样子：

```
var header = {  
        // The signing algorithm.
        "alg": "HS256",
        // The type (typ) property says it's "JWT",
        // because with JWS you can sign any type of data.
        "typ": "JWT"
    },
    // Base64 representation of the header object.
    headerB64 = btoa(JSON.stringify(header)),
    // The payload here is our JWT claims.
    payload = {
        "name": "John Doe",
        "admin": true
    },
    // Base64 representation of the payload object.
    payloadB64 = btoa(JSON.stringify(payload)),
    // The signature is calculated on the base64 representation
    // of the header and the payload.
    signature = signatureCreatingFunction(headerB64 + '.' + payloadB64),
    // Base64 representation of the signature.
    signatureB64 = btoa(signature),
    // Finally, the whole JWS - all base64 parts glued together with a '.'
    jwt = headerB64 + '.' + payloadB64 + '.' + signatureB64; 
```

由此得到的 JWS 结果看上去整洁而优雅，有点像这样：

```
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiSm9obiBEb2UiLCJhZG1pbiI6dHJ1ZX0.OLvs36KmqB9cmsUrMpUutfhV52_iSz4bQMYJjkI_TLQ` 
```

你也可以试着在 [jwt.io](https://jwt.io/#debugger) 这个网站上来创建令牌试试。

有一点相当重要，那就是签名是依据头部和负载计算出来的。因此头部和负载的授权也很容易同样被检验：

```
[headerB64, payloadB64, signatureB64] = jwt.split('.');

if (atob(signatureB64) === signatureCreatingFunction(headerB64 + '.' + payloadB64) {  
    // good
} else
    // no good
} 
```

### JWT 头部中可以存放什么？

事实上，JWT 头部被称为 JOSE 头部。JOSE 表示的是 JSON 对象的签名和加密。也正如你期望的那样，JWS 和 JWE 都是这样的一个头部，然而它们各自之间存在着一套稍微不同的注册参数。下面是在 JWS 中使用的头部注册参数列表。所有的参数除了第一个参数（alg）以外，其他参数都是可选的：

*   **alg** 算法 (必选项)

*   **typ** 类型 (如果是 JWT 那么就带有一个值 `JWT`，如果存在的话)

*   **kid** 密钥 ID

*   **cty** 内容类型

*   **jku** JWK 指定 URL

*   **jwk** JSON 网络值

*   **x5u** X.509 URL

*   **x5c** X.509 证书链

*   **x5t** X.509 证书 SHA-1 指纹

*   **x5t#S256** X.509 证书 SHA-256 指纹

*   **crit** 临界值

前两个参数是最常用的，所以典型的头部看起来有点类似下面这样：

```
{
    "alg": "HS256",
    "typ": "JWT"
} 
```

上面列出的第三个参数 `kid` 是基于安全原因使用的。`cty` 参数在另一方面应该只被用于处理嵌套的 JWT。剩下的参数你可以在[规范文档](https://tools.ietf.org/html/rfc7515#section-4.1)中阅读了解，我认为它们不适合在这篇文章中被提及。

#### `alg` (算法)

`alg` 参数的值可以是 [JSON 网络算法(JWA)](https://tools.ietf.org/html/rfc7518#section-3.1)中的任意指定值 - 这是我所知道的另一个规范。下面是 JWS 的注册列表：

*   **HS256** - HMAC 使用 SHA-256 算法

*   **HS384** - HMAC 使用 SHA-384 算法

*   **HS512** - HMAC 使用 SHA-512 算法

*   **RS256** - RSASSA-PKCS1-v1_5 使用 SHA-256 算法

*   **RS384** - RSASSA-PKCS1-v1_5 使用 SHA-384 算法

*   **RS512** - RSASSA-PKCS1-v1_5 使用 SHA-512 算法

*   **ES256** - ECDSA 使用 P-256 和 SHA-256 算法

*   **ES384** - ECDSA 使用 P-384 和 SHA-384 算法

*   **ES512** - ECDSA 使用 P-521 和 SHA-512 算法

*   **PS256** - RSASSA-PSS 使用 SHA-256 和基于 SHA-256 算法的 MGF1 

*   **PS384** - RSASSA-PSS 使用 SHA-384 和基于 SHA-384 算法的 MGF1 

*   **PS512** - RSASSA-PSS 使用 SHA-512 和基于 SHA-512 算法的 MGF1 

*   **none** - 没有数字签名或 MAC 执行

请注意最后一个值 `none`，从安全性的角度来看这是最有趣的。[这是已知的被用来进行降级防御攻击的方法](https://auth0.com/blog/2015/03/31/critical-vulnerabilities-in-json-web-token-libraries/)。它是如何工作的呢？想象一个客户端生成的带有一些声明的 JWT 。它在头部指定 `none` 值的签名算法并进行发送验证。如果攻击者比较单纯，那么它会使 `alg` 参数为真来确保被授权通过，然而实际上则是不会被允许的。

底线是，你的应用的安全层应该总是对头部的 `alg` 参数进行校验。那里就是 `kid` 参数用的上的地方。

#### `typ` (类型)

这一个参数非常简单。如果它是已知的，那么它就是 JWT，因为应用不会去索取其他的值，如果这个参数没有值就会被忽视掉。因此它是可选的。如果需要被指定值，它应该按大写字母拼写 - `JWT` 。

在某些情况下，当应用程序接受到没有 JWT 类型的请求却又包含了 JWT 时，去重新指定它是很重要的，因为这样应用程序才不会崩溃。

#### `kid` (密钥 id)

如果你的应用程序中的安全层只使用了一个算法来签名 JWTs，你不用太担心 `alg` 参数，因为你会总是使用相同的密钥和算法来校验令牌的完整性。但是，如果你的应用程序使用了一堆不同的算法和密钥，你就需要能够分辨出是由谁签署的令牌。

正如我们之前看到的，单独依靠 `alg` 参数可能会导致一些...不便。然而，如果你的应用维护了一个密钥/算法的列表，并且每一对都有一个名称(id)，你可以添加这个密钥 id 到头部，这样在之后验证 JWT 时你会有更多的信心去选择算法。这就是头部参数 `kid` - 你的应用中用来签名令牌所使用的密钥 id 。这个 id 是由你来任意指定的。最重要的是 - 这是你给的 id ，所以你可以验证。

#### `cty` (内容类型)

这里把[规范](https://tools.ietf.org/html/rfc7519#section-5.2)介绍的很清楚，所以这里我就只是引用了：

> 在通常情况下，在不使用嵌套签名或是加密操作时，是不推荐使用这个头部参数的。而在使用嵌套签名或加密时，这个头部参数必须存在；在这种情况下，它的值必须是 "JWT"，来表明这是一个在 JWT 中嵌套的 JWT。虽然媒体类型名字对大小写并不敏感，但这里为了与现有遗留实现兼容还是推荐始终用 "JWT" 大写字母来拼写。

### 在 JWT 声明中可以有什么？

"claims" 这个名称是否让你感到困惑？在最初它也确实让我很困惑。我相信你需要重复读几次来尝试适应它。简而言之，claims 是 JWT 的主要内容 - 是我们十分关心的签名的数据。它被叫做 "claims" 是因为通常它就是声明这个意思 - 客户端声明了用户名，用户角色或者其他什么的来让它可以获得对资源的访问。

还记得我在最开始提到的那个可爱的故事吗？你的公民资格就是你的声明而你的护照则就是 - JWT

你可以在声明中放置任何你想要的参数，这儿有一个[注册表](https://tools.ietf.org/html/rfc7519#section-4.1)应当被视为公认的参考实现方法。请注意这里的每一个参数都是可选的并且大多数是应用程序特定的，下面就是这个列表：

*   **exp** - 过期时间

*   **nbf** - 有效起始日期
   
*   **iat** - 发行时间

*   **sub** - 主题

*   **iss** - 发行者

*   **aud** - 受众

*   **jti** - JWT ID

值得注意的是，除了最后三个(issuer ，audience 和 JWT ID)参数通常是在更复杂的情况下(例如包含多个发行者时)才被使用。下面让我们来讨论一下它们吧。

#### `exp` (过期时间)

`exp` 是时间戳值表示着在什么时候令牌会失效。规范上要求"当前日期/时间"必须在指定的 `exp` 值之前，从而保证令牌可以得到处理。这里也表明了存在一些余地（几分钟）来应对时间差。

#### `nbf` (有效起始时间)

`nbf` 是时间戳值表示着在什么时候令牌开始生效。规范上要求"当前日期/时间"必须与指定的 `nbf` 值相等或在其之后，从而保证令牌可以得到处理。这里也表明了存在一些余地（几分钟）来应对时间差。

#### `iat` (发行时间)

`iat` 是时间戳值表示什么时候令牌被发行。

#### `sub` (主题)

`sub` 在规范上被要求"是JWT 中的声明中通常用于陈述主题的值"。这里主题必须是内容中唯一的发行者或全局上的唯一值。`sub` 声明可以用来鉴别用户，例如 [JIRA](https://developer.atlassian.com/static/connect/docs/latest/concepts/understanding-jwt.html#token-structure-claims) 文档上那样。

#### `iss` (发行者)

`iss` 是被用来确认令牌的发行者的字符串值。如果值中包含 `:` 那么它就是一个 URI。如果有很多的发行者而在一个安全层中应用程序需要去识别发行人时，它将会是有用的。例如 [Salesforce](https://help.salesforce.com/HTViewHelpDoc?id=remoteaccess_oauth_jwt_flow.htm) 要求了去使用 OAuth client_id 来作为 `iss` 的值。

#### `aud` (受众)

`aud` 是被用来确认令牌的可能接受者的字符串值或数组。如果值中包含 `:` 那么它就是一个 URI。
通常使用 URI 资源的声明是有效的。例如，在 [OAuth](https://tools.ietf.org/html/rfc7523#section-3) 中，接受者是授权服务器。应用程序处理令牌时，在针对不同的接受者的情况下，必须验证接受者是否是正确的或者拒绝令牌。

#### `jti` (JWT id)

令牌的唯一标识符。每个发布的令牌的 `jti` 必须是唯一的，即使有很多发行人也是一样。`jti` 声明可以用于一次性的不能重放的令牌。

## 如何在我的应用中使用 JWT ?

在最常见的场景中，客户端的浏览器将在认证服务中认证并接受返回的 JWT。然后客户端用某种方式(如内存，localStorage)存储这个令牌并与受保护的资源一起发送返回。通常令牌发送时是作为 cookie 或是 HTTP 请求中 `Authorization` 头部。

```
GET /api/secured-resource HTTP/1.1  
Host: example.com  
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiSm9obiBEb2UiLCJhZG1pbiI6dHJ1ZX0.OLvs36KmqB9cmsUrMpUutfhV52_iSz4bQMYJjkI_TLQ 
```

首选头部方法是出于安全的原因 - cookies 会很容易受 [CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery) (跨站请求伪造)的影响，除非 CSRF 令牌是使用过的。

其次，cookies 只能发送返回到被发出的相同的域下(或者最多二级域下)。如果身份验证服务驻留在不同的域下，那么 cookies 得需要更强烈的创造性才行。

### 如何通过 JWT 登出?

因为没有 session 数据存储在服务端了，所以不能再通过破坏 session 来注销了。因此登出成为了客户端的职责 - 一旦客户丢失了令牌不能再被授权，就可以被认为是登出了。

## 总结

我认为 JWTs 是一个在脱离 sessions 的情况下非常聪明的授权方式。它允许创建真正的服务端无状态的基于 RESTful 的服务，这也意味着不需要 session 存储。

与浏览器自动发送 session cookie 到任意匹配域/路径组合(老实说，在大多数情况下这里只有域的情况)的 URL 不一样的是，JWTs 可以选择性的只向需要身份授权的资源来发送。

对于客户端和服务端来说，它的实现非常简单，特别是已经有[专门的库](https://jwt.io/#libraries-io)来制造签名和验证令牌了。

感谢阅读！

如果你喜欢这篇文章的话，欢迎分享它。同样也十分欢迎你对它进行评论！