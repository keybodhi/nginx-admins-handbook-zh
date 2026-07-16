<div align="center">
  <h1>Nginx 管理员手册</h1>
</div>

<div align="center">
  <b><code>我关于 NGINX 管理基础、技巧与陷阱、注意事项的笔记。</code></b>
</div>

<br>

<p align="center">
  <a href="https://www.hostingadvice.com/how-to/nginx-vs-apache/">
    <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/nginx_meme.png" alt="Meme">
  </a>
</p>

<br>

<p align="center">
  <sup>
    <i>
      Hi-diddle-diddle, he played on his<br>
      fiddle and danced with lady pigs.<br>
      Number three said, "Nicks on tricks!<br>
      I'll build my house with <b>EN-jin-EKS</b>!".<br>
      <a href="https://g.co/kgs/HCcQVz">The Three Little Pigs: Who's Afraid of the Big Bad Wolf?</a>
    </i>
  </sup>
</p>

<br>

<p align="center">
  <a href="https://github.com/trimstray/nginx-admins-handbook/pulls">
    <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg?longCache=true" alt="Pull Requests">
  </a>
  <a href="LICENSE.md">
    <img src="https://img.shields.io/badge/License-MIT-lightgrey.svg?longCache=true" alt="MIT License">
  </a>
</p>

<br>

****

<a id="table-of-contents"></a>
# 目录

- **[介绍](#introduction)**<a id="toc-introduction"></a>
  * [序言](#prologue)
  * [为什么创建这本手册](#why-i-created-this-handbook)
  * [这本手册的受众](#who-this-handbook-is-for)
  * [开始之前](#before-you-start)
  * [贡献与支持](#contributing--support)
  * [RSS 订阅与更新](#rss-feed--updates)
  * [统领一切的清单](#checklist-to-rule-them-all)
- **[额外内容](#bonus-stuff)**<a id="toc-bonus-stuff"></a>
  * [配置报告](#configuration-reports)
    * [SSL Labs](#ssl-labs)
    * [Mozilla Observatory](#mozilla-observatory)
  * [可打印的加固速查表](#printable-hardening-cheatsheets)
  * [全自动安装](#fully-automatic-installation)
  * [静态错误页面生成器](#static-error-pages-generator)
  * [服务器名称解析器](#server-names-parser)
- **[书籍](#books)**<a id="toc-books"></a>
  * [Nginx Essentials](#nginx-essentials)
  * [Nginx Cookbook](#nginx-cookbook)
  * [Nginx HTTP Server](#nginx-http-server)
  * [Nginx High Performance](#nginx-high-performance)
  * [Mastering Nginx](#mastering-nginx)
  * [ModSecurity 3.0 and NGINX: Quick Start Guide](#modsecurity-30-and-nginx-quick-start-guide)
  * [Cisco ACE to NGINX: Migration Guide](#cisco-ace-to-nginx-migration-guide)
- **[外部资源](#external-resources)**<a id="toc-external-resources"></a>
  * [Nginx 官方](#nginx-official)
  * [Nginx 发行版](#nginx-distributions)
  * [对比评测](#comparison-reviews)
  * [速查表与参考](#cheatsheets--references)
  * [性能与加固](#performance--hardening)
  * [演示与视频](#presentations--videos)
  * [操练场](#playgrounds)
  * [配置生成器](#config-generators)
  * [配置解析器](#config-parsers)
  * [配置管理器](#config-managers)
  * [静态分析器](#static-analyzers)
  * [日志分析器](#log-analyzers)
  * [性能分析器](#performance-analyzers)
  * [构建工具](#builder-tools)
  * [基准测试工具](#benchmarking-tools)
  * [调试工具](#debugging-tools)
  * [安全与 Web 测试工具](#security--web-testing-tools)
  * [开发](#development)
  * [在线与 Web 工具](#online--web-tools)
  * [其他资源](#other-stuff)
- **[下一步是什么？](#whats-next)**

<details>
<summary><b>其他章节</b></summary><br>

- **[HTTP 基础](doc/HTTP_BASICS.md#http-basics)**<a id="toc-http-basics"></a>
  * [介绍](doc/HTTP_BASICS.md#introduction-1)
  * [特性与架构](doc/HTTP_BASICS.md#features-and-architecture)
  * [HTTP/2](doc/HTTP_BASICS.md#http2)
    * [如何调试 HTTP/2？](doc/HTTP_BASICS.md#how-to-debug-http2)
  * [HTTP/3](doc/HTTP_BASICS.md#http3)
  * [URI vs URL](doc/HTTP_BASICS.md#uri-vs-url)
  * [连接 vs 请求](doc/HTTP_BASICS.md#connection-vs-request)
  * [HTTP 头部](doc/HTTP_BASICS.md#http-headers)
    * [头部压缩](#header-compression)
  * [HTTP 方法](doc/HTTP_BASICS.md#http-methods)
  * [请求](doc/HTTP_BASICS.md#request)
    * [请求行](doc/HTTP_BASICS.md#request-line)
      * [方法](doc/HTTP_BASICS.md#methods)
      * [请求 URI](doc/HTTP_BASICS.md#request-uri)
      * [HTTP 版本](doc/HTTP_BASICS.md#http-version)
    * [请求头部字段](doc/HTTP_BASICS.md#request-header-fields)
    * [消息体](doc/HTTP_BASICS.md#message-body)
    * [生成请求](doc/HTTP_BASICS.md#generate-requests)
  * [响应](doc/HTTP_BASICS.md#response)
    * [状态行](doc/HTTP_BASICS.md#status-line)
      * [HTTP 版本](doc/HTTP_BASICS.md#http-version-1)
      * [状态码与原因短语](doc/HTTP_BASICS.md#status-codes-and-reason-phrase)
    * [响应头部字段](doc/HTTP_BASICS.md#response-header-fields)
    * [消息体](doc/HTTP_BASICS.md#message-body-1)
  * [HTTP 客户端](doc/HTTP_BASICS.md#http-client)
    * [IP 地址简写](doc/HTTP_BASICS.md#ip-address-shortcuts)
  * [后端 Web 架构](doc/HTTP_BASICS.md#back-end-web-architecture)
  * [有用的视频资源](doc/HTTP_BASICS.md#useful-video-resources)
- **[SSL/TLS 基础](doc/SSL_TLS_BASICS.md#ssltls-basics)**<a id="toc-ssltls-basics"></a>
  * [介绍](doc/SSL_TLS_BASICS.md#introduction-2)
  * [TLS 版本](doc/SSL_TLS_BASICS.md#tls-versions)
  * [TLS 握手](doc/SSL_TLS_BASICS.md#tls-handshake)
    * [在 TCP/IP 协议栈中，TLS 位于哪一层？](doc/SSL_TLS_BASICS.md#in-which-layer-is-tls-situated-within-the-tcpip-stack)
  * [RSA 和 ECC 密钥/证书](doc/SSL_TLS_BASICS.md#rsa-and-ecc-keyscertificates)
  * [密码套件](doc/SSL_TLS_BASICS.md#cipher-suites)
    * [认证加密 (AEAD) 密码套件](doc/SSL_TLS_BASICS.md#authenticated-encryption-aead-cipher-suites)
    * [为什么密码套件很重要？](doc/SSL_TLS_BASICS.md#why-cipher-suites-are-important)
    * [不安全、弱、安全和推荐分别意味着什么？](doc/SSL_TLS_BASICS.md#what-does-insecure-weak-secure-and-recommended-mean)
    * [NGINX 和 TLS 1.3 密码套件](doc/SSL_TLS_BASICS.md#nginx-and-tls-13-cipher-suites)
  * [Diffie-Hellman 密钥交换](doc/SSL_TLS_BASICS.md#diffie-hellman-key-exchange)
    * [这些 DH 参数的确切目的是什么？](doc/SSL_TLS_BASICS.md#what-exactly-is-the-purpose-of-these-dh-parameters)
  * [证书](doc/SSL_TLS_BASICS.md#certificates)
    * [信任链](doc/SSL_TLS_BASICS.md#chain-of-trust)
      * [中间 CA 的主要目的是什么？](doc/SSL_TLS_BASICS.md#what-is-the-main-purpose-of-the-intermediate-ca)
    * [单域名](doc/SSL_TLS_BASICS.md#single-domain)
    * [多域名](doc/SSL_TLS_BASICS.md#multi-domain)
    * [通配符](doc/SSL_TLS_BASICS.md#wildcard)
    * [通配符 SSL 不处理根域名？](doc/SSL_TLS_BASICS.md#wildcard-ssl-doesnt-handle-root-domain)
    * [使用自签名证书的 HTTPS 与 HTTP](doc/SSL_TLS_BASICS.md#https-with-self-signed-certificate-vs-http)
  * [TLS 服务器名称指示](doc/SSL_TLS_BASICS.md#tls-server-name-indication)
  * [验证你的 SSL、TLS 和密码实现](doc/SSL_TLS_BASICS.md#verify-your-ssl-tls--ciphers-implementation)
  * [有用的视频资源](doc/SSL_TLS_BASICS.md#useful-video-resources)
- **[NGINX 基础](doc/NGINX_BASICS.md#nginx-basics)**<a id="toc-nginx-basics"></a>
  * [目录和文件](doc/NGINX_BASICS.md#directories-and-files)
  * [命令](doc/NGINX_BASICS.md#commands)
  * [进程](doc/NGINX_BASICS.md#processes)
    * [CPU 绑定](doc/NGINX_BASICS.md#cpu-pinning)
    * [工作进程的关闭](doc/NGINX_BASICS.md#shutdown-of-worker-processes)
  * [配置语法](doc/NGINX_BASICS.md#configuration-syntax)
    * [注释](doc/NGINX_BASICS.md#comments)
    * [行尾](doc/NGINX_BASICS.md#end-of-lines)
    * [变量、字符串和引号](doc/NGINX_BASICS.md#variables-strings-and-quotes)
    * [指令、块和上下文](doc/NGINX_BASICS.md#directives-blocks-and-contexts)
    * [外部文件](doc/NGINX_BASICS.md#external-files)
    * [度量单位](doc/NGINX_BASICS.md#measurement-units)
    * [PCRE 正则表达式](doc/NGINX_BASICS.md#regular-expressions-with-pcre)
    * [启用语法高亮](doc/NGINX_BASICS.md#enable-syntax-highlighting)
  * [连接处理](doc/NGINX_BASICS.md#connection-processing)
    * [事件驱动架构](doc/NGINX_BASICS.md#event-driven-architecture)
    * [多进程](doc/NGINX_BASICS.md#multiple-processes)
    * [并发连接](doc/NGINX_BASICS.md#simultaneous-connections)
    * [HTTP Keep-Alive 连接](doc/NGINX_BASICS.md#http-keep-alive-connections)
    * [sendfile、tcp_nodelay 和 tcp_nopush](doc/NGINX_BASICS.md#sendfile-tcp_nodelay-and-tcp_nopush)
  * [请求处理阶段](doc/NGINX_BASICS.md#request-processing-stages)
  * [服务器块逻辑](doc/NGINX_BASICS.md#server-blocks-logic)
    * [处理传入连接](doc/NGINX_BASICS.md#handle-incoming-connections)
    * [匹配 location](doc/NGINX_BASICS.md#matching-location)
    * [rewrite vs return](doc/NGINX_BASICS.md#rewrite-vs-return)
    * [URL 重定向](doc/NGINX_BASICS.md#url-redirections)
    * [try_files 指令](doc/NGINX_BASICS.md#try_files-directive)
    * [if、break 和 set](doc/NGINX_BASICS.md#if-break-and-set)
    * [root vs alias](doc/NGINX_BASICS.md#root-vs-alias)
    * [internal 指令](doc/NGINX_BASICS.md#internal-directive)
    * [外部和内部重定向](doc/NGINX_BASICS.md#external-and-internal-redirects)
    * [allow 和 deny](doc/NGINX_BASICS.md#allow-and-deny)
    * [uri vs request_uri](doc/NGINX_BASICS.md#uri-vs-request_uri)
  * [压缩与解压](doc/NGINX_BASICS.md#compression-and-decompression)
    * [NGINX 压缩 gzip 级别的最佳值是多少？](doc/NGINX_BASICS.md#what-is-the-best-nginx-compression-gzip-level)
  * [哈希表](doc/NGINX_BASICS.md#hash-tables)
    * [服务器名称哈希表](doc/NGINX_BASICS.md#server-names-hash-table)
  * [日志文件](doc/NGINX_BASICS.md#log-files)
    * [条件日志](doc/NGINX_BASICS.md#conditional-logging)
    * [手动日志轮转](doc/NGINX_BASICS.md#manually-log-rotation)
    * [错误日志严重级别](doc/NGINX_BASICS.md#error-log-severity-levels)
    * [如何记录请求的开始时间？](doc/NGINX_BASICS.md#how-to-log-the-start-time-of-a-request)
    * [如何记录 HTTP 请求体？](doc/NGINX_BASICS.md#how-to-log-the-http-request-body)
    * [NGINX upstream 变量返回 2 个值](doc/NGINX_BASICS.md#nginx-upstream-variables-returns-2-values)
  * [反向代理](doc/NGINX_BASICS.md#reverse-proxy)
    * [传递请求](doc/NGINX_BASICS.md#passing-requests)
    * [尾部斜杠](doc/NGINX_BASICS.md#trailing-slashes)
    * [传递头部到后端](doc/NGINX_BASICS.md#passing-headers-to-the-backend)
      * [Host 头部的重要性](doc/NGINX_BASICS.md#importance-of-the-host-header)
      * [重定向和 X-Forwarded-Proto](doc/NGINX_BASICS.md#redirects-and-x-forwarded-proto)
      * [关于 X-Forwarded-For 的警告](doc/NGINX_BASICS.md#a-warning-about-the-x-forwarded-for)
      * [使用 Forwarded 提高可扩展性](doc/NGINX_BASICS.md#improve-extensibility-with-forwarded)
    * [响应头部](doc/NGINX_BASICS.md#response-headers)
  * [负载均衡算法](doc/NGINX_BASICS.md#load-balancing-algorithms)
    * [后端参数](doc/NGINX_BASICS.md#backend-parameters)
    * [带 SSL 的上游服务器](doc/NGINX_BASICS.md#upstream-servers-with-ssl)
    * [轮询 (Round Robin)](doc/NGINX_BASICS.md#round-robin)
    * [加权轮询 (Weighted Round Robin)](doc/NGINX_BASICS.md#weighted-round-robin)
    * [最少连接 (Least Connections)](doc/NGINX_BASICS.md#least-connections)
    * [加权最少连接 (Weighted Least Connections)](doc/NGINX_BASICS.md#weighted-least-connections)
    * [IP Hash](doc/NGINX_BASICS.md#ip-hash)
    * [通用 Hash](doc/NGINX_BASICS.md#generic-hash)
    * [其他方法](doc/NGINX_BASICS.md#other-methods)
  * [速率限制](doc/NGINX_BASICS.md#rate-limiting)
    * [变量](doc/NGINX_BASICS.md#variables)
    * [指令、键和区域](doc/NGINX_BASICS.md#directives-keys-and-zones)
    * [Burst 和 nodelay 参数](doc/NGINX_BASICS.md#burst-and-nodelay-parameters)
  * [NAXSI Web 应用防火墙](doc/NGINX_BASICS.md#naxsi-web-application-firewall)
  * [OWASP ModSecurity 核心规则集 (CRS)](doc/NGINX_BASICS.md#owasp-modsecurity-core-rule-set-crs)
  * [核心模块](doc/NGINX_BASICS.md#core-modules)
    * [ngx_http_geo_module](doc/NGINX_BASICS.md#ngx_http_geo_module)
  * [第三方模块](doc/NGINX_BASICS.md#3rd-party-modules)
    * [ngx_set_misc](doc/NGINX_BASICS.md#ngx_set_misc)
    * [ngx_http_geoip_module](doc/NGINX_BASICS.md#ngx_http_geoip_module)
- **[辅助工具](doc/HELPERS.md#helpers)**<a id="toc-helpers"></a>
  * [从预构建包安装](doc/HELPERS.md#installing-from-prebuilt-packages)
    * [RHEL7 或 CentOS 7](doc/HELPERS.md#rhel7-or-centos-7)
    * [Debian 或 Ubuntu](doc/HELPERS.md#debian-or-ubuntu)
    * [FreeBSD](doc/HELPERS.md#freebsd)
  * [从源码安装](doc/HELPERS.md#installing-from-source)
    * [在 RHEL/Debian/BSD 上自动安装](doc/HELPERS.md#automatic-installation-on-rheldebianbsd)
    * [Nginx 包](doc/HELPERS.md#nginx-package)
    * [依赖](doc/HELPERS.md#dependencies)
    * [补丁](doc/HELPERS.md#patches)
    * [第三方模块](doc/HELPERS.md#3rd-party-modules)
    * [配置选项](doc/HELPERS.md#cconfigure-options)
    * [编译器和链接器](doc/HELPERS.md#compiler-and-linker)
      * [调试符号](doc/HELPERS.md#debugging-symbols)
    * [SystemTap](doc/HELPERS.md#systemtap)
      * [stapxx](doc/HELPERS.md#stapxx)
    * [在 CentOS 7 上安装 Nginx](doc/HELPERS.md#installation-nginx-on-centos-7)
      * [安装前任务](doc/HELPERS.md#pre-installation-tasks)
      * [依赖](doc/HELPERS.md#dependencies)
      * [获取 Nginx 源码](doc/HELPERS.md#get-nginx-sources)
      * [下载第三方模块](doc/HELPERS.md#download-3rd-party-modules)
      * [构建 Nginx](doc/HELPERS.md#build-nginx)
      * [安装后任务](doc/HELPERS.md#post-installation-tasks)
    * [在 CentOS 7 上安装 OpenResty](doc/HELPERS.md#installation-openresty-on-centos-7)
    * [在 Ubuntu 18.04 上安装 Tengine](doc/HELPERS.md#installation-tengine-on-ubuntu-1804)
    * [在 FreeBSD 11.3 上安装 Nginx](doc/HELPERS.md#installation-nginx-on-freebsd-113)
    * [在 FreeBSD 11.3 上安装 Nginx（从 ports）](doc/HELPERS.md#installation-nginx-on-freebsd-113-from-ports)
  * [分析配置](doc/HELPERS.md#analyse-configuration)
  * [监控](doc/HELPERS.md#monitoring)
    * [GoAccess](doc/HELPERS.md#goaccess)
      * [构建与安装](doc/HELPERS.md#build-and-install)
      * [分析日志文件并启用所有记录的统计信息](doc/HELPERS.md#analyse-log-file-and-enable-all-recorded-statistics)
      * [分析压缩日志文件](doc/HELPERS.md#analyse-compressed-log-file)
      * [远程分析日志文件](doc/HELPERS.md#analyse-log-file-remotely)
      * [分析日志文件并生成 html 报告](doc/HELPERS.md#analyse-log-file-and-generate-html-report)
    * [Ngxtop](doc/HELPERS.md#ngxtop)
      * [分析日志文件](doc/HELPERS.md#analyse-log-file)
      * [分析日志文件并打印 4xx 和 5xx 的请求](doc/HELPERS.md#analyse-log-file-and-print-requests-with-4xx-and-5xx)
      * [远程分析日志文件](doc/HELPERS.md#analyse-log-file-remotely-1)
  * [测试](doc/HELPERS.md#testing)
    * [构建 OpenSSL 1.0.2-chacha 版本](doc/HELPERS.md#build-openssl-102-chacha-version)
    * [发送请求并显示响应头部](doc/HELPERS.md#send-request-and-show-response-headers)
    * [使用 http 方法、user-agent、跟随重定向发送请求并显示响应头部](doc/HELPERS.md#send-request-with-http-method-user-agent-follow-redirects-and-show-response-headers)
    * [发送多个请求](doc/HELPERS.md#send-multiple-requests)
    * [测试 SSL 连接](doc/HELPERS.md#testing-ssl-connection)
    * [测试 SSL 连接（调试模式）](doc/HELPERS.md#testing-ssl-connection-debug-mode)
    * [使用 SNI 支持测试 SSL 连接](doc/HELPERS.md#testing-ssl-connection-with-sni-support)
    * [使用特定 SSL 版本测试 SSL 连接](doc/HELPERS.md#testing-ssl-connection-with-specific-ssl-version)
    * [使用特定密码测试 SSL 连接](doc/HELPERS.md#testing-ssl-connection-with-specific-cipher)
    * [测试 OCSP Stapling](doc/HELPERS.md#testing-ocsp-stapling)
    * [验证 0-RTT](doc/HELPERS.md#verify-0-rtt)
    * [测试 SCSV](doc/HELPERS.md#testing-scsv)
    * [使用 ApacheBench (ab) 进行负载测试](doc/HELPERS.md#load-testing-with-apachebench-ab)
      * [标准测试](doc/HELPERS.md#standard-test)
      * [使用 Keep-Alive 头部测试](doc/HELPERS.md#test-with-keep-alive-header)
    * [使用 wrk2 进行负载测试](doc/HELPERS.md#load-testing-with-wrk2)
      * [标准场景](doc/HELPERS.md#standard-scenarios)
      * [POST 调用（使用 Lua）](doc/HELPERS.md#post-call-with-lua)
      * [随机路径（使用 Lua）](doc/HELPERS.md#random-paths-with-lua)
      * [多路径（使用 Lua）](doc/HELPERS.md#multiple-paths-with-lua)
      * [每个线程的随机服务器地址（使用 Lua）](doc/HELPERS.md#random-server-address-to-each-thread-with-lua)
      * [多个 json 请求（使用 Lua）](doc/HELPERS.md#multiple-json-requests-with-lua)
      * [调试模式（使用 Lua）](doc/HELPERS.md#debug-mode-with-lua)
      * [分析进出线程的数据](doc/HELPERS.md#analyse-data-pass-to-and-from-the-threads)
      * [解析 wrk 结果并生成报告](doc/HELPERS.md#parsing-wrk-result-and-generate-report)
    * [使用 locust 进行负载测试](doc/HELPERS.md#load-testing-with-locust)
      * [多路径](doc/HELPERS.md#multiple-paths)
      * [不同用户会话的多路径](doc/HELPERS.md#multiple-paths-with-different-user-sessions)
    * [TCP SYN 洪水拒绝服务攻击](doc/HELPERS.md#tcp-syn-flood-denial-of-service-attack)
    * [HTTP 拒绝服务攻击](doc/HELPERS.md#tcp-syn-flood-denial-of-service-attack)
  * [调试](doc/HELPERS.md#debugging)
    * [显示进程信息](doc/HELPERS.md#show-information-about-nginx-processes)
    * [检查内存使用](doc/HELPERS.md#check-memoryusage)
    * [显示打开的文件](doc/HELPERS.md#show-open-files)
    * [检查段错误消息](doc/HELPERS.md#check-segmentation-fault-messages)
    * [转储配置](doc/HELPERS.md#dump-configuration)
    * [获取 configure 参数列表](doc/HELPERS.md#get-the-list-of-configure-arguments)
    * [检查模块是否已编译](doc/HELPERS.md#check-if-the-module-has-been-compiled)
    * [显示访问最多的 IP 地址](doc/HELPERS.md#show-the-most-accessed-ip-addresses)
    * [显示访问最多的 IP 地址（IP 和 URL）](doc/HELPERS.md#show-the-most-accessed-ip-addresses-ip-and-url)
    * [显示访问最多的 IP 地址（方法、状态码、IP 和 URL）](doc/HELPERS.md#show-the-most-accessed-ip-addresses-method-code-ip-and-url)
    * [显示前 5 位访客（IP 地址）](doc/HELPERS.md#show-the-top-5-visitors-ip-addresses)
    * [显示最常请求的 URL](doc/HELPERS.md#show-the-most-requested-urls)
    * [显示包含特定字符串的最常请求的 URL](doc/HELPERS.md#show-the-most-requested-urls-containing-string)
    * [显示带有 HTTP 方法的最常请求的 URL](doc/HELPERS.md#show-the-most-requested-urls-with-http-methods)
    * [显示最常访问的响应状态码](doc/HELPERS.md#show-the-most-accessed-response-codes)
    * [分析 Web 服务器日志并仅显示 2xx HTTP 状态码](doc/HELPERS.md#analyse-web-server-log-and-show-only-2xx-http-codes)
    * [分析 Web 服务器日志并仅显示 5xx HTTP 状态码](doc/HELPERS.md#analyse-web-server-log-and-show-only-5xx-http-codes)
    * [显示导致 502 的请求，并按 URL 对请求数量排序](doc/HELPERS.md#show-requests-which-result-502-and-sort-them-by-number-per-requests-by-url)
    * [显示导致 404 的 PHP 文件请求，并按 URL 对请求数量排序](doc/HELPERS.md#show-requests-which-result-404-for-php-files-and-sort-them-by-number-per-requests-by-url)
    * [计算 HTTP 响应状态码数量](doc/HELPERS.md#calculating-amount-of-http-response-codes)
    * [计算每秒请求数](doc/HELPERS.md#calculating-requests-per-second)
    * [计算带 IP 地址的每秒请求数](doc/HELPERS.md#calculating-requests-per-second-with-ip-addresses)
    * [计算带 IP 地址和 URL 的每秒请求数](doc/HELPERS.md#calculating-requests-per-second-with-ip-addresses-and-urls)
    * [获取最近 n 小时内的条目](doc/HELPERS.md#get-entries-within-last-n-hours)
    * [获取两个时间戳之间的条目（日期范围）](doc/HELPERS.md#get-entries-between-two-timestamps-range-of-dates)
    * [从 Web 服务器日志获取行速率](doc/HELPERS.md#get-line-rates-from-web-server-log)
    * [跟踪所有进程的网络流量](doc/HELPERS.md#trace-network-traffic-for-all-nginx-processes)
    * [列出 NGINX 访问的所有文件](doc/HELPERS.md#list-all-files-accessed-by-a-nginx)
    * [检查 gzip_static 模块是否工作](doc/HELPERS.md#check-that-the-gzip_static-module-is-working)
    * [哪个工作进程正在处理当前请求](doc/HELPERS.md#which-worker-processing-current-request)
    * [仅捕获 HTTP 数据包](doc/HELPERS.md#capture-only-http-packets)
    * [从 HTTP 数据包中提取 User Agent](doc/HELPERS.md#extract-user-agent-from-the-http-packets)
    * [仅捕获 HTTP GET 和 POST 数据包](doc/HELPERS.md#capture-only-http-get-and-post-packets)
    * [按源 IP 和目标端口捕获请求并过滤](doc/HELPERS.md#capture-requests-and-filter-by-source-ip-and-destination-port)
    * [实时捕获 HTTP 请求/响应，按 GET、HEAD 过滤并保存到文件](doc/HELPERS.md#capture-http-requests--responses-in-real-time-filter-by-get-head-and-save-to-a-file)
    * [转储进程的内存](doc/HELPERS.md#dump-a-processs-memory)
    * [GNU 调试器 (gdb)](doc/HELPERS.md#gnu-debugger-gdb)
      * [从运行进程转储配置](doc/HELPERS.md#dump-configuration-from-a-running-process)
      * [显示内存中的调试日志](doc/HELPERS.md#show-debug-log-in-memory)
      * [核心转储回溯](doc/HELPERS.md#core-dump-backtrace)
    * [调试 socket 泄漏](doc/HELPERS.md#debugging-socket-leaks)
  * [Shell 别名](doc/HELPERS.md#shell-aliases)
  * [配置片段](doc/HELPERS.md#configuration-snippets)
    * [移除 Nginx 服务器头部](doc/HELPERS.md#nginx-server-header-removal)
    * [自定义日志格式](doc/HELPERS.md#custom-log-formats)
    * [仅记录 4xx/5xx](doc/HELPERS.md#log-only-4xx5xx)
    * [使用基本认证限制访问](doc/HELPERS.md#restricting-access-with-basic-authentication)
    * [使用客户端证书限制访问](doc/HELPERS.md#restricting-access-with-client-certificate)
    * [按地理位置限制访问](doc/HELPERS.md#restricting-access-by-geographical-location)
      * [GeoIP 2 数据库](doc/HELPERS.md#geoip-2-database)
    * [使用 SSI 的动态错误页面](doc/HELPERS.md#dynamic-error-pages-with-ssi)
    * [阻止/允许 IP 地址](doc/HELPERS.md#blockingallowing-ip-addresses)
    * [阻止引荐来源垃圾](doc/HELPERS.md#blocking-referrer-spam)
    * [限制引荐来源垃圾](doc/HELPERS.md#limiting-referrer-spam)
    * [阻止 User-Agent](doc/HELPERS.md#blocking-user-agent)
    * [限制 User-Agent](doc/HELPERS.md#limiting-user-agent)
    * [使用突发模式限制请求速率](doc/HELPERS.md#limiting-the-rate-of-requests-with-burst-mode)
    * [使用突发模式和 nodelay 限制请求速率](doc/HELPERS.md#limiting-the-rate-of-requests-with-burst-mode-and-nodelay)
    * [使用 geo 和 map 按 IP 限制请求速率](doc/HELPERS.md#limiting-the-rate-of-requests-per-ip-with-geo-and-map)
    * [限制连接数](doc/HELPERS.md#limiting-the-number-of-connections)
    * [使用尾部斜杠](doc/HELPERS.md#using-trailing-slashes)
    * [正确将所有 HTTP 请求重定向到 HTTPS](doc/HELPERS.md#properly-redirect-all-http-requests-to-https)
    * [添加和移除 www 前缀](doc/HELPERS.md#adding-and-removing-the-www-prefix)
    * [代理/重写并保留原始 URL](doc/HELPERS.md#proxyrewrite-and-keep-the-original-url)
    * [代理/重写并保留部分原始 URL](doc/HELPERS.md#proxyrewrite-and-keep-the-part-of-original-url)
    * [代理/重写而不改变原始 URL（在浏览器中）](doc/HELPERS.md#proxyrewrite-without-changing-the-original-url-in-browser)
    * [修改 301/302 响应体](doc/HELPERS.md#modify-301302-response-body)
    * [将带负载的 POST 请求重定向到外部端点](doc/HELPERS.md#redirect-post-request-with-payload-to-external-endpoint)
    * [根据 HTTP 方法路由到不同的后端](doc/HELPERS.md#route-to-different-backends-based-on-HTTP-method)
    * [使用 CORS 头部允许多个跨域](doc/HELPERS.md#allow-multiple-cross-domains-using-the-cors-headers)
    * [设置 X-Forwarded-Proto 中的正确 scheme](doc/HELPERS.md#set-correct-scheme-passed-in-x-forwarded-proto)
  * [其他片段](doc/HELPERS.md#other-snippets)
    * [重新创建基础目录](doc/HELPERS.md#recreate-base-directory)
    * [创建临时静态后端](doc/HELPERS.md#create-a-temporary-static-backend)
    * [创建带 SSL 支持的临时静态后端](doc/HELPERS.md#create-a-temporary-static-backend-with-ssl-support)
    * [使用 htpasswd 命令生成密码文件](doc/HELPERS.md#generate-password-file-with-htpasswd-command)
    * [生成无口令的私钥](doc/HELPERS.md#generate-private-key-without-passphrase)
    * [生成带口令的私钥](doc/HELPERS.md#generate-private-key-with-passphrase)
    * [从私钥中移除口令](doc/HELPERS.md#remove-passphrase-from-private-key)
    * [用口令加密现有私钥](doc/HELPERS.md#encrypt-existing-private-key-with-a-passphrase)
    * [生成 CSR](doc/HELPERS.md#generate-csr)
    * [生成 CSR（从现有证书获取元数据）](doc/HELPERS.md#generate-csr-metadata-from-existing-certificate)
    * [使用 -config 参数生成 CSR](doc/HELPERS.md#generate-csr-with--config-param)
    * [生成私钥和 CSR](doc/HELPERS.md#generate-private-key-and-csr)
    * [列出可用的 EC 曲线](doc/HELPERS.md#list-available-ec-curves)
    * [打印 ECDSA 私钥和公钥](doc/HELPERS.md#print-ecdsa-private-and-public-keys)
    * [生成 ECDSA 私钥](doc/HELPERS.md#generate-ecdsa-private-key)
    * [生成私钥和 CSR (ECC)](doc/HELPERS.md#generate-private-key-and-csr-ecc)
    * [生成自签名证书](doc/HELPERS.md#generate-self-signed-certificate)
    * [从现有私钥生成自签名证书](doc/HELPERS.md#generate-self-signed-certificate-from-existing-private-key)
    * [从现有私钥和 CSR 生成自签名证书](doc/HELPERS.md#generate-self-signed-certificate-from-existing-private-key-and-csr)
    * [生成多域名证书 (Certbot)](doc/HELPERS.md#generate-multidomain-certificate-certbot)
    * [生成通配符证书 (Certbot)](doc/HELPERS.md#generate-wildcard-certificate-certbot)
    * [生成 4096 位私钥的证书 (Certbot)](doc/HELPERS.md#generate-certificate-with-4096-bit-private-key-certbot)
    * [生成 DH 公共参数](doc/HELPERS.md#generate-dh-public-parameters)
    * [显示 DH 公共参数](doc/HELPERS.md#display-dh-public-parameters)
    * [从 pfx 提取私钥](doc/HELPERS.md#extract-private-key-from-pfx)
    * [从 pfx 提取私钥和证书](doc/HELPERS.md#extract-private-key-and-certs-from-pfx)
    * [从 p7b 提取证书](doc/HELPERS.md#extract-certs-from-p7b)
    * [将 DER 转换为 PEM](doc/HELPERS.md#convert-der-to-pem)
    * [将 PEM 转换为 DER](doc/HELPERS.md#convert-pem-to-der)
    * [验证证书的支持用途](doc/HELPERS.md#verification-of-the-certificates-supported-purposes)
    * [检查私钥](doc/HELPERS.md#check-private-key)
    * [验证私钥](doc/HELPERS.md#verification-of-the-private-key)
    * [从私钥获取公钥](doc/HELPERS.md#get-public-key-from-private-key)
    * [验证公钥](doc/HELPERS.md#verification-of-the-public-key)
    * [验证证书](doc/HELPERS.md#verification-of-the-certificate)
    * [验证 CSR](doc/HELPERS.md#verification-of-the-csr)
    * [检查私钥和证书是否匹配](doc/HELPERS.md#check-the-private-key-and-the-certificate-are-match)
    * [检查私钥和 CSR 是否匹配](doc/HELPERS.md#check-the-private-key-and-the-csr-are-match)
    * [TLSv1.3 和 CCM 密码](doc/HELPERS.md#tlsv13-and-ccm-ciphers)
- **[基础规则 (16)](doc/RULES.md#base-rules)**<a id="toc-base-rules"></a>
  * [组织 Nginx 配置](doc/RULES.md#beginner-organising-nginx-configuration)
  * [格式化、美化和缩进 Nginx 代码](doc/RULES.md#beginner-format-prettify-and-indent-your-nginx-code)
  * [使用 reload 选项动态更改配置](doc/RULES.md#beginner-use-reload-option-to-change-configurations-on-the-fly)
  * [为 80 和 443 端口分别使用 listen 指令](doc/RULES.md#beginner-separate-listen-directives-for-80-and-443-ports)
  * [使用 address:port 对定义 listen 指令](doc/RULES.md#beginner-define-the-listen-directives-with-addressport-pair)
  * [防止处理未定义服务器名称的请求](doc/RULES.md#beginner-prevent-processing-requests-with-undefined-server-names)
  * [绝不在 listen 或 upstream 指令中使用主机名](doc/RULES.md#beginner-never-use-a-hostname-in-a-listen-or-upstream-directives)
  * [正确使用 add_header 和 proxy_*_header 指令设置 HTTP 头部](doc/RULES.md#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)
  * [为 listen 指令仅使用一个 SSL 配置](doc/RULES.md#beginner-use-only-one-ssl-config-for-the-listen-directive)
  * [使用 geo/map 模块替代 allow/deny](doc/RULES.md#beginner-use-geomap-modules-instead-of-allowdeny)
  * [映射万物...](doc/RULES.md#beginner-map-all-the-things)
  * [为未匹配的 location 设置全局根目录](doc/RULES.md#beginner-set-global-root-directory-for-unmatched-locations)
  * [使用 return 指令进行 URL 重定向 (301, 302)](doc/RULES.md#beginner-use-return-directive-for-url-redirection-301-302)
  * [配置日志轮转策略](doc/RULES.md#beginner-configure-log-rotation-policy)
  * [使用简单的自定义错误页面](doc/RULES.md#beginner-use-simple-custom-error-pages)
  * [不要重复 index 指令，仅在 http 块中使用](doc/RULES.md#beginner-dont-duplicate-index-directive-use-it-only-in-the-http-block)
- **[调试 (5)](doc/RULES.md#debugging)**<a id="toc-debugging"></a>
  * [使用自定义日志格式](doc/RULES.md#beginner-use-custom-log-formats)
  * [使用调试模式追踪意外行为](doc/RULES.md#beginner-use-debug-mode-to-track-down-unexpected-behaviour)
  * [通过禁用守护进程、主进程和除一个之外的所有工作进程来改进调试](doc/RULES.md#beginner-improve-debugging-by-disable-daemon-master-process-and-all-workers-except-one)
  * [使用核心转储找出 NGINX 持续崩溃的原因](doc/RULES.md#beginner-use-core-dumps-to-figure-out-why-nginx-keep-crashing)
  * [使用 mirror 模块将请求复制到另一个后端](doc/RULES.md#beginner-use-mirror-module-to-copy-requests-to-another-backend)
- **[性能 (13)](doc/RULES.md#performance)**<a id="toc-performance"></a>
  * [调整工作进程数](doc/RULES.md#beginner-adjust-worker-processes)
  * [使用 HTTP/2](doc/RULES.md#beginner-use-http2)
  * [维护 SSL 会话](doc/RULES.md#beginner-maintaining-ssl-sessions)
  * [启用 OCSP Stapling](doc/RULES.md#beginner-enable-ocsp-stapling)
  * [尽可能在 server_name 指令中使用精确名称](doc/RULES.md#beginner-use-exact-names-in-a-server_name-directive-if-possible)
  * [避免使用 if 指令检查 server_name](doc/RULES.md#beginner-avoid-checks-server_name-with-if-directive)
  * [使用  避免使用正则表达式](doc/RULES.md#beginner-use-request_uri-to-avoid-using-regular-expressions)
  * [使用 try_files 指令确保文件存在](doc/RULES.md#beginner-use-try_files-directive-to-ensure-a-file-exists)
  * [使用 return 指令替代 rewrite 进行重定向](doc/RULES.md#beginner-use-return-directive-instead-of-rewrite-for-redirects)
  * [启用 PCRE JIT 加速正则表达式处理](doc/RULES.md#beginner-enable-pcre-jit-to-speed-up-processing-of-regular-expressions)
  * [激活到上游服务器的连接缓存](doc/RULES.md#beginner-activate-the-cache-for-connections-to-upstream-servers)
  * [使用精确 location 匹配加速选择过程](doc/RULES.md#beginner-make-an-exact-location-match-to-speed-up-the-selection-process)
  * [使用 limit_conn 改善下载速度限制](doc/RULES.md#beginner-use-limit_conn-to-improve-limiting-the-download-speed)
- **[加固 (31)](doc/RULES.md#hardening)**<a id="toc-hardening"></a>
  * [始终保持 NGINX 为最新版本](doc/RULES.md#beginner-always-keep-nginx-up-to-date)
  * [以非特权用户身份运行](doc/RULES.md#beginner-run-as-an-unprivileged-user)
  * [禁用不必要的模块](doc/RULES.md#beginner-disable-unnecessary-modules)
  * [保护敏感资源](doc/RULES.md#beginner-protect-sensitive-resources)
  * [注意 ACL 规则](doc/RULES.md#beginner-take-care-about-your-acl-rules)
  * [隐藏 Nginx 版本号](doc/RULES.md#beginner-hide-nginx-version-number)
  * [隐藏 Nginx 服务器签名](doc/RULES.md#beginner-hide-nginx-server-signature)
  * [隐藏上游代理头部](doc/RULES.md#beginner-hide-upstream-proxy-headers)
  * [移除对传统和有风险的 HTTP 请求头部的支持](doc/RULES.md#beginner-remove-support-for-legacy-and-risky-http-request-headers)
  * [仅使用最新支持的 OpenSSL 版本](doc/RULES.md#beginner-use-only-the-latest-supported-openssl-version)
  * [强制所有连接使用 TLS](doc/RULES.md#beginner-force-all-connections-over-tls)
  * [使用至少 2048 位 RSA 和 256 位 ECC](doc/RULES.md#beginner-use-min-2048-bit-for-rsa-and-256-bit-for-ecc)
  * [仅保留 TLS 1.3 和 TLS 1.2](doc/RULES.md#beginner-keep-only-tls-13-and-tls-12)
  * [仅使用强密码](doc/RULES.md#beginner-use-only-strong-ciphers)
  * [使用更安全的 ECDH 曲线](doc/RULES.md#beginner-use-more-secure-ecdh-curve)
  * [使用具有完美前向保密的强密钥交换](doc/RULES.md#beginner-use-strong-key-exchange-with-perfect-forward-secrecy)
  * [防止零往返时间的重放攻击](doc/RULES.md#beginner-prevent-replay-attacks-on-zero-round-trip-time)
  * [防御 BEAST 攻击](doc/RULES.md#beginner-defend-against-the-beast-attack)
  * [缓解 CRIME/BREACH 攻击](doc/RULES.md#beginner-mitigation-of-crimebreach-attacks)
  * [启用 HTTP 严格传输安全](doc/RULES.md#beginner-enable-http-strict-transport-security)
  * [降低 XSS 风险 (Content-Security-Policy)](doc/RULES.md#beginner-reduce-xss-risks-content-security-policy)
  * [控制 Referer 头部的行为 (Referrer-Policy)](doc/RULES.md#beginner-control-the-behaviour-of-the-referer-header-referrer-policy)
  * [提供点击劫持保护 (X-Frame-Options)](doc/RULES.md#beginner-provide-clickjacking-protection-x-frame-options)
  * [防止某些类别的 XSS 攻击 (X-XSS-Protection)](doc/RULES.md#beginner-prevent-some-categories-of-xss-attacks-x-xss-protection)
  * [防止嗅探 MIME 类型中间件 (X-Content-Type-Options)](doc/RULES.md#beginner-prevent-sniff-mimetype-middleware-x-content-type-options)
  * [拒绝使用浏览器功能 (Feature-Policy)](doc/RULES.md#beginner-deny-the-use-of-browser-features-feature-policy)
  * [拒绝不安全的 HTTP 方法](doc/RULES.md#beginner-reject-unsafe-http-methods)
  * [防止缓存敏感数据](doc/RULES.md#beginner-prevent-caching-of-sensitive-data)
  * [限制并发连接](doc/RULES.md#beginner-limit-concurrent-connections)
  * [控制缓冲区溢出攻击](doc/RULES.md#beginner-control-buffer-overflow-attacks)
  * [缓解慢速 HTTP DoS 攻击（关闭慢速连接）](doc/RULES.md#beginner-mitigating-slow-http-dos-attacks-closing-slow-connections)
- **[反向代理 (8)](doc/RULES.md#reverse-proxy)**<a id="toc-reverse-proxy"></a>
  * [使用与后端协议兼容的 pass 指令](doc/RULES.md#beginner-use-pass-directive-compatible-with-backend-protocol)
  * [注意 proxy_pass 指令中的尾部斜杠](doc/RULES.md#beginner-be-careful-with-trailing-slashes-in-proxy_pass-directive)
  * [仅使用 System.Management.Automation.Internal.Host.InternalHost 变量设置和传递 Host 头部](doc/RULES.md#beginner-set-and-pass-host-header-only-with-host-variable)
  * [正确设置 X-Forwarded-For 头部的值](doc/RULES.md#beginner-set-properly-values-of-the-x-forwarded-for-header)
  * [不要在反向代理后使用 X-Forwarded-Proto 与 ](doc/RULES.md#beginner-dont-use-x-forwarded-proto-with-scheme-behind-reverse-proxy)
  * [始终将 Host、X-Real-IP 和 X-Forwarded 头部传递到后端](doc/RULES.md#beginner-always-pass-host-x-real-ip-and-x-forwarded-headers-to-the-backend)
  * [使用不带 X- 前缀的自定义头部](doc/RULES.md#beginner-use-custom-headers-without-x--prefix)
  * [在 proxy_pass 中始终使用  替代 ](doc/RULES.md#beginner-always-use-request_uri-instead-of-uri-in-proxy_pass)
- **[负载均衡 (2)](doc/RULES.md#load-balancing)**<a id="toc-load-balancing"></a>
  * [调整被动健康检查](doc/RULES.md#beginner-tweak-passive-health-checks)
  * [不要通过注释禁用后端，使用 down 参数](doc/RULES.md#beginner-dont-disable-backends-by-comments-use-down-parameter)
- **[其他 (4)](doc/RULES.md#others)**<a id="toc-others"></a>
  * [正确设置证书链](doc/RULES.md#beginner-set-the-certificate-chain-correctly)
  * [启用 DNS CAA 策略](doc/RULES.md#beginner-enable-dns-caa-policy)
  * [使用 security.txt 定义安全策略](doc/RULES.md#beginner-define-security-policies-with-securitytxt)
  * [使用 tcpdump 诊断和排查 HTTP 问题](doc/RULES.md#beginner-use-tcpdump-to-monitor-http-traffic)
- **[配置示例](doc/EXAMPLES.md#configuration-examples)**<a id="toc-configuration-examples"></a>
  * [反向代理](doc/EXAMPLES.md#reverse-proxy)
    * [安装](doc/EXAMPLES.md#installation)
    * [配置](doc/EXAMPLES.md#configuration)
    * [导入配置](doc/EXAMPLES.md#import-configuration)
    * [设置绑定 IP 地址](doc/EXAMPLES.md#set-bind-ip-address)
    * [设置域名](doc/EXAMPLES.md#set-your-domain-name)
    * [重新生成私钥和证书](doc/EXAMPLES.md#regenerate-private-keys-and-certs)
    * [更新模块列表](doc/EXAMPLES.md#update-modules-list)
    * [生成必要的错误页面](doc/EXAMPLES.md#generating-the-necessary-error-pages)
    * [添加新域名](doc/EXAMPLES.md#add-new-domain)
    * [测试配置](doc/EXAMPLES.md#test-your-configuration)

</details>

<a id="introduction"></a>
# 介绍

<br>

<p align="center">
  <a href="https://www.nginx.com/">
    <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/nginx_admins_handbook_logo.png">
  </a>
</p>

<br>

  > 在开始使用 NGINX 之前，请先阅读官方的 **[入门指南](http://nginx.org/en/docs/beginners_guide.html)**。这对每个人来说都是一份很棒的介绍。

**Nginx**（_/ˌɛndʒɪnˈɛks/ EN-jin-EKS_，风格化为 NGINX 或 nginx）是一款开源 HTTP 和反向代理服务器、邮件代理服务器以及通用 TCP/UDP 代理服务器，重点关注高并发、高性能和低内存占用。它由 [Igor Sysoev](http://sysoev.ru/en/) 最初编写。

长期以来，它一直运行在许多高负载的俄罗斯网站上，包括 Yandex、Mail.Ru、VK 和 Rambler。目前，一些知名公司使用 NGINX，包括 Cisco、DuckDuckGo、Facebook、GitLab、Google、Twitter、Apple、Intel 等。在 2019 年 9 月，它是最常用的 HTTP 服务器（参见 [Netcraft 调查](https://news.netcraft.com/archives/category/web-server-survey/)）。

NGINX 是一款快速、轻量且功能强大的 Web 服务器，也可用作：

- 快速的 HTTP 反向代理
- 可靠的负载均衡器
- 高性能缓存服务器
- 功能齐全的 Web 平台

简而言之，它提供了完整 Web 堆栈的核心，旨在帮助构建可扩展的 Web 应用程序。在性能方面，NGINX 可以轻松处理大量流量。NGINX 的另一个主要优势是它允许你以不同的方式完成同一件事。

与传统 HTTP 服务器不同，NGINX 不依赖线程来处理请求，并且其架构设计理念与众不同——这种架构更适合并发连接数和每秒请求数的非线性扩展。

NGINX 也被称为 _Apache Killer_（主要是因为其轻量级和更低的 RAM 消耗）。它基于事件驱动，因此不遵循 Apache 为每个 Web 页面请求创建新进程或线程的风格。总的来说，它旨在解决 [C10K 问题](http://www.kegel.com/c10k.html)。

对我来说，它是我在系统管理员生涯中使用过的最好、最重要的服务之一。

----

以下这些重要文档应该是你的主要知识来源：

- **[入门指南](https://www.nginx.com/resources/wiki/start/)**
- **[NGINX 文档](https://nginx.org/en/docs/)**
- **[开发指南](http://nginx.org/en/docs/dev/development_guide.html)**
- **[安全控制](https://docs.nginx.com/nginx/admin-guide/security-controls/)**

此外，我想推荐三份专注于 HTTP 协议概念的好文档：

- **[HTTP Made Really Easy](https://www.jmarshall.com/easy/http/)**
- **[超文本传输协议规范](https://www.w3.org/Protocols/)**
- **[Web 技术开发者文档 - HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)**
如果你热爱安全，请关注这个：[Cryptology ePrint Archive](https://eprint.iacr.org/)。它提供了密码学最新研究的访问，并探讨了许多安全主题（例如，密码、算法、SSL/TLS 协议）。一篇涵盖密码学核心概念的优秀入门读物是 [Practical Cryptography for Developers](https://cryptobook.nakov.com/)。我还推荐阅读 [Bulletproof SSL and TLS](https://www.feistyduck.com/books/bulletproof-ssl-and-tls/)。是的，这绝对是我所读过的最全面的关于部署 TLS 的书籍。

另一个必不可少的知识来源是 [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)。你应该将其视为优秀的安全指南。[Burp Scanner - Issue Definitions](https://portswigger.net/kb/issues) 介绍了 Web 应用程序和安全漏洞。最后，[The Web Security Academy](https://portswigger.net/web-security) 是一个免费的 Web 应用安全在线培训中心，提供高质量的阅读材料和不同难度的互动实验室。所有这些都是开始学习 Web 应用安全的好资源。

当然，始终浏览官方的 [Nginx 安全公告](http://nginx.org/en/security_advisories.html) 和 CVE 数据库，如 [CVE Details](https://www.cvedetails.com/vendor/10048/Nginx.html) 或 [CVE - The MITRE Corporation](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=NGINX)——以保持对 NGINX 漏洞的最新了解。

<a id="prologue"></a>
## 序言

当我学习 HTTP 服务器架构时，我开始对 NGINX 感兴趣。在研究过程中，我一直在做笔记。我找到了大量相关信息，例如，关于各种问题的论坛帖子非常棒。然而，我从未找到一本能够以合适形式涵盖最重要内容的指南。我有点失望。

我对所有方面都感兴趣：NGINX 内部机制、函数、安全最佳实践、性能优化、技巧与窍门、黑客技巧和规则，但对我来说，一些文档对主题的处理过于肤浅。

当然，[NGINX 官方文档](https://nginx.org/en/docs/) 是最好的地方，但我知道我们还有其他很棒的资源：

- [agentzh's Nginx Tutorials](https://openresty.org/download/agentzh-nginx-tutorials-en.html)
- [Nginx Guts](http://www.nginxguts.com/)
- [Nginx discovery journey](http://www.nginx-discovery.com/)
- [Nginx Secure Web Server](https://calomel.org/nginx.html)
- [Emiller's Guide To Nginx Module Development](https://www.evanmiller.org/nginx-modules-guide.html)
- [Emiller's Advanced Topics In Nginx Module Development](https://www.evanmiller.org/nginx-modules-guide-advanced.html)

这些绝对是我们的最佳资源，你应该首先在那里寻求帮助。此外，为了提高你的知识，请参阅[书籍](#books)章节——其中包含关于 NGINX 的顶级文献。

<a id="why-i-created-this-handbook"></a>
## 为什么创建这本手册

然而，对我来说，一直缺少一份真正深入且足够简单的速查表，来描述 HTTP 服务器的各种配置和重要的跨领域主题。NGINX 的配置有时可能很棘手，你真的需要深入了解语法和概念才能理解技巧、漏洞和机制。该文档不如其他项目那么精美，并且肯定应该包含更丰富的示例。

  > 本手册是一套针对 NGINX 开源 HTTP 服务器的规则和建议。它还包含了最佳实践、笔记和带有无数示例的辅助工具。其中许多参考了外部资源。

你可以做很多事情来改进你的 NGINX 实例，本指南将尝试尽可能多地涵盖这些内容。在大多数情况下，它包含了对我来说关于 NGINX 最重要的内容。我认为你所提供的配置无需\"魔法\"就能正常工作。这就是我创建这个仓库的原因。

通过本手册，你将探索 NGINX 的许多特性和功能。例如，你将了解如何测试性能或如何解决调试问题。你将学习配置指南、安全设计模式、处理常见问题的方法以及如何避免这些问题。我在这里解释了一些避免陷阱和配置错误的最佳技巧。

我还添加了一套指南和示例，以帮助你管理 NGINX。它们也能让我们深入了解 NGINX 内部机制。

大多数情况下，我将这里介绍的规则应用于作为反向代理工作的 NGINX。但这并不妨碍它们被用于独立服务器模式的 NGINX。

<a id="who-this-handbook-is-for"></a>
## 这本手册的受众

如果你没有时间阅读数百篇文章（就像我一样），这本多功能手册可能对你有用。我创建它是希望它特别对系统管理员和基于 Web 的应用程序的专家有用。

本手册并未涵盖 NGINX 的所有方面。此外，本指南中描述的某些内容可能相当基础，因为我们大多数人并不是每天配置 NGINX，很容易忘记基本/琐碎的事情。另一方面，它也讨论了重量级话题，因此也为高级用户提供了一些内容。我尝试在本手册的许多地方放置外部资源，以消除可能存在的任何疑虑。

我尽力使本手册保持单一且一致（但现在我知道这真的很难）。它按照我认为逻辑合理的顺序组织。我认为它也可以很好地补充官方文档和其他优秀文档。这里描述的许多主题当然可以做得更好或不同。当然，我还有很多地方[需要改进和去做](#contributing--support)。我希望你喜欢并享受使用它的过程。

不要将本手册及其中的笔记视为天启知识。在阅读本文档时，你应该采取科学的方法。如果你有任何疑问或不同意我的观点，请指出我的错误。你应该通过提出问题、仔细收集和检查证据、以及查看所有可用信息是否能组合成逻辑答案来发现因果关系。

我创建这本手册还有一个原因。我不是从零开始，而是制定了一个计划来回答你的问题，帮助你找到做事情的最佳方式，并确保你不会重蹈我过去的覆辙。

那么，最重要的是：

- 对你观察到的事情提出问题
- 进行背景研究
- 通过实验进行测试
- 分析和得出结论
- 交流结果（为我们！）

最后，你应该知道我不是 NGINX 专家，但我喜欢了解事物是如何工作的以及它们为什么以这种方式工作。[我不是密码学专家……但我确实知道\"椭圆曲线\"这个词](https://twitter.com/ErikVoorhees/status/1004313761224757248)（我非常喜欢这句话！）。不需要成为专家就能找出原因，只需要知道为什么使用了这个而不是那个，或者为什么某件事以这种方式而不是另一种方式工作。理解你所热衷主题的推荐和细微差别，感觉很好。

<a id="before-you-start"></a>
## 开始之前

请记住以下最重要的事情：

  > **盲目部署此处描述的规则可能会破坏你的 Web 应用程序！**

  > **不要仅仅为了达到 100% 的某项目标而遵循指南。思考你实际在服务器上做了什么！**

  > **复制粘贴不是最好的学习方式。在采纳本手册的规则之前请三思。**

  > **没有对每个人都完美的设置。**

  > **始终思考什么对你更重要：安全性 vs 可用性/兼容性。**

  > **安全主要是指最小化风险。**

  > **改变一件事可能会引发一系列全新的问题。**

  > **阅读关于事物如何工作的内容，以及哪些值被认为是足够安全的（以及出于什么目的）。**

  > **唯一正确的方法是了解你的暴露面，进行测量和调整。**

`diff
+ 安全性出于道德原因很重要。合规性出于法律原因很重要。
+ 工作满意度的关键在于理解它们彼此无关。
+ 两者都很重要，但一个不能导致另一个（合规 != 安全）。
author: unknown

+ 无论是什么类型的网站，始终需要安全性。它可以是静态 HTML
+ 或完全动态的，攻击者仍然可以在传输过程中将恶意内容注入页面
+ 以攻击用户。
author: Scott Helme

+ 不要仅仅因为佛罗里达的 Karen 还在使用她 2001 年购买的电脑
+ 就启用旧的已弃用协议。
author: thisinterestsmeblog
`

我认为，在钓鱼攻击、网络攻击、勒索软件等盛行的时代，你应该尽可能严格地保护基础设施的安全，但永远不要忘记这一点……

<br>

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/crypto_nerds.png">
</p>

最后，我想引用在网络上发现的两条关于合规性以及人为因素在安全中的重要性的重要评论：

  > _有意义的法规通常不是描述性的——捕捉规则的意图和范围通常需要技术专长。更重要的是，这是大多数组织不具备的专业知识。而这些公司（可能构成行业的绝大多数）非但没有提升自己，反而向监管机构请愿，希望获得一份安全的、可勾选的技术缓解措施清单，以保持合规。[...]公司不是做正确的事情并实现计划的意图，而是盲目地、无意识地、脱离现实地勾选监管机构和审计员要求的无意义的选项框。_ - 来自 [bostik](https://news.ycombinator.com/user?id=bostik)

  > _在考虑安全性时，人为因素几乎总是与技术方面同等重要，甚至更为重要。政策和程序需要考虑人为因素，并努力确保这些政策和程序的结构能够帮助员工做正确的事情，即使他们可能不完全理解为什么需要这样做。_ - 来自 [Tim X](https://security.stackexchange.com/users/13958/tim-x)
<a id="contributing--support"></a>
## 贡献与支持

  > _然而，真正的社区只有在其成员以有意义的方式互动，加深彼此的理解并促进学习时才存在。_

如果你发现任何不合理或看起来不正确的地方，请提交 pull request，并请添加关于你的更改或评论的有效且合理的解释。

在提交 pull request 之前，请参阅 **[贡献指南](.github/CONTRIBUTING.md)**。

<a id="code-contributors"></a>
## 代码贡献者

本项目感谢所有贡献者。

<a href="https://github.com/trimstray/nginx-admins-handbook/graphs/contributors"><img src="https://opencollective.com/nginx-admins-handbook/contributors.svg?width=890&button=false"></a>

<a id="todo"></a>
### 待办事项

需要做什么？请看以下待办列表：

新章节：

- [x] **额外内容**
- [x] **HTTP 基础**
- [x] **SSL/TLS 基础**
- [x] **反向代理**
- [ ] **缓存**
- [x] **核心模块**
- [x] **第三方模块**
- [ ] **Web 应用防火墙**
- [ ] **ModSecurity**
- [x] **调试**

现有章节：

<details>
<summary><b>介绍</b></summary><br>

  - [x] _序言_
  - [x] _为什么创建这本手册_
  - [x] _这本手册的受众_
  - [x] _开始之前_
  - [x] _贡献与支持_
  - [x] _RSS 订阅与更新_
  - [x] _统领一切的清单_

</details>

<details>
<summary><b>额外内容</b></summary><br>

  - [x] _全自动安装_
  - [x] _静态错误页面生成器_
  - [x] _服务器名称解析器_

</details>

<details>
<summary><b>书籍</b></summary><br>

  - [x] _ModSecurity 3.0 and NGINX: Quick Start Guide_
  - [x] _Cisco ACE to NGINX: Migration Guide_

</details>

<details>
<summary><b>外部资源</b></summary><br>

  - _Nginx 官方_
    - [x] _Nginx 论坛_
    - [x] _Nginx 邮件列表_
    - [x] _NGINX-Demos_
  - _演示与视频_
    - [x] _NGINX: Basics and Best Practices_
    - [x] _NGINX Installation and Tuning_
    - [x] _Nginx Internals (by Joshua Zhu)_
    - [x] _Nginx internals (by Liqiang Xu)_
    - [x] _How to secure your web applications with NGINX_
    - [x] _Tuning TCP and NGINX on EC2_
    - [x] _Extending functionality in nginx, with modules!_
    - [x] _Nginx - Tips and Tricks._
    - [x] _Nginx Scripting - Extending Nginx Functionalities with Lua_
    - [x] _How to handle over 1,200,000 HTTPS Reqs/Min_
    - [x] _Using ngx_lua / lua-nginx-module in pixiv_
  - _速查表与参考_
    - [x] _Nginx configurations for most popular CMS/CMF/Frameworks based on PHP_
  - _性能与加固_
    - [x] _Memorable site for testing clients against bad SSL configs_
  - _配置解析器_
    - [x] _Quick and reliable way to convert NGINX configurations into JSON and back_
    - [x] _Parses nginx configuration with Pyparsing_
  - _配置管理器_
    - [x] _Ansible role to install and manage nginx configuration_
    - [x] _Ansible Role - Nginx_
    - [x] _Ansible role for NGINX_
    - [x] _Puppet Module to manage NGINX on various UNIXes_
  - _静态分析器_
    - [x] _nginx-minify-conf_
  - _对比评测_
    - [x] _NGINX vs. Apache (Pro/Con Review, Uses, & Hosting for Each)_
    - [x] _Web cache server performance benchmark: nuster vs nginx vs varnish vs squid_
  - _构建工具_
    - [x] _Nginx-builder_
  - _基准测试工具_
    - [x] _wrk2_
    - [x] _httperf_
    - [x] _slowloris_
    - [x] _slowhttptest_
    - [x] _GoldenEye_
  - _调试工具_
    - [x] _strace_
    - [x] _GDB_
    - [x] _SystemTap_
    - [x] _stapxx_
    - [x] _htrace.sh_
  - _安全与 Web 测试工具_
    - [x] _Burp Suite_
    - [x] _w3af_
    - [x] _nikto_
    - [x] _ssllabs-scan_
    - [x] _http-observatory_
    - [x] _testssl.sh_
    - [x] _sslyze_
    - [x] _cipherscan_
    - [x] _O-Saft_
    - [x] _Nghttp2_
    - [x] _h2spec_
    - [x] _http2fuzz_
    - [x] _Arjun_
    - [x] _Corsy_
    - [x] _XSStrike_
  - _在线与 Web 工具_
    - [x] _ssltools_
  - _其他资源_
    - [x] _OWASP Cheat Sheet Series_
    - [x] _Mozilla Web Security_
    - [x] _Application Security Wiki_
    - [x] _OWASP ASVS 4.0_
    - [x] _The System Design Primer_
    - [x] _awesome-scalability_
    - [x] _Web Architecture 101_

</details>

<details>
<summary><b>HTTP 基础</b></summary><br>

  - [x] _特性与架构_
  - [x] _HTTP/2_
    - [x] _如何调试 HTTP/2？_
  - [x] _HTTP/3_
  - [x] _URI vs URL_
  - [x] _连接 vs 请求_
  - [x] _HTTP 头部_
    - [x] _头部压缩_
  - [x] _HTTP 方法_
  - [x] _请求_
    - [x] _请求行_
      - [x] _方法_
      - [x] _请求 URI_
      - [x] _HTTP 版本_
    - [x] _请求头部字段_
    - [x] _消息体_
    - [x] _生成请求_
  - [x] _响应_
    - [x] _状态行_
      - [x] _HTTP 版本_
      - [x] _状态码与原因短语_
    - [x] _响应头部字段_
    - [x] _消息体_
  - [x] _HTTP 客户端_
    - [x] _IP 地址简写_
  - [x] _后端 Web 架构_
  - [x] _有用的视频资源_

</details>

<details>
<summary><b>SSL/TLS 基础</b></summary><br>

  - [x] _TLS 版本_
  - [x] _TLS 握手_
    - [x] _在 TCP/IP 协议栈中，TLS 位于哪一层？_
  - [x] _RSA 和 ECC 密钥/证书_
  - [x] _密码套件_
    - [x] _认证加密 (AEAD) 密码套件_
    - [x] _为什么密码套件很重要？_
    - [x] _NGINX 和 TLS 1.3 密码套件_
  - [x] _Diffie-Hellman 密钥交换_
  - [x] _证书_
    - [x] _信任链_
      - [x] _中间 CA 的主要目的是什么？_
    - [x] _单域名_
    - [x] _多域名_
    - [x] _通配符_
    - [x] _通配符 SSL 不处理根域名？_
  - [x] _TLS 服务器名称指示_
  - [x] _验证你的 SSL、TLS 和密码实现_
  - [x] _有用的视频资源_

</details>

<details>
<summary><b>NGINX 基础</b></summary><br>

  - _进程_
    - [x] _CPU 绑定_
    - [x] _工作进程的关闭_
  - _配置语法_
    - [x] _注释_
    - [x] _行尾_
    - [x] _变量、字符串和引号_
    - [x] _指令、块和上下文_
    - [x] _外部文件_
    - [x] _度量单位_
    - [x] _PCRE 正则表达式_
    - [x] _启用语法高亮_
  - _连接处理_
    - [x] _事件驱动架构_
    - [x] _多进程_
    - [x] _并发连接_
    - [x] _HTTP Keep-Alive 连接_
    - [x] _sendfile、tcp_nodelay 和 tcp_nopush_
  - _服务器块逻辑_
    - [x] _匹配 location_
      - [ ] _location 中的 if_
      - [ ] _嵌套 location_
    - [x] _rewrite vs return_
    - [x] _try_files 指令_
    - [x] _if、break 和 set_
    - [x] _root vs alias_
    - [x] _internal 指令_
    - [x] _外部和内部重定向_
    - [x] _allow 和 deny_
    - [x] _uri vs request_uri_
  - _压缩与解压_
    - [x] _NGINX 压缩 gzip 级别的最佳值是多少？_
  - _哈希表_
    - [x] _服务器名称哈希表_
  - _日志文件_
    - [x] _条件日志_
    - [x] _手动日志轮转_
    - [x] _NGINX upstream 变量返回 2 个值_
  - _反向代理_
    - [x] _传递请求_
    - [x] _尾部斜杠_
    - [ ] _处理头部_
    - [x] _传递头部_
      - [x] _Host 头部的重要性_
      - [x] _重定向和 X-Forwarded-Proto_
      - [x] _关于 X-Forwarded-For 的警告_
      - [x] _使用 Forwarded 提高可扩展性_
    - [x] _响应头部_
  - _负载均衡算法_
    - [x] _后端参数_
    - [x] _带 SSL 的上游服务器_
    - [x] _轮询 (Round Robin)_
    - [x] _加权轮询 (Weighted Round Robin)_
    - [x] _最少连接 (Least Connections)_
    - [x] _加权最少连接 (Weighted Least Connections)_
    - [x] _IP Hash_
    - [x] _通用 Hash_
    - [ ] _Fair 模块_
    - [x] _其他方法_
  - _速率限制_
    - [x] _变量_
    - [x] _指令、键和区域_
    - [x] _Burst 和 nodelay 参数_
  - _NAXSI Web 应用防火墙_
  - _OWASP ModSecurity 核心规则集 (CRS)_
  - _其他主题_
    - [ ] _使用 NGINX 安全分发 SSL 私钥_
  - _核心模块_
    - [x] _ngx_http_geo_module_
  - _第三方模块_
    - [x] _ngx_set_misc_
    - [x] _ngx_http_geoip_module_

</details>

<details>
<summary><b>辅助工具</b></summary><br>

  - _从源码安装_
    - [x] _在 RHEL/Debian/BSD 上自动安装_
    - [x] _编译器和链接器_
      - [x] _调试符号_
    - [x] _SystemTap_
      - [x] _stapxx_
    - [x] _分离和改进了安装方法_
    - [x] _在 CentOS 7 上安装 Nginx_
    - [x] _在 CentOS 7 上安装 OpenResty_
    - [x] _在 Ubuntu 18.04 上安装 Tengine_
    - [x] _在 FreeBSD 11.3 上安装 Nginx_
    - [x] _在 FreeBSD 11.3 上安装 Nginx（从 ports）_
  - _监控_
    - [ ] _CollectD、Prometheus 和 Grafana_
      - [ ] _nginx-vts-exporter_
    - [ ] _CollectD、InfluxDB 和 Grafana_
    - [ ] _Telegraf、InfluxDB 和 Grafana_
  - _测试_
    - [x] _构建 OpenSSL 1.0.2-chacha 版本_
    - [x] _发送请求并显示响应头部_
    - [x] _使用 http 方法、user-agent、跟随重定向发送请求并显示响应头部_
    - [x] _发送多个请求_
    - [x] _测试 SSL 连接_
    - [x] _测试 SSL 连接（调试模式）_
    - [x] _使用 SNI 支持测试 SSL 连接_
    - [x] _使用特定 SSL 版本测试 SSL 连接_
    - [x] _使用特定密码测试 SSL 连接_
    - [x] _验证 0-RTT_
    - [x] _测试 SCSV_
    - _使用 ApacheBench (ab) 进行负载测试_
      - [x] _标准测试_
      - [x] _使用 Keep-Alive 头部测试_
    - _使用 wrk2 进行负载测试_
      - [x] _标准场景_
      - [x] _POST 调用（使用 Lua）_
      - [x] _随机路径（使用 Lua）_
      - [x] _多路径（使用 Lua）_
      - [x] _每个线程的随机服务器地址（使用 Lua）_
      - [x] _多个 json 请求（使用 Lua）_
      - [x] _调试模式（使用 Lua）_
      - [x] _分析进出线程的数据_
      - [x] _解析 wrk 结果并生成报告_
    - _使用 locust 进行负载测试_
      - [x] _多路径_
      - [x] _不同用户会话的多路径_
    - [x] _TCP SYN 洪水拒绝服务攻击_
    - [x] _HTTP 拒绝服务攻击_
  - _调试_
    - [x] _显示进程信息_
    - [x] _检查内存使用_
    - [x] _显示打开的文件_
    - [x] _检查段错误消息_
    - [x] _转储配置_
    - [x] _获取 configure 参数列表_
    - [x] _检查模块是否已编译_
    - [x] _显示访问最多的 IP 地址（IP 和 URL）_
    - [x] _显示带有 HTTP 方法的最常请求的 URL_
    - [x] _显示最常访问的响应状态码_
    - [x] _计算带 IP 地址和 URL 的每秒请求数_
    - [x] _检查 gzip_static 模块是否工作_
    - [x] _哪个工作进程正在处理当前请求_
    - [x] _仅捕获 HTTP 数据包_
    - [x] _从 HTTP 数据包中提取 User Agent_
    - [x] _仅捕获 HTTP GET 和 POST 数据包_
    - [x] _按源 IP 和目标端口捕获请求并过滤_
    - [x] _实时捕获 HTTP 请求/响应，按 GET、HEAD 过滤并保存到文件_
    - [ ] _服务器端包含 (SSI) 调试_
    - [x] _转储进程的内存_
    - _GNU 调试器 (gdb)_
      - [x] _从运行进程转储配置_
      - [x] _显示内存中的调试日志_
      - [x] _核心转储回溯_
    - [x] _调试 socket 泄漏_
    - _SystemTap 速查表_
      - [x] _stapxx_
  - _错误与问题_
    - [ ] _常见错误_
  - _配置片段_
    - [x] _移除 Nginx 服务器头部_
    - [x] _自定义日志格式_
    - [x] _仅记录 4xx/5xx_
    - [x] _使用客户端证书限制访问_
    - [x] _按地理位置限制访问_
      - [x] _GeoIP 2 数据库_
    - [ ] _自定义错误页面_
    - [x] _使用 SSI 的动态错误页面_
    - [x] _使用 geo 和 map 按 IP 限制请求速率_
    - [x] _使用尾部斜杠_
    - [x] _正确将所有 HTTP 请求重定向到 HTTPS_
    - [x] _添加和移除 www 前缀_
    - [x] _代理/重写并保留原始 URL_
    - [x] _代理/重写并保留部分原始 URL_
    - [x] _代理/重写而不改变原始 URL（在浏览器中）_
    - [x] _修改 301/302 响应体_
    - [x] _将带负载的 POST 请求重定向到外部端点_
    - [x] _根据 HTTP 方法路由到不同的后端_
    - [ ] _将特定 IP 的用户重定向到特殊位置_
    - [x] _使用 CORS 头部允许多个跨域_
    - [x] _设置 X-Forwarded-Proto 中的正确 scheme_
    - [ ] _使用 Secure Link 模块保护 URL_
    - [ ] _高负载流量测试的技巧和方法（速查表）_
    - [ ] _Location 匹配示例_
    - [ ] _向后端传递请求_
      - [ ] _HTTP 后端服务器_
      - [ ] _uWSGI 后端服务器_
      - [ ] _FastCGI 后端服务器_
      - [ ] _memcached 后端服务器_
      - [ ] _Redis 后端服务器_
    - [ ] _到上游服务器的 HTTPS 流量_
    - [ ] _TCP 和 UDP 负载均衡_
    - [ ] _Lua 片段_
    - [ ] _nginscripts 片段_
  - _其他片段_
    - [x] _重新创建基础目录_
    - [x] _创建临时静态后端_
    - [x] _创建带 SSL 支持的临时静态后端_
    - [x] _使用 htpasswd 命令生成密码文件_
    - [x] _生成无口令的私钥_
    - [x] _生成带口令的私钥_
    - [x] _从私钥中移除口令_
    - [x] _用口令加密现有私钥_
    - [x] _生成 CSR_
    - [x] _生成 CSR（从现有证书获取元数据）_
    - [x] _使用 -config 参数生成 CSR_
    - [x] _生成私钥和 CSR_
    - [x] _列出可用的 EC 曲线_
    - [x] _生成 ECDSA 私钥_
    - [x] _生成私钥和 CSR (ECC)_
    - [x] _生成自签名证书_
    - [x] _从现有私钥生成自签名证书_
    - [x] _从现有私钥和 CSR 生成自签名证书_
    - [x] _生成多域名证书 (Certbot)_
    - [x] _生成通配符证书 (Certbot)_
    - [x] _生成 4096 位私钥的证书 (Certbot)_
    - [x] _生成 DH 公共参数_
    - [x] _显示 DH 公共参数_
    - [x] _从 p7b 提取证书_
    - [x] _将 DER 转换为 PEM_
    - [x] _将 PEM 转换为 DER_
    - [x] _验证证书的支持用途_
    - [x] _验证私钥_
    - [x] _检查私钥_
    - [x] _从私钥获取公钥_
    - [x] _验证公钥_
    - [x] _验证证书_
    - [x] _验证 CSR_
    - [x] _检查私钥和证书是否匹配_
    - [x] _TLSv1.3 和 CCM 密码_

</details>

<details>
<summary><b>基础规则</b></summary><br>

  - [x] _格式化、美化和缩进 Nginx 代码_
  - [x] _绝不在 listen 或 upstream 指令中使用主机名_
  - [x] _正确使用 add_header 和 proxy_*_header 指令设置 HTTP 头部_
  - [ ] _使重写绝对（带 scheme）_
  - [x] _使用 return 指令进行 URL 重定向 (301, 302)_
  - [x] _使用简单的自定义错误页面_
  - [x] _配置日志轮转策略_
  - [x] _不要重复 index 指令，仅在 http 块中使用_

</details>

<details>
<summary><b>调试</b></summary><br>

  - [x] _通过禁用守护进程、主进程和除一个之外的所有工作进程来改进调试_
  - [x] _使用核心转储找出 NGINX 持续崩溃的原因_
  - [x] _使用 mirror 模块将请求复制到另一个后端_
  - [ ] _使用 echo 模块进行动态调试_
  - [ ] _使用 SSI 进行动态调试_

</details>

<details>
<summary><b>性能</b></summary><br>

  - [x] _启用 OCSP Stapling_
  - [ ] _避免多个 index 指令_
  - [x] _使用  避免使用正则表达式_
  - [x] _使用 try_files 指令确保文件存在_
  - [ ] _不要将所有请求传递到后端 - 使用 try_files_
  - [x] _使用 return 指令替代 rewrite 进行重定向_
  - [x] _启用 PCRE JIT 加速正则表达式处理_
  - [ ] _为正常负载和高负载设置代理超时_
  - [ ] _为高负载流量配置内核参数_
  - [x] _激活到上游服务器的连接缓存_

</details>

<details>
<summary><b>加固</b></summary><br>

  - [x] _保持 NGINX 为最新版本_
  - [x] _注意 ACL 规则_
  - [x] _仅使用最新支持的 OpenSSL 版本_
  - [x] _移除对传统和有风险的 HTTP 请求头部的支持_
  - [x] _防止零往返时间的重放攻击_
  - [x] _防止缓存敏感数据_
  - [x] _限制并发连接_
  - [ ] _正确设置路径上的文件和目录权限（包括 ACL）_
  - [ ] _在 cookie 上实现 HTTPOnly 和安全属性_

</details>

<details>
<summary><b>反向代理</b></summary><br>

  - [x] _使用与后端协议兼容的 pass 指令_
  - [x] _注意 proxy_pass 指令中的尾部斜杠_
  - [x] _仅使用 System.Management.Automation.Internal.Host.InternalHost 变量设置和传递 Host 头部_
  - [x] _正确设置 X-Forwarded-For 头部的值_
  - [x] _不要在反向代理后使用 X-Forwarded-Proto 与 
  - [x] _始终将 Host、X-Real-IP 和 X-Forwarded 头部传递到后端_
  - [x] _使用不带 X- 前缀的自定义头部_
  - [x] _在 proxy_pass 中始终使用  替代 
  - [ ] _设置代理缓冲区和超时_

</details>

<details>
<summary><b>其他</b></summary><br>

  - [x] _正确设置证书链_
  - [x] _使用 security.txt 定义安全策略_
  - [x] _使用 tcpdump 诊断和排查 HTTP 问题_

</details>

如果你有任何想法，请反馈给我或提交 pull request。
<a id="rss-feed--updates"></a>
## RSS 订阅与更新

GitHub 公开了提交的 [RSS/Atom](https://github.com/trimstray/nginx-admins-handbook/commits.atom) 订阅源，如果你希望随时了解所有变更，这也可能很有用。

<a id="checklist-to-rule-them-all"></a>
## 统领一切的清单

这份清单是 _nginx-admins-handbook_ 的主要目标。它包含了一套关于如何正确配置和维护 NGINX 的最佳实践和建议。

  > 此清单包含本手册中的[所有规则 (79 条)](doc/RULES.md)。

总的来说，我认为这些原则每一条都很重要，应该予以考虑。我将它们分为四个优先级，以帮助指导你的决策。

| <b>优先级</b> | <b>名称</b> | <b>数量</b> | <b>描述</b> |
| :---:        | :---         | :---:        | :---         |
| ![高](static/img/priorities/high.png) | <i>关键</i> | 33 | 务必使用此规则，否则会给你的 NGINX 安全、性能等带来高风险 |
| ![中](static/img/priorities/medium.png) | <i>重要</i> | 26 | 也非常重要但不关键，应尽早处理 |
| ![低](static/img/priorities/low.png) | <i>正常</i> | 12 | 不需要立即实施，但值得考虑，因为它可以改善 NGINX 的工作和功能 |
| ![信息](static/img/priorities/info.png) | <i>次要</i> | 8 | 作为可选项实施或使用（非必需） |

请记住，这些只是指导原则。我的观点可能与你的不同，因此如果你觉得这些优先级不能反映你对安全、性能等的重视程度，你可以根据需要调整它们。

| <b>规则</b> | <b>章节</b> | <b>优先级</b> |
| :---         | :---         | :---:        |
| [使用 address:port 对定义 listen 指令](doc/RULES.md#beginner-define-the-listen-directives-with-addressport-pair)<br><sup>防止可能难以调试的软错误。</sup> | 基础规则 | ![高](static/img/priorities/high.png) |
| [防止处理未定义服务器名称的请求](doc/RULES.md#beginner-prevent-processing-requests-with-undefined-server-names)<br><sup>防止配置错误，例如流量转发到不正确的后端。</sup> | 基础规则 | ![高](static/img/priorities/high.png) |
| [绝不在 listen 或 upstream 指令中使用主机名](doc/RULES.md#beginner-never-use-a-hostname-in-a-listen-or-upstream-directives)<br><sup>虽然这可能有效，但会带来大量问题。</sup> | 基础规则 | ![高](static/img/priorities/high.png) |
| [正确使用 add_header 和 proxy_*_header 指令设置 HTTP 头部](doc/RULES.md#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)<br><sup>为所有上下文设置正确的安全头部。</sup> | 基础规则 | ![高](static/img/priorities/high.png) |
| [配置日志轮转策略](doc/RULES.md#beginner-configure-log-rotation-policy)<br><sup>避免 Web 服务器出现问题：配置适当的日志记录策略。</sup> | 基础规则 | ![高](static/img/priorities/high.png) |
| [使用简单的自定义错误页面](doc/RULES.md#beginner-use-simple-custom-error-pages)<br><sup>默认错误页面会泄露信息，导致信息泄露漏洞。</sup> | 基础规则 | ![高](static/img/priorities/high.png) |
| [使用 HTTP/2](doc/RULES.md#beginner-use-http2)<br><sup>HTTP/2 将使我们的应用程序更快、更简单、更健壮。</sup> | 性能 | ![高](static/img/priorities/high.png) |
| [始终保持 NGINX 为最新版本](doc/RULES.md#beginner-always-keep-nginx-up-to-date)<br><sup>使用最新的 NGINX 包来修复漏洞、错误并使用新特性。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [以非特权用户身份运行](doc/RULES.md#beginner-run-as-an-unprivileged-user)<br><sup>使用最小权限原则。这样只有主进程以 root 身份运行。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [保护敏感资源](doc/RULES.md#beginner-protect-sensitive-resources)<br><sup>隐藏的目录和文件永远不应可通过 Web 访问。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [注意 ACL 规则](doc/RULES.md#beginner-take-care-about-your-acl-rules)<br><sup>测试你的访问控制列表以保持安全。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [隐藏上游代理头部](doc/RULES.md#beginner-hide-upstream-proxy-headers)<br><sup>不要暴露服务器上运行的软件版本。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [移除对传统和有风险的 HTTP 请求头部的支持](doc/RULES.md#beginner-remove-support-for-legacy-and-risky-http-request-headers)<br><sup>应移除对有问题的头部的支持。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [强制所有连接使用 TLS](doc/RULES.md#beginner-force-all-connections-over-tls)<br><sup>保护你的网站以处理敏感通信。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [使用至少 2048 位 RSA 和 256 位 ECC](doc/RULES.md#beginner-use-min-2048-bit-for-rsa-and-256-bit-for-ecc)<br><sup>2048 位 (RSA) 或 256 位 (ECC) 密钥足以用于商业用途。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [仅保留 TLS 1.3 和 TLS 1.2](doc/RULES.md#beginner-keep-only-tls-13-and-tls-12)<br><sup>使用具有现代加密算法且无协议弱点的 TLS。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [仅使用强密码](doc/RULES.md#beginner-use-only-strong-ciphers)<br><sup>仅使用强且不脆弱的密码套件。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [使用更安全的 ECDH 曲线](doc/RULES.md#beginner-use-more-secure-ecdh-curve)<br><sup>根据 NIST 建议使用 ECDH 曲线。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [使用具有完美前向保密的强密钥交换](doc/RULES.md#beginner-use-strong-key-exchange-with-perfect-forward-secrecy)<br><sup>在双方之间建立可用于秘密通信的共享密钥。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [防御 BEAST 攻击](doc/RULES.md#beginner-defend-against-the-beast-attack)<br><sup>应优先选择服务器密码而不是客户端密码。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [启用 HTTP 严格传输安全](doc/RULES.md#beginner-enable-http-strict-transport-security)<br><sup>告诉浏览器只能使用 HTTPS 访问，而不是 HTTP。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [降低 XSS 风险 (Content-Security-Policy)](doc/RULES.md#beginner-reduce-xss-risks-content-security-policy)<br><sup>CSP 最适合用作纵深防御。它减少了恶意注入可能造成的危害。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [控制 Referer 头部的行为 (Referrer-Policy)](doc/RULES.md#beginner-control-the-behaviour-of-the-referer-header-referrer-policy)<br><sup>引荐者泄露的默认行为使网站面临隐私和安全漏洞的风险。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [提供点击劫持保护 (X-Frame-Options)](doc/RULES.md#beginner-provide-clickjacking-protection-x-frame-options)<br><sup>防御点击劫持攻击。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [防止某些类别的 XSS 攻击 (X-XSS-Protection)](doc/RULES.md#beginner-prevent-some-categories-of-xss-attacks-x-xss-protection)<br><sup>如果检测到潜在的 XSS 反射攻击，阻止渲染页面。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [防止嗅探 MIME 类型中间件 (X-Content-Type-Options)](doc/RULES.md#beginner-prevent-sniff-mimetype-middleware-x-content-type-options)<br><sup>告诉浏览器不要嗅探 MIME 类型。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [拒绝不安全的 HTTP 方法](doc/RULES.md#beginner-reject-unsafe-http-methods)<br><sup>只允许你实际提供服务的 HTTP 方法。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [防止缓存敏感数据](doc/RULES.md#beginner-prevent-caching-of-sensitive-data)<br><sup>帮助防止关键数据（例如信用卡详细信息或用户名）泄露。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [限制并发连接](doc/RULES.md#beginner-limit-concurrent-connections)<br><sup>限制并发连接，防止恶意用户反复连接并独占 NGINX。</sup> | 加固 | ![高](static/img/priorities/high.png) |
| [使用与后端协议兼容的 pass 指令](doc/RULES.md#beginner-use-pass-directive-compatible-with-backend-protocol)<br><sup>仅设置与兼容后端层协议一起使用的 pass 指令。</sup> | 反向代理 | ![高](static/img/priorities/high.png) |
| [正确设置 X-Forwarded-For 头部的值](doc/RULES.md#beginner-set-properly-values-of-the-x-forwarded-for-header)<br><sup>识别与位于代理后面的服务器通信的客户端。</sup> | 反向代理 | ![高](static/img/priorities/high.png) |
| [不要在反向代理后使用 X-Forwarded-Proto 与 ](doc/RULES.md#beginner-dont-use-x-forwarded-proto-with-scheme-behind-reverse-proxy)<br><sup>防止传递此头部的不正确值。</sup> | 反向代理 | ![高](static/img/priorities/high.png) |
| [在 proxy_pass 中始终使用  替代 ](doc/RULES.md#beginner-always-use-request_uri-instead-of-uri-in-proxy_pass)<br><sup>应始终将未更改的 URI 传递到后端层。</sup> | 反向代理 | ![高](static/img/priorities/high.png) |
| [组织 Nginx 配置](doc/RULES.md#beginner-organising-nginx-configuration)<br><sup>组织良好的代码更容易理解和维护。</sup> | 基础规则 | ![中](static/img/priorities/medium.png) |
| [格式化、美化和缩进 Nginx 代码](doc/RULES.md#beginner-format-prettify-and-indent-your-nginx-code)<br><sup>格式化代码更易于维护、调试，并且可以在短时间内阅读和理解。</sup> | 基础规则 | ![中](static/img/priorities/medium.png) |
| [使用 reload 选项动态更改配置](doc/RULES.md#beginner-use-reload-option-to-change-configurations-on-the-fly)<br><sup>优雅地重新加载配置，无需停止服务器且不丢失任何数据包。</sup> | 基础规则 | ![中](static/img/priorities/medium.png) |
| [使用 return 指令进行 URL 重定向 (301, 302)](doc/RULES.md#beginner-use-return-directive-for-url-redirection-301-302)<br><sup>迄今为止最简单和最快的方法，因为不需要评估正则表达式。</sup> | 基础规则 | ![中](static/img/priorities/medium.png) |
| [维护 SSL 会话](doc/RULES.md#beginner-maintaining-ssl-sessions)<br><sup>从客户端的角度提高性能。</sup> | 性能 | ![中](static/img/priorities/medium.png) |
| [启用 OCSP Stapling](doc/RULES.md#beginner-enable-ocsp-stapling)<br><sup>启用以减少 OCSP 验证的成本。</sup> | 性能 | ![中](static/img/priorities/medium.png) |
| [尽可能在 server_name 指令中使用精确名称](doc/RULES.md#beginner-use-exact-names-in-a-server_name-directive-if-possible)<br><sup>帮助加速使用精确名称的搜索。</sup> | 性能 | ![中](static/img/priorities/medium.png) |
| [避免使用 if 指令检查 server_name](doc/RULES.md#beginner-avoid-checks-server_name-with-if-directive)<br><sup>降低 NGINX 处理需求。</sup> | 性能 | ![中](static/img/priorities/medium.png) |
| [使用  避免使用正则表达式](doc/RULES.md#beginner-use-request_uri-to-avoid-using-regular-expressions)<br><sup>默认情况下，正则会消耗资源并降低性能。</sup> | 性能 | ![中](static/img/priorities/medium.png) |
| [使用 try_files 指令确保文件存在](doc/RULES.md#beginner-use-try_files-directive-to-ensure-a-file-exists)<br><sup>如果你需要搜索文件时使用它，同时也能避免重复代码。</sup> | 性能 | ![中](static/img/priorities/medium.png) |
| [使用 return 指令替代 rewrite 进行重定向](doc/RULES.md#beginner-use-return-directive-instead-of-rewrite-for-redirects)<br><sup>使用 return 指令比 rewrite 响应速度更快。</sup> | 性能 | ![中](static/img/priorities/medium.png) |
| [启用 PCRE JIT 加速正则表达式处理](doc/RULES.md#beginner-enable-pcre-jit-to-speed-up-processing-of-regular-expressions)<br><sup>带 PCRE JIT 的 NGINX 比不带快得多。</sup> | 性能 | ![中](static/img/priorities/medium.png) |
| [激活到上游服务器的连接缓存](doc/RULES.md#beginner-activate-the-cache-for-connections-to-upstream-servers)<br><sup>Nginx 现在可以重用其每个上游的现有连接 (keepalive)。</sup> | 性能 | ![中](static/img/priorities/medium.png) |
| [禁用不必要的模块](doc/RULES.md#beginner-disable-unnecessary-modules)<br><sup>限制漏洞，提高性能和内存效率。</sup> | 加固 | ![中](static/img/priorities/medium.png) |
| [隐藏 Nginx 版本号](doc/RULES.md#beginner-hide-nginx-version-number)<br><sup>不要泄露关于 NGINX 的敏感信息。</sup> | 加固 | ![中](static/img/priorities/medium.png) |
| [隐藏 Nginx 服务器签名](doc/RULES.md#beginner-hide-nginx-server-signature)<br><sup>不要泄露关于 NGINX 的敏感信息。</sup> | 加固 | ![中](static/img/priorities/medium.png) |
| [仅使用最新支持的 OpenSSL 版本](doc/RULES.md#beginner-use-only-the-latest-supported-openssl-version)<br><sup>保持免受 SSL 安全威胁的影响，并且不错过新功能。</sup> | 加固 | ![中](static/img/priorities/medium.png) |
| [防止零往返时间的重放攻击](doc/RULES.md#beginner-prevent-replay-attacks-on-zero-round-trip-time)<br><sup>0-RTT 默认禁用，但你应该知道启用此选项会带来重大安全风险。</sup> | 加固 | ![中](static/img/priorities/medium.png) |
| [缓解 CRIME/BREACH 攻击](doc/RULES.md#beginner-mitigation-of-crimebreach-attacks)<br><sup>禁用 HTTP 压缩或仅压缩零敏感内容。</sup> | 加固 | ![中](static/img/priorities/medium.png) |
| [拒绝使用浏览器功能 (Feature-Policy)](doc/RULES.md#beginner-deny-the-use-of-browser-features-feature-policy)<br><sup>一种允许和拒绝使用浏览器功能的机制。</sup> | 加固 | ![中](static/img/priorities/medium.png) |
| [控制缓冲区溢出攻击](doc/RULES.md#beginner-control-buffer-overflow-attacks)<br><sup>防止以覆盖 NGINX 进程内存片段为特征的错误。</sup> | 加固 | ![中](static/img/priorities/medium.png) |
| [缓解慢速 HTTP DoS 攻击（关闭慢速连接）](doc/RULES.md#beginner-mitigating-slow-http-dos-attack-closing-slow-connections)<br><sup>防止攻击者缓慢地发送 HTTP 请求片段的攻击。</sup> | 加固 | ![中](static/img/priorities/medium.png) |
| [仅使用 System.Management.Automation.Internal.Host.InternalHost 变量设置和传递 Host 头部](doc/RULES.md#beginner-set-and-pass-host-header-only-with-host-variable)<br><sup>使用 System.Management.Automation.Internal.Host.InternalHost 是唯一保证具有合理值的变量。</sup> | 反向代理 | ![中](static/img/priorities/medium.png) |
| [始终将 Host、X-Real-IP 和 X-Forwarded 头部传递到后端](doc/RULES.md#beginner-always-pass-host-x-real-ip-and-x-forwarded-headers-to-the-backend)<br><sup>让你对转发的头部有更多控制。</sup> | 反向代理 | ![中](static/img/priorities/medium.png) |
| [正确设置证书链](doc/RULES.md#beginner-set-the-certificate-chain-correctly)<br><sup>向客户端发送完整的证书链。</sup> | 其他 | ![中](static/img/priorities/medium.png) |
| [启用 DNS CAA 策略](doc/RULES.md#beginner-enable-dns-caa-policy)<br><sup>允许域名持有者向 CA 指示他们是否有权颁发数字证书。</sup> | 其他 | ![中](static/img/priorities/medium.png) |
| [为 80 和 443 端口分别使用 listen 指令](doc/RULES.md#beginner-separate-listen-directives-for-80-and-443-ports)<br><sup>帮助你维护和修改配置。</sup> | 基础规则 | ![低](static/img/priorities/low.png) |
| [为 listen 指令仅使用一个 SSL 配置](doc/RULES.md#beginner-use-only-one-ssl-config-for-the-listen-directive)<br><sup>防止在同一监听地址上出现多个配置。</sup> | 基础规则 | ![低](static/img/priorities/low.png) |
| [使用 geo/map 模块替代 allow/deny](doc/RULES.md#beginner-use-geomap-modules-instead-of-allowdeny)<br><sup>提供了阻止无效访问者的完美方法。</sup> | 基础规则 | ![低](static/img/priorities/low.png) |
| [为未匹配的 location 设置全局根目录](doc/RULES.md#beginner-set-global-root-directory-for-unmatched-locations)<br><sup>为未定义的 location 指定根目录。</sup> | 基础规则 | ![低](static/img/priorities/low.png) |
| [不要重复 index 指令，仅在 http 块中使用](doc/RULES.md#beginner-dont-duplicate-index-directive-use-it-only-in-the-http-block)<br><sup>注意重复相同的规则。</sup> | 基础规则 | ![低](static/img/priorities/low.png) |
| [调整工作进程数](doc/RULES.md#beginner-adjust-worker-processes)<br><sup>你可以调整此值以在高并发下获得最大吞吐量。</sup> | 性能 | ![低](static/img/priorities/low.png) |
| [使用精确 location 匹配加速选择过程](doc/RULES.md#beginner-make-an-exact-location-match-to-speed-up-the-selection-process)<br><sup>精确 location 匹配通常用于加速选择过程。</sup> | 性能 | ![低](static/img/priorities/low.png) |
| [使用 limit_conn 改善下载速度限制](doc/RULES.md#beginner-use-limit_conn-to-improve-limiting-the-download-speed)<br><sup>限制每个连接的 NGINX 下载速度。</sup> | 性能 | ![低](static/img/priorities/low.png) |
| [注意 proxy_pass 指令中的尾部斜杠](doc/RULES.md#beginner-be-careful-with-trailing-slashes-in-proxy_pass-directive)<br><sup>不正确的设置可能会导致奇怪的 URL。</sup> | 反向代理 | ![低](static/img/priorities/low.png) |
| [使用不带 X- 前缀的自定义头部](doc/RULES.md#beginner-use-custom-headers-without-x--prefix)<br><sup>不鼓励使用带 X- 前缀的自定义头部。</sup> | 反向代理 | ![低](static/img/priorities/low.png) |
| [调整被动健康检查](doc/RULES.md#beginner-tweak-passive-health-checks)<br><sup>改进被动健康检查的行为。</sup> | 负载均衡 | ![低](static/img/priorities/low.png) |
| [使用 security.txt 定义安全策略](doc/RULES.md#beginner-define-security-policies-with-securitytxt)<br><sup>帮助使公司和安全研究人员的工作更轻松。</sup> | 其他 | ![低](static/img/priorities/low.png) |
| [映射万物...](doc/RULES.md#beginner-map-all-the-things)<br><sup>Map 模块为清晰地解析大量正则表达式列表提供了更优雅的解决方案。</sup> | 基础规则 | ![信息](static/img/priorities/info.png) |
| [使用自定义日志格式](doc/RULES.md#beginner-use-custom-log-formats)<br><sup>这对于调试特定的 location 指令非常有帮助。</sup> | 调试 | ![信息](static/img/priorities/info.png) |
| [使用调试模式追踪意外行为](doc/RULES.md#beginner-use-debug-mode-to-track-down-unexpected-behaviour)<br><sup>可能包含比你想要的更多细节，但这有时可以救命。</sup> | 调试 | ![信息](static/img/priorities/info.png) |
| [通过禁用守护进程、主进程和除一个之外的所有工作进程来改进调试](doc/RULES.md#beginner-improve-debugging-by-disable-daemon-master-process-and-all-workers-except-one)<br><sup>这简化了调试，并允许快速测试配置。</sup> | 调试 | ![信息](static/img/priorities/info.png) |
| [使用核心转储找出 NGINX 持续崩溃的原因](doc/RULES.md#beginner-use-core-dumps-to-figure-out-why-nginx-keep-crashing)<br><sup>当 NGINX 实例遇到意外错误或崩溃时，启用核心转储。</sup> | 调试 | ![信息](static/img/priorities/info.png) |
| [使用 mirror 模块将请求复制到另一个后端](doc/RULES.md#beginner-use-mirror-module-to-copy-requests-to-another-backend)<br><sup>使用镜像来调查和调试任何原始请求。</sup> | 调试 | ![信息](static/img/priorities/info.png) |
| [不要通过注释禁用后端，使用 down 参数](doc/RULES.md#beginner-dont-disable-backends-by-comments-use-down-parameter)<br><sup>这是将服务器标记为永久不可用的好方法。</sup> | 负载均衡 | ![信息](static/img/priorities/info.png) |
| [使用 tcpdump 诊断和排查 HTTP 问题](doc/RULES.md#beginner-use-tcpdump-to-diagnose-and-troubleshoot-the-http-issues)<br><sup>使用 tcpdump 监控 HTTP。</sup> | 其他 | ![信息](static/img/priorities/info.png) |
<a id="bonus-stuff"></a>
# 额外内容

你可以在这里找到一些我创作并添加到这个仓库中的不同内容。希望这些额外内容对你有用。

<a id="configuration-reports"></a>
## 配置报告

这些配方中的许多已应用于我旧的私人网站的配置。

  > [配置示例](#configuration-examples)章节中有一个示例配置。它也基于[这个](https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/cheatsheets/nginx-hardening-cheatsheet-tls13.png)版本的可打印高分辨率加固速查表。

<a id="ssl-labs"></a>
### SSL Labs

  > 请阅读 [这里](https://community.qualys.com/docs/DOC-6321-ssl-labs-grading-2018) 关于 SSL Labs 评分的内容（SSL Labs Grading 2018）。

简短的 SSL Labs 等级说明：

  > _A+ 显然是理想的等级，A 和 B 等级都是可接受的，并且能提供足够的商业安全性。特别是 B 等级，可能适用于旨在支持非常广泛受众（针对旧客户端）的配置。_

我最终获得了 **A+** 等级和以下分数：

- 证书 = **100%**
- 协议支持 = **100%**
- 密钥交换 = **90%**
- 密码强度 = **90%**

也请查看以下建议。我相信正确的 NGINX 配置应该获得以下 SSL Labs 分数，并为大多数情况提供最佳安全性：

- **推荐**

  - A/A+
  - 证书：100/100
  - 协议支持：95/100
  - 密钥交换：90/100
  - 密码强度：90/100

- **完美但严格**

  - A+
  - 证书：100/100
  - 协议支持：100/100
  - 密钥交换：100/100
  - 密码强度：100/100

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/blkcipher_ssllabs_preview.png" alt="blkcipher_ssllabs_preview">
</p>

关于 SSL Labs 评分机制的一些看法（这是一个有趣的观点）：

  > _整个评分机制更多的是宣传和公关，而不是实际安全性。如果你想要良好的安全性，那么你必须关注细节并理解事物内部的工作原理。如果你想要好成绩，那么你应该不惜一切代价获得好成绩。来自 SSL Labs 的 \"A+\" 是一个很酷的东西，可以添加到报告的末尾，但它并不真正等同于拥有坚如磐石的安全性。获得 \"A+\" 等同于能够说\"我有 A+\"。_ - 来自 [Tom Leek](https://security.stackexchange.com/users/5411/tom-leek) 在 [此](https://security.stackexchange.com/a/112539) 的回答。

<a id="mozilla-observatory"></a>
### Mozilla Observatory

  > 请阅读 [这里](https://observatory.mozilla.org/faq/) 关于 Mozilla Observatory 的内容，以及 [这里](https://github.com/mozilla/http-observatory/blob/master/httpobs/docs/scoring.md) 关于 Observatory 评分方法的内容。

我还在 Observatory 上获得了最高的总结评分 (**A+**)，测试分数非常高（120/100，最高 135/100）：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/blkcipher_mozilla_observatory_preview.png" alt="blkcipher_mozilla_observatory_preview">
</p>

<a id="printable-hardening-cheatsheets"></a>
## 可打印的加固速查表

我创建了两个版本的可打印海报，包含基于本手册配方的加固速查表（高分辨率 5000x8800）：

  > 有关 xcf 和 pdf 格式，请参见 [此](https://github.com/trimstray/nginx-admins-handbook/tree/master/static/img) 目录。

- **A+**，在 @ssllabs 上获得所有 **100%**，在 @mozilla observatory 上获得 **120/100**：

  > 它提供了 SSL Labs 测试的最高分数。配置非常严格，使用 4096 位私钥，仅 TLS 1.2，以及现代严格的 TLS 密码套件（非 128 位）。请仔细考虑其用途（无 TLS 1.3，严格的密码套件），在我看来，它仅适用于获得最高可能评分，并且似乎有些不切实际。

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/cheatsheets/nginx-hardening-cheatsheet-tls12-100p.png" alt="nginx-hardening-cheatsheet-100p" width="92%" height="92%">
</p>

- **A+**，在 @ssllabs 上获得 **120/100**，在 @mozilla observatory 上获得 **120/100**，支持 TLS 1.3：

  > 它提供了不那么严格的设置，使用 2048 位 RSA 密钥或 256 位 ECC 密钥，TLS 1.3 和 1.2，现代严格的 TLS 密码套件（128/256 位），以及 Mozilla 推荐的 2048 位预定义 DH 组。最终评分也符合行业标准和指南。建议使用这个，对我来说，这是一个非常合理的配置。

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/cheatsheets/nginx-hardening-cheatsheet-tls13.png" alt="nginx-hardening-cheatsheet-tls13" width="92%" height="92%">
</p>

<a id="fully-automatic-installation"></a>
## 全自动安装

我创建了一组脚本，用于从原始未编译代码无人值守安装 NGINX。它允许你轻松安装、创建依赖项（如 zlib 或 openssl）的设置，并使用安装参数进行自定义。

更多信息请参见 [从源码安装 - 自动安装](https://github.com/trimstray/nginx-admins-handbook/tree/master/lib) 章节，其中描述了在 Ubuntu、Debian、CentOS 和 FreeBSD 等系统/发行版上安装 NGINX。

<a id="static-error-pages-generator"></a>
## 静态错误页面生成器

我创建了一个易于使用的静态页面生成器，用于替换 NGINX 等 Web 服务器自带的默认错误页面。

更多信息请参见 [HTTP 静态错误页面生成器](https://github.com/trimstray/nginx-admins-handbook/tree/master/lib/nginx/snippets/http-error-pages#http-static-error-pages-generator)。

<a id="server-names-parser"></a>
## 服务器名称解析器

我添加了用于在配置中快速搜索多个域名的脚本。这些工具获取特定的 server_name 匹配项，并将其作为 server { ... } 块打印在屏幕上。如果你确实有大量域名，或者想要从文件或活动配置中列出特定的虚拟主机，这两个工具都非常有用。

你必须遵循一个重要规则才能使用它。你的服务器块必须具有以下结构：

`
ginx
server {

  server_name example.com example.org;

  ... # 其他指令

}
`

使用示例：

`
./snippets/server-name-parser/check-server-name.sh example.com
Searching 'example.com' in '/usr/local/etc/nginx' (from disk)

/usr/local/etc/nginx/domains/example.com/servers.conf:79: return 301 https://example.com;
/usr/local/etc/nginx/domains/example.com/servers.conf:252: return 301 https://example.com;
/usr/local/etc/nginx/domains/example.com/servers.conf:3825: server_name example.com;

Searching 'example.com' in server contexts (from a running process)

>>>>>>>> BEG >>>>>>>>>>
server {

  include listen/192.168.252.10/https.example.com.conf;

  server_name example.com;

  location / {

    return 204 "RFC 792";

  }

  access_log /var/log/nginx/example.com/access.log standard;
  error_log /var/log/nginx/example.com/error.log warn;

}
<<<<<<<<<< END <<<<<<<<<<
`

更多信息请参见 [snippets/server-name-parser](https://github.com/trimstray/nginx-admins-handbook/tree/master/lib/nginx/snippets/server-name-parser) 目录。
<a id="books"></a>
# 书籍

#### [Nginx Essentials](https://www.amazon.com/Nginx-Essentials-Valery-Kholodkov/dp/1785289535)

作者：**Valery Kholodkov**

_通过学习如何在实际应用中使用其最重要的功能，快速精通 Nginx。_

- _学习如何设置、配置和操作 Nginx 安装以便日常使用_
- _探索 Nginx 的广泛功能，像专业人士一样管理它，并成功使用它来运行你的网站_
- _基于示例的指南，充分发挥 Nginx 的优势，减少资源占用_

<sup><i>此简短介绍来自本书或商店。</i></sup>

#### [Nginx Cookbook](https://www.oreilly.com/library/view/nginx-cookbook/9781492049098/)

作者：**Derek DeJonghe**

_你将找到以下配方：_

- _流量管理和 A/B 测试_
- _使用动态模板和 NGINX Plus API 管理可编程性和自动化_
- _通过加密流量、安全链接、HTTP 认证子请求等保障访问安全_
- _将 NGINX 部署到 AWS、Azure 和 Google 云计算服务_
- _使用 Docker 部署容器和微服务_
- _调试和故障排除、性能调优以及实用操作技巧_

<sup><i>此简短介绍来自本书或商店。</i></sup>

#### [Nginx HTTP Server](https://www.amazon.com/Nginx-HTTP-Server-Harness-infrastructure/dp/178862355X)

作者：**Martin Fjordvald**、**Clement Nedelcu**

_充分利用 Nginx 的力量，最大化你的基础设施效能，以前所未有的速度提供页面服务。_

- _发现 Nginx 与 Apache 之间的可能交互，以充分利用两者的优势_
- _学习利用 Nginx 提供的功能来增强你的 Web 应用程序_
- _获取最新版本的 Nginx (1.13.2) 以满足所有 Web 管理需求_

<sup><i>此简短介绍来自本书或商店。</i></sup>

#### [Nginx High Performance](https://www.amazon.com/Nginx-High-Performance-Rahul-Sharma/dp/1785281836)

作者：**Rahul Sharma**

_优化 NGINX 以实现高性能、可扩展的 Web 应用程序。_

- _通过配置示例和解释，为 Nginx 配置最佳性能_
- _使用开源软件进行性能测试的分步教程_
- _调整 TCP 堆栈以充分利用可用基础设施_

<sup><i>此简短介绍来自本书或商店。</i></sup>

#### [Mastering Nginx](https://www.amazon.com/Mastering-Nginx-Dimitri-Aivaliotis/dp/1849517444)

作者：**Dimitri Aivaliotis**

_本书面向经验丰富的系统管理员和工程师，从头开始教你如何为任何情况配置 Nginx。分步说明和真实代码片段即使是最复杂的领域也能清晰阐述。_

<sup><i>此简短介绍来自本书或商店。</i></sup>

#### [ModSecurity 3.0 and NGINX: Quick Start Guide](https://www.nginx.com/resources/library/modsecurity-3-nginx-quick-start-guide/)

作者：**Faisal Memon**、**Owen Garrett**、**Michael Pleshakov**

_在这本电子书中学习如何开始使用 ModSecurity——全球部署最广泛的 Web 应用防火墙 (WAF)，现在可用于 NGINX 和 NGINX Plus。_

<sup><i>此简短介绍来自本书或商店。</i></sup>

#### [Cisco ACE to NGINX: Migration Guide](https://www.nginx.com/resources/library/cisco-ace-nginx-migration-guide/)

作者：**Faisal Memon**

_这本电子书提供了用 NGINX 和现成服务器替换 Cisco ACE 的分步说明。NGINX 帮助你降低成本并实现现代化。_

_在这本电子书中，你将学习：_

- _如何将 Cisco ACE 配置迁移到 NGINX，附有详细示例_
- _为什么应该选择软件负载均衡器，而不是硬件_

<sup><i>此简短介绍来自本书或商店。</i></sup>

<a id="external-resources"></a>
# 外部资源

<a id="nginx-official"></a>
##### Nginx 官方

<p>
&nbsp;&nbsp;:black_small_square: <a href="https://www.nginx.com/"><b>Nginx Project</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://nginx.org/en/docs/"><b>Nginx Documentation</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.nginx.com/resources/wiki/"><b>Nginx Wiki</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://docs.nginx.com/nginx/admin-guide/"><b>Nginx Admin's Guide</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/"><b>Nginx Pitfalls and Common Mistakes</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="http://nginx.org/en/docs/dev/development_guide.html"><b>Development Guide</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://forum.nginx.org/"><b>Nginx Forum</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="http://nginx.org/en/security_advisories.html"><b>Nginx Security Advisories</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://docs.nginx.com/nginx/admin-guide/security-controls/"><b>Nginx Security Controls</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://mailman.nginx.org/mailman/listinfo/nginx"><b>Nginx Mailing List</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/nginx/nginx"><b>Nginx Read-only Mirror</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/nginxinc/NGINX-Demos"><b>NGINX-Demos
</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.nginx.com/blog/thread-pools-boost-performance-9x/"><b>Thread Pools in NGINX Boost Performance 9x!</b></a><br>
</p>

<a id="nginx-distributions"></a>
##### Nginx 发行版

<p>
&nbsp;&nbsp;:black_small_square: <a href="https://openresty.org/"><b>OpenResty</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://tengine.taobao.org/"><b>The Tengine Web Server</b></a><br>
</p>

<a id="comparison-reviews"></a>
##### 对比评测

<p>
&nbsp;&nbsp;:black_small_square: <a href="https://www.hostingadvice.com/how-to/nginx-vs-apache/"><b>NGINX vs. Apache (Pro/Con Review, Uses, & Hosting for Each)</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/jiangwenyuan/nuster/wiki/Web-cache-server-performance-benchmark:-nuster-vs-nginx-vs-varnish-vs-squid"><b>Web cache server performance benchmark: nuster vs nginx vs varnish vs squid</b></a><br>
</p>

<a id="cheatsheets--references"></a>
##### 速查表与参考

<p>
&nbsp;&nbsp;:black_small_square: <a href="https://openresty.org/download/agentzh-nginx-tutorials-en.html"><b>agentzh's Nginx Tutorials</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="http://agentzh.org/misc/slides/nginx-conf-scripting/nginx-conf-scripting.html#1"><b>Introduction to nginx.conf scripting</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="http://www.nginx-discovery.com/"><b>Nginx discovery journey</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="http://www.nginxguts.com/"><b>Nginx Guts</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://gist.github.com/carlessanagustin/9509d0d31414804da03b"><b>Nginx Cheatsheet</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="http://www.scalescale.com/tips/nginx/"><b>Nginx Tutorials, Linux Sysadmin Configuration & Optimizing Tips and Tricks</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/h5bp/server-configs-nginx"><b>Nginx boilerplate configs</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/nginx-boilerplate/nginx-boilerplate"><b>Awesome Nginx configuration template</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/SimulatedGREG/nginx-cheatsheet"><b>Nginx Quick Reference</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/fcambus/nginx-resources"><b>A collection of resources covering Nginx and more</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/lebinh/nginx-conf"><b>A collection of useful Nginx configuration snippets</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/elasticweb/nginx-configs"><b>Nginx configurations for most popular CMS/CMF/Frameworks based on PHP</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/wmnnd/nginx-certbot"><b>Boilerplate configuration for nginx and certbot with docker-compose</b></a><br>
</p>

<a id="performance--hardening"></a>
##### 性能与加固

<p>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/denji/nginx-tuning"><b>Nginx Tuning For Best Performance by Denji</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://thoughts.t37.net/nginx-optimization-understanding-sendfile-tcp-nodelay-and-tcp-nopush-c55cdd276765"><b>Nginx Optimization: understanding sendfile, tcp_nodelay and tcp_nopush</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://blog.cloudflare.com/how-we-scaled-nginx-and-saved-the-world-54-years-every-day/"><b>How we scaled nginx and saved the world 54 years every day</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://istlsfastyet.com/"><b>TLS has exactly one performance problem: it is not used widely enough</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.ssllabs.com/projects/best-practices/"><b>SSL/TLS Deployment Best Practices</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.ssllabs.com/projects/rating-guide/index.html"><b>SSL Server Rating Guide</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.ssllabs.com/ssl-pulse/"><b>SSL Pulse</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.upguard.com/blog/how-to-build-a-tough-nginx-server-in-15-steps"><b>How to Build a Tough NGINX Server in 15 Steps</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.cyberciti.biz/tips/linux-unix-bsd-nginx-webserver-security.html"><b>Top 25 Nginx Web Server Best Security Practices</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://calomel.org/nginx.html"><b>Nginx Secure Web Server</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html"><b>Strong SSL Security on Nginx</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://enable-cors.org/index.html"><b>Enable cross-origin resource sharing (CORS)</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://github.com/nbs-system/naxsi"><b>NAXSI - WAF for Nginx</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://geekflare.com/install-modsecurity-on-nginx/"><b>ModSecurity for Nginx</b></a><br>
</p>

<a id="presentations--videos"></a>
##### 演示与视频

<p>
&nbsp;&nbsp;:black_small_square: <a href="https://www.slideshare.net/Nginx/nginx-basics-and-best-practices"><b>NGINX: Basics and Best Practices</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.slideshare.net/Nginx/nginx-installation-and-tuning"><b>NGINX Installation and Tuning</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.slideshare.net/joshzhu/nginx-internals"><b>Nginx Internals (by Joshua Zhu)</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.slideshare.net/feifengxlq/nginx-internals-10514355"><b>Nginx internals (by Liqiang Xu)</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.slideshare.net/wallarm/how-to-secure-your-web-applications-with-nginx"><b>How to secure your web applications with NGINX</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.slideshare.net/chartbeat/tuning-tcp-and-nginx-on-ec2"><b>Tuning TCP and NGINX on EC2</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.slideshare.net/trygvevea/extending-functionality-in-nginx-with-modules"><b>Extending functionality in nginx, with modules!</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.slideshare.net/tuxtoti/nginx-tips-and-tricks-13087831"><b>Nginx - Tips and Tricks.</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.slideshare.net/TonyFabeen/nginx-scripting-extending-nginx-functionalities-with-lua"><b>Nginx Scripting - Extending Nginx Functionalities with Lua</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.slideshare.net/kazeburo/advanced-nginx-in-mercari-how-to-handle-over-1200000-https-reqsmin"><b>How to handle over 1,200,000 HTTPS Reqs/Min</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.slideshare.net/harukayon/ngx-lua-public"><b>Using ngx_lua / lua-nginx-module in pixiv</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://mdounin.ru/files/mdounin-nginx-whatsnew-nginxconf2018.pdf"><b>Reading nginx CHANGES together</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://mdounin.ru/files/mdounin-dynamic-modules-nginxconf2016.pdf"><b>Dynamic modules:how it works</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.youtube.com/playlist?list=PLGz_X9w9raXewvc6tjIGGFZ6DBKHEld3k"><b>NGINX Conf 2014</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.youtube.com/playlist?list=PLGz_X9w9raXdED9BR6GQ61A6d3fBzjpbn"><b>NGINX Conf 2015</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.youtube.com/playlist?list=PLGz_X9w9raXcOsB_dT26iu0BvbSxWYG1g"><b>NGINX Conf 2016</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.youtube.com/playlist?list=PLGz_X9w9raXeT-z_rcZ9yF0kV5SENZ-yt"><b>NGINX Conf 2017</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.youtube.com/playlist?list=PLGz_X9w9raXeHhKRX6ZS7vmFKN12iYOw9"><b>NGINX Conf 2018 | Deep Dive Track</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.youtube.com/playlist?list=PLGz_X9w9raXe_Vc708VKvr5KJ4gnf1WxS"><b>NGINX Conf 2018 | Keynotes and Sessions</b></a><br>
&nbsp;&nbsp;:black_small_square: <a href="https://www.youtube.com/watch?v=iHxD-G0YjiU"><b>Making HTTPS Fast(er): Ilya Grigorik @ nginx.conf 2014</b></a><br>
</p>
