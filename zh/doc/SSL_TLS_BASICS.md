<a id="ssltls-basics"></a>
# SSL/TLS 基础

返回 **[目录](https://github.com/trimstray/nginx-admins-handbook#table-of-contents)** 或 **[下一步？](https://github.com/trimstray/nginx-admins-handbook#whats-next)** 部分。

- **[≡ SSL/TLS 基础](#ssltls-basics)**
  * [介绍](#introduction)
  * [TLS 版本](#tls-versions)
  * [TLS 握手](#tls-handshake)
    * [在 TCP/IP 协议栈中，TLS 位于哪一层？](#in-which-layer-is-tls-situated-within-the-tcpip-stack)
  * [RSA 和 ECC 密钥/证书](#rsa-and-ecc-keyscertificates)
  * [密码套件](#cipher-suites)
    * [认证加密 (AEAD) 密码套件](#authenticated-encryption-aead-cipher-suites)
    * [为什么密码套件很重要？](#why-cipher-suites-are-important)
    * [不安全、弱、安全和推荐分别意味着什么？](#what-does-insecure-weak-secure-and-recommended-mean)
    * [NGINX 与 TLS 1.3 密码套件](#nginx-and-tls-13-cipher-suites)
  * [Diffie-Hellman 密钥交换](#diffie-hellman-key-exchange)
    * [这些 DH 参数的目的到底是什么？](#what-exactly-is-the-purpose-of-these-dh-parameters)
  * [证书](#certificates)
    * [信任链](#chain-of-trust)
      * [中间 CA 的主要目的是什么？](#what-is-the-main-purpose-of-the-intermediate-ca)
    * [单域名](#single-domain)
    * [多域名](#multi-domain)
    * [通配符](#wildcard)
    * [通配符 SSL 不涵盖根域名？](#wildcard-ssl-doesnt-handle-root-domain)
    * [使用自签名证书的 HTTPS 与 HTTP](#https-with-self-signed-certificate-vs-http)
  * [TLS 服务器名称指示](#tls-server-name-indication)
  * [验证你的 SSL、TLS 和密码实现](#verify-your-ssl-tls--ciphers-implementation)
  * [有用的视频资源](#useful-video-resources)

<a id="introduction"></a>
#### 介绍

TLS 代表 _传输层安全性 (Transport Layer Security)_。它是一种在两个通信应用之间提供隐私和数据完整性的协议。它是当今使用最广泛的安全协议，取代了 _安全套接层 (SSL)_，并用于 Web 浏览器和其他需要通过网络安全交换数据的应用。

TLS 通过加密和端点身份验证，确保与远程端点的连接是预期的端点。迄今为止，TLS 的版本包括 TLS 1.3、1.2、1.1 和 1.0。

我不会详述 SSL/TLS 协议的每一个细节，因此请将此视为一个入门介绍。我将只讨论最重要的内容，因为我们有一些非常优秀的文档已经详细描述了该协议：

- [Bulletproof SSL and TLS](https://www.feistyduck.com/books/bulletproof-ssl-and-tls/)
- [Cryptology ePrint Archive](https://eprint.iacr.org/)
- [Practical Cryptography for Developers](https://cryptobook.nakov.com/)
- [SSL/TLS for dummies](https://www.wst.space/tag/https/)
- [Every byte of a TLS connection explained and reproduced - TLS 1.2](https://tls.ulfheim.net/)
- [Every byte of a TLS connection explained and reproduced - TLS 1.3](https://tls13.ulfheim.net/)
- [Transport Layer Security (TLS) - High Performance Browser Networking](https://hpbn.co/transport-layer-security-tls/)
- [The Sorry State Of SSL](https://hynek.me/talks/tls/)
- [How does SSL/TLS work?](https://security.stackexchange.com/questions/20803/how-does-ssl-tls-work)
- [What is TLS and how does it work?](https://www.comparitech.com/blog/information-security/tls-encryption/)
- [TLSeminar - Understanding and Securing TLS](https://tlseminar.github.io/)
- [A study of the TLS ecosystem](https://pdfs.semanticscholar.org/12fb/86fcbf0564ab11552c516539c91c6c8ff4d6.pdf) <sup>[pdf]</sup>
- [Inspecting TLS/SSL](https://www.java2depth.com/2019/04/transport-layer-security-tls-and-secure.html)
- [TLS in HTTP/2](https://daniel.haxx.se/blog/2015/03/06/tls-in-http2/)
- [Keyless SSL: The Nitty Gritty Technical Details](https://blog.cloudflare.com/keyless-ssl-the-nitty-gritty-technical-details/)
- [Nuts and Bolts of Transport Layer Security (TLS)](https://medium.facilelogin.com/nuts-and-bolts-of-transport-layer-security-tls-2c5af298c4be)
- [SSL and TLS Deployment Best Practices](https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices)
- [TLS Security 1: What Is SSL/TLS (set of articles by Acunetix)](https://www.acunetix.com/blog/articles/tls-security-what-is-tls-ssl-part-1/)
- [How to deploy modern TLS in 2019?](https://blog.probely.com/how-to-deploy-modern-tls-in-2018-1b9a9cafc454?gi=7e9d841a4d9d)
- [Cryptographic Right Answers](https://latacora.micro.blog/2018/04/03/cryptographic-right-answers.html)
- [The First Few Milliseconds of an HTTPS Connection](http://www.moserware.com/2009/06/first-few-milliseconds-of-https.html)
- [Best Practices for ACME Client Operations](https://docs.https.dev/acme-ops)
- [SSL Labs Grading 2018](https://discussions.qualys.com/docs/DOC-6321-ssl-labs-grading-2018)
- [SSL Testing Methods](https://securityboulevard.com/2020/02/ssl-testing-methods/)
- [SSL/TLS Threat Model](https://blog.ivanristic.com/SSL_Threat_Model.png)
- [Baseline requirements documents (SSL/TLS server certificates)](https://cabforum.org/baseline-requirements-documents/)

如果你对你的 SSL 配置有任何异议，请将你的网站提交到 [SSL Labs](https://www.ssllabs.com/)。它是验证 HTTP 服务器 SSL/TLS 配置的最佳工具之一（如果不是最佳的话）。我还推荐 [ImmuniWeb SSL Security Test](https://www.immuniweb.com/ssl/) 和 [CryptCheck](https://tls.imirhil.fr/)。它们每个都会告诉你是需要修复还是更新配置。

为了针对（不良）SSL 配置测试客户端，并了解真实世界 TLS 客户端的能力，我经常使用 [sslClientInfo](https://suche.org/sslClientInfo)、[How's My SSL](https://www.howsmyssl.com/) 和 [badssl.com](https://badssl.com/)。对于 SSL/TLS 质量监控，我推荐 [SSL Pulse](https://www.ssllabs.com/ssl-pulse/)。这是一个持续更新的仪表板，旨在展示 SSL 生态系统的状态，并基于 Alexa 全球最受欢迎网站列表，追踪超过 150,000 个支持 SSL/TLS 的网站随时间的变化。

另请参阅本章的[有用的视频资源](#useful-video-resources)部分。

最后，请看 [Julien Vehent](https://jve.linuxwall.info/blog/) 提供的这个[精彩答案](https://jve.linuxwall.info/blog/index.php?post/TLS_Survey)。它告诉你为什么正确的 TLS 配置如此重要：

  > _我们能做些什么？教育是关键：TLS 是一个复杂的主题，大多数管理员和网站所有者没有时间和知识去翻阅数十个邮件列表和博客文章来找到最佳的配置选择。_

  > _这正是 Server Side TLS 和 Better Crypto 等文档的主要动机。我们中的一些人正在努力改进这些文档。但我们还需要一支大军来传播信息，在会议、邮件列表和用户组中培训管理员，并推动网站所有者对其网站应用更安全的配置。_

  > _我们可以借助你的力量：走出去，去教授 TLS！_

<a id="tls-versions"></a>
#### TLS 版本

  > **:bookmark: [仅保留 TLS 1.3 和 TLS 1.2 - 加固 - P1](RULES.md#beginner-keep-only-tls-13-and-tls-12)**

| <b>协议</b> | <b>RFC</b> | <b>发布时间</b> | <b>状态</b> |
| :---:        | :---:        | :---:        | :---         |
| SSL 1.0 | | 未发布 | 未发布 |
| SSL 2.0 | | 1995 | 于 2011 年弃用 ([RFC 6176](https://tools.ietf.org/html/rfc6176)) <sup>[IETF]</sup> |
| SSL 3.0 | | 1996 | 于 2015 年弃用 ([RFC 7568](https://tools.ietf.org/html/rfc7568)) <sup>[IETF]</sup> |
| TLS 1.0 | [RFC 2246](https://tools.ietf.org/html/rfc2246) <sup>[IETF]</sup> | 1999 | 计划于 2020 年弃用 |
| TLS 1.1 | [RFC 4346](https://tools.ietf.org/html/rfc4346) <sup>[IETF]</sup> | 2006 | 计划于 2020 年弃用 |
| TLS 1.2 | [RFC 5246](https://tools.ietf.org/html/rfc5246) <sup>[IETF]</sup> | 2008 | 仍然安全 |
| TLS 1.3 | [RFC 8446](https://tools.ietf.org/html/rfc8446) <sup>[IETF]</sup> | 2018 | 仍然安全 |

某些 SSL/TLS 服务器实现不能正确地协商协议版本，如果客户端尝试协商服务器不支持的协议版本，则会以致命警报终止连接。这种情况可能发生在握手协议的三个步骤中的任何一个：

- 当服务器解析客户端 hello 消息时（该消息应包含客户端支持的最高协议版本）。正确实现的服务器应存储客户端版本值，即使服务器不支持该版本

- 当解析包含 RSA 加密的预主密钥的客户端密钥交换消息时（该消息的前两个八位组应包含客户端支持的最高协议版本）。此版本字段应与上述客户端版本值进行比较，而不是与协商后的版本进行比较

- 当验证客户端 finished 消息时（该消息应是所有握手消息的哈希值，包括包含客户端支持的最高协议版本的客户端 hello 消息）

现在，如果服务器由于某种原因未能按照 TLS 1.0、1.1 或 1.2 标准处理这些步骤，而是以其他方式（例如要求客户端支持的最高协议版本具有特定值，或不超过服务器支持的最高协议版本），则客户端有两个选择：通过向用户显示错误消息来终止连接，或者尝试在禁用最高协议版本的情况下重新连接。

<sup>此答案来自 [Why is TLS susceptible to protocol downgrade attacks?](https://crypto.stackexchange.com/questions/10493/why-is-tls-susceptible-to-protocol-downgrade-attacks/10495#10495)。</sup>

<a id="tls-handshake"></a>
#### TLS 握手

TLS 1.2 和 TLS 1.3 之间的差异如下面的插图所示（每个字节都得到了解释和再现——非常出色的工作！）：

- [The New Illustrated TLS Connection TLS 1.2](https://tls.ulfheim.net/)
- [The New Illustrated TLS Connection TLS 1.3](https://tls13.ulfheim.net/)

完整的 TLS 1.2 握手过程如下所示：

<p align="center">
  <a href="https://ldapwiki.com/wiki/How%20SSL-TLS%20Works">
    <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/tls/tls-handshake.png" alt="tls-handshake">
  </a>
</p>

<sup><i>此信息图来自 [ldapwiki - How SSL-TLS Works](https://ldapwiki.com/wiki/How%20SSL-TLS%20Works)。</i></sup>

TLS 1.3 则不同：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/tls/tls-1.3-handshake.png" alt="tls-1.3-handshake">
</p>

顺便一提，ldapwiki 上的 [How SSL-TLS Works](https://ldapwiki.com/wiki/How%20SSL-TLS%20Works) 是一个绝妙的解释。

<a id="in-which-layer-is-tls-situated-within-the-tcpip-stack"></a>
##### 在 TCP/IP 协议栈中，TLS 位于哪一层？

请看此图：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/tls/tcp_tls_http.png" alt="tcp_tls_http">
</p>

SSL 协议工作在 `TCP/IP` 协议和诸如 `HTTP` 之类的高层协议之间。其设计允许你在公钥密码学的基础上启动任何网站，从而在网络中实现安全的数据传输。

另请参阅 [What Happens in a TLS Handshake?](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)。

<a id="rsa-and-ecc-keyscertificates"></a>
#### RSA 和 ECC 密钥/证书

`RSA` 密钥对包括一个私钥和一个公钥。它是一种成熟的公钥密码学方法，基于使用两个大质数。`RSA` 私钥用于生成数字签名，`RSA` 公钥用于验证数字签名。`RSA` 需要更大的密钥长度来实现加密（目前最低为 2048 位）。当我们有两个物理或地理上不同的端点时，这种加密方式非常出色。

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/tls/rsa_ecc_lengths.png" alt="rsa_ecc_lengths">
</p>

`ECC` 是最新的加密方法。`ECC` 的主要卖点在于使用非常短的系统参数即可达到同等级别的安全水平（与 `RSA` 相比需要更短的密钥长度），因此速度更快。基于椭圆曲线的算法使用的密钥大小明显小于非椭圆曲线等价算法。`ECC` 密钥对也包括一个私钥和一个公钥。`ECC` 私钥用于生成数字签名，`ECC` 公钥用于验证数字签名。

根据大多数建议，密钥长度可以比 `RSA` 或经典离散对数密码学中的对应长度短约 12 倍；具体来说，`Curve25519` 使用约 256 位的密钥，而等效的 `RSA` 实例则需要 3072 位的密钥大小。

默认情况下，`ECC` 密钥对使用椭圆曲线数字签名算法（参见 [ECDSA: The digital signature algorithm of a better internet](https://blog.cloudflare.com/ecdsa-the-digital-signature-algorithm-of-a-better-internet/)）。该算法使用椭圆曲线密码学（一种基于椭圆曲线特性的加密系统）来提供数字签名算法 (`DSA`) 的变体，并应用于 `ECC` 以使其适用于安全加密。

有关更多信息，请参阅这个精彩的演示：[ECC vs RSA: Battle of the Crypto-Ninjas](https://www.slideshare.net/JamesMcGivern/ecc-vs-rsa-battle-of-the-cryptoninjas)。另见 [Diffie-Hellman, RSA, DSA, ECC and ECDSA – Asymmetric Key Algorithms](https://www.ssl2buy.com/wiki/diffie-hellman-rsa-dsa-ecc-and-ecdsa-asymmetric-key-algorithms)。

看看这个密钥长度对比表：

```
+===+=================+========================+=================+=============+==============+
|   |    对称          | RSA 和 Diffie-Hellman   | 椭圆曲线         |   密钥大小   |  保护年限    |
|   | 密钥大小 (位)    | 密钥大小 (位)           | 密钥大小 (位)    |    比率     |              |
+===+=================+========================+=================+=============+==============+
| 1 |       80        |          1024          |     160-223     |    ~1:6     |    2010      |
+---+-----------------+------------------------+-----------------+=============+==============+
| 2 |      112        |         [2048]         |     224-255     |    ~1:9     |    2030      |
+---+-----------------+------------------------+-----------------+=============+==============+
| 3 |      128        |          3072          |   [256]-383     |    ~1:12    |    2031+     |
+---+-----------------+------------------------+-----------------+=============+==============+
| 4 |      192        |          7680          |     384-511     |    ~1:20    |              |
+---+-----------------+------------------------+-----------------+=============+==============+
| 5 |      256        |         15360          |      521+       |    ~1:30    |              |
+---+-----------------+------------------------+-----------------+=============+==============+
```

最后，我推荐阅读 [Maarten Bodewes](https://crypto.stackexchange.com/users/1172/maarten-bodewes) 的这个[回答](https://crypto.stackexchange.com/questions/61248/aes-and-ecdh-key)：

  > _RSA——一种非对称算法——需要更大的密钥大小，因为其数值计算基于大数。对于 RSA，攻击者可以尝试分解模数以找到私钥组件。因此，攻击 RSA 比尝试 $2^X$ 个值来破解 $X$ 位密钥要容易得多。在上表中，你可以看到破解 3072 位密钥大约需要 $2^{128}$ 次测试。因此，128 位的 AES 密钥和 3072 位的 RSA 密钥都具有 128 位的强度。_

  > _椭圆曲线密码学允许比 RSA 更小的密钥大小来实现相同强度的非对称密钥对。通常，密钥对的有效密钥大小需要翻倍才能达到与对称密钥相同的强度。因此我们在同一行中看到了 ECC 的值 256。列出的曲线大小是由 Certicom 首次创建、后来由 NIST 标准化的命名曲线：P-160、P-224、P-256、P-384 和 P-521（这不是笔误，不是 512）。_

<a id="cipher-suites"></a>
#### 密码套件

  > **:bookmark: [仅使用强密码 - 加固 - P1](RULES.md#beginner-use-only-strong-ciphers)**<br>
  > **:bookmark: [使用更安全的 ECDH 曲线 - 加固 - P1](RULES.md#beginner-use-more-secure-ecdh-curve)**

为了确保数据传输的安全，TLS/SSL 使用一个或多个密码套件。密码套件是身份验证、加密和消息认证码 (MAC) 算法的组合。它们在 TLS/SSL 连接的安全设置协商期间以及数据传输期间使用。

  > TLS 1.1 使用的密码与 TLS 1.0 相同，因此 OpenSSL 不区分两者。当它支持 TLS 1.1 的某个密码套件时，它也支持 TLS 1.0 的该套件，反之亦然。TLS 1.2 和 TLS 1.3 拥有自己的一套密码套件。在 TLS 1.3 中，它们在 OpenSSL 中配置，默认启用，并且自动选择（无需在配置中设置）。

在 SSL 握手中，客户端首先告知服务器它支持哪些密码。密码套件通常按安全级别排列。然后，服务器将这些密码套件与其自身启用的密码套件进行比较。一旦找到匹配项，就通知客户端，并调用所选密码套件的算法。

  > 注意：客户端建议密码套件，但由服务器选择。密码套件的决定权在服务器手中。服务器协商并选择特定的密码套件用于通信。如果服务器准备不使用客户端通告的任何密码套件，那么它将不允许该会话。

在建立 TLS 连接期间和之后，会使用各种加密算法。TLS 1.2 密码套件本质上包含 4 个不同的部分：

- **密钥交换** - 使用什么非对称密码来交换密钥？<br>
  例如：`RSA`、`DH`、`ECDH`、`DHE`、`ECDHE`
- **身份验证/数字签名算法** - 使用什么密码来验证服务器的真实性？<br>
  例如：`RSA`、`DSA`、`ECDSA`
- **密码/批量加密算法** - 使用什么对称密码来加密数据？<br>
  例如：`AES`、`3DES`、`CHACHA20`、`Camellia`、`ARIA`
- **MAC** - 使用什么哈希函数来确保消息完整性？<br>
  例如：`MD5`、`SHA-256`、`POLY1305`

这四种类型的算法组合成所谓的密码套件/集，例如 `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256_P256` 使用临时椭圆曲线 Diffie-Hellman (`ECDHE`) 来交换密钥，提供前向保密性。由于参数是临时的，使用后即被丢弃，没有它们就无法从流量流中恢复交换的密钥。`RSA_WITH_AES_128_CBC_SHA256`——这意味着 RSA 密钥交换与 `AES-128-CBC`（对称密码）结合使用，`SHA256` 哈希用于消息认证。`P256` 是一种椭圆曲线类型（TLS 密码套件和椭圆曲线有时通过这样的单个字符串进行配置）。

  > 要使用 `ECDSA` 密码套件，你需要 `ECDSA` 证书和密钥。要使用 `RSA` 密码套件，你需要 `RSA` 证书和密钥。推荐使用 `ECDSA` 证书而非 `RSA` 证书。我认为，最低配置是 `ECDSA`（`P-256`）或 `RSA`（2048 位）。

请看以下对 `TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256` 的分解说明：

| <b>协议</b> | <b>密钥交换</b> | <b>身份验证</b> | <b>加密</b> | <b>哈希</b> |
| :---:        | :---:        | :---:        | :---:        | :---:        |
| `TLS` | `ECDHE` | `ECDSA` | `AES_128_GCM` | `SHA256` |

`TLS` 是协议（标准起始点）。`ECDHE` 表明在握手期间，密钥将通过临时椭圆曲线 Diffie-Hellman 进行交换——使用临时密钥的椭圆曲线版 Diffie-Hellman 密钥交换。`ECDSA` 是用于对密钥交换参数进行签名的身份验证算法，对于 RSA 则省略。`AES_128_GCM` 是批量加密算法：`AES` 运行伽罗瓦/计数器模式，使用 `128 位` 密钥大小（一种现代认证加密及相关数据 (`AEAD`) 操作模式，用于消息的机密性和完整性/真实性，在此示例中用于 128 位块的块密码）。`AES_256` 表示 256 位密钥。对于 `GCM`，只有 `AES`、`CAMELLIA` 和 `ARIA` 是可能的，其中 `AES` 显然是最流行和部署最广泛的选择（已在硬件中实现）。最后，`SHA-256` 是哈希算法——用作 TLS 协议中从主密钥派生密钥的基础，以及用于验证 finished 消息的哈希函数。

客户端和服务器在 TLS 连接开始时协商使用哪个密码套件（客户端发送它支持的密码套件列表，服务器选择一个并告知客户端）。`ECDH` 的椭圆曲线选择不是密码套件编码的一部分。曲线是单独协商的（同样，客户端提议，服务器决定）。

好的，现在看看这个[最后一个例子](https://security.stackexchange.com/a/137297)，其中解释了 `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` 和 `TLS_RSA_WITH_AES_128_GCM_SHA256` 之间的差异：

- 两者都使用 `RSA` 证书来验证服务器（以及可能的客户端）
- 两者都使用 `AES-128` 伽罗瓦/计数器模式进行加密
- 两者都使用 `HMAC-SHA256` 进行消息完整性

它们的区别在于密钥交换方法。`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` 使用临时椭圆曲线 Diffie-Hellman 来交换密钥，提供前向保密性。由于参数是临时的，使用后即被丢弃，没有它们就无法从流量流中恢复交换的密钥。而 `TLS_RSA_WITH_AES_128_GCM_SHA256` 使用服务器证书中的 `RSA` 密钥来交换密钥。这仍然是强加密（假设密钥足够大），但可以使用服务器的私钥从流量流中恢复交换的会话密钥，而私钥显然不能频繁丢弃。

  > 如果你想获取大量有关可用密码的有用信息，请参阅 [TLS Cipher Suite Search](https://ciphersuite.info/) 引擎。更多信息，另请参阅 SSL 和 TLS 协议的[密码套件定义](https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.gska100/csdcwh.htm)。

我推荐阅读 [Cipher Suites: Ciphers, Algorithms and Negotiating Security Settings](https://www.thesslstore.com/blog/cipher-suites-algorithms-security-settings/) 以及关于 [Role of the chosen ciphersuite in an SSL/TLS connection](https://security.stackexchange.com/questions/160429/role-of-the-chosen-ciphersuite-in-an-ssl-tls-connection/160445#160445) 的优秀回答（作者 [dave_thompson_085](https://security.stackexchange.com/users/39571/dave-thompson-085)）。

<a id="authenticated-encryption-aead-cipher-suites"></a>
##### 认证加密 (AEAD) 密码套件

AEAD 算法通常附带安全证明。它们提供了称为认证加密 (AE) 模式或有时称为带关联数据的认证加密 (AEAD) 的专用块密码操作模式。这些模式一次性处理加密和认证，通常使用单个密钥（在一个算法中结合加密和完整性验证）。

这些安全证明当然依赖于底层原语，但它无论如何都增加了对完整方案的信心。AEAD 密码——无论内部结构如何——应该能够避免 authenticate-then-encrypt 方式引起的问题（参见 [How to choose an Authenticated Encryption mode](https://blog.cryptographyengineering.com/2012/05/19/how-to-choose-authenticated-encryption/) 作为很好的解释）。

AE(AD) 模式被开发出来，旨在使认证问题对实现者变得“容易”。此外，其中一些模式速度极快，或者至少允许你利用并行化来加速。

AEAD 密码的优势：

- 只信任一种算法，而不是两种
- 只需执行一次遍历（AEAD 的理想情况，而非必然结果）
- 节省代码量，有时也节省计算量

任何带有 `GCM`、`CCM` 或 `POLY1305` 的都是 AEAD 密码套件，例如：

```
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
TLS_ECDHE_ECDSA_WITH_AES_128_CCM
TLS_ECDHE_ECDSA_WITH_AES_256_CCM
TLS_ECDHE_ECDSA_WITH_AES_128_CCM_8
TLS_ECDHE_ECDSA_WITH_AES_256_CCM_8
TLS_ECDHE_ECDSA_WITH_CAMELLIA_128_GCM_SHA256
TLS_ECDHE_ECDSA_WITH_CAMELLIA_256_GCM_SHA384
TLS_ECDHE_ECDSA_WITH_ARIA_128_GCM_SHA256
TLS_ECDHE_ECDSA_WITH_ARIA_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
TLS_ECDHE_RSA_WITH_CAMELLIA_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_CAMELLIA_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_ARIA_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_ARIA_256_GCM_SHA384
TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256
TLS_DHE_RSA_WITH_AES_128_CCM
TLS_DHE_RSA_WITH_AES_256_CCM
TLS_DHE_RSA_WITH_AES_128_CCM_8
TLS_DHE_RSA_WITH_AES_256_CCM_8
TLS_DHE_RSA_WITH_CAMELLIA_128_GCM_SHA256
TLS_DHE_RSA_WITH_CAMELLIA_256_GCM_SHA384
TLS_DHE_RSA_WITH_ARIA_128_GCM_SHA256
TLS_DHE_RSA_WITH_ARIA_256_GCM_SHA384
```

但以下 AEAD 密码套件是推荐的：

| <b>名称</b> | <b>别名</b> | <b>密钥大小</b> | <b>盐大小</b> | <b>Nonce 大小</b> | <b>标签大小</b> |
| :---:        | :---:        | :---:        | :---:        | :---:        | :---:        |
| `AEAD_CHACHA20_POLY1305` | `chacha20-ietf-poly1305` | 32 | 32 | 12 | 16 |
| `AEAD_AES_256_GCM` | `aes-256-gcm` | 32 | 32 | 12 | 16 |
| `AEAD_AES_192_GCM` | `aes-192-gcm` | 24 | 24 | 12 | 16 |
| `AEAD_AES_128_GCM` | `aes-128-gcm` | 16 | 16 | 12 | 16 |

这些是当前不会触发 [ROBOT](https://robotattack.org/) 警告的 AEAD 密码。

<a id="why-cipher-suites-are-important"></a>
##### 为什么密码套件很重要？

你的 HTTPS 流量（你的数据和用户数据的安全性）的安全级别取决于你的 Web 服务器使用哪些密码套件。拥有广泛的高安全性密码套件列表对于高安全性的 SSL/TLS 拦截非常重要。你的 HTTPS 流量的兼容性（谁将看到错误、警告或遇到其他问题）取决于你的 Web 服务器使用的密码套件。你的 HTTPS 流量的性能（用户看到页面的速度）也取决于你的 Web 服务器使用的密码套件。

<a id="what-does-insecure-weak-secure-and-recommended-mean"></a>
##### 不安全、弱、安全和推荐分别意味着什么？

- **不安全** - 这些密码非常古老，在任何情况下都不应使用。如今只需最少的努力就能破解它们的保护

- **弱** - 这些密码较旧，如果你在设置新服务器等情况下应禁用它们。仅在你有特殊用例需要支持较旧的操作系统、浏览器或应用程序时才启用它们

- **安全** - 安全密码被认为是当前最先进的，如果你想保护你的 Web 服务器，你当然应该从这组中选择。只有非常旧的操作系统、浏览器或应用程序无法处理它们

- **推荐** - 所有“推荐”密码在定义上都是“安全”密码。推荐意味着这些密码还支持 PFS（完美前向保密性），并且如果你想要最高级别的安全性，它们应该是你的首选。但是，你可能会遇到不支持 PFS 密码的旧客户端的一些兼容性问题

<sup><i>此解释来自 [TLS Cipher Suite Search](https://ciphersuite.info/page/faq/)。</i></sup>

<a id="nginx-and-tls-13-cipher-suites"></a>
##### NGINX 与 TLS 1.3 密码套件

我们目前无法在没有 NGINX 支持使用新 API 的情况下控制 TLS 1.3 密码套件（这就是为什么今天你不能指定 TLSv1.3 密码套件，应用程序仍需适配）。NGINX 无法影响这一点，因此目前所有可用的密码始终处于开启状态（即使你从 NGINX 禁用了潜在的弱密码）。另一方面，领先的加密专家已将 TLSv1.3 中的密码限制为一小部分完全安全的密码。

如果你想使用 `TLS_AES_128_CCM_SHA256` 和 `TLS_AES_128_CCM_8_SHA256` 密码（例如在通常各方面都受限的受限系统上），请参阅 [TLSv1.3 和 `CCM` 密码](HELPERS.md#tlsv13-and-ccm-ciphers)。但请记住：对于大多数需要认证加密的应用，`GCM` 应该被认为优于 `CCM`。

<a id="diffie-hellman-key-exchange"></a>
#### Diffie-Hellman 密钥交换

  > **:bookmark: [使用具有完美前向保密性的强密钥交换 - 加固 - P1](RULES.md#beginner-use-strong-key-exchange-with-perfect-forward-secrecy)**

Diffie-Hellman 密钥交换 (DHKE) 的目标是让两个用户获得一个共享的秘密密钥，而其他任何用户都不知道该密钥。该交换在公共网络上进行，即两个用户之间发送的所有消息都可以被任何其他用户截获和读取。

该协议利用模算术，特别是指数运算。该协议的安全性依赖于这样一个事实：当使用足够大的数值时，求解离散对数（指数的逆运算）实际上是不可能的。

`DHE`（根据 [RFC 5246 - 密码套件](https://tools.ietf.org/html/rfc5246#appendix-A.5) <sup>[IETF]</sup>）和 `EDH` 是相同的（在 OpenSSL 术语中为 `EDH`，其他地方为 `DHE`）。`EDH` 不是标准的表述方式，但它没有其他常见含义。`ECC` 可以代表“椭圆曲线证书”或“椭圆曲线密码学”。椭圆曲线证书通常称为 `ECDSA`。椭圆曲线密钥交换称为 `ECDH`。如果你在后者中再加一个 'E'（`ECDHE`），就得到了临时版本。

| <b>类型</b> | <b>椭圆曲线</b> | <b>临时</b> | <b>密钥轮换</b> | <b>PFS</b> | <b>DHPARAM 文件</b> |
| :---:        | :---:        | :---:        | :---:        | :---:        | :---:        |
| `DHE` | 否 | 是 | 是 | 是 | 是 |
| `ECDH` | 是 | 否 | 否 | 否 | 否 |
| `ECDHE` | 是 | 是 | 是 | 是 | 否 |

使用 `DHE` 意味着 `g` 和 `p` 参数也可能是随机生成的，但由于这是一个非常昂贵的过程（因为你需要找到一个安全素数），从计算角度来看，通常不会这样做。

然而，当你进行 DH（不带 "E"）密钥交换时，服务器拥有一个证书，其中嵌入了静态的公共 Diffie-Hellman 密钥，即 `gxmodp` 以及 `g` 和 `p`，这允许你省去额外的签名（例如使用 `RSA` 或 `ECDSA`）来验证 DH 参数的真实性。因此，虽然服务器的 DH 值是静态的，但客户端通常仍会出于安全和存储减少的原因选择一个随机值。显然，将参数固定在证书中意味着它们不能在运行时更改。

临时 Diffie-Hellman (`ECDHE/DHE`) 为每次交换生成一个新密钥（即时生成），从而实现完美前向保密性 (PFS)。然后，它使用服务器的 `RSA`、`DSA` 或 `ECDSA` 私钥对公钥进行签名，并将其发送给客户端。DH 密钥是临时的，意味着服务器永远不会将其存储在磁盘上；它在会话期间保存在 RAM 中，并在使用后丢弃。由于从未存储，因此之后无法被窃取，这就是 PFS 的由来。

前向保密性是临时版 Diffie-Hellman 的主要特性，这意味着如果服务器的私钥泄露，其过去的通信仍然是安全的。临时 Diffie-Hellman 本身不提供身份验证，因为密钥每次都不相同。因此，任何一方都不能确保该密钥来自预期的对方。

`ECDHE` 是 Diffie-Hellman 协议的一个变体，它使用椭圆曲线密码学来降低计算、存储和内存需求。`DHE` 提供的完美前向保密性是有代价的：更多的计算。`ECDHE` 变体使用椭圆曲线密码学来减少这种计算成本。

而固定 Diffie-Hellman (`ECDH` 和 `DH`) 每次都使用相同的 Diffie-Hellman 密钥。没有任何 DH 交换，你只能使用 `RSA` 加密模式。

这些参数不是秘密，可以重复使用；而且生成它们需要几秒钟。`openssl dhparam ...` 步骤提前生成 DH 参数（主要是一个大质数），然后存储起来供服务器使用。

<a id="what-exactly-is-the-purpose-of-these-dh-parameters"></a>
##### 这些 DH 参数的目的到底是什么？

我将引用一些[精彩答案](https://security.stackexchange.com/questions/94390/whats-the-purpose-of-dh-parameters)：

  > _这些参数定义了 OpenSSL 如何执行 Diffie-Hellman (DH) 密钥交换。正如你正确指出的，它们包含一个域素数 p 和一个生成元 g。允许自定义这些参数的目的是让每个人都能使用自己的参数。这可以用于防止受到 Logjam 攻击的影响（不过这并不真正适用于 4096 位的域素数）。_

  > _参数 `p` 和 `g` 定义了此密钥交换的安全性。更大的 `p` 将使找到共享密钥 `K` 变得更加困难，从而防御被动攻击者。_

  > _找到这样的素数在计算上非常密集，无法在每个连接上都承担得起，因此它们被预先计算。_

  > _发布它们没有风险。事实上，在每次涉及 Diffie-Hellman (DH) 密钥交换的交换中，它们都会被发送出去。甚至有一些这样的参数在 [RFC 5114 - 用于 IETF 标准的附加 Diffie-Hellman 组](https://tools.ietf.org/html/rfc5114) <sup>[IETF]</sup> 中进行了标准化。_

<a id="certificates"></a>
#### 证书

SSL 证书包含公钥及其所有者信息，通过加密信息并保护从你的网站传输和传输到你的网站的敏感数据（登录详细信息、注册信息、地址和付款信息），验证网站的身份并允许从 Web 服务器到浏览器的安全连接。

证书的真实性和完整性可以通过密码学方法进行检查。数字证书包含验证它所需的数据。

  > 如果没有 SSL 证书，你网站收集的任何数据都容易被第三方截获。

证书允许你使用单个 SSL 证书保护你的主域及其所有子域（如 `example.com` 和 `api.example.com`）。

另请参阅 [What is an SSL Certificate?](https://www.cloudflare.com/learning/ssl/what-is-an-ssl-certificate/) 和 [EV Certificates Make The Web Slow and Unreliable](https://www.aaronpeters.nl/blog/ev-certificates-make-the-web-slow-and-unreliable/)。

<a id="chain-of-trust"></a>
##### 信任链

  > **:bookmark: [正确设置证书链 - 其他 - P2](RULES.md#beginner-set-the-certificate-chain-correctly)**

证书链的验证是任何基于证书的身份验证过程中的关键部分。如果系统不遵循证书到根服务器的信任链，则证书作为信任度量标准将失去所有用处。

  > 在 [RFC 5280](https://tools.ietf.org/html/rfc5280) 中，证书链或信任链被定义为“认证路径”。

证书链由验证终端证书所标识主体所需的所有证书组成。在实践中，这包括终端证书、中间 CA 的证书以及链中所有各方都信任的根 CA 的证书。链中的每个中间 CA 都持有由其信任层次结构中上一级 CA 颁发的证书。

服务器的证书及其链不是为服务器准备的。服务器对自己的证书没有用处。证书始终是为其他人（这里指客户端）准备的。服务器使用的是其私钥（与其证书中的公钥相对应）。特别是，服务器不需要信任自己的证书或任何为其颁发证书的 CA。

看下面的示意图：

```
ROOT_CERT (isCA=yes)
|
|---INTERMEDIATE_CERT_1 (isCA=yes)
     |
     |---INTERMEDIATE_CERT_2 (isCA=yes)
         |
         |---LEAF_CERT valid for example.com (isCA=no)
```

  > 当 CA 签署证书时，它们不仅签署网站的公钥，而且还签署大量的元数据。例如，这些元数据包括证书何时过期等信息。在我们的案例中，这些信息以 X.509 定义的数据格式保存。[...] 它还包含某些“约束”，即对证书所能执行操作的限制。维基百科文章列出了概述，但对我们来说重要的是：它还说明了该证书是否可以签署其他“（子）证书”并因此“认证”它们。 - [TLS: Clarification on trust in the certificate trust chain](https://security.stackexchange.com/questions/210672/tls-clarification-on-trust-in-the-certificate-trust-chain/210700#210700)。

根 CA 和中间 CA/证书都设置了此项。因此，它们被允许签署其他证书。显然，叶子证书不得拥有签署其他证书的权限。

如果证书直接由受信任的根 CA 签署，则无需在证书链中添加任何额外/中间证书。根 CA 为自己颁发证书。

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/tls/chain_of_trust.png" alt="chain_of_trust">
</p>

<sup><i>此信息图来自 [维基百科](https://en.wikipedia.org/wiki/Chain_of_trust)。</i></sup>

列表中的最后一个证书是信任锚：一个你通过某些可信程序获得的、你信任的证书。信任锚是依赖方用作路径验证起点的 CA 证书（或更准确地说，CA 的公钥验证密钥）。

如果信任链断裂，则无法验证托管数据的服务器是正确的（预期的）服务器——无法确定服务器是否与其声称的一致（你将失去验证连接安全性或信任它的能力）。

  > 如果缺少中间证书，客户端将无法验证证书是否有效。

连接仍然是安全的，但主要问题是修复该证书链。你应该通过按顺序连接从证书到受信任根证书的所有证书（不包括根证书，按此顺序）来手动解决证书链不完整的问题，以防止此类问题。不过，流量仍将被加密。

信任链可能被破坏的方式有多种，包括但不限于：

- 链中的任何证书是自签名的，除非它是根证书
- 并非从原始证书一直追溯到根证书的每个中间证书都经过检查
- 由 CA 签署的中间证书没有预期的基础约束或其他重要扩展
- 根证书已被泄露或被授权给了错误的方

看看[这里](https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices#21-use-complete-certificate-chains)的精彩解释：

  > _在大多数部署中，仅服务器证书是不够的；需要两个或更多证书来构建完整的信任链。一种常见的配置问题发生在部署了有效证书但没有所有必要的中间证书的服务器上。为避免这种情况，只需按相同顺序使用 CA 提供给你的所有证书即可。_
  >
  > _无效的证书链实际上会使服务器证书无效，并导致浏览器警告。在实践中，这个问题有时难以诊断，因为某些浏览器可以重建不完整的链，而有些则不能。所有浏览器都倾向于缓存和重复使用中间证书。_

要测试证书链的验证，请使用以下工具之一：

- [SSL Checker by sslshopper](https://www.sslshopper.com/ssl-checker.html)
- [SSL Checker by namecheap](https://decoder.link/sslchecker/)
- [SSL Server Test by Qualys](https://www.ssllabs.com/ssltest/analyze.html)

有关更多信息，请参阅 [What is the SSL Certificate Chain?](https://support.dnsimple.com/articles/what-is-ssl-certificate-chain/) 和 [Get your certificate chain right](https://medium.com/@superseb/get-your-certificate-chain-right-4b117a9c0fce)。另见 [What happens to code sign certificates when when root CA expires?](https://serverfault.com/questions/878919/what-happens-to-code-sign-certificates-when-when-root-ca-expires)。

<a id="what-is-the-main-purpose-of-the-intermediate-ca"></a>
###### 中间 CA 的主要目的是什么？

出于增强安全性的目的（额外的安全级别），如今大多数最终用户证书由中间证书颁发机构颁发。使用中间 CA 证书更安全，因为这样根 CA 可以离线（牺牲便利性以换取安全性）。因此，如果中间 CA 被攻破，不会影响根 CA。

如果服务器没有随主域证书一起发送中间证书，浏览器将开始抛出错误，显示 `NET:ERR_CERT_AUTHORITY_INVALID`（在 Chrome 中），因为它期望的是签署了域证书的中间证书，但只收到了域证书。

  > 如果你不信任证书颁发者，请不要执行以下操作！

请参阅 [Lie Ryan](https://security.stackexchange.com/users/2755/lie-ryan) 的这篇[精彩回答](https://security.stackexchange.com/questions/128779/why-is-it-more-secure-to-use-intermediate-ca-certificates/128800#128800)：

  > _由于根证书被入侵而部署新的根证书，比替换中间证书被入侵的证书要困难得多。[...] 在短时间内做到这一点极其困难。人们更新浏览器的频率不够高。某些软件（如浏览器）具有快速广播已撤销根证书的机制，某些软件供应商在发现产品中存在严重安全漏洞时会紧急发布，但你可以确信，他们不一定认为添加新根证书值得紧急更新。人们也不会急于更新他们的软件以获取新的根证书。_

以及 [gowenfawr](https://security.stackexchange.com/users/3365/gowenfawr) 的这篇[回答](https://security.stackexchange.com/questions/128779/why-is-it-more-secure-to-use-intermediate-ca-certificates/128791#128791)：

  > _根 CA 离线是为了缓慢、笨拙但更安全地处理请求。使用多个中间 CA 允许将“在线且可访问”的风险分散到不同的证书集中；鸡蛋被分到了不同的篮子里。_

<a id="single-domain"></a>
##### 单域名

当证书只有一个 SAN 字段且包含对单个子域名/主机名的引用时，它就是单域名证书（它不能保护任何其他域名）。

<a id="multi-domain"></a>
##### 多域名

多域名证书通常也称为 SAN 证书（使用 SSL 证书的 SAN 功能），可用于多个域名。

当用户尝试访问受多域名/SAN 证书保护的网站时，浏览器将检查证书以查看 URL 是否与其中包含的 SAN 名称之一匹配。如果匹配，将建立到服务器的安全连接。

多域名证书有时有 100 个或更多的 SAN 字段，其中一些或全部字段可能包含通配符，从而创建混合的“多域名通配符”证书。

<a id="wildcard"></a>
##### 通配符

当你希望使用单个证书保护无限数量的子域时，可以使用通配符证书。

作者在此简要解释了[为什么你可能不应该使用通配符证书](https://gist.github.com/joepie91/7e5cad8c0726fd6a5e90360a754fc568)，因为它将使你的安全面临风险。

<a id="wildcard-ssl-doesnt-handle-root-domain"></a>
##### 通配符 SSL 不涵盖根域名？

不，这是不可能的。默认情况下，通配符证书仅对 `*.example.com` 有效，而非 `example.com`。名称中的通配符仅反映单个标签，且通配符只能位于最左侧。因此 `*.*.example.org` 或 `www.*.example.org` 是不可能的。而 `*.example.org` 既不匹配 `example.org`，也不匹配 `www.subdomain.example.org`，只匹配 `sub.example.org`。

  > 为了保护域名本身和域内的主机，你需要获取在 SAN 扩展中包含这些名称的证书。

从技术上讲，通配符证书是基于子域的未知子级颁发的。大多数通配符证书是为 3 部分域名（`*.domain.com`）颁发的，但也很常见到它们用于 4 部分域名（例如 `*.domain.co.uk`）。

规范性答案应在 [RFC 2818 - 服务器身份](https://tools.ietf.org/html/rfc2818#section-3.1) <sup>[IETF]</sup> 中：

  > _匹配使用 RFC 2459 指定的匹配规则。如果证书中存在多个给定类型的身份（例如，多个 dNSName 名称，则集合中任何一个匹配即视为可接受。）名称可以包含通配符字符 `*`，该字符被认为匹配任何单个域名组件或组件片段。例如，`*.a.com` 匹配 `foo.a.com` 但不匹配 `bar.foo.a.com`。`f*.com` 匹配 `foo.com` 但不匹配 `bar.com`。_

[RFC 2459 - 服务器身份检查](https://tools.ietf.org/html/rfc2595#section-2.4) <sup>[IETF]</sup> 规定：

  > _证书中可以将 "`*`" 通配符字符用作**最左侧的名称组件**。例如，`*.example.com` 将匹配 `a.example.com`、`foo.example.com` 等，但**不匹配** `example.com`。_

本质上，标准规定 `*` 应匹配 1 个或多个非点字符。因此根域名需要作为备用名称才能验证通过。

对于 `*.example.com` 证书：

- `a.example.com` 应通过
- `www.example.com` 应通过
- `example.com` 不应通过
- `a.b.example.com` 不应通过

有时，某些 SSL 提供商会自动将根域名作为使用者备用名称添加到通配符 SSL 证书中，例如：

```bash
issuer: RapidSSL RSA CA 2018 (DigiCert Inc)
cn: example.com
san: *.example.com example.com
```

另一个有趣的事情是，你可以在同一证书中包含多个通配符名称，也就是说，你可以在同一证书中包含 `*.example.org` 和 `*.subdomain.example.org`。你应该不难找到愿意颁发此类证书的证书颁发机构，并且大多数客户端应该接受它。

<a id="https-with-self-signed-certificate-vs-http"></a>
##### 使用自签名证书的 HTTPS 与 HTTP

| <b>特性</b> | <b>HTTP</b> | <b>使用自签名证书的 HTTPS</b> |
| :---         | :---         | :---         |
| 加密 | 否 | **是** |
| 授权 | 否 | 否（如果你隐式信任该证书的颁发者，则为**是**） |
| 隐私 | 否 | 否（如果你隐式信任该证书的颁发者，则为**是**） |
| 性能 | **快** | **比 HTTP 更快**（在某些条件下） |

看看 [Kevin Cox](https://stackoverflow.com/users/1166181/kevin-cox) 这个[精彩的解释](https://stackoverflow.com/a/20578199)：

  > _自签名证书并不严格劣于由信誉良好的 CA 签名的证书，并且在所有技术方面，它们都优于纯 HTTP。从签名和加密的角度来看，它们是相同的。两者都可以对流量进行签名和加密，使得其他人无法窃听或进行修改。_

  > _区别在于证书被指定为受信任的方式。对于 CA 签名的证书，用户信任的是其浏览器/操作系统中安装的一组受信任的 CA。如果他们看到由其中某个 CA 签名的证书，就会接受它，一切正常。如果不是（例如自签名的情况），你会看到一个很大的可怕警告。_

  > _对自签名证书显示此警告的原因是浏览器不知道谁控制着该证书。浏览器信任的 CA 以验证其仅签署网站所有者的证书而闻名。因此，浏览器通过扩展信任该证书对应的私钥由网站运营商控制（并且希望如此）。对于自签名证书，浏览器无法知道该证书是由网站所有者生成的，还是由想要读取你流量的中间人生成的。为了安全起见，浏览器拒绝该证书，除非被证明有效……然后你会看到一个大的红色警告。_

  > _如果你不验证证书，那么与未加密的 HTTP 相比，你没有任何收获，因为你和服务器之间的任何人都可以生成他们自己的证书，而你却浑然不知。这甚至可能被认为比纯 HTTP 更糟糕，因为我们这些情感脆弱的人类可能会被误导，认为我们的连接是安全的，但与 HTTP 相比，唯一的技术缺点就是一些浪费的 CPU 周期。_

- **安全性**

对我来说，自签名证书适用于测试目的和内部服务，前提是你可以信任证书的颁发者（但你仍然需要通过手动验证证书颁发机构服务器是安全的来隐式授权颁发者；无法知道谁签署了证书或它是否应该被信任）。否则，它们只会制造安全的假象（仅提供加密和 HTTPS 连接），仅此而已，因此原则上，自签名证书应始终引起怀疑，并且只能在受控环境中使用。

- **性能**

因此，需要牢记的重要事项是性能。在我看来，HTTP 慢于带有 HTTP/2（例如一个 TCP 连接、多路复用、HPACK 标头压缩）、HSTS、OCSP Stapling 及其他几项改进的 HTTPS，但初始 TLS 握手需要额外的两次往返除外（但我认为 TLS 的性能影响已不像过去那么重要）。另请参见 [HTTP vs HTTPS Test](http://www.httpvshttps.com/) 和 [TLS has exactly one performance problem: it is not used widely enough](https://istlsfastyet.com/)。

<a id="tls-server-name-indication"></a>
#### TLS 服务器名称指示

SNI 是 SSL/TLS 协议的一个扩展，允许客户端（例如浏览器）在 TLS 握手过程开始时提供它试图连接的精确主机名（它指示浏览器在握手过程开始时正在联系哪个主机名）。在 HTTP 服务器端，它允许多个连接使用相同的 IP 地址和端口号，而无需使用多个 IP 地址。

请参阅以下带有 SNI 扩展的通信示例图：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/tls/with_sni.png" alt="with_sni">
</p>

<sup><i>此信息图来自 [Supporting virtual servers with Server Name Indication](https://nnc3.com/mags/LM10/Magazine/Archive/2008/92/072-074_SNI/article.html)。</i></sup>

来自 NGINX 文档：

  > _这是由 SSL 协议行为导致的。SSL 连接在浏览器发送 HTTP 请求之前建立，而 nginx 不知道所请求服务器的名称。因此，它可能只提供默认服务器的证书。_

请看这个：

  > _在单个 IP 地址上运行多个 HTTPS 服务器的更通用解决方案是 TLS 服务器名称指示扩展 (SNI，[RFC 6066](https://tools.ietf.org/html/rfc6066) <sup>[IETF]</sup>)，它允许浏览器在 SSL 握手期间传递所请求的服务器名称，因此服务器将知道应该为连接使用哪个证书。_

有关更多信息，请参阅 [What Is SNI? How TLS Server Name Indication Works](https://www.cloudflare.com/learning/ssl/what-is-sni/) 和 [Supporting virtual servers with Server Name Indication](https://nnc3.com/mags/LM10/Magazine/Archive/2008/92/072-074_SNI/article.html)。

<a id="verify-your-ssl-tls--ciphers-implementation"></a>
#### 验证你的 SSL、TLS 和密码实现

| <b>工具</b> | <b>描述</b> |
| :---         | :---         |
| **[SSL Labs by Qualys](https://www.ssllabs.com/ssltest/)** | 检查所有最新的漏洞和错误配置 |
| **[ImmuniWeb SSL Security Test](https://www.immuniweb.com/ssl/)** | 验证配置是否符合 PCI DSS、HIPAA 和 NIST |

<a id="useful-video-resources"></a>
#### 有用的视频资源

- [Transport Layer Security, TLS 1.2 and 1.3 (Explained by Example)](https://youtu.be/AlE5X1NlHgg)
- [35C3 - The Rocky Road to TLS 1.3 and better Internet Encryption](https://youtu.be/i6mGfZrypP4)
- [SSL/TLS in action with Wireshark](https://youtu.be/u4ht-E-Kihk)
- [SF18US - 35: Examining SSL encryption/decryption using Wireshark (Ross Bagurdes)](https://youtu.be/0X2BVwNX4ks)
- [Breaking Down the TLS Handshake](https://youtu.be/cuR05y_2Gxc)
- [SSL/TLS handshake Protocol](https://youtu.be/sEkw8ZcxtFk)
- [What is a TLS Cipher Suite?](https://youtu.be/ZM3tXhPV8v0)
- [Strong vs. Weak TLS Ciphers](https://youtu.be/k_C2HcJbgMc)
- [Perfect Forward Secrecy](https://youtu.be/IkM3R-KDu44)
- [How SSL certificate works?](https://youtu.be/33VYnE7Bzpk)
- [Intro to Digital Certificates](https://youtu.be/qXLD2UHq2vk)
- [Digital Certificates: Chain of Trust](https://youtu.be/heacxYUnFHA)
- [Elliptic Curves - Computerphile](https://youtu.be/NF1pwjL9-DE)
- [Elliptic Curve Cryptography Overview](https://youtu.be/dCvB-mhkT0w)
- [Secret Key Exchange (Diffie-Hellman) - Computerphile](https://youtu.be/NmM9HA2MQGI)
- [The Cryptographers' Panel](https://youtu.be/gMc9fHvc78Y)
- [The Cryptographers' Panel 2015](https://youtu.be/9RtZrNPP26w)
