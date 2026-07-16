<a id="nginx-basics"></a>
# NGINX 基础

转到 **[目录](https://github.com/trimstray/nginx-admins-handbook#table-of-contents)** 或 **[下一步？](https://github.com/trimstray/nginx-admins-handbook#whats-next)** 章节。

- **[≡ NGINX 基础](#nginx-basics)**
  * [目录和文件](#directories-and-files)
  * [命令](#commands)
  * [进程](#processes)
    * [CPU 绑定](#cpu-pinning)
    * [工作进程关闭](#shutdown-of-worker-processes)
  * [配置语法](#configuration-syntax)
    * [注释](#comments)
    * [行尾](#end-of-lines)
    * [变量、字符串和引号](#variables-strings-and-quotes)
    * [指令、块和上下文](#directives-blocks-and-contexts)
    * [外部文件](#external-files)
    * [度量单位](#measurement-units)
    * [使用 PCRE 的正则表达式](#regular-expressions-with-pcre)
    * [启用语法高亮](#enable-syntax-highlighting)
  * [连接处理](#connection-processing)
    * [事件驱动架构](#event-driven-architecture)
    * [多进程](#multiple-processes)
    * [并发连接](#simultaneous-connections)
    * [HTTP Keep-Alive 连接](#http-keep-alive-connections)
    * [sendfile、tcp_nodelay 和 tcp_nopush](#sendfile-tcp_nodelay-and-tcp_nopush)
  * [请求处理阶段](#request-processing-stages)
  * [Server 块逻辑](#server-blocks-logic)
    * [处理传入连接](#handle-incoming-connections)
    * [匹配 location](#matching-location)
    * [rewrite vs return](#rewrite-vs-return)
    * [URL 重定向](#url-redirections)
    * [try_files 指令](#try_files-directive)
    * [if、break 和 set](#if-break-and-set)
    * [root vs alias](#root-vs-alias)
    * [internal 指令](#internal-directive)
    * [外部和内部重定向](#external-and-internal-redirects)
    * [allow 和 deny](#allow-and-deny)
    * [uri vs request_uri](#uri-vs-request_uri)
  * [压缩和解压缩](#compression-and-decompression)
    * [什么是最佳的 NGINX 压缩 gzip 级别？](#what-is-the-best-nginx-compression-gzip-level)
  * [哈希表](#hash-tables)
    * [服务器名称哈希表](#server-names-hash-table)
  * [日志文件](#log-files)
    * [条件日志记录](#conditional-logging)
    * [手动日志轮转](#manually-log-rotation)
    * [错误日志严重级别](#error-log-severity-levels)
    * [如何记录请求的开始时间？](#how-to-log-the-start-time-of-a-request)
    * [如何记录 HTTP 请求体？](#how-to-log-the-http-request-body)
    * [NGINX upstream 变量返回 2 个值](#nginx-upstream-variables-returns-2-values)
  * [反向代理](#reverse-proxy)
    * [传递请求](#passing-requests)
    * [尾部斜杠](#trailing-slashes)
    * [向后端传递请求头](#passing-headers-to-the-backend)
      * [Host 请求头的重要性](#importance-of-the-host-header)
      * [重定向和 X-Forwarded-Proto](#redirects-and-x-forwarded-proto)
      * [关于 X-Forwarded-For 的警告](#a-warning-about-the-x-forwarded-for)
      * [使用 Forwarded 提高可扩展性](#improve-extensibility-with-forwarded)
    * [响应头](#response-headers)
  * [负载均衡算法](#load-balancing-algorithms)
    * [后端参数](#backend-parameters)
    * [使用 SSL 的上游服务器](#upstream-servers-with-ssl)
    * [轮询](#round-robin)
    * [加权轮询](#weighted-round-robin)
    * [最少连接](#least-connections)
    * [加权最少连接](#weighted-least-connections)
    * [IP 哈希](#ip-hash)
    * [通用哈希](#generic-hash)
    * [其他方法](#other-methods)
  * [速率限制](#rate-limiting)
    * [变量](#variables)
    * [指令、键和区域](#directives-keys-and-zones)
    * [Burst 和 nodelay 参数](#burst-and-nodelay-parameters)
  * [NAXSI Web 应用防火墙](#naxsi-web-application-firewall)
  * [OWASP ModSecurity 核心规则集 (CRS)](#owasp-modsecurity-core-rule-set-crs)
  * [核心模块](#core-modules)
    * [ngx_http_geo_module](#ngx_http_geo_module)
  * [第三方模块](#3rd-party-modules)
    * [ngx_set_misc](#ngx_set_misc)
    * [ngx_http_geoip_module](#ngx_http_geoip_module)
<a id="directories-and-files"></a>
#### 目录和文件

  > 如果你使用默认参数编译 NGINX，所有文件和目录都位于 `/usr/local/nginx` 位置。

对于上游 NGINX 打包，路径可能如下（取决于系统/发行版的类型）：

- `/etc/nginx` - 是 NGINX 服务的默认配置根目录
  * 其他位置：`/usr/local/etc/nginx`, `/usr/local/nginx/conf`

- `/etc/nginx/nginx.conf` - 是 NGINX 服务使用的默认配置入口点，包含顶级 http 块以及所有其他配置上下文和文件
  * 其他位置：`/usr/local/etc/nginx/nginx.conf`, `/usr/local/nginx/conf/nginx.conf`

- `/usr/share/nginx` - 是请求的默认根目录，包含 `html` 目录和基本静态文件
  * 其他位置：根目录中的 `html/`

- `/var/log/nginx` - 是 NGINX 的默认日志（访问和错误日志）位置
  * 其他位置：根目录中的 `logs/`

- `/var/cache/nginx` - 是 NGINX 的默认临时文件位置
  * 其他位置：`/var/lib/nginx`

- `/etc/nginx/conf` - 包含自定义/vhosts 配置文件
  * 其他位置：`/etc/nginx/conf.d`, `/etc/nginx/sites-enabled`（我无法忍受这种 debian/apache 式约定）

- `/var/run/nginx` - 包含有关 NGINX 进程的信息
  * 其他位置：`/usr/local/nginx/logs`，根目录中的 `logs/`

另请参阅 [安装和编译时选项 - 文件和权限](https://www.nginx.com/resources/wiki/start/topics/tutorials/installoptions/#files-and-permissions)。

<a id="commands"></a>
#### 命令

  > **:bookmark: [使用 reload 选项动态更改配置 - 基本规则 - P2](RULES.md#beginner-use-reload-option-to-change-configurations-on-the-fly)**

- `nginx -h` - 显示帮助
- `nginx -v` - 显示 NGINX 版本
- `nginx -V` - 显示 NGINX 的扩展信息：版本、构建参数和配置参数
- `nginx -t` - 测试 NGINX 配置
- `nginx -c <filename>` - 设置配置文件（默认：`/etc/nginx/nginx.conf`）
- `nginx -p <directory>` - 设置前缀路径（默认：`/etc/nginx/`）
- `nginx -T` - 测试 NGINX 配置并在屏幕上打印验证后的配置
- `nginx -s <signal>` - 向 NGINX 主进程发送信号：
  - `stop` - 立即终止 NGINX 进程
  - `quit` - 在处理完当前请求后停止 NGINX 进程
  - `reload` - 重新加载配置而不停止进程
  - `reopen` - 指示 NGINX 重新打开日志文件
- `nginx -g <directive>` - 在配置文件之外设置[全局指令](https://nginx.org/en/docs/ngx_core_module.html)

一些用于管理 NGINX 守护进程的有用代码片段：

- 测试配置：

  ```bash
  /usr/sbin/nginx -t -c /etc/nginx/nginx.conf
  /usr/sbin/nginx -t -q -g 'daemon on; master_process on;' # ; echo $?

  /usr/local/etc/rc.d/nginx status
  ```

- 启动守护进程：

  ```bash
  /usr/sbin/nginx -g 'daemon on; master_process on;'

  service nginx start
  systemctl start nginx

  /usr/local/etc/rc.d/nginx start

  # 你也可以从 start-stop-daemon 脚本启动 NGINX：
  /sbin/start-stop-daemon --quiet --start --exec /usr/sbin/nginx --background --retry QUIT/5 --pidfile /run/nginx.pid
  ```

- 停止守护进程：

  ```bash
  # 优雅关闭（等待工作进程完成当前请求的处理）
  /usr/sbin/nginx -s quit
  # 快速关闭（立即终止连接）
  /usr/sbin/nginx -s stop

  service nginx stop
  systemctl stop nginx

  /usr/local/etc/rc.d/nginx stop

  # 你也可以从 start-stop-daemon 脚本停止 NGINX：
  /sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
  ```

- 重新加载守护进程：

  ```bash
  /usr/sbin/nginx -g 'daemon on; master_process on;' -s reload

  service nginx reload
  systemctl reload nginx

  /usr/local/etc/rc.d/nginx reload

  kill -HUP $(cat /var/run/nginx.pid)
  kill -HUP $(pgrep -f "nginx: master")
  ```

- 重启守护进程：

  ```bash
  service nginx restart
  systemctl restart nginx

  /usr/local/etc/rc.d/nginx restart
  ```

关于测试配置的一些说明：

  > 你不能测试未完成的配置。例如，你在一个单独的文件中为你的域名定义了一个 server 块。任何测试此类文件的尝试都会抛出错误。该文件必须在所有方面都是完整的。

<a id="configuration-syntax"></a>
#### 配置语法

  > **:bookmark: [组织 Nginx 配置 - 基本规则 - P2](RULES.md#beginner-organising-nginx-configuration)**<br>
  > **:bookmark: [格式化、美化和缩进你的 Nginx 代码 - 基本规则 - P2](RULES.md#beginner-format-prettify-and-indent-your-nginx-code)**

NGINX 在配置文件中使用一种微编程语言。这种语言的设计深受 Perl 和 Bourne Shell 的影响。配置语法、格式和定义遵循所谓的 C 风格约定。对我来说，NGINX 配置具有简单且非常透明的结构。

<a id="comments"></a>
##### 注释

NGINX 配置文件不支持注释块，它们只接受行首的 `#` 作为注释。

<a id="end-of-lines"></a>
##### 行尾

包含指令的行必须以分号（`;`）结束，否则 NGINX 将无法加载配置并报告错误。

<a id="variables-strings-and-quotes"></a>
##### 变量、字符串和引号

变量以 `$` 开头，并为每个请求自动设置。在运行时设置变量并基于它们控制逻辑流程的能力是 rewrite 模块的一部分，而不是 NGINX 的通用特性。默认情况下，我们不能修改内置变量，如 `$host` 或 `$request_uri`。

  > 有些指令不支持变量，例如 `access_log`（实际上是例外，因为可以包含有限制的变量）或 `error_log`。变量可能不能（也不应该，因为它们在每个请求的处理过程中在运行时评估，与纯静态配置相比相当昂贵）在任何地方声明，只有极少数例外：`root` 指令可以包含变量，`server_name` 指令只允许严格的 `$hostname` 内置值作为类似变量的表示法（但它更像是一个魔术常量）。如果你在 `if` 上下文中使用变量，你只能在 `if` 条件（以及可能的 rewrite 指令）中设置它们。不要试图在其他地方使用它们。

要给变量赋值，你应该使用 `set` 指令：

```nginx
set $var "value";
```

  > 参见 [`if`、`break` 和 `set`](#if-break-and-set) 章节以了解更多关于变量的信息。

关于变量的一些有趣的事情：

  > 确保阅读 [agentzh 的 Nginx 教程](https://openresty.org/download/agentzh-nginx-tutorials-en.html) —— 这是关于 NGINX 技巧和诀窍的。这个人是 NGINX 大师和 OpenResty 的创建者。在这些教程中，他详细描述了变量等。我还推荐 [nginx built-in variables](http://siwei.me/blog/posts/nginx-built-in-variables) 文章。

- NGINX 中的大多数变量只在运行时存在，而不是在配置时存在
- 变量的作用域遍布整个配置
- 变量赋值发生在实际服务请求时
- 变量的生命周期与相应请求完全相同
- 每个请求都有自己版本的所有那些变量的容器（不同的容器值）
- 即使请求引用了同名的变量，也不会相互干扰
- 赋值操作只在访问 location 的请求中执行

字符串可以不用引号输入，除非它们包含空格、分号或花括号，这时需要用反斜杠转义或用单引号/双引号括起来。

对于包含空格和/或其他特殊字符的值，需要引号，否则 NGINX 将无法识别它们。你可以引用或使用 `\` 转义字符串中的一些特殊字符，如 `" "` 或 `";"`（这些字符会使语句的含义变得模糊）。所以以下指令是相同的：

```nginx
# 1)
add_header My-Header "nginx web server;";

# 2)
add_header My-Header nginx\ web\ server\;;
```

除非 `$` 被转义，否则引号字符串中的变量会被正常展开。

<a id="directives-blocks-and-contexts"></a>
##### 指令、块和上下文

  > 阅读这篇关于 [NGINX 配置继承模型](https://blog.martinfjordvald.com/understanding-the-nginx-configuration-inheritance-model/) 的优秀文章，作者 [Martin Fjordvald](https://blog.martinfjordvald.com/about/)。

配置选项称为指令。我们有四种类型的指令：

- 标准指令 - 每个上下文一个值：

  ```nginx
  worker_connections 512;
  ```

- 数组指令 - 每个上下文多个值：

  ```nginx
  error_log /var/log/nginx/localhost/localhost-error.log warn;
  ```

- 动作指令 - 不仅仅进行配置：

  ```nginx
  rewrite ^(.*)$ /msie/$1 break;
  ```

- `try_files` 指令：

  ```nginx
  try_files $uri $uri/ /test/index.html;
  ```

有效指令以变量名开头，然后指定一个参数或用空格分隔的一系列参数。

指令被组织成称为**块**或**上下文**的组。通常，上下文是一个块指令，可以在花括号内包含其他指令。它似乎以树状结构组织，由一组括号——`{` 和 `}`——定义。

  > 花括号实际上表示一个新的配置上下文。

作为一般规则，如果一个指令在多个嵌套作用域中有效，则更广泛上下文中的声明将作为默认值传递给任何子上下文。子上下文可以随意覆盖这些值。

  > 放在任何上下文之外的配置文件中的指令被视为在全局/主上下文中。

  > 应特别注意与某些指令相关的一些奇怪行为。更多信息请参见 [正确设置带有 add_header 和 proxy_*_header 指令的 HTTP 头](RULES.md#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly) 规则。

指令只能在其设计的上下文中使用。NGINX 在读取包含在错误上下文中声明的指令的配置文件时会报错。

  > 如果你想查看所有指令，请参见 [指令字母索引](https://nginx.org/en/docs/dirindex.html)。

上下文可以相互嵌套（继承层级）。它们的结构如下：

```
Global/Main Context
        |
        |
        +-----» Events Context
        |
        |
        +-----» HTTP Context
        |          |
        |          |
        |          +-----» Server Context
        |          |          |
        |          |          |
        |          |          +-----» Location Context
        |          |
        |          |
        |          +-----» Upstream Context
        |
        |
        +-----» Mail Context
```

最重要的上下文如下所示。这些将是你大部分时间要处理的：

- `global` - 包含全局配置指令；用于全局设置 NGINX 的设置，是唯一没有用花括号包围的上下文

- `events` - 事件模块的配置；用于设置连接处理的全局选项；包含影响连接处理的指令

- `http` - 控制使用 HTTP 模块的所有方面，并包含处理 HTTP 和 HTTPS 流量的指令；此上下文中的指令可以分组为：

  - HTTP 客户端指令
  - HTTP 文件 I/O 指令
  - HTTP 哈希指令
  - HTTP 套接字指令

- `server` - 定义虚拟主机设置，并描述与特定域名或 IP 地址关联的一组资源的逻辑分离

- `location` - 定义处理客户端请求的指令，并指示来自客户端或内部重定向的 URI

- `upstream` - 定义 NGINX 可以代理请求的后端服务器池；通常用于定义用于负载均衡的 Web 服务器集群

NGINX 还提供其他上下文（例如用于映射），例如：

- `map` - 用于根据另一个变量的值设置变量的值。它提供变量值的映射，以确定第二个变量应设置为什么

- `geo` - 用于指定映射。但是，此映射专门用于分类客户端 IP 地址。它根据连接 IP 地址设置变量的值

- `types` - 用于将 MIME 类型映射到应与之关联的文件扩展名

- `if` - 提供在其中定义的指令的条件处理，如果给定测试返回 `true`，则执行包含的指令

- `limit_except` - 用于限制在 location 上下文中使用某些 HTTP 方法

另请查看下面的图表。它显示了最重要的上下文及其与配置的关系：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/nginx_contexts.png" alt="nginx-contexts">
</p>

对于 HTTP，NGINX 查找从 http 块开始，然后通过一个或多个 server 块，接着是 location 块。

<a id="external-files"></a>
##### 外部文件

`include` 指令可以出现在任何上下文中，以执行条件包含。它附加另一个文件或匹配指定掩码的文件：

```nginx
include /etc/nginx/proxy.conf;

# 或者：
include /etc/nginx/conf/*.conf;
```

  > 你不能在 NGINX 配置文件包含中使用变量。这是因为包含操作在任何变量被评估之前就已处理。

另请参见[此链接](http://nginx.org/en/docs/faq/variables_in_config.html)：

  > _变量不应被用作模板宏。变量在每个请求的处理过程中在运行时被评估，因此与纯静态配置相比相当昂贵。使用变量存储静态字符串也是一个坏主意。相反，应使用宏扩展和 "include" 指令来更轻松地生成配置，这可以通过外部工具完成，例如 sed + make 或任何其他常见的模板机制。_

<a id="measurement-units"></a>
##### 度量单位

  > 建议始终指定后缀，以确保清晰和一致。

大小可以指定为：

- 无后缀：Bytes
- `k` 或 `K`：Kilobytes
- `m` 或 `M`：Megabytes
- `g` 或 `G`：Gigabytes

```nginx
client_max_body_size 2m;
```

时间间隔可以指定为：

- 无后缀：Seconds
- `ms`：Milliseconds
- `s`：Seconds
- `m`：Minutes
- `h`：Hours
- `d`：Days
- `w`：Weeks
- `M`：Months (30 days)
- `y`：Years (365 days)

```nginx
proxy_read_timeout 20; # =20s，默认
```

某些时间间隔只能以秒精度指定。你还应该记住：

  > _可以通过按从最重要到最不重要的顺序指定多个单位，并可选地用空格分隔，来组合成一个值。例如，`1h 30m` 指定的时间与 `90m` 或 `5400s` 相同。_

<a id="regular-expressions-with-pcre"></a>
##### 使用 PCRE 的正则表达式

  > **:bookmark: [启用 PCRE JIT 以加速正则表达式处理 - 性能 - P2](RULES.md#beginner-enable-pcre-jit-to-speed-up-processing-of-regular-expressions)**

在开始阅读接下来的章节之前，你应该了解什么是正则表达式以及它们如何工作（它们实际上不是黑魔法）。我推荐两篇由 [Jonny Fox](https://medium.com/@jonny.fox) 编写的关于正则表达式的优秀简短文章：

- [Regex 教程——快速备忘单（附示例）](https://medium.com/factory-mind/regex-tutorial-a-simple-cheatsheet-by-examples-649dc1c3f285)
- [Regex 手册——十大最受欢迎的 regex](https://medium.com/factory-mind/regex-cookbook-most-wanted-regex-aa721558c3c1)

为什么？正则表达式可以在 `server_name` 和 `location`（以及其他）指令中使用，有时你必须具备良好的阅读它们的能力。我认为你应该创建最具可读性的正则表达式，不要让它们成为无法调试和维护的意大利面条式代码。

NGINX 使用 [PCRE](https://www.pcre.org/) 库来对你的 `location` 块执行复杂操作，并使用强大的 `rewrite` 指令。要使用正则表达式进行字符串匹配，首先需要编译它，这通常在配置阶段完成。

你还可以启用 `pcre_jit` 以在执行期间（运行时）进行动态翻译，而不是在执行之前。此选项可以提高性能，但是，在某些情况下，`pcre_jit` 可能会产生负面影响。因此，在启用它之前，我建议你阅读这份优秀文档：[PCRE 性能项目](https://zherczeg.github.io/sljit/pcre.html)。

以下是关于正则表达式和 PCRE 的一些有趣内容：

- [在 Y 分钟内学习 PCRE](https://learnxinyminutes.com/docs/pcre/)
- [PCRE 正则表达式备忘单](https://www.debuggex.com/cheatsheet/regex/pcre)
- [正则表达式备忘单 - PCRE](https://github.com/niklongstone/regular-expression-cheat-sheet)
- [Regex 备忘单](https://remram44.github.io/regex-cheatsheet/regex.html)
- [Perl 中的正则表达式](http://jkorpela.fi/perl/regexp.html)
- [Regexp 安全备忘单](https://github.com/attackercan/regexp-security-cheatsheet)
- [面向所有 regex 讨厌者（和爱好者）的 regex 备忘单](https://dev.to/catherinecodes/a-regex-cheatsheet-for-all-those-regex-haters-and-lovers--2cj1)

你还可以使用外部工具来测试正则表达式。更多信息请参见[在线工具](https://github.com/trimstray/nginx-admins-handbook#online-tools)章节。

如果你擅长此道，可以尝试这些非常有趣且烧脑的正则表达式挑战：

- [RegexGolf](https://alf.nu/RegexGolf)
- [Regex Crossword](https://regexcrossword.com/)

<a id="enable-syntax-highlighting"></a>
##### 启用语法高亮

<a id="vivim"></a>
###### vi/vim

```bash
# 1) 下载 NGINX 的 vim 插件：

# 官方 NGINX vim 插件：
mkdir -p ~/.vim/syntax/

wget "http://www.vim.org/scripts/download_script.php?src_id=19394" -O ~/.vim/syntax/nginx.vim

# 改进的 NGINX vim 插件（包括语法高亮），使用 Pathogen：
mkdir -p ~/.vim/{autoload,bundle}/

curl -LSso ~/.vim/autoload/pathogen.vim https://tpo.pe/pathogen.vim
echo -en "\nexecute pathogen#infect()\n" >> ~/.vimrc

git clone https://github.com/chr4/nginx.vim ~/.vim/bundle/nginx.vim

# 2) 设置 NGINX 配置文件的位置：
cat > ~/.vim/filetype.vim << __EOF__
au BufRead,BufNewFile /etc/nginx/*,/etc/nginx/conf.d/*,/usr/local/nginx/conf/*,*/conf/nginx.conf if &ft == '' | setfiletype nginx | endif
__EOF__
```

  > 你可能感兴趣：[在 Vim 中高亮不安全的 SSL 配置](https://github.com/chr4/sslsecure.vim)。

<a id="sublime-text"></a>
###### Sublime Text

安装 `cabal`——用于构建和打包 Haskell 库和程序的系统（在 Ubuntu 上）：

```bash
add-apt-repository -y ppa:hvr/ghc
apt-get update

apt-get install -y cabal-install-1.22 ghc-7.10.2

# 将此添加到你的 shell 主配置文件中：
export PATH=$HOME/.cabal/bin:/opt/cabal/1.22/bin:/opt/ghc/7.10.2/bin:$PATH
source $HOME/.<shellrc>

cabal update
```

- `nginx-lint`：

  ```bash
  git clone https://github.com/temoto/nginx-lint

  cd nginx-lint && cabal install --global
  ```

- `sublime-nginx` + `SublimeLinter-contrib-nginx-lint`：

  打开 _Command Palette_ 并输入 `install`。在命令中你应该看到 _Package Control: Install Package_。输入 `nginx` 安装 [sublime-nginx](https://github.com/brandonwamboldt/sublime-nginx)，然后再次执行上述操作以安装 [SublimeLinter-contrib-nginx-lint](https://github.com/irvinlim/SublimeLinter-contrib-nginx-lint)：输入 `SublimeLinter-contrib-nginx-lint`。

<a id="processes"></a>
#### 进程

  > **:bookmark: [调整工作进程数 - 性能 - P3](RULES.md#beginner-adjust-worker-processes)**<br>
  > **:bookmark: [通过禁用守护进程、主进程和除一个外的所有工作进程来改进调试 - 调试 - P4](RULES.md#beginner-improve-debugging-by-disable-daemon-master-process-and-all-workers-except-one)**

NGINX 有**一个主进程**和**一个或多个工作进程**。如果你启用缓存，它还有缓存加载器和缓存管理器进程。

主进程的主要目的是读取和评估配置文件，以及维护工作进程（在工作进程死亡时重新生成）、处理信号、通知工作进程、打开日志文件，当然还有绑定端口。

主进程应以 root 用户身份启动，因为这将允许 NGINX 打开低于 1024 的套接字（它需要能够监听 HTTP 的 80 端口和 HTTPS 的 443 端口）。

  > 要定义工作进程的数量，你应该设置 `worker_processes` 指令。

工作进程执行实际的请求处理并从主进程接收命令。它们在事件循环中运行（注册事件并在事件发生时响应），处理网络连接，读写磁盘内容，并与上游服务器通信。它们由主进程生成，用户和组按指定设置（非特权）。

  > 工作进程大部分时间只是休眠并等待新事件（它们在 `top` 中处于 `S` 状态）。

以下信号可以发送到 NGINX 主进程：

| <b>信号</b> | <b>编号</b> | <b>描述</b> |
| :---         | :---:        | :---         |
| `TERM`, `INT` | **15**, **2** | 快速关闭 |
| `QUIT` | **3** | 优雅关闭 |
| `KILL` | **9** | 终止顽固进程 |
| `HUP` | **1** | 配置重新加载，启动新工作进程，优雅关闭旧工作进程 |
| `USR1` | **10** | 重新打开日志文件 |
| `USR2` | **12** | 动态升级可执行文件 |
| `WINCH` | **28** | 优雅关闭工作进程 |

无需自己控制工作进程。但是，它们也支持一些信号：

| <b>信号</b> | <b>编号</b> | <b>描述</b> |
| :---         | :---:        | :---         |
| `TERM`, `INT` | **15**, **2** | 快速关闭 |
| `QUIT` | **3** | 优雅关闭 |
| `USR1` | **10** | 重新打开日志文件 |

<a id="cpu-pinning"></a>
###### CPU 绑定

此外，重要的是要提到 `worker_cpu_affinity` 指令（仅在 GNU/Linux 上支持）。CPU 亲和性用于控制 NGINX 为各个工作进程利用哪些 CPU。默认情况下，工作进程不绑定到任何特定的 CPU。更重要的是，系统可能会将所有工作进程调度到同一个 CPU 上运行，这可能效率不高。

CPU 亲和性表示为位掩码（以十六进制给出），最低位对应第一个逻辑 CPU，最高位对应最后一个逻辑 CPU。

[这里](https://www.kutukupret.com/2010/11/18/nginx-worker_cpu_affinity/) 有一个关于此问题的精彩解释。有一个 NGINX 的 [worker_cpu_affinity 配置生成器](https://github.com/cubicdaiya/nwcagen)。毕竟，我建议让 OS 调度程序来完成工作，因为在正常操作期间没有理由设置它。

<a id="shutdown-of-worker-processes"></a>
###### 工作进程关闭

如果你想调整 NGINX 的关闭过程，这应该很有用，特别是当其他服务器或负载均衡器依赖于可预测的重启时间，或者关闭工作进程需要很长时间时。

`worker_shutdown_timeout` 指令配置一个超时时间，用于优雅关闭工作进程。当计时器到期时，NGINX 将尝试关闭当前打开的所有连接以促进关闭。

NGINX 的 [Maxim Dounin](https://twitter.com/mdounin) 解释说：

  > _`worker_shutdown_timeout` 指令不会在没有活动连接时延迟关闭。它的引入是为了限制可能的关闭时间，即即使有活动连接，也确保足够快的关闭。_

当工作进程进入 "exiting" 状态时，它会做以下几件事：

  - 将自己标记为退出进程
  - 如果定义了 `worker_shutdown_timeout`，设置关闭计时器
  - 关闭监听套接字
  - 关闭空闲连接

然后，如果设置了关闭计时器，在 `worker_shutdown_timeout` 间隔后，所有连接都将关闭。

  > 默认情况下，NGINX 会在完全关闭连接之前等待并处理来自客户端的额外数据，但这仅当启发式算法表明客户端可能正在发送更多数据时。

有时，你会在日志文件中看到 `nginx: worker process is shutting down`。问题出现在重新加载配置时——NGINX 通常会优雅地退出现有的工作进程，但有时关闭这些进程需要数小时。每次配置重新加载都可能产生僵尸工作进程，永久消耗你系统的所有内存。在这种情况下，快速关闭工作进程可能是一种解决方案。

此外，设置 `worker_shutdown_timeout` 也可以解决这个问题：

```nginx
worker_shutdown_timeout 60s;
```

测试连接超时以及服务器处理请求所需的时间，然后根据这些值调整 `worker_shutdown_timeout` 值。60 秒是一个有充足余量的值，任何有效的请求都不应超过这个时间。

根据我的经验，如果你有多个工作进程处于关闭状态，也许你应该首先检查可能导致工作进程挂起问题的已加载模块。

<a id="connection-processing"></a>
#### 连接处理

NGINX 支持多种[连接处理方法](https://nginx.org/en/docs/events.html)，具体取决于所使用的平台。

一般来说，有四种类型的事件多路复用：

- `select` - 已经过时且不推荐，但在所有平台上都作为后备安装
- `poll` - 已经过时且不推荐

以及最高效的非阻塞 I/O 实现：

- `epoll` - 如果你使用 GNU/Linux，推荐使用
- `kqueue` - 如果你使用 BSD，推荐使用（技术上优于 `epoll`）

`select` 方法可以使用 `--with-select_module` 或 `--without-select_module` 配置参数启用或禁用。类似地，`poll` 方法可以使用 `--with-poll_module` 或 `--without-poll_module` 配置参数启用或禁用。

  > `epoll` 是一种在 Linux 2.6+ 上可用的高效连接处理方法。`kqueue` 是一种在 FreeBSD 4.1+、OpenBSD 2.9+ 和 NetBSD 2.0+ 上可用的高效连接处理方法。

通常无需显式指定它，因为 NGINX 默认会使用最高效的方法。但如果你想设置它：

```nginx
use epoll;
```

还有一些优秀的资源（也进行了比较）：

- [Kqueue：一种通用且可扩展的事件通知机制](https://people.freebsd.org/~jlemon/papers/kqueue.pdf)
- [poll vs select vs 基于事件](https://daniel.haxx.se/docs/poll-vs-select.html)
- [select/poll/epoll：系统架构师的实际差异](http://www.ulduzsoft.com/2014/01/select-poll-epoll-practical-difference-for-system-architects/)
- [可扩展的事件多路复用：epoll vs. kqueue](https://people.eecs.berkeley.edu/~sangjin/2012/12/21/epoll-vs-kqueue.html)
- [Linux 上的异步 I/O：select、poll 和 epoll](https://jvns.ca/blog/2017/06/03/async-io-on-linux--select--poll--and-epoll/)
- [select(2) 简史](https://idea.popcount.org/2016-11-01-a-brief-history-of-select2/)
- [Select 从根本上被破坏](https://idea.popcount.org/2017-01-06-select-is-fundamentally-broken/)
- [Epoll 从根本上被破坏](https://idea.popcount.org/2017-02-20-epoll-is-fundamentally-broken-12/)
- [使用 epoll 和 kqueue 系统调用的 I/O 多路复用](https://austingwalters.com/io-multiplexing/)
- [BSD 和 Linux 基准测试](http://bulk.fefe.de/scalability/)
- [C10K 问题](http://www.kegel.com/c10k.html)

再看看 libevent 基准测试（阅读关于 [libevent – 一个事件通知库](http://libevent.org/)）：

<p align="center">
  <a href="https://www.nginx.com/resources/library/infographic-inside-nginx/">
    <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/libevent-benchmark.jpg" alt="libevent-benchmark">
  </a>
</p>

<sup><i>此信息图来自 [daemonforums - 一个有趣的基准测试（kqueue vs. epoll）](http://daemonforums.org/showthread.php?t=2124)。</i></sup>

你还可以了解为什么大玩家在 FreeBSD 上使用 NGINX 而不是在 GNU/Linux 上：

- [FreeBSD NGINX 性能](https://devinteske.com/wp/freebsd-nginx-performance/)
- [为什么 Netflix 使用 NGINX 和 FreeBSD 构建自己的 CDN？](https://www.youtube.com/watch?v=KP_bKvXkoC4)

NGINX 对连接的定义如下（以下状态信息由 `ngx_http_stub_status_module` 提供）：

- **Active connections** - 当前活动（开放）客户端连接的数量，包括等待连接和到后端的连接
  - **accepts** - 已接受的客户端连接总数
  - **handled** - 已处理的连接总数。通常，此参数值与 `accepts` 相同，除非达到了某些资源限制（例如，`worker_connections` 限制）
  - **requests** - 客户端请求总数
- **Reading** - NGINX 正在读取请求头的当前连接数
- **Writing** - NGINX 正在向客户端写回响应的当前连接数（读取请求体、处理请求或向客户端写入响应）
- **Waiting** - 等待请求的空闲客户端连接数，即连接仍然打开，等待新请求或 keepalive 过期（实际上是 Active - (Reading + Writing)）

  > Waiting 连接是那些 keepalive 连接。它们通常不是问题，但如果想减少它们，请设置较低的 `keepalive_timeout` 指令值。

请务必推荐阅读[此内容](https://trac.nginx.org/nginx/ticket/1610#comment:1)：

  > Writing 连接计数器增加可能表示以下情况之一：
  >
  >   - 工作进程崩溃或被终止。但在你的情况下不太可能，因为这也会导致其他值增长，特别是 `Waiting`
  >   - 某处存在真正的套接字泄漏。这通常会导致套接字处于 `CLOSE_WAIT` 状态（等待终止连接的 FIN 包），尝试查看不带 `grep -v CLOSE_WAIT` 过滤器的 `netstat` 输出。泄漏的套接字会在工作进程优雅关闭时（例如，在配置重新加载后）被 NGINX 报告——如果有任何泄漏的套接字，NGINX 会在错误日志中写入 `open socket ... left in connection ...` 警报
  >
  > 要进一步调查，请执行以下操作：
  >   - 升级到最新的主线版本，不带任何第三方模块，并检查你是否能够重现该问题
  >   - 尝试禁用 HTTP/2 以查看是否解决了问题
  >   - 检查在配置重新加载时是否看到 `open socket ... left in connection ...`（套接字泄漏）警报

另请参见[调试套接字泄漏（来自本手册）](HELPERS.md#debugging-socket-leaks)。

<a id="event-driven-architecture"></a>
##### 事件驱动架构

  > [NGINX 中的线程池将性能提升 9 倍！](https://www.nginx.com/blog/thread-pools-boost-performance-9x/)——这篇官方文章是关于线程池以及通常关于处理连接的精彩解释。我还推荐 [Inside NGINX：我们如何为性能和规模设计](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale)。两者都非常棒。

NGINX 使用事件驱动架构，严重依赖于非阻塞 I/O。非阻塞/异步操作的一个优点是可以最大化单个 CPU 以及内存的使用，因为你的线程可以并行继续工作。最终结果是，即使负载增加，内存和 CPU 使用率仍然可控。

  > 有一个非常好的简短[总结](https://stackoverflow.com/questions/8546273/is-non-blocking-i-o-really-faster-than-multi-threaded-blocking-i-o-how)关于非阻塞 I/O 和多线程阻塞 I/O，作者是 [Werner Henze](https://stackoverflow.com/users/1023911/werner-henze)。我还推荐 [asynchronous vs non-blocking](https://stackoverflow.com/a/2625565)，作者是 [Daniel Earwicker](https://stackoverflow.com/users/27423/daniel-earwicker)。

看看这个简单的图示：

<p align="center">
  <a href="http://faculty.salina.k-state.edu/tim/ossg/Device/blocking.html">
    <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/io/blocking_non-blocking.jpg" alt="blocking_non-blocking">
  </a>
</p>

<sup><i>此信息图来自 [堪萨斯州立理工大学网站](http://faculty.salina.k-state.edu/tim/ossg/Device/blocking.html)。</i></sup>

阻塞 I/O 系统调用（a）直到 I/O 完成才返回。非阻塞 I/O 系统调用立即返回。进程稍后在 I/O 完成时得到通知。

I/O 的形式和 POSIX 函数示例：

| <b>阻塞</b> | <b>非阻塞</b> | <b>异步</b> |
| :---         | :---         | :---         |
| `write`, `read` | `write`, `read` + `poll/select` | `aio_write`, `aio_read` |

再看看官方文档对此的描述：

  > _众所周知，NGINX 使用异步、事件驱动的方法来处理连接。这意味着它不会为每个请求创建另一个专用进程或线程（像传统架构的服务器那样），而是在一个工作进程中处理多个连接和请求。为了实现这一点，NGINX 以非阻塞模式处理套接字，并使用高效的方法，如 epoll 和 kqueue。_

  > _由于全重量进程的数量很少（通常每个 CPU 核心只有一个）且保持不变，因此消耗的内存少得多，CPU 周期也不会浪费在任务切换上。这种方法的优点通过 NGINX 本身的例子已广为人知。它成功地处理了数百万个并发请求，并且扩展性非常好。_

我绝不能忘记在这里提到关于非阻塞和第三方模块（也来自官方文档）：

  > _不幸的是，许多第三方模块使用阻塞调用，用户（有时甚至是模块的开发者）没有意识到其缺点。阻塞操作会破坏 NGINX 的性能，必须不惜一切代价避免。_

为了用单个工作进程处理并发请求，NGINX 使用[反应器设计模式](https://stackoverflow.com/questions/5566653/simple-explanation-for-the-reactor-pattern-with-its-applications)。基本上，它是单线程的，但可以 fork 多个进程以利用多个核心。

然而，NGINX 并不是一个单线程应用程序。每个工作进程都是单线程的，可以处理数千个并发连接。工作进程用于跨多个核心实现请求并行性。当一个请求阻塞时，该工作进程将处理另一个请求。

NGINX 不会为每个连接/请求创建新进程/线程，而是在启动期间启动几个工作线程。它使用一个线程异步完成此操作，而不是使用多线程编程（它使用带有异步 I/O 的事件循环）。

这样，I/O 和网络操作就不会成为非常大的瓶颈（记住，你的 CPU 会花费大量时间等待网络接口等）。这是因为 NGINX 只使用一个线程来服务所有请求。当请求到达服务器时，它们被逐一服务。然而，当被服务的代码需要做其他事情时，它会将回调发送到另一个队列，主线程将继续运行（它不会等待）。

现在你明白为什么 NGINX 可以完美地处理大量请求（且没有任何问题）了。

更多信息请参考以下资源：

- [异步、非阻塞 I/O](https://medium.com/@entzik/on-asynchronous-non-blocking-i-o-4a2ac0af5c50)
- [异步编程。阻塞 I/O 和非阻塞 I/O](https://luminousmen.com/post/asynchronous-programming-blocking-and-non-blocking)
- [阻塞 I/O 和非阻塞 I/O](https://medium.com/coderscorner/tale-of-client-server-and-socket-a6ef54a74763)
- [非阻塞 I/O](https://www.hellsoft.se/non-blocking-io/)
- [关于高并发、NGINX 架构和内部原理](http://www.aosabook.org/en/nginx.html)
- [一个小节日礼物：与 Nginx 达到 10,000 reqs/sec！](https://blog.webfaction.com/2008/12/a-little-holiday-present-10000-reqssec-with-nginx-2/)
- [Nginx vs Apache：它快吗？如果快，为什么？](http://planetunknown.blogspot.com/2011/02/why-nginx-is-faster-than-apache.html)
- [Nginx 在任务或线程方面如何处理请求？](https://softwareengineering.stackexchange.com/questions/256510/how-is-nginx-handling-its-requests-in-terms-of-tasks-or-threading)
- [为什么 nginx 比 Apache 快，以及为什么你不一定在乎](https://djangodeployment.com/2016/11/15/why-nginx-is-faster-than-apache-and-why-you-neednt-necessarily-care/)
- [我们如何扩展 nginx 并每天拯救世界 54 年](https://blog.cloudflare.com/how-we-scaled-nginx-and-saved-the-world-54-years-every-day/)

最后，看看这些精彩的预览：

<p align="center">
  <a href="https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/">
    <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/io/NGINX_non-blocking.png" alt="NGINX_non-blocking">
  </a>
</p>

<sup><i>两个信息图均来自 [Inside NGINX：我们如何为性能和规模设计](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)。</i></sup>

<a id="multiple-processes"></a>
##### 多进程

NGINX 只使用异步 I/O，这使得阻塞不成问题。NGINX 使用多进程的唯一原因是充分利用多核、多 CPU 和超线程系统。NGINX 只需要足够的工作进程即可获得对称多处理（SMP）的全部好处。

官方文档指出：

  > _大多数情况下推荐的 NGINX 配置——每个 CPU 核心运行一个工作进程——能最高效地利用硬件资源。_

NGINX 使用专为 NGINX 设计的自定义事件循环——所有连接都在有限数量的单线程进程（称为工作进程）中以高度高效的运行循环处理。工作进程从共享监听套接字接受新请求并执行循环。NGINX 中没有专门的连接分发到工作进程；这项工作由 OS 内核机制完成，它通知工作进程。

  > _启动时，会创建一组初始的监听套接字。工作进程随后持续接受、读取和写入套接字，同时处理 HTTP 请求和响应。_——来自 [开源应用程序的架构 - NGINX](http://aosabook.org/en/nginx.html)。

多路复用的工作原理是使用循环逐块递增地遍历程序，每个连接/对象每次循环迭代处理一个数据/新连接/其他内容。这一切都基于事件多路复用，如 `epoll()` 或 `kqueue()`。在每个工作进程中，NGINX 每秒可以处理成千上万的并发连接和请求。

  > 参见 [Nginx 内部原理](https://www.slideshare.net/joshzhu/nginx-internals) 演示文稿，其中包含大量关于 NGINX 内部的精彩内容。

NGINX 不会为每个连接 fork 一个进程或线程（如 Apache），因此在绝大多数情况下内存使用非常保守且极其高效。NGINX 比 Apache 更快，消耗更少的内存，并且在高负载下表现非常出色。它对 CPU 也非常友好，因为没有持续的进程或线程创建-销毁模式。

最后总结：

- 使用非阻塞 "Event-Driven" 架构
- 使用单线程反应器模式处理并发请求
- 使用高度高效的连接处理循环
- 不是单线程应用程序，因为它启动多个工作进程（以处理多个连接和请求）

<a id="simultaneous-connections"></a>
##### 并发连接

好的，那么 NGINX 可以处理多少个并发连接？

```
worker_processes * worker_connections = 最大连接数
```

据此：如果你运行 **4** 个工作进程，每个工作进程有 **4,096** 个工作连接，你将能够服务 **16,384** 个连接。当然，这些是受内核限制的 NGINX 设置（连接数、打开文件数或进程数）。

  > 在这里，我想提一下[理解 TCP 中的套接字和端口](https://medium.com/fantageek/understanding-socket-and-port-in-tcp-2213dc2e9b0c)。这是一个很棒且简短的解释。我还推荐阅读[现代 Linux 盒子可以拥有的最大打开 TCP 连接数](https://stackoverflow.com/questions/2332741/what-is-the-theoretical-maximum-number-of-open-tcp-connections-that-a-modern-lin)。

我见过一些管理员直接将 `worker_processes` 和 `worker_connections` 的总和解释为可以同时服务的客户端数量。在我看来，这是一个错误，因为某些客户端（例如，具有不同值的浏览器）**会打开多个并行连接**（请参见[此链接](https://stackoverflow.com/questions/985431/max-parallel-http-connections-in-a-browser)以确认我的话）。客户端通常会建立 4-8 个 TCP 连接，以便可以并行下载资源（以下载组成网页的各种组件，例如图像、脚本等）。这增加了有效带宽并减少了延迟。

  > 这是 HTTP/1.1 并发 HTTP 调用的限制（6-8 个）。提高性能的最佳解决方案（无需升级硬件和在中介处使用缓存（例如 CDN、Varnish））是使用 HTTP/2（[RFC 7540](https://tools.ietf.org/html/rfc7540) <sup>[IETF]</sup>）而不是 HTTP/1.1。
  >
  > HTTP/2 在单个连接上多路复用多个 HTTP 请求。当 HTTP/1.1 有大约 6-8 个的限制时，HTTP/2 没有标准限制，但说："_建议此值（`SETTINGS_MAX_CONCURRENT_STREAMS`）不小于 100_"（RFC 7540）。这个数字比 6-8 好。

此外，你必须知道 `worker_connections` 指令**包括每个工作进程的所有连接**（例如，连接结构用于监听套接字、NGINX 进程之间的内部控制套接字、与代理服务器的连接以及上游连接），而不仅仅是来自客户端的传入连接。

  > 请注意，每个工作连接（在休眠状态）需要 256 字节的内存，因此你可以轻松增加它。

连接数尤其受系统上最大打开文件数（`RLIMIT_NOFILE`）的限制（你可以在这篇[精彩解释](https://stackoverflow.com/questions/2423628/whats-the-difference-between-a-file-descriptor-and-file-pointer)中阅读关于文件描述符和文件句柄的内容）。原因是操作系统需要内存来管理每个打开的文件，而内存是有限的资源。此限制仅影响当前进程的限制。当前进程的限制也会传给子进程，但每个进程有单独的计数。

要更改最大文件描述符（单个工作进程可以打开的）的限制，你还可以编辑 `worker_rlimit_nofile` 指令。有了这个，NGINX 提供了非常强大的动态配置能力，无需重启服务。

  > 文件描述符的数量不是连接数量的唯一限制——还要记住内核网络（TCP/IP 栈）参数和最大进程数。

我不太喜欢 NGINX 文档的这部分内容。也许我遗漏了什么，但它说 `worker_rlimit_nofile` 是工作进程最大打开文件数的限制。我相信它关联的是单个工作进程。

如果你设置 `RLIMIT_NOFILE` 为 25,000，`worker_rlimit_nofile` 为 12,000，NGINX 会将工作进程的最大打开文件数限制设置为 `worker_rlimit_nofile`。但主进程将具有 `RLIMIT_NOFILE` 的设置值。`worker_rlimit_nofile` 指令的默认值是 `none`，因此默认情况下 NGINX 从系统限制设置最大打开文件数的初始值。

```bash
# 在 GNU/Linux 上（或 /usr/lib/systemd/system/nginx.service）：
grep "LimitNOFILE" /lib/systemd/system/nginx.service
LimitNOFILE=5000

grep "worker_rlimit_nofile" /etc/nginx/nginx.conf
worker_rlimit_nofile 256;

   PID       SOFT HARD
 24430       5000 5000
 24431        256 256
 24432        256 256
 24433        256 256
 24434        256 256

# 在 FreeBSD 上检查文件描述符：
sysctl kern.maxfiles kern.maxfilesperproc kern.openfiles
kern.maxfiles: 64305
kern.maxfilesperproc: 57870
kern.openfiles: 143
```

这也受操作系统的控制，因为工作进程不是服务器上唯一运行的进程。如果你的工作进程用完了所有进程可用的文件描述符，那将非常糟糕，所以不要设置可能导致这种情况的限制。

在我看来，依赖 `RLIMIT_NOFILE`（以及其他系统上的替代方案）比依赖 `worker_rlimit_nofile` 值更易理解和预测。老实说，使用哪种方法设置并不重要，但你应该持续关注限制的优先级。

  > 如果你没有手动设置 `worker_rlimit_nofile` 指令，那么操作系统设置将决定 NGINX 可以使用多少个文件描述符。

我认为文件描述符耗尽的可能性很小，但在高流量网站上可能是一个大问题。

好的，那么 NGINX 会打开多少个文件描述符？

- 客户端活动连接的一个文件句柄
- 代理连接的一个文件句柄（将打开处理这些请求到远程或本地主机/进程的套接字）
- 打开文件的一个文件句柄（例如，静态文件）
- 用于内部连接、共享库、日志文件和套接字的其他文件句柄

同样重要的是：

  > NGINX 每个完整的连接最多可以使用两个文件描述符。

另请查看这些图示：

- 1 个文件句柄用于与客户端的连接，1 个文件句柄用于 NGINX 服务的静态文件：

  ```
  # 1 个连接，2 个文件句柄

                       +-----------------+
  +----------+         |                 |
  |          |    1    |                 |
  |  CLIENT <---------------> NGINX      |
  |          |         |        ^        |
  +----------+         |        |        |
                       |      2 |        |
                       |        |        |
                       | +------v------+ |
                       | | 静态文件    | |
                       | +-------------+ |
                       +-----------------+
  ```

- 1 个文件句柄用于与客户端的连接，1 个文件句柄用于到远程或本地主机/进程的开放套接字：

  ```
  # 2 个连接，2 个文件句柄

                       +-----------------+
  +----------+         |                 |         +-----------+
  |          |    1    |                 |    2    |           |
  |  CLIENT <---------------> NGINX <---------------> 后端     |
  |          |         |                 |         |           |
  +----------+         |                 |         +-----------+
                       +-----------------+
  ```

- 2 个文件句柄用于来自同一客户端的两个并发连接（1、4），1 个文件句柄用于与其他客户端的连接（3），2 个文件句柄用于静态文件（2、5），1 个文件句柄用于到远程或本地主机/进程的开放套接字（6），因此总共是 6 个文件描述符：

  ```
  # 4 个连接，6 个文件句柄

                    4
        +-----------------------+
        |              +--------|--------+
  +-----v----+         |        |        |
  |          |    1    |        v        |  6
  |  CLIENT <-----+---------> NGINX <---------------+
  |          |    |    |        ^        |    +-----v-----+
  +----------+    |    |        |        |    |           |
                 3 |    |      2 | 5      |    |  后端    |
  +----------+    |    |        |        |    |           |
  |          |    |    |        |        |    +-----------+
  |  CLIENT  <----+    | +------v------+ |
  |          |         | | 静态文件    | |
  +----------+         | +-------------+ |
                       +-----------------+
  ```

在前两个例子中：我们可以认为 NGINX 每个完整连接需要 2 个文件句柄（但仍然使用 2 个工作连接）。在第三个例子中，NGINX 每个完整连接仍然需要 2 个文件句柄（即使客户端使用并行连接）。

所以，总结一下，我认为每个工作进程的所有连接的 `worker_rlimit_nofile` 的正确值应大于 `worker_connections`。

在我看来，`worker_rlimit_nofile`（和系统限制）的安全值是：

```
# 1 个连接对应 1 个文件句柄：
worker_connections + (共享库、日志文件、事件池等) = worker_rlimit_nofile

# 1 个连接对应 2 个文件句柄：
(worker_connections * 2) + (共享库、日志文件、事件池等) = worker_rlimit_nofile
```

这大概是每个工作进程可以打开的文件数，并且应该大于每个工作进程的连接数（根据上述公式）。

在大多数文章和教程中，我们可以看到此参数的值类似于 NGINX 所有打开文件的最大数量（甚至更多）。如果我们假设此参数分别适用于每个工作进程，那么这些值总体上是过度的。

然而，经过更深入的思考，它们是合理的，因为它们允许一个工作进程使用所有文件描述符，这样如果其他工作进程出现问题，它们不会受到限制。但请记住，我们仍然受限于每个工作进程的连接数。请允许我提醒你，任何连接至少打开一个文件。

那么，NGINX 的最大打开文件数应为：

```
(worker_processes * worker_connections * 2) + (共享库、日志文件、事件池等) = 最大打开文件数
```

  > 为了服务所有工作进程的 **16,384** 个连接（每个工作进程 4,096 个连接），并考虑到 NGINX 使用的其他句柄，这种情况下最大文件句柄数的合理值可能是 **35,000**。我认为这绰绰有余。

考虑到以上情况，要更改/改进限制，你应该：

1. 编辑内核在阻塞之前将分配的最大总全局文件描述符数（此步骤是可选的，我认为只有在流量非常高时才应更改此值）：

    ```bash
    # 查找系统范围的最大文件句柄数：
    sysctl fs.file-max

    # 显示内核内存中当前所有文件描述符的数量：
    #   第一个值：<已分配的文件句柄>
    #   第二个值：<已分配但未使用的文件句柄>
    #   第三个值：<系统范围的最大文件句柄数> # fs.file-max
    sysctl fs.file-nr

    # 手动临时设置：
    sysctl -w fs.file-max=150000

    # 永久设置：
    echo "fs.file-max = 150000" > /etc/sysctl.d/99-fs.conf

    # 加载内核参数的新值：
    sysctl -p       # 对于 /etc/sysctl.conf
    sysctl --system # 对于 /etc/sysctl.conf 和所有系统配置文件
    ```

2. 编辑单个进程可以打开的最大文件描述符数的系统范围值：

    - 对于非 systemd 系统：

      ```bash
      # 设置通过 PAM 登录的用户的最大文件描述符数：
      #   /etc/security/limits.conf
      nginx       soft    nofile    35000
      nginx       hard    nofile    35000
      ```

    - 对于 systemd 系统：

      ```bash
      # 设置通过 systemd 启动的服务的最大文件描述符数（硬限制）：
      #   /etc/systemd/system.conf          - 全局配置（所有单元的默认值）
      #   /etc/systemd/user.conf            - 指定进一步的每个用户限制
      #   /lib/systemd/system/nginx.service - NGINX 服务的默认单元
      #   /etc/systemd/system/nginx.service - 你自己的 NGINX 服务实例
      [Service]
      # ...
      LimitNOFILE=35000

      # 重新加载单元文件并重启 NGINX 服务：
      systemctl daemon-reload && systemctl restart nginx
      ```

3. 调整 NGINX 工作进程的系统打开文件数限制。最大值不能大于 `LimitNOFILE`（在此示例中：35,000）。你可以随时更改它：

    ```bash
    # 设置单个工作进程的文件描述符限制（根据需要更改）：
    #   nginx.conf 在主上下文中
    worker_rlimit_nofile 10000;

    # 你需要重新加载 NGINX 服务：
    nginx -s reload
    ```

要显示应用于 NGINX 进程的当前软限制和硬限制（使用 `nofile`、`LimitNOFILE` 或 `worker_rlimit_nofile`）：

```bash
for _pid in $(pgrep -f "nginx: [master,worker]") ; do

  echo -en "$_pid "
  grep "Max open files" /proc/${_pid}/limits | awk '{print $4" "$5}'

done | xargs printf '%6s %10s\t%s\n%6s %10s\t%s\n' "PID" "SOFT" "HARD"
```

或使用以下命令：

```bash
# 要确定进程的 OS 限制，请读取文件 /proc/$pid/limits。
# $pid 对应进程的 PID：
for _pid in $(pgrep -f "nginx: [master,worker]") ; do

  echo -en ">>> $_pid\\n"
  cat /proc/$_pid/limits

done
```

要列出每个 NGINX 进程的当前打开文件描述符：

```bash
for _pid in $(pgrep -f "nginx: [master,worker]") ; do

  _fds=$(find /proc/${_pid}/fd/*)
  _fds_num=$(echo "$_fds" | wc -l)

  echo -en "\n\n##### PID: $_pid ($_fds_num fds) #####\n\n"

  # 列出 proc/{pid}/fd 目录中的所有文件：
  echo -en "$_fds\n\n"

  # 列出所有打开的文件（日志文件、内存映射文件、库）：
  lsof -as -p $_pid | awk '{if(NR>1)print}'

done
```

你还应该记住以下规则：

- `worker_rlimit_nofile` 用于动态更改 NGINX 工作进程可以处理的最大文件描述符数，通常由系统的软限制（`ulimit -Sn`）定义

- `worker_rlimit_nofile` 仅在进程级别工作，受限于系统的硬限制（`ulimit -Hn`）

- 如果你启用了 SELinux，你将需要运行 `setsebool -P httpd_setrlimit 1`，以便 NGINX 具有设置其 rlimit 的权限。要诊断 SELinux 拒绝和尝试，你可以使用 `sealert -a /var/log/audit/audit.log`，或 `audit2why` 和 `audit2allow` 工具

总结此示例：

- 每个 NGINX 进程（主进程 + 工作进程）能够创建最多 35,000 个文件
- 对于所有工作进程，最大文件描述符数为 140,000（每个工作进程的 `LimitNOFILE`）
- 对于每个工作进程，初始/当前文件描述符数为 10,000（`worker_rlimit_nofile`）

```
nginx: master process         = LimitNOFILE (35,000)
  \_ nginx: worker process    = LimitNOFILE (35,000), worker_rlimit_nofile (10,000)
  \_ nginx: worker process    = LimitNOFILE (35,000), worker_rlimit_nofile (10,000)
  \_ nginx: worker process    = LimitNOFILE (35,000), worker_rlimit_nofile (10,000)
  \_ nginx: worker process    = LimitNOFILE (35,000), worker_rlimit_nofile (10,000)

                              = master (35,000), 所有工作进程：
                                                 - LimitNOFILE: 140,000
                                                 - worker_rlimit_nofile: 40,000
```

另请参阅这篇关于[优化 Nginx 以应对高流量负载](https://blog.martinfjordvald.com/optimizing-nginx-for-high-traffic-loads/)的优秀文章。


<a id="http-keep-alive-connections"></a>
##### HTTP Keep-Alive 连接

  > **:bookmark: [激活到上游服务器的连接缓存 - 性能 - P2](RULES.md#beginner-activate-the-cache-for-connections-to-upstream-servers)**

在开始本节之前，我建议阅读以下文章：

- [HTTP Keepalive 连接和 Web 性能](https://www.nginx.com/blog/http-keepalives-and-web-performance/)
- [优化 HTTP：Keep-alive 和管道化](https://www.igvita.com/2011/10/04/optimizing-http-keep-alive-and-pipelining/)
- [HTTP 的演变 — HTTP/0.9、HTTP/1.0、HTTP/1.1、Keep-Alive、Upgrade 和 HTTPS](https://medium.com/platform-engineer/evolution-of-http-69cfe6531ba0)

HTTP 的原始模型，也是 HTTP/1.0 中的默认模型，是短连接。每个 HTTP 请求在其自己的连接上完成；这意味着在每个 HTTP 请求之前都要进行 TCP 握手，并且这些握手是串行的。客户端为每个事务创建一个新的 TCP 连接（事务完成后连接被拆除）。

HTTP Keep-Alive 连接或持久连接是使用单个 TCP 连接发送和接收多个 HTTP 请求/响应的概念（Keep Alive 在请求之间工作），而不是为每个请求/响应对打开一个新连接。

使用 keep alive 时，浏览器不必建立多个连接（请记住建立连接开销很大），而是使用已经建立的连接并控制其保持活动/打开的时间。因此，keep alive 是一种减少创建连接开销的方法，因为大多数情况下，用户会在站点中导航等（再加上单个页面中的多个请求，以下载 css、javascript、图像等）。

建立 TCP 连接需要三次握手，因此，当客户端和服务器之间存在可感知的延迟时，keepalive 通过重用现有连接可以大大加快速度。

这种机制在 HTTP 事务完成后保持客户端和服务器之间的 TCP 连接打开。这一点很重要，因为 NGINX 需要不时地关闭连接，即使你配置 NGINX 允许无限的 keep alive 超时和大量每连接可接受的请求，以返回结果以及错误和成功消息。

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/closed_vs_keepalive.png" alt="closed_vs_keepalive">
</p>

持久连接模型在连续请求之间保持连接打开，减少了打开新连接所需的时间。HTTP 管道化模型更进一步，通过发送多个连续请求而无需等待应答，大大减少了网络中的延迟。

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/http_connections.png" alt="http_connections">
</p>

<sup><i>此信息图来自 [Mozilla MDN - HTTP/1.x 中的连接管理](https://developer.mozilla.org/en-US/docs/Web/HTTP/Connection_management_in_HTTP_1.x)。</i></sup>

然而，目前浏览器并未使用管道化 HTTP 请求。更多信息请参见[为什么现代浏览器中禁用了管道化？](https://stackoverflow.com/questions/30477476/why-is-pipelining-disabled-in-modern-browsers)。

另请参见这个示例，展示了如何使用 Keep-Alive 请求头：

`
 Client                        Proxy                         Server
   |                             |                              |
   +- Keep-Alive: timeout=600 -->|                              |
   |  Connection: Keep-Alive     |                              |
   |                             +- Keep-Alive: timeout=1200 -->|
   |                             |  Connection: Keep-Alive      |
   |                             |                              |
   |                             |<-- Keep-Alive: timeout=300 --+
   |                             |    Connection: Keep-Alive    |
   |<- Keep-Alive: timeout=5000 -+                              |
   |    Connection: Keep-Alive   |                              |
   |                             |                              |
`

NGINX 官方文档说明：

  > _所有连接都是独立协商的。客户端指示超时时间为 600 秒（10 分钟），但代理只准备保留连接至少 120 秒（2 分钟）。在代理和服务器之间的链接上，代理请求超时时间为 1200 秒，服务器将其减少到 300 秒。正如本示例所示，代理维护的超时策略对于每个连接是不同的。每个连接跳是独立的。_

Keepalive 连接减少了开销，特别是在使用 SSL/TLS 时，但它们也有缺点；即使在空闲时也会消耗服务器资源，并且在重负载下可能发生 DoS 攻击。在这种情况下，使用一旦空闲就关闭的非持久连接可以提供更好的性能。因此，如果客户端正在进行多个请求，Keep-Alive 将大幅提高 SSL/TLS 性能，但如果你没有资源处理它们，它们会拖垮你的服务器。

  > 当达到 worker_connections 限制时，NGINX 会关闭 keepalive 连接（连接保留在缓存中，直到源服务器关闭它们）。

为了更好地理解 Keep-Alive 的工作原理，请参见 [Barry Pollard](https://stackoverflow.com/users/2144578/barry-pollard) 的精彩[解释](https://stackoverflow.com/a/38190172)。

NGINX 提供两个层来启用 Keep-Alive：

<a id="client-layer"></a>
###### 客户端层

- 客户端在给定连接上可以进行的最大 keepalive 请求数，这意味着客户端可以在一个 keepalive 连接内成功发出例如 256 个请求：

  `
ginx
  # 默认：100
  keepalive_requests 256;
  `

- 服务器将在此时间后关闭连接。当流量较大时，可能需要更高的值以确保不会频繁重新建立 TCP 连接。如果设置较低，你将无法在大多数请求上利用 keep-alive，从而减慢客户端速度：

  `
ginx
  # 默认：75s
  keepalive_timeout 10s;

  # 或者通过添加可选的第二个超时时间告诉浏览器何时应关闭连接
  # 该值在发送给浏览器的请求头中（某些浏览器不关心此请求头）：
  keepalive_timeout 10s 25s;
  `

  > 增加此值可以保持 keepalive 连接更长时间打开，从而使后续请求更快。然而，设置过高会导致资源浪费（主要是内存），因为即使没有流量，连接也会保持打开，可能会显著影响性能。我认为这个值应尽可能接近你的平均响应时间。你也可以一点一点地减少超时时间（75s -> 50s，然后 25s...），看看服务器的表现。

<a id="upstream-layer"></a>
###### 上游层

- 每个工作进程保留的空闲 keepalive 连接数。connections 参数设置每个工作进程缓存中保留到上游服务器的最大空闲 keepalive 连接数（超过此数量时，最近最少使用的连接将被关闭）：

  `
ginx
  # 默认：禁用
  keepalive 32;
  `

NGINX 默认只使用 HTTP/1.0 与上游服务器通信。要保持 TCP 连接活动，上游部分和源服务器都应配置为不终止连接。

  > 请记住，keepalive 是 HTTP 1.1 的一个特性，NGINX 默认对上游使用 HTTP 1.0。

默认情况下不会重用连接，因为 upstream 部分的 keepalive 意味着没有 keepalive（每次你都可以看到 TCP 流数量随着每个到源服务器的请求而增加）。

在 NGINX 上游服务器中启用 HTTP keepalive 可以减少延迟，从而提高性能，并减少 NGINX 耗尽临时端口的可能性。

  > connections 参数应设置为一个足够小的数字，以便上游服务器也能处理新的传入连接。

更新你的上游配置以使用 keepalive：

`
ginx
upstream bk_x8080 {

  ...

  # 设置每个工作进程缓存中保留到上游服务器的最大空闲 keepalive 连接数。
  keepalive 16;

}
`

并在所有上游请求中启用 HTTP/1.1 协议：

`
ginx
server {

  ...

  location / {

    # 默认是 HTTP/1，keepalive 仅在 HTTP/1.1 中启用：
    proxy_http_version 1.1;
    # 如果客户端发送了 Connection 请求头，则将其删除，
    # 它可能是 "close" 用于关闭 keepalive 连接：
    proxy_set_header Connection "";

    proxy_pass http://bk_x8080;

  }

...

}
`

保持连接活动真正有益的两个基本场景：

- 快速后端，在非常短的时间内产生响应，可与 TCP 握手相媲美
- 远程后端，当 TCP 握手耗时较长，可与后端响应时间相媲美

查看测试结果：

- 上游无 keepalive：

`ash
wrk -c 500 -t 6 -d 60s -R 15000 -H "Host: example.com" https://example.com/
Running 1m test @ https://example.com/
  6 threads and 500 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    24.13s    10.68s   49.55s    59.06%
    Req/Sec   679.21     42.44   786.00     78.95%
  228421 requests in 1.00m, 77.98MB read
  Socket errors: connect 0, read 0, write 0, timeout 1152
  Non-2xx or 3xx responses: 4
Requests/sec:   3806.96
Transfer/sec:      1.30MB
`

- 上游有 keepalive：

`ash
wrk -c 500 -t 6 -d 60s -R 15000 -H "Host: example.com" https://example.com/
Running 1m test @ https://example.com/
  6 threads and 500 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    23.40s     9.53s   47.25s    60.67%
    Req/Sec     0.86k    50.19     0.94k    60.00%
  294148 requests in 1.00m, 100.41MB read
  Socket errors: connect 0, read 0, write 0, timeout 380
Requests/sec:   4902.24
Transfer/sec:      1.67MB
`


<a id=""sendfile"-"tcp_nodelay"-and-"tcp_nopush""></a>
##### sendfile、	cp_nodelay 和 	cp_nopush

在开始阅读之前，请先查看：

- [Nginx 优化，理解 SENDFILE、TCP_NODELAY 和 TCP_NOPUSH](https://thoughts.t37.net/nginx-optimization-understanding-sendfile-tcp-nodelay-and-tcp-nopush-c55cdd276765)
- [Nginx 教程 #2：性能](https://www.netguru.com/codestories/nginx-tutorial-performance)

在进行这些更改时，请密切关注你的网络流量，看看每次调整对拥塞的影响。

<a id=""sendfile""></a>
###### sendfile

  > _默认情况下，NGINX 自己处理文件传输，并在发送前将文件复制到缓冲区。启用 sendfile 指令消除了将数据复制到缓冲区的步骤，并实现了直接从文件描述符到文件描述符的数据复制。_

通常，当需要发送文件时，需要以下步骤：

- malloc - 分配用于存储对象数据的本地缓冲区
- ead - 检索并将对象复制到本地缓冲区
- write - 将对象从本地缓冲区复制到套接字缓冲区

看看这个精彩的解释（来自 [Nginx 教程 #2：性能](https://www.netguru.com/codestories/nginx-tutorial-performance)）：

  > _这涉及两个上下文切换（读、写），这使得同一对象的第二次复制变得不必要。如你所见，这不是最优的方式。幸运的是，还有另一种改进文件发送的系统调用，它叫做（惊喜！）：sendfile(2)。这个调用将对象检索到文件缓存中，并将指针（而不复制整个对象）直接传递给套接字描述符。Netflix 表示，使用 sendfile(2) 将网络吞吐量从 [6Gbps 提高到 30Gbps](https://people.freebsd.org/~rrs/asiabsd_2015_tls.pdf)。_

当文件由进程传输时，内核首先缓冲数据，然后将数据发送到进程缓冲区。进程再将数据发送到目的地。

NGINX 采用了一种解决方案，使用 sendfile 系统调用来执行从磁盘到套接字的零拷贝数据流，并节省了读/写时从用户空间的上下文切换。sendfile 指示 NGINX 如何缓冲或读取文件（尝试将内容直接塞入网络槽位，或首先缓冲其内容）。

这是一种改进的数据传输方法，其中数据在 OS 内核空间内的文件描述符之间复制，即无需将数据传输到应用程序缓冲区。不需要额外的缓冲区或数据副本，数据永远不会离开内核内存地址空间。

在我看来，除非 NGINX 从可以映射到虚拟内存空间的内容（如文件）中读取（即数据在缓存中），否则启用此功能不会产生任何区别。但请不要让我影响你——你应该首先关注这份文档：[在 FreeBSD 中优化高带宽应用的 TLS](https://people.freebsd.org/~rrs/asiabsd_2015_tls.pdf) <sup>[pdf]</sup>。

默认情况下 NGINX 禁用 sendfile：

`
ginx
# http, server, location, if in location 上下文
# 启用 sendfile（我的建议）：
sendfile on;

# 禁用 sendfile：
sendfile off;     # 默认
`

另请参见 sendfile_max_chunk 指令。NGINX 文档说明：

  > _当设置为非零值时，限制在单个 sendfile() 调用中传输的数据量。如果没有限制，一个快速连接可能会独占整个工作进程。_

在快速的本地连接上，Linux 中的 sendfile() 可能每次系统调用发送数十兆字节，从而阻塞其他连接。sendfile_max_chunk 允许限制每个 sendfile() 操作的最大大小。因此，NGINX 可以减少在阻塞的 sendfile() 调用中花费的最大时间，因为 NGINX 不会尝试一次性发送整个文件，而是分块发送。例如：

`
ginx
sendfile on;
sendfile_max_chunk 512k;
`

<a id=""tcp_nodelay""></a>
###### 	cp_nodelay

我建议阅读 [TCP_NODELAY 的注意事项](https://eklitzke.org/the-caveats-of-tcp-nodelay) 和 [重新思考 TCP Nagle 算法](http://ccr.sigcomm.org/archive/2001/jan01/ccr-200101-mogul.pdf) <sup>[pdf]</sup>。这些优秀的论文描述了关于 TCP_NODELAY 和 TCP_NOPUSH 的非常有趣的主题。

	cp_nodelay 用于管理 Nagle 算法，该算法是一种通过减少在网络上发送的小数据包数量来提高 TCP 效率的机制。如果你设置 	cp_nodelay on;，NGINX 在打开新套接字时会添加 TCP_NODELAY 选项。

  > 此选项仅影响 keep-alive 连接。否则，当 NGINX 在最后一个不完整的 TCP 数据包中发送响应尾部时，会有 100ms 的延迟。此外，它在 SSL 连接、非缓冲代理和 WebSocket 代理上启用。

也许你应该考虑启用 Nagle 算法（	cp_nodelay off;），但这实际上取决于你的特定工作负载和服务的主要流量模式。对于现代 Web，	cp_nodelay on; 更合理，TCP 的整个延迟业务对于终端来说是合理的。通常，局域网比广域网更少有流量拥塞问题。如果 TCP/IP 流量是由用户输入零星生成的，而不是由使用流导向协议（如 HTTP 流量）的应用程序生成的，则 Nagle 算法最有效。

所以，对我来说，规则很简单：

- 批量发送或 HTTP 流量
- 需要更低延迟的应用程序
- 非交互式类型的流量

不需要使用 Nagle 算法。

你还应该知道 [Nagle 算法作者的评论](https://news.ycombinator.com/item?id=9045125)：

  > _如果你正在进行批量文件传输，你永远不会遇到这个问题。如果你发送足够多的数据来填充传出缓冲区，就没有延迟。如果你发送所有数据并关闭 TCP 连接，最后一个数据包后就没有延迟。如果你发送、应答、发送、应答，就没有延迟。如果你进行批量发送，就没有延迟。如果你发送、发送、应答，会有延迟。_

  > _真正的问题在于 ACK 延迟。200ms 的 "ACK 延迟" 定时器是大约 1985 年 Berkeley 的某个人因为不理解问题而塞入 BSD 的坏主意。延迟 ACK 是一种赌博，认为在 200ms 内会有来自应用层的回复。即使每次都在输掉这场赌博，TCP 仍会继续使用延迟 ACK。_

我认为如果你处理的是非交互式类型的流量或批量传输，如 HTTP/Web 流量，那么启用 TCP_NODELAY 以禁用 Nagle 算法可能是有用的（这是 NGINX 的默认行为）。这尤其适用于运行某些时候只有高度交互流量和聊天协议的应用或环境。

默认情况下 NGINX 启用 TCP_NODELAY 选项：

`
ginx
# http, server, location 上下文
# 启用 tcp_nodelay 同时禁用 Nagle 算法
#（我的建议，除非你启用了 tcp_nopush）：
tcp_nodelay on;   # 默认

# 禁用 tcp_nodelay 同时启用 Nagle 算法：
tcp_nodelay off;
`

<a id=""tcp_nopush""></a>
###### 	cp_nopush

此选项仅在你使用 sendfile 时可用（NGINX 对使用 sendfile 服务的请求使用 	cp_nopush）。它使 NGINX 尝试将 HTTP 响应头发送在一个数据包中，而不是使用部分帧。这对于在调用 sendfile 之前添加请求头或进行吞吐量优化非常有用。

  > 通常，将 	cp_nopush 与 sendfile 一起使用是非常好的。然而，有些情况下它可能会减慢速度（特别是来自缓存系统的请求），因此，运行你自己的测试，看看它是否在这种情况下有用。

	cp_nopush 启用 TCP_CORK（更具体地说，FreeBSD 上的 TCP_NOPUSH 套接字选项或 Linux 上的 TCP_CORK 套接字选项），它积极累积数据，并告诉 TCP 等待应用程序移除软木塞后再发送任何数据包。

如果套接字上启用了 TCP_NOPUSH/TCP_CORK（它们不一样！），它将不会发送数据，直到缓冲区填充到固定限制（允许应用程序控制数据包的构建，例如用完整的 HTTP 响应填充一个数据包）。要了解更多并深入了解此选项的细节，请阅读 [TCP_CORK：比你想象中想知道的更多](https://baus.net/on-tcp_cork/)。

我曾读到过 	cp_nopush 与 	cp_nodelay 相反。我不同意这一点，因为根据我的理解，第一个是基于缓冲区压力聚合数据，而 Nagle 算法在等待返回 ACK 时聚合数据，而后者禁用了这一点。

	cp_nopush 和 	cp_nodelay 看起来是互斥的，但如果所有指令都打开，NGINX 会非常明智地管理它们：

- 确保数据包在发送给客户端之前是完整的
- 对于最后一个数据包，	cp_nopush 将被移除，允许 TCP 立即发送它，没有 200ms 的延迟

也让我们记住（看看 [Tony Finch 的笔记](http://dotat.at/writing/nopush.html)——此人开发了使 TCP_NOPUSH 像 TCP_CORK 一样工作的 FreeBSD 内核补丁）：

- 在 Linux 上，sendfile() 依赖于 TCP_CORK 套接字选项来避免不良的数据包边界
- FreeBSD 有一个类似的选项叫 TCP_NOPUSH
- 当 TCP_CORK 关闭时，任何缓冲的数据都会立即发送，但 TCP_NOPUSH 的情况并非如此

默认情况下 NGINX 禁用 TCP_NOPUSH 选项：

`
ginx
# http, server, location 上下文
# 启用 tcp_nopush（我的建议）：
tcp_nopush on;

# 禁用 tcp_nopush：
tcp_nopush off;   # 默认
`

<a id=""mixing-all-together""></a>
###### 全部组合使用

对此有许多看法。我的建议是将所有选项设置为 on。然而，我引用了一个有趣的评论（[混用 sendfile、tcp_nodelay 和 tcp_nopush 不合逻辑？](https://github.com/denji/nginx-tuning/issues/5)），它应该能消除任何疑虑：

  > _当设置时，指示始终排队非完整帧。随后用户清除此选项，我们传输队列中任何待处理的 partial 帧。这旨在与 sendfile() 一起使用，以便在用户（例如）必须先用 write() 调用写入请求头，然后使用 sendfile 发送数据部分时正确填充帧。TCP_CORK 可以与 TCP_NODELAY 一起设置，并且它比 TCP_NODELAY 更强。_

总结：

- 	cp_nodelay on; 通常与 	cp_nopush on; 不一致，因为它们是互斥的
- NGINX 有特殊行为：如果你有 sendfile on;，它对除最后一个包之外的所有包使用 TCP_NOPUSH
- 然后关闭 TCP_NOPUSH 并启用 TCP_NODELAY 以避免 200ms 的 ACK 延迟

因此，最重要的更改如下：

`
ginx
sendfile on;
tcp_nopush on;    # 有了这个，tcp_nodelay 就无关紧要了
`


<a id=""request-processing-stages""></a>
#### 请求处理阶段

  > 在构建过滤规则（例如使用 llow/deny）时，你应该始终记住测试它们，并了解每个阶段发生了什么（使用了哪些模块）。关于潜在问题的更多信息，请查看 [allow 和 deny](#allow-and-deny) 章节以及 [注意你的 ACL 规则 - 加固 - P1](RULES.md#beginner-take-care-about-your-acl-rules)。

NGINX 处理（处理）一个请求时，总共可以有 11 个阶段：

- NGX_HTTP_POST_READ_PHASE - 第一阶段，读取请求头
  - 示例模块：[ngx_http_realip_module](https://nginx.org/en/docs/http/ngx_http_realip_module.html)

- NGX_HTTP_SERVER_REWRITE_PHASE - 实现 server 块中定义的 rewrite 指令；使用 PCRE 正则表达式更改请求 URI、返回重定向并有条件地选择配置
  - 示例模块：[ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)

- NGX_HTTP_FIND_CONFIG_PHASE - 根据 URI 替换 location（location 查找）

- NGX_HTTP_REWRITE_PHASE - 在 location 级别进行 URI 转换
  - 示例模块：[ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)

- NGX_HTTP_POST_REWRITE_PHASE - URI 转换后处理（请求被重定向到新的 location）
  - 示例模块：[ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)

- NGX_HTTP_PREACCESS_PHASE - 认证预处理请求限制、连接限制（访问限制）
  - 示例模块：[ngx_http_limit_req_module](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html)、[ngx_http_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)、[ngx_http_realip_module](https://nginx.org/en/docs/http/ngx_http_realip_module.html)

- NGX_HTTP_ACCESS_PHASE - 验证客户端（认证过程，限制访问）
  - 示例模块：[ngx_http_access_module](https://nginx.org/en/docs/http/ngx_http_access_module.html)、[ngx_http_auth_basic_module](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)

- NGX_HTTP_POST_ACCESS_PHASE - 访问限制检查后处理阶段，认证过程，处理 satisfy any 指令
  - 示例模块：[ngx_http_access_module](https://nginx.org/en/docs/http/ngx_http_access_module.html)、[ngx_http_auth_basic_module](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)

- NGX_HTTP_PRECONTENT_PHASE - 生成内容
  - 示例模块：[ngx_http_try_files_module](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files)

- NGX_HTTP_CONTENT_PHASE - 内容处理
  - 示例模块：[ngx_http_index_module](https://nginx.org/en/docs/http/ngx_http_index_module.html)、[ngx_http_autoindex_module](https://nginx.org/en/docs/http/ngx_http_autoindex_module.html)、[ngx_http_gzip_module](https://nginx.org/en/docs/http/ngx_http_gzip_module.html)

- NGX_HTTP_LOG_PHASE - 日志处理
  - 示例模块：[ngx_http_log_module](https://nginx.org/en/docs/http/ngx_http_log_module.html)

你现在可能感到困惑（我也是……），所以我允许自己放置这个精彩且简单的预览：

<p align="center">
  <a href="https://www.nginx.com/resources/library/infographic-inside-nginx/">
    <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/request-flow.png" alt="request-flow">
  </a>
</p>

<sup><i>此信息图来自 [NGINX 内部](https://www.nginx.com/resources/library/infographic-inside-nginx/) 官方库。</i></sup>

在每个阶段，你可以注册任意数量的处理程序。每个阶段都有一个与之关联的处理程序列表。

我建议阅读关于 [Nginx 中 HTTP 请求处理阶段](http://scm.zoomquiet.top/data/20120312173425/index.html) 的精彩解释，当然还有官方的[开发指南](http://nginx.org/en/docs/dev/development_guide.html)。我还准备了一个简单的图表，可以帮助你了解每个阶段使用了哪些模块。它还包含来自官方开发指南的简短描述：

<p align="center">
  <a href="http://nginx.org/en/docs/dev/development_guide.html#http_phases">
    <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/nginx_phases.png" alt="nginx_phases">
  </a>
</p>


<a id=""server-blocks-logic""></a>
#### Server 块逻辑

  > NGINX 有 **server 块**（类似于 Apache 中的虚拟主机），它使用 listen 指令绑定到 TCP 套接字，并使用 server_name 指令标识虚拟主机。

以下是两个 server 块上下文及多个正则表达式的简短示例：

`
ginx
http {

  index index.html;
  root /var/www/example.com/default;

  server {

    listen 10.10.250.10:80;
    server_name www.example.com;

    access_log logs/example.access.log main;

    root /var/www/example.com/public;

    location ~ ^/(static|media)/ { ... }

    location ~* /[0-9][0-9](-.*)(\.html)$ { ... }

    location ~* \.(jpe?g|png|gif|ico)$ { ... }

    location ~* (?<begin>.*app)/(?<end>.+\.php)$ { ... }

    ...

  }

  server {

    listen 10.10.250.11:80;
    server_name "~^(api.)?example\.com api.de.example.com";

    access_log logs/example.access.log main;

    location ~ ^(/[^/]+)/api(.*)$ { ... }

    location ~ ^/backend/id/([a-z]\.[a-z]*) { ... }

    ...

  }

}
`

<a id=""handle-incoming-connections""></a>
##### 处理传入连接

  > **:bookmark: [使用 address:port 对定义 listen 指令 - 基本规则 - P1](RULES.md#beginner-define-the-listen-directives-with-addressport-pair)**<br>
  > **:bookmark: [阻止处理具有未定义服务器名称的请求 - 基本规则 - P1](RULES.md#beginner-prevent-processing-requests-with-undefined-server-names)**<br>
  > **:bookmark: [绝不在 listen 或 upstream 指令中使用主机名 - 基本规则 - P1](RULES.md#beginner-never-use-a-hostname-in-a-listen-or-upstream-directives)**<br>
  > **:bookmark: [尽可能在 server_name 指令中使用确切名称 - 性能 - P2](RULES.md#beginner-use-exact-names-in-a-server_name-directive-if-possible)**<br>
  > **:bookmark: [为 80 和 443 端口分别设置 listen 指令 - 基本规则 - P3](RULES.md#beginner-separate-listen-directives-for-80-and-443-ports)**<br>
  > **:bookmark: [为 listen 指令仅使用一个 SSL 配置 - 基本规则 - P3](RULES.md#beginner-use-only-one-ssl-config-for-the-listen-directive)**

NGINX 使用以下逻辑来确定应该使用哪个虚拟服务器（server 块）：

1. 将 ddress:port 对匹配到 listen 指令——可能有多个具有相同特异性的 listen 指令的 server 块可以处理请求

    > NGINX 使用 ddress:port 组合来处理传入连接。这对被分配给 listen 指令。

    listen 指令可以设置为：

      - IP 地址/端口组合（127.0.0.1:80;）

      - 单独的 IP 地址，如果只给出地址，则使用端口 80（127.0.0.1;）——变为 127.0.0.1:80;

      - 单独的端口，将监听该端口上的所有接口（80; 或 *:80;）——变为  .0.0.0:80;

      - UNIX 域套接字的路径（unix:/var/run/nginx.sock;）

    如果 listen 指令不存在，则使用 *:80（以超级用户权限运行时），否则使用 *:8000。

    用 listen 指令操作时，NGINX 必须遵循以下步骤：

      - NGINX 通过用默认值替换缺失的值来转换所有不完整的 listen 指令（见上文）

      - NGINX 尝试收集基于 ddress:port 最具体匹配请求的 server 块列表

      - 如果存在匹配的块列出特定 IP，则任何功能上使用  .0.0.0 的块都不会被选中

      - 如果只有一个最具体的匹配，则使用该 server 块来服务请求

      - 如果有多个具有相同匹配级别的 server 块，NGINX 随后开始评估每个 server 块的 server_name 指令

    看这个简短的例子：

    `
ginx
    # 客户端：
    GET / HTTP/1.0
    Host: api.random.com

    # 服务器端：
    server {

      # 此块将被处理：
      listen 192.168.252.10;  # --> 192.168.252.10:80

      ...

    }

    server {

      listen 80;  # --> *:80 --> 0.0.0.0:80
      server_name api.random.com;

      ...

    }
    `

2. 将 Host 请求头字段作为字符串匹配到 server_name 指令（确切名称哈希表）

3. 将 Host 请求头字段匹配到开头带有通配符的 server_name 指令（以星号开头的通配符名称哈希表）

  > 如果找到匹配，该块将用于服务请求。如果找到多个匹配，将使用最长匹配来服务请求。

4. 将 Host 请求头字段匹配到结尾带有通配符的 server_name 指令（以星号结尾的通配符名称哈希表）

  > 如果找到匹配，该块用于服务请求。如果找到多个匹配，将使用最长匹配来服务请求。

5. 将 Host 请求头字段作为正则表达式匹配到 server_name 指令

  > 第一个具有与 Host 请求头匹配的正则表达式的 server_name 将用于服务请求。

6. 如果所有 Host 请求头都不匹配，则转到标记为 default_server 的 listen 指令（使该 server 块响应所有不匹配任何 server 块的请求）

7. 如果所有 Host 请求头都不匹配且没有 default_server，则转到第一个具有满足第一步的 listen 指令的 server

8. 最后，NGINX 进入 location 上下文

<sup><i>此列表基于 [掌握 Nginx - 虚拟服务器部分](https://github.com/trimstray/nginx-admins-handbook#mastering-nginx)。</i></sup>


<a id=""matching-location""></a>
##### 匹配 location

  > **:bookmark: [使用精确 location 匹配来加速选择过程 - 性能 - P3](RULES.md#beginner-make-an-exact-location-match-to-speed-up-the-selection-process)**

  > 对于每个请求，NGINX 会经过一个过程来选择将用于服务该请求的最佳 location 块。

location 块使你能在 server 块内处理多种类型的 URI/路由（基于 URL 的第 7 层路由）。语法如下：

`
location optional_modifier location_match { ... }
`

上面的 location_match 定义了 NGINX 应检查请求 URI 的依据。下面的 optional_modifier 将导致关联的 location 块被解释如下（此时顺序无关紧要）：

- (none)：如果没有修饰符，则 location 被解释为前缀匹配。为了确定匹配，location 将与 URI 的开头进行匹配

- =：是精确匹配，不带任何通配符、前缀匹配或正则表达式；强制请求 URI 与 location 参数之间进行字面匹配

- ~：如果存在波浪号修饰符，则此 location 必须用于区分大小写的匹配（RE 匹配）

- ~*：如果使用波浪号和星号修饰符，则该 location 必须用于不区分大小写的匹配（RE 匹配）

- ^~：假设此块是最佳非 RE 匹配，则插入符后跟波浪号修饰符意味着不会进行 RE 匹配

现在，简要介绍 location 优先级：

- 精确匹配是最佳优先级（首先处理）；如果匹配则停止搜索

- 前缀匹配是第二优先级；有两种类型的前缀：^~ 和 (none)，如果此匹配使用了 ^~ 前缀，搜索停止

- 正则表达式匹配优先级最低；有两种类型的前缀：~ 和 ~*；按它们在配置文件中定义的顺序

- 如果正则表达式搜索产生了匹配，则使用该结果，否则使用前缀搜索的匹配

所以，看这个例子，它来自 [Nginx 文档 - ngx_http_core_module](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)：

`
location = / {
  # 只匹配 / 查询。
  [ configuration A ]
}
location / {
  # 匹配任何查询，因为所有查询都以 / 开头，但正则表达式
  # 和任何更长的传统块将首先被匹配。
  [ configuration B ]
}
location /documents/ {
  # 匹配任何以 /documents/ 开头的查询，并继续搜索，
  # 因此将检查正则表达式。仅当正则表达式找不到匹配时
  # 才会匹配此配置。
  [ configuration C ]
}
location ^~ /images/ {
  # 匹配任何以 /images/ 开头的查询，并停止搜索，
  # 因此不会检查正则表达式。
  [ configuration D ]
}
location ~* \.(gif|jpg|jpeg)$ {
  # 匹配任何以 gif、jpg 或 jpeg 结尾的请求。然而，
  # 对 /images/ 目录的所有请求将由配置 D 处理。
  [ configuration E ]
}
`

为了帮助你理解 location 匹配是如何工作的：

- [Nginx location 匹配测试器](https://nginx.viraptor.info/)
- [Nginx location 匹配可视化](https://detailyang.github.io/nginx-location-match-visible/)
- [NGINX 正则表达式测试器](https://github.com/nginxinc/NGINX-Demos/tree/master/nginx-regex-tester)

选择 NGINX location 块的过程如下（详细说明）：

1. NGINX 搜索精确匹配。如果 = 修饰符（例如 location = foo { ... }）与请求 URI 完全匹配，则立即选择此特定 location 块

   - 处理此块
   - 匹配搜索停止

2. 基于前缀的 NGINX location 匹配（无正则表达式）。每个 location 将与请求 URI 进行检查。如果未找到精确（即没有 = 修饰符）location 块，NGINX 将继续使用非精确前缀。它从该 URI 的最长匹配前缀 location 开始，采用以下方法：

   - 如果最长匹配前缀 location 具有 ^~ 修饰符（例如 location ^~ foo { ... }），NGINX 将立即停止搜索并选择此 location

     - 处理这些匹配中最长（最显式）的块
     - 匹配搜索停止

   - 假设最长匹配前缀 location 未使用 ^~ 修饰符，则暂时存储匹配并继续处理

  > 我不确定顺序。在官方文档中没有明确说明，外部指南的解释也不同。先检查最长匹配前缀 location 似乎是合乎逻辑的。

3. 一旦选择了最长匹配前缀 location 并存储，NGINX 继续评估区分大小写（例如 location ~ foo { ... }）和不区分大小写的正则表达式（例如 location ~* foo { ... }）location。第一个适合 URI 的正则表达式 location 被立即选中来处理请求

   - 处理第一个匹配的正则表达式的块（按配置文件从上到下的顺序解析时）
   - 匹配搜索停止

4. 如果没有找到与请求 URI 匹配的正则表达式 location，则选择先前存储的前缀 location（例如 location foo { ... }）来服务请求

   - location / 是一种 catch-all location
   - 处理这些匹配中最长（最显式）的块
   - 匹配搜索停止

你还应该知道，非正则表达式匹配类型是完全声明式的——定义顺序在配置中无关紧要——但获胜的正则表达式匹配（如果处理到了那一步）完全基于其在配置文件中的输入顺序。

为了更好地理解这个过程，请参阅这份简短的备忘单，它将允许你以可预测的方式设计 location 块：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/nginx_location_cheatsheet.png" alt="nginx-location-cheatsheet">
</p>

  > 我建议使用外部工具测试正则表达式。更多信息请参见[在线工具](https://github.com/trimstray/nginx-admins-handbook#online-tools)章节。

好的，这是一个更复杂的配置：

`
ginx
server {

 listen 80;
 server_name xyz.com www.xyz.com;

 location ~ ^/(media|static)/ {
  root /var/www/xyz.com/static;
  expires 10d;
 }

 location ~* ^/(media2|static2) {
  root /var/www/xyz.com/static2;
  expires 20d;
 }

 location /static3 {
  root /var/www/xyz.com/static3;
 }

 location ^~ /static4 {
  root /var/www/xyz.com/static4;
 }

 location = /api {
  proxy_pass http://127.0.0.1:8080;
 }

 location / {
  proxy_pass http://127.0.0.1:8080;
 }

 location /backend {
  proxy_pass http://127.0.0.1:8080;
 }

 location ~ logo.xcf$ {
  root /var/www/logo;
  expires 48h;
 }

 location ~* .(png|ico|gif|xcf)$ {
  root /var/www/img;
  expires 24h;
 }

 location ~ logo.ico$ {
  root /var/www/logo;
  expires 96h;
 }

 location ~ logo.jpg$ {
  root /var/www/logo;
  expires 48h;
 }

}
`

查看结果表：

| <b>URL</b> | <b>找到的 LOCATION</b> | <b>最终匹配</b> |
| :---         | :---         | :---         |
| / | <sup>1)</sup> 前缀匹配 / | / |
| /css | <sup>1)</sup> 前缀匹配 / | / |
| /api | <sup>1)</sup> 精确匹配 /api | /api |
| /api/ | <sup>1)</sup> 前缀匹配 / | / |
| /backend | <sup>1)</sup> 前缀匹配 /<br><sup>2)</sup> 前缀匹配 /backend | /backend |
| /static | <sup>1)</sup> 前缀匹配 / | / |
| /static/header.png | <sup>1)</sup> 前缀匹配 /<br><sup>2)</sup> 区分大小写正则表达式匹配 ^/(media|static)/ | ^/(media|static)/ |
| /static/logo.jpg | <sup>1)</sup> 前缀匹配 /<br><sup>2)</sup> 区分大小写正则表达式匹配 ^/(media|static)/ | ^/(media|static)/ |
| /media2 | <sup>1)</sup> 前缀匹配 /<br><sup>2)</sup> 不区分大小写正则表达式匹配 ^/(media2|static2) | ^/(media2|static2) |
| /media2/ | <sup>1)</sup> 前缀匹配 /<br><sup>2)</sup> 不区分大小写正则表达式匹配 ^/(media2|static2) | ^/(media2|static2) |
| /static2/logo.jpg | <sup>1)</sup> 前缀匹配 /<br><sup>2)</sup> 不区分大小写正则表达式匹配 ^/(media2|static2) | ^/(media2|static2) |
| /static2/logo.png | <sup>1)</sup> 前缀匹配 /<br><sup>2)</sup> 不区分大小写正则表达式匹配 ^/(media2|static2) | ^/(media2|static2) |
| /static3/logo.jpg | <sup>1)</sup> 前缀匹配 /static3<br><sup>2)</sup> 前缀匹配 /<br><sup>3)</sup> 区分大小写正则表达式匹配 logo.jpg$ | logo.jpg$ |
| /static3/logo.png | <sup>1)</sup> 前缀匹配 /static3<br><sup>2)</sup> 前缀匹配 /<br><sup>3)</sup> 不区分大小写正则表达式匹配 .(png|ico|gif|xcf)$ | .(png|ico|gif|xcf)$ |
| /static4/logo.jpg | <sup>1)</sup> 优先级前缀匹配 /static4<br><sup>2)</sup> 前缀匹配 / | /static4 |
| /static4/logo.png | <sup>1)</sup> 优先级前缀匹配 /static4<br><sup>2)</sup> 前缀匹配 / | /static4 |
| /static5/logo.jpg | <sup>1)</sup> 前缀匹配 /<br><sup>2)</sup> 区分大小写正则表达式匹配 logo.jpg$ | logo.jpg$ |
| /static5/logo.png | <sup>1)</sup> 前缀匹配 /<br><sup>2)</sup> 不区分大小写正则表达式匹配 .(png|ico|gif|xcf)$ | .(png|ico|gif|xcf)$ |
| /static5/logo.xcf | <sup>1)</sup> 前缀匹配 /<br><sup>2)</sup> 区分大小写正则表达式匹配 logo.xcf$ | logo.xcf$ |
| /static5/logo.ico | <sup>1)</sup> 前缀匹配 /<br><sup>2)</sup> 不区分大小写正则表达式匹配 .(png|ico|gif|xcf)$ | .(png|ico|gif|xcf)$ |


<a id=""rewrite-vs-return""></a>
##### ewrite vs eturn

通常有两种在 NGINX 中实现重定向的方法：使用 ewrite 指令和 eturn 指令。

这些指令（来自 
gx_http_rewrite_module）非常有用，但（根据 NGINX 文档）在 location 上下文中的 if 内唯一 100% 安全的事情是：

- eturn ...;
- ewrite ... last;

其他任何操作都可能导致不可预测的行为，包括潜在的 SIGSEGV。

<a id=""rewrite-directive""></a>
###### ewrite 指令

ewrite 指令按其在配置文件中出现的顺序依次执行。它比 eturn 慢（但仍然非常快），并且在所有情况下都返回 HTTP 302，与 permanent 无关。

ewrite 指令只更改请求 URI，而不是请求的响应。重要的是，只有与正则表达式匹配的原始 URL 部分会被重写。它可用于临时 URL 更改。

我有时使用 ewrite 来捕获原始 URL 中的元素、更改或添加路径中的元素，以及通常在做更复杂的事情时使用：

`
ginx
location / {

  ...

  rewrite ^/users/(.*)$ /user.php?username= last;

  # 或者：
  rewrite ^/users/(.*)/items$ /user.php?username=&page=items last;

}
`

  > 你必须知道 rewrite 只返回 301 或 302 代码。

ewrite 指令接受可选标志：

- reak - 基本完成 rewrite 指令的处理，停止处理，并通过不进行任何 location 查找和内部跳转来打破 location 查找循环

  - 如果在 location 块内使用 reak 标志：

    - 不再解析 rewrite 条件
    - 内部引擎继续解析当前的 location 块

    _在 location 块内，使用 reak，NGINX 只停止处理更多的 rewrite 条件。_

  - 如果在 location 块外使用 reak 标志：

    - 不再解析 rewrite 条件
    - 内部引擎进入下一阶段（搜索 location 匹配）

    _在 location 块外，使用 reak，NGINX 停止处理更多的 rewrite 条件。_

- last - 基本完成 rewrite 指令的处理，停止处理，并开始搜索与更改后的 URI 匹配的新 location

  - 如果在 location 块内使用 last 标志：

    - 不再解析 rewrite 条件
    - 内部引擎**开始查找**基于 rewrite 结果的其他 location 匹配
    - 即使在下一个 location 匹配中，也不再解析 rewrite 条件

    _在 location 块内，使用 last，NGINX 停止处理更多的 rewrite 条件，然后**开始查找**新的 location 块匹配。NGINX 也会忽略新 location 块中的任何 rewrite。_

  - 如果在 location 块外使用 last 标志：

    - 不再解析 rewrite 条件
    - 内部引擎进入下一阶段（搜索 location 匹配）

    _在 location 块外，使用 last，NGINX 停止处理更多的 rewrite 条件。_

- edirect - 返回带有 302 HTTP 响应代码的临时重定向
- permanent - 返回带有 301 HTTP 响应代码的永久重定向

注意：

- 在 location 块外，last 和 reak 实际上是相同的
- 服务器级别的 rewrite 指令处理可以通过 reak 停止，但 location 查找无论如何都会进行

<sup><i>此解释基于 [Pothi Kalimuthu](https://serverfault.com/users/102173/pothi-kalimuthu) 对 [nginx url rewriting: difference between break and last](https://serverfault.com/a/829148) 的精彩回答。</i></sup>

官方文档有关于[创建 NGINX Rewrite 规则](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)和[转换 rewrite 规则](https://nginx.org/en/docs/http/converting_rewrite_rules.html)的优秀教程。我还推荐[使用 Nginx 进行 Clean URL 重写](https://www.codesmite.com/article/clean-url-rewrites-using-nginx)。

最后，看看 last 和 reak 标志在实际中的区别：

- last 指令：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/rewrites/last_01.jpeg" alt="last">
</p>

- reak 指令：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/rewrites/break_01.jpeg" alt="break">
</p>

<sup><i>此信息图来自 [内部重写 - nginx](https://www.linkedin.com/pulse/internal-rewrite-nginx-ivan-dabi%C4%87)，作者 [Ivan Dabic](https://www.linkedin.com/in/ivan-dabic)。</i></sup>

<a id=""return-directive""></a>
###### eturn 指令

  > **:bookmark: [使用 return 指令进行 URL 重定向 (301, 302) - 基本规则 - P2](RULES.md#beginner-use-return-directive-for-url-redirection-301-302)**<br>
  > **:bookmark: [使用 return 指令代替 rewrite 进行重定向 - 性能 - P2](RULES.md#beginner-use-return-directive-instead-of-rewrite-for-redirects)**

另一种方式是 eturn 指令。它比 rewrite 更快，因为没有需要评估的正则表达式。它会停止处理并向客户端返回 HTTP 301（默认情况）（告诉 NGINX 直接响应请求），整个 URL 被重定向到指定的 URL。

我在以下情况下使用 eturn 指令：

- 强制从 http 重定向到 https：

  `
ginx
  server {

    ...

    return 301 https://example.com;

  }
  `

- 从 www 重定向到非 www 或反之：

  `
ginx
  server {

    ...

    # 这只是示例。你不应在以下情况下使用 'if' 语句：
    if (System.Management.Automation.Internal.Host.InternalHost = www.example.com) {

      return 301 https://example.com;

    }

  }
  `

- 关闭连接并内部记录：

  `
ginx
  server {

    ...

    return 444;

  }
  `

- 向客户端发送 4xx HTTP 响应而不执行其他任何操作：

  `
ginx
  server {

    ...

    if ( = POST) {

      return 405;

    }

    # 或者：
    if () {

      return 403;

    }

    # 或者：
    if ( ~ "^/app/(.+)$") {

      return 403;

    }

    # 或者：
    location ~ ^/(data|storage) {

      return 403;

    }

  }
  `

- 偶尔用于回复 HTTP 代码而不提供文件或响应体：

  `
ginx
  server {

    ...

    # NGINX 不允许 200 没有响应体（200 需要在响应中附带资源。
    # '204 No Content' 意为 "我已经完成了请求，但没有要返回的主体")：
    return 204 "it's all okay";
    # 或者不带主体：
    return 204;

    # 因为默认 Content-Type 是 application/octet-stream，浏览器会提供 "保存文件"。
    # 如果你想在浏览器中看到回复，你应该添加适当的 Content-Type：
    # add_header Content-Type text/plain;

  }
  `

对于最后一个示例：如果你使用这种配置进行健康检查，请注意。虽然 204 HTTP 代码对于健康检查（成功指示，无内容）在语义上是完美的，但某些服务不认为它是成功的。

<a id=""url-redirections""></a>
###### URL 重定向

  > **:bookmark: [使用 return 指令进行 URL 重定向 (301, 302) - 基本规则 - P2](RULES.md#beginner-use-return-directive-for-url-redirection-301-302)**<br>
  > **:bookmark: [使用 return 指令代替 rewrite 进行重定向 - 性能 - P2](RULES.md#beginner-use-return-directive-instead-of-rewrite-for-redirects)**

HTTP 允许服务器将客户端请求重定向到不同的位置。这在你将内容移动到新 URL、删除页面或更改域名或合并网站时非常有用。

URL 重定向的原因有多种：

- 用于 URL 缩短
- 防止网页移动时出现死链
- 允许属于同一所有者的多个域名指向同一个网站
- 引导进出网站的导航
- 为隐私保护
- 用于恶意目的，如钓鱼攻击或恶意软件分发

<sup><i>来自 [维基百科 - URL 重定向](https://en.wikipedia.org/wiki/URL_redirection)。</i></sup>

我建议阅读：

- [HTTP 中的重定向](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections)
- [301 101：重定向如何工作](https://www.digitalthirdcoast.com/blog/301-101-redirects-work)
- [修改 301/302 响应体（来自本手册）](HELPERS.md#modify-301302-response-body)
- [将带有负载的 POST 请求重定向到外部端点（来自本手册）](HELPERS.md#redirect-post-request-with-payload-to-external-endpoint)


<a id=""try_files-directive""></a>
##### 	ry_files 指令

我们还有一个非常有趣且重要的指令：	ry_files（来自 
gx_http_core_module）。该指令告诉 NGINX 检查一组命名文件或目录的存在（有条件地检查文件，成功时停止）。

我认为最好的解释来自官方文档：

  > _	ry_files 按指定顺序检查文件的存在，并使用第一个找到的文件进行请求处理；处理在当前上下文中执行。文件的路径是根据 oot 和 lias 指令从文件参数构造的。可以通过在名称末尾指定斜杠来检查目录的存在，例如 $uri/。如果没有找到任何文件，则进行内部重定向到最后一个参数中指定的 URI。_

通常它可以在一个指令中检查磁盘上的文件、重定向到代理或内部 location 以及返回错误代码。

看看下面的例子：

`
ginx
server {

  ...

  root /var/www/example.com;

  location / {

    try_files  / /frontend/index.html;

  }

  location ^~ /images {

    root /var/www/static;
    try_files  / =404;

  }

  ...
`

- 所有 location 的默认根目录是 /var/www/example.com

- location / - 匹配所有没有更具体 location 的 location，例如确切名称

  - 	ry_files  - 当你收到一个与此块匹配的 URI 时，首先尝试 $uri

    > 例如：https://example.com/tools/en.js - NGINX 将尝试检查 /tools 目录下是否存在名为 en.js 的文件，如果找到，首先提供它。

  - 	ry_files  / - 如果第一个条件未找到，则尝试将 URI 作为目录

    > 例如：https://example.com/backend/ - NGINX 将首先检查是否存在名为 ackend 的文件，如果找不到，则转到第二个检查 $uri/，看看是否存在名为 ackend 的目录，然后尝试提供它。

  - 	ry_files  / /frontend/index.html - 如果文件和目录都未找到，NGINX 发送 /frontend/index.html

- location ^~ /images - 处理任何以 /images 开头的查询并停止搜索

  - 此 location 的默认根目录是 /var/www/static

  - 	ry_files  - 当你收到一个与此块匹配的 URI 时，首先尝试 $uri

    > 例如：https://example.com/images/01.gif - NGINX 将尝试检查 /images 目录下是否存在名为  1.gif 的文件，如果找到，首先提供它。

  - 	ry_files  / - 如果第一个条件未找到，则尝试将 URI 作为目录

    > 例如：https://example.com/images/ - NGINX 将首先检查是否存在名为 images 的文件，如果找不到，则转到第二个检查 $uri/，看看是否存在名为 images 的目录，然后尝试提供它。

  - 	ry_files  / =404 - 如果文件和目录都未找到，NGINX 发送 HTTP 404（未找到）

另一方面，	ry_files 相对原始。当遇到时，NGINX 会在与 location 块匹配的目录中查找指定的任何文件物理上是否存在。如果它们不存在，NGINX 会执行内部重定向到指令中的最后一个条目。

此外，考虑不要检查目录的存在：

`
ginx
# 使用此方法省去额外的文件系统 stat()：
try_files  @index;

# 而不是这样：
try_files  / @index;
`

<a id=""if-break-and-set""></a>
##### if、reak 和 set

  > **:bookmark: [避免使用 if 指令检查 server_name - 性能 - P2](RULES.md#beginner-avoid-checks-server_name-with-if-directive)**


gx_http_rewrite_module 还提供了其他指令：

- reak - 停止处理，如果在 location 内指定，请求的进一步处理在此 location 中继续：

  `
ginx
  # 用于以下情况：
  if () {

    limit_rate 50k;
    break;

  }
  `

- if - 你可以在 server 内使用 if，但不能反过来，还要注意你不应在 location 内使用 if，因为它可能无法按预期工作。例如，if 语句不是设置自定义请求头的好方法，因为它们可能导致 if 块外的语句被忽略。NGINX 文档说：

  > _有些情况下你无法避免使用 if，例如如果你需要测试一个没有等效指令的变量。_

  你还应该记住：

  > _NGINX 中的 if 上下文由 rewrite 模块提供，这是该上下文的主要预期用途。由于 NGINX 会通过许多其他专用指令来测试请求条件，因此 if **不应**用于大多数形式的条件执行。这是一个非常重要的说明，NGINX 社区创建了一个名为 [if is evil](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/) 的页面（是的，它确实是邪恶的，并且在大多数情况下不需要）。_

  很久以前我发现了这个：

  > _这实际上是不正确的，表明你不了解问题所在。当 if 语句以 eturn 指令结束时，没有问题，使用是安全的。_

  另一方面，官方文档说：

  > _指令 if 在 location 上下文中使用时存在问题，在某些情况下它不会按预期工作，而是做一些完全不同的东西。在某些情况下它甚至会段错误。一般来说，尽可能避免使用它是个好主意。_

- set - 为指定的变量设置一个值。该值可以包含文本、变量及其组合

if 和 set 指令的使用示例：

`
ginx
# 来自：https://gist.github.com/jrom/1760790：
if ( = /) {

  set  A;

}

if (System.Management.Automation.Internal.Host.InternalHost ~* example.com) {

  set  "B";

}

if ( !~* "auth_token") {

  set  "C";

}

if ( = ABC) {

  proxy_pass http://cms.example.com;
  break;

}
`

<a id=""root-vs-alias""></a>
##### oot vs lias

  > 在 location 块中放置 oot 或 lias 指令会覆盖在更高作用域应用的 oot 或 lias 指令。

使用 lias，你可以映射到另一个文件名。使用 oot 强制你在服务器上命名你的文件。在第一种情况下，NGINX 将 URL 路径中的字符串前缀（例如 /robots.txt）替换为（例如 /var/www/static/robots.01.txt），然后使用结果作为文件系统路径。在第二种情况下，NGINX 将字符串（例如 /var/www/static/）插入到 URL 路径的开头，然后使用结果作为文件系统路径。

看这个。当 lias 用于整个目录时，会有区别：

`
ginx
location ^~ /data/ { alias /home/www/static/data/; }
`

但以下代码不行：

`
ginx
location ^~ /data/ { root /home/www/static/data/; }
`

这必须是：

`
ginx
location ^~ /data/ { root /home/www/static/; }
`

oot 指令通常放置在 server 和 location 块中。将 oot 指令放在 server 块中会使 oot 指令对同一 server 块内的所有 location 块可用。

此指令告诉 NGINX 获取请求 URL 并将其附加到指定目录后面。例如，使用以下配置块：

`
ginx
server {

  server_name example.com;
  listen 10.250.250.10:80;

  index index.html;
  root /var/www/example.com;

  location / {

    try_files  / =404;

  }

  location ^~ /images {

    root /var/www/static;
    try_files  / =404;

  }

}
`

NGINX 将请求映射到：

- http://example.com/images/logo.png 映射到文件路径 /var/www/static/images/logo.png
- http://example.com/contact.html 映射到文件路径 /var/www/example.com/contact.html
- http://example.com/about/us.html 映射到文件路径 /var/www/example.com/about/us.html

就像你想将所有以 /static 开头的请求转发到 /var/www/static，你应该设置：

- 第一个路径：/var/www
- 最后一个路径：/static
- 完整路径：/var/www/static

`
ginx
location <last path> {

  root <first path>;

  ...

}
`

NGINX 关于 lias 指令的文档建议，当 location 与指令值的最后部分匹配时，最好使用 oot 而不是 lias。

lias 指令只能放置在 location 块中。以下是说明如何应用 lias 指令的一组配置：

`
ginx
server {

  server_name example.com;
  listen 10.250.250.10:80;

  index index.html;
  root /var/www/example.com;

  location / {

    try_files  / =404;

  }

  location ^~ /images {

    alias /var/www/static;
    try_files  / =404;

  }

}
`

NGINX 将请求映射到：

- http://example.com/images/logo.png 映射到文件路径 /var/www/static/logo.png
- http://example.com/images/ext/img.png 映射到文件路径 /var/www/static/ext/img.png
- http://example.com/contact.html 映射到文件路径 /var/www/example.com/contact.html
- http://example.com/about/us.html 映射到文件路径 /var/www/example.com/about/us.html

当 location 与指令值的最后部分匹配时，最好使用 root 指令（这似乎是一个任意的风格选择，因为作者根本没有证明该指令的合理性）。看这个来自官方文档的例子：

`
ginx
location /images/ {

  alias /data/w3/images/;

}

# 更好的解决方案：
location /images/ {

  root /data/w3;

}
`


<a id=""internal-directive""></a>
##### internal 指令

此指令指定 location 块是内部的。换句话说，外部请求无法访问指定的资源。

另一方面，它指定了如何处理外部重定向，即像 http://example.com/app.php/some-path 这样的 location；设置后，它们应返回 404，只允许内部重定向。简而言之，这告诉 NGINX 它不能从外部访问（它不重定向任何内容）。

作为内部重定向处理的条件在 internal 指令的文档中列出。指定给定 location 只能用于内部请求，条件如下：

- 由 error_page、index、andom_index 和 	ry_files 指令重定向的请求
- 由上游服务器的 X-Accel-Redirect 响应头字段重定向的请求
- 由 
gx_http_ssi_module 模块的 include virtual 命令、
gx_http_addition_module 模块指令以及 uth_request 和 mirror 指令形成的子请求
- 由 ewrite 指令更改的请求

示例 1：

`
ginx
error_page 404 /404.html;

location = /404.html {

  internal;

}
`

示例 2：

文件从目录 /srv/hidden-files 通过路径前缀 /hidden-files/ 提供。非常简单。internal 声明告诉 NGINX，此路径只能通过 NGINX 配置中的 rewrite 或通过代理响应中的 X-Accel-Redirect 请求头进行访问。

要使用此功能，只需返回一个包含该请求头的空响应。请求头的内容应该是你想要重定向到的 location：

`
ginx
location /hidden-files/ {

  internal;
  alias /srv/hidden-files/;

}
`

示例 3：

NGINX 中内部重定向的另一个用例是隐藏凭据。通常你需要向第三方服务发出请求。例如，你想要发送短信或访问付费地图服务器。最有效的方法是直接从你的 JavaScript 前端发送这些请求。然而，这样做意味着你必须在前端嵌入访问令牌。这意味着精明的用户可以提取此令牌并以你的帐户发出请求。

一个简单的修复方法是在后端中创建一个发起实际请求的端点。我们可以在后端中使用 HTTP 客户端库。然而，这又会占满工作进程，特别是如果你预计会有大量请求并且第三方服务响应非常慢。

`
ginx
location /external-api/ {

  internal;
  set  "";
  set  "";

  # 为了性能：
  proxy_buffering off;
  # 传递来自后端的密钥：
  proxy_set_header Authorization ;
  # 使用后端确定的 URI：
  proxy_pass ;

}
`

<sup><i>示例 2 和 3（都很棒！）来自 [如何在 NGINX 中使用内部重定向](https://clubhouse.io/developer-how-to/how-to-use-internal-redirects-in-nginx/)。</i></sup>

  > 每个请求有 10 个内部重定向的限制，以防止可能发生在错误配置中的请求处理循环。如果达到此限制，将返回错误 _HTTP 500 Internal Server Error_。在这种情况下，可以在错误日志中看到 ewrite or internal redirection cycle 消息。

另请参阅官方文档中的[基于子请求结果的认证](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-subrequest-authentication/)。

<a id=""external-and-internal-redirects""></a>
##### 外部和内部重定向

外部重定向直接来自客户端。因此，如果客户端获取了 https://example.com/directory，它将直接落入前面的 location 块。

内部重定向意味着它不会向客户端发送 302 响应，它只是执行 URL 的隐式重写，并尝试处理它，就像用户最初键入了新的 URL 一样。

内部重定向与由 HTTP 响应代码 302 和 301 定义的外部重定向不同，客户端浏览器不会更新其 URI 地址。

要开始内部重写，我们应该解释重定向和内部重写之间的区别。当源指向目标域之外的 destination 时，我们称之为重定向，因为你的请求将从源到达外部域/目标。

使用内部重写，你基本上也是在执行相同的操作，只是目标是在同一域下的本地路径，而不是外部 location。

还有关于内部重定向的[精彩解释](https://openresty.org/download/agentzh-nginx-tutorials-en.html#02-nginxdirectiveexecorder06)：

  > _内部重定向（例如通过 echo_exec 或 ewrite 指令）是一种操作，它使 NGINX 在处理请求时从一个 location 跳转到另一个 location（非常类似于 C 语言中的 goto 语句）。这种 "跳转" 完全发生在服务器内部。_

有两种不同类型的内部请求：

- **内部重定向** - 内部重定向客户端请求。URI 被更改，请求可能因此匹配另一个 location 块并适用于不同的设置。内部重定向最常见的例子是使用 ewrite 指令，它允许你重写请求 URI

- **子请求** - 内部触发的额外请求，用于生成（插入或附加到原始请求的正文中）与主请求互补的内容（ddition 或 ssi 模块）

<a id=""allow-and-deny""></a>
##### llow 和 deny

  > **:bookmark: [注意你的 ACL 规则 - 加固 - P1](RULES.md#beginner-take-care-about-your-acl-rules)**<br>
  > **:bookmark: [拒绝不安全的 HTTP 方法 - 加固 - P1](RULES.md#beginner-reject-unsafe-http-methods)**

两者都来自 
gx_http_access_module 模块，允许限制对某些客户端地址的访问。你可以组合 llow/deny 规则。

  > deny 总是返回 403 错误代码。

最简单的方法是从拒绝所有访问开始，然后只授予对那些你想要的位置的访问权限。例如：

`
ginx
location / {

  # 没有 'satisfy any' 两者都必须通过：
  satisfy any;
  allow 192.168.0/0/16;
  deny all;

  # sh -c "echo -n 'user:' >> /etc/nginx/.secret"
  # sh -c "openssl passwd -apr1 >> /etc/nginx/.secret"
  auth_basic "Restricted Area";
  auth_basic_user_file /etc/nginx/.secret;

  root   /usr/share/nginx/html;
  index  index.html index.htm;

}
`

在你的配置中放置 satisfy any; 告诉 NGINX 接受 HTTP 认证或 IP 限制。默认情况下，当你同时定义两者时，它将期望两者都通过。

另请参见[此答案](https://serverfault.com/a/748373)：

  > _正如你所发现的，不建议将认证设置放在服务器级别，因为它们将适用于所有 location。虽然可以关闭 basic auth，但似乎没有清除现有 IP 白名单的方法。_
  >
  > _更好的解决方案是将认证添加到 / location，这样就不会被 /hello 继承。_
  >
  > _如果你有其他需要 basic auth 和 IP 白名单的 location，则可能需要考虑将认证组件移动到 include 文件或将它们嵌套在 / 下。_

两个指令都可能出现意外行为！看下面的例子：

`
ginx
server {

  server_name example.com;

  deny all;

  location = /test {
    return 200 "it's all okay";
    more_set_headers 'Content-Type: text/plain';
  }

}
`

如果你生成一个请求：

`ash
curl -i https://example.com/test
HTTP/2 200
date: Wed, 11 Nov 2018 10:02:45 GMT
content-length: 13
server: Unknown
content-type: text/plain

it's all okay
`

为什么？请看[请求处理阶段](#request-processing-stages)章节。这是因为 NGINX 分阶段处理请求，ewrite 阶段（eturn 所在）在 ccess 阶段（deny 工作）之前。

<a id=""uri-vs-request_uri""></a>
##### uri vs equest_uri

  > **:bookmark: [使用 $request_uri 以避免使用正则表达式 - 性能 - P2](RULES.md#beginner-use-request_uri-to-avoid-using-regular-expressions)**

$request_uri 是原始请求（例如 /foo/bar.php?arg=baz 包含参数且不能被修改），但 $uri 指的是更改后的 URI，因此 $uri 不等同于 $request_uri。

看 [Richard Smith](https://stackoverflow.com/users/4862445/richard-smith) 的这个[精彩且简短的解释](https://stackoverflow.com/a/48709976)：

  > _$uri 变量设置为 NGINX 当前正在处理的 URI——但也会进行规范化，包括：_
  >
  > _- 删除 ? 和查询字符串_
  > _- 连续的 / 字符被替换为单个 /_
  > _- URL 编码的字符被解码_
  >
  > _$request_uri 的值始终是原始 URI，不受上述任何规范化的影响。_
  >
  > _大多数时候你会使用 $uri，因为它是规范化的。在错误的地方使用 $request_uri 可能导致 URL 编码的字符被双重编码。_

两者都不包含 schema（https:// 和端口（上面的两个示例中都是隐式 443）），如 [RFC 2616 - http URL](https://tools.ietf.org/html/rfc2616#section-3.2.2) <sup>[IETF]</sup> 为 URL 定义的：

`
http_URL = "http(s):" "//" host [ ":" port ] [ abs_path [ "?" query ]]
`

查看下表：

| <b>URL</b> | <b><code></code></b> | <b><code></code></b> |
| :---         | :---         | :---         |
| https://example.com/foo | /foo | /foo |
| https://example.com/foo/bar | /foo/bar | /foo/bar |
| https://example.com/foo/bar/ | /foo/bar/ | /foo/bar/ |
| https://example.com/foo/bar? | /foo/bar? | /foo/bar |
| https://example.com/foo/bar?do=test | /foo/bar?do=test | /foo/bar |
| https://example.com/rfc2616-sec3.html#sec3.2 | /rfc2616-sec3.html | /rfc2616-sec3.html |

另一种重复 location 的方法是使用 proxy_pass 指令，这非常简单：

`
ginx
location /app/ {

  proxy_pass http://127.0.0.1:5000;

  # 或者：
  proxy_pass http://127.0.0.1:5000/api/app/;

}
`

| <b>LOCATION</b> | <b><code>proxy_pass</code></b> | <b>请求</b> | <b>上游接收</b> |
| :---         | :---         | :---         | :---         |
| /app/ | http://localhost:5000/api | /app/foo?bar=baz | /api/webapp/foo?bar=baz |
| /app/ | http://localhost:5000/api | /app/foo?bar=baz | /api/webapp/foo |


<a id="english-anchor"></a>
#### 压缩和解压缩

  > **:bookmark: [缓解 CRIME/BREACH 攻击 - 加固规则 - P2](RULES.md#beginner-mitigation-of-crimebreach-attacks)**

默认情况下，NGINX 仅使用 `gzip` 方法压缩 MIME 类型为 text/html 的响应。因此，如果你发送包含 `Accept-Encoding: gzip` 请求头的请求，你将不会在响应中看到 `Content-Encoding: gzip`。

要启用 `gzip` 压缩：

```nginx
gzip on;
```

要压缩其他 MIME 类型的响应，请包含 `gzip_types` 指令并列出其他类型：

```nginx
gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/xml;
```

  > 请记住：默认情况下，NGINX 不会使用其按请求的 gzip 模块压缩图像文件。

我强烈建议你阅读以下内容（这是 [Barry Pollard](https://serverfault.com/users/268936/barry-pollard) 关于 gzip 和性能的有趣观察）：

  > _老实说，如今 gzip 并不十分消耗处理器资源，即时压缩（然后在浏览器中解压）通常是常态。Web 浏览器非常擅长这一点。_
  >
  > _因此，除非你拥有巨大的流量，否则你可能不会注意到即时 gzip 压缩对大多数 Web 文件造成的性能或 CPU 负载影响。_

要测试 HTTP 和 Gzip 压缩，我推荐两个外部工具：

- [HTTP Compression Test](https://www.whatsmyip.org/http-compression-test/)
- [HTTP Gzip Compression Test](http://www.visiospark.com/gzip-compression-test/)

NGINX 还会压缩大文件并避免压缩小文件（如图像、可执行文件等），因为非常小的文件几乎无法从压缩中受益。你可以告诉 NGINX 不压缩小于例如 128 字节的文件：

```nginx
gzip_min_length 128;
```

更多信息请参阅 [Finding the Nginx gzip_comp_level Sweet Spot](https://mjanja.ch/2015/03/finding-the-nginx-gzip_comp_level-sweet-spot/)。

即时压缩资源每次提供资源时都会增加 CPU 负载和延迟（等待压缩完成）。NGINX 还通过 static 模块提供静态压缩。这更好，原因有两个：

- 无需为每个请求进行 gzip 压缩
- 可以使用更高的 gzip 级别

例如：

```nginx
# 启用静态 gzip 压缩：
location ^~ /assets/ {

  gzip_static on;

  ...

}
```

你应该将 `gzip_static on;` 放在配置静态文件的块内，但如果你只运行一个站点，将其放在 http 块中也是安全的。

  > NGINX 不会自动为你压缩文件。你需要自己完成此操作。

要手动压缩文件：

```bash
cd assets/
while IFS='' read -r -d '' _fd; do

  gzip -N4c ${_fd} > ${_fd}.gz

done < <(find . -maxdepth 1 -type f -regex ".*\.\(css\|js\|jpg\|gif\|png\|jpeg\)" -print0)
```

因此，例如，要处理对 `/foo/bar/file` 的请求，NGINX 会尝试查找并直接发送文件 `/foo/bar/file.gz`，因此不会为你的请求增加额外的 CPU 成本或延迟，从而加快应用程序的响应速度。

<a id="english-anchor"></a>
##### 什么是最佳的 NGINX 压缩 gzip 级别？

gzip 的压缩级别只是在 1-9 的范围内决定数据的压缩程度，其中 9 是最高压缩级别。折衷之处在于，最高压缩的数据通常需要最多的计算来进行压缩/解压缩，但也可以参考[这个](https://stackoverflow.com/questions/28452429/does-gzip-compression-level-have-any-impact-on-decompression/37892065#37892065)精彩的回答。作者解释了 gzip 压缩级别不会影响解压缩的难度。

我认为理想的压缩级别似乎在 4 到 6 之间。以下指令设置了文件的压缩程度：

```nginx
gzip_comp_level 6;
```

<a id="english-anchor"></a>
#### 哈希表

  > 在开始阅读本章之前，我推荐阅读 [Hash tables explained](https://yourbasic.org/algorithms/hash-tables-explained/)。

为了协助快速处理请求，NGINX 使用哈希表。NGINX 的哈希虽然在原则上与典型的哈希列表相同，但有显著的区别。

它们并非用于动态添加和删除元素的应用程序，而是专门设计用于保存一组在初始化时排列在哈希列表中的元素。所有放入哈希列表的元素在创建哈希列表本身时都是已知的。这里无法进行动态添加或删除。

该哈希表在重启或重新加载期间构建并编译，之后运行速度非常快。其主要目的似乎是加速一次性添加元素的查找。

查看官方文档中的 [Setting up hashes](http://nginx.org/en/docs/hash.html)：

  > _为了快速处理静态数据集，如服务器名称、map 指令的值、MIME 类型、请求头字符串的名称，NGINX 使用哈希表。在启动和每次重新配置期间，NGINX 会选择哈希表的最小可能大小，使得存储具有相同哈希值的键的桶大小不超过配置的参数（哈希桶大小）。表的大小以桶为单位。调整会持续进行，直到表大小超过哈希最大大小参数。大多数哈希都有相应的指令允许更改这些参数。_

我还推荐阅读 [Optimizations](https://www.nginx.com/resources/wiki/start/topics/tutorials/optimizations/) 部分和 [nginx - Hashing scheme](http://netsecinfo.blogspot.com/2010/01/nginx-hashing-scheme.html) 的解释。

一些重要信息（基于 [brablc](https://serverfault.com/users/94256/brablc) 的[这项出色研究](https://serverfault.com/questions/419847/nginx-setting-server-names-hash-max-size-and-server-names-hash-bucket-size/786726#786726)）：

- 一般建议是将两个值保持为尽可能小且尽可能少的冲突（在启动和每次重新配置时，NGINX 会选择哈希表的最小可能大小）

- 这取决于你的设置，你可以减少表中的服务器数量，然后执行 `reload` 而不是 `restart`

- 如果 NGINX 发出需要增加 `hash_max_size` 或 `hash_bucket_size` 的通信，则首先需要增加第一个参数

- 更大的 `hash_max_size` 使用更多内存，更大的 `hash_bucket_size` 在查找期间使用更多 CPU 周期以及从主内存到缓存的更多传输。如果你有足够的内存，请增加 `hash_max_size` 并尽量保持 `hash_bucket_size` 尽可能低

- 每个哈希表条目在桶中占用空间。所需空间是键的长度（加上一些存储域实际长度的开销），例如域名

  > 由于 `stage.api.example.com` 是 21 个字符，所有条目在桶中至少消耗 24 字节，大多数消耗 32 字节或更多。

- 当你增加条目数量时，你必须增加哈希表的大小和/或表中哈希桶的数量

  > 如果 NGINX 报错，首先增加 `hash_max_size` 直到它不再报错。如果数量超过了某个大数（例如 32769），则将 `hash_bucket_size` 增加到平台默认值的倍数，直到它不再报错。如果它不再报错，则减少 `hash_max_size` 直到它刚好不再报错。这样你就为你的服务器名称集获得了最佳设置（每组服务器名称可能需要不同的设置）。

- 当哈希桶大小为 64 或 128 时，桶在 4 或 5 个条目哈希到它后就会满

- `hash_max_size` 与服务器名称数量没有直接关系，如果服务器数量翻倍，你可能需要将 `hash_max_size` 增加 10 倍甚至更多以避免冲突。如果无法避免冲突，则必须增加 `hash_bucket_size`

- 如果你的 `hash_max_size` 小于 10000 且 `hash_bucket_size` 较小，则可能会遇到较长的加载时间，因为 NGINX 会循环尝试寻找最佳的哈希大小（参见 [src/core/ngx_hash.c](https://github.com/nginx/nginx/blob/c3aed0a23392a509f64b740064f5f6633e8c89d8/src/core/ngx_hash.c#L289)）

- 如果你的 `hash_max_size` 大于 10000，则只会在报错前执行 1000 次循环

<a id="english-anchor"></a>
##### 服务器名称哈希表

服务器名称的哈希由以下指令控制（在 `http` 上下文中）：

- `server_names_hash_max_size` - 设置服务器名称哈希表的最大大小；默认值：512
- `server_names_hash_bucket_size` - 设置服务器名称哈希表的桶大小；默认值：32、64 或 128（默认值取决于处理器缓存行的大小）

  > 参数 `server_names_hash_bucket_size` 总是与处理器缓存行大小对齐为倍数。

如果服务器名称定义为 `too.long.server.name.example.com`，则 NGINX 将启动失败并显示如下错误消息：

```
nginx: [emerg] could not build server_names_hash, you should increase server_names_hash_bucket_size: 64
```

要修复此问题，你应该 `reload` NGINX 或将 `server_names_hash_bucket_size` 指令值增加到下一个 2 的幂（在本例中为 128）。

如果定义了大量的服务器名称，并且 NGINX 报如下错误：

```
nginx: [emerg] could not build the server_names_hash, you should increase either server_names_hash_max_size: 512 or server_names_hash_bucket_size: 32
```

尝试将 `server_names_hash_max_size` 设置为接近服务器名称数量的数字。仅当此方法无效，或 NGINX 启动时间过长时，才尝试增加 `server_names_hash_bucket_size` 参数。

<a id="english-anchor"></a>
#### 日志文件

  > **:bookmark: [使用自定义日志格式 - 调试 - P4](RULES.md#beginner-use-custom-log-formats)**

日志文件是 NGINX 管理的关键部分。它在请求处理完毕后（在最后阶段：`NGX_HTTP_LOG_PHASE`）将客户端请求的信息写入访问日志。

默认情况下：

- 访问日志位于 `logs/access.log`，但我建议你将其放在 `/var/log/nginx` 目录下
- 数据以预定义的 `combined/main` 格式写入
- `access.log` 存储每个请求的记录，日志格式完全可配置
- `error.log` 包含重要的操作消息

这相当于以下配置：

```nginx
# 在 nginx.conf 中（默认日志格式）：
http {

  ...

  log_format main
                  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

  # 但我建议你改为：
  log_format main
                  '$remote_addr - $remote_user [$time_local] '
                  '"$request_method $scheme://$host$request_uri '
                  '$server_protocol" $status $body_bytes_sent '
                  '"$http_referer" "$http_user_agent" '
                  '$request_time';

}
```

更多信息请参阅 [Configuring Logging](https://docs.nginx.com/nginx/admin-guide/monitoring/logging/)。

  > 设置 `access log off;` 可完全关闭日志记录。

  > 如果你不希望 404 错误显示在 NGINX 错误日志中，应设置 `log_not_found off;`。

  > 如果你想在 `access_log` 中启用子请求的日志记录，应设置 `log_subrequest on;` 并更改默认日志格式（你需要记录 `$uri` 以查看差异）。关于如何在 NGINX 日志文件中识别子请求，这里有[很好的解释](https://serverfault.com/questions/904396/how-to-identify-subrequests-in-nginx-log-files/922956#922956)。

我还推荐阅读：

- [ngx_http_log_module](http://nginx.org/en/docs/http/ngx_http_log_module.html)
- [ngx_http_upstream_module](http://nginx.org/en/docs/http/ngx_http_upstream_module.html)

<a id="english-anchor"></a>
##### 条件日志记录

有时某些条目只是为了填满日志或造成混乱。当我想要更有效地调试日志文件时，我有时会排除某些请求 - 按客户端 IP 或其他条件。

因此，在此示例中，如果 `$error_codes` 变量的值为 0 - 则不记录任何内容（默认操作），但如果为 1（例如后端返回的 `404` 或 `503`）- 则将此请求保存到日志：

```nginx
# 在 http 上下文中定义 map：
http {

  ...

  map $status $error_codes {

    default   1;
    ~^[23]    0;

  }

  ...

  # 向 access log 添加 if 条件：
  access_log /var/log/nginx/example.com-access.log combined if=$error_codes;

}
```

<a id="english-anchor"></a>
##### 手动日志轮转

  > **:bookmark: [配置日志轮转策略 - 基本规则 - P1](RULES.md#beginner-configure-log-rotation-policy)**

NGINX 会在收到 `USR1` 信号时重新打开其日志文件：

```bash
cd /var/log/nginx

mv access.log access.log.0
kill -USR1 $(cat /var/run/nginx.pid) && sleep 1

# >= gzip-1.6：
gzip -k access.log.0
# 任意版本：
gzip < access.log.0 > access.log.0.gz

# 测试完整性并在通过测试后删除：
gzip -t access.log.0 && rm -fr access.log.0
```

<a id="english-anchor"></a>
##### 错误日志严重级别

  > 你不能指定自己的格式，但 NGINX 内置了多个 `error_log` 级别。

以下是所有严重级别的列表：

| <b>类型</b> | <b>描述</b> |
| :---         | :---         |
| `debug` | 有助于定位问题所在的信息 |
| `info` | 非必需但可能值得了解的信息性消息 |
| `notice` | 已发生的值得注意的正常事件 |
| `warn` | 发生了意外情况，但无需担忧 |
| `error` | 某些操作未成功，包含限制规则的操作（默认） |
| `crit` | 需要处理的重要问题 |
| `alert` | 需要立即采取行动的严重情况 |
| `emerg` | 系统处于不可用状态，需要立即关注 |

例如：如果你设置 `crit` 错误日志级别，则会记录 `crit`、`alert` 和 `emerg` 级别的消息。

  > 要使调试日志记录生效，NGINX 需要使用 `--with-debug` 构建。

错误级别的默认值：

- 在 main 部分 - `error`
- 在 HTTP 部分 - `crit`
- 在 server 部分 - `crit`

<a id="english-anchor"></a>
##### 如何记录请求的开始时间？

大多数日志记录信息需要请求完成后才能获取（状态码、发送的字节数、持续时间等）。如果你想在 NGINX 中记录请求的开始时间，应应用一个[补丁](https://gist.github.com/rkbodenner/318681)，该补丁将请求开始时间暴露为一个变量。

`$time_local` 变量包含日志条目写入时的时间，因此当 HTTP 请求头被读取时，NGINX 会查找关联的虚拟服务器配置。如果找到虚拟服务器，请求将经历六个阶段：

- server rewrite 阶段
- location 阶段
- location rewrite 阶段（可能将请求带回前一阶段）
- access control 阶段
- `try_files` 阶段
- log 阶段

由于 log 阶段是最后一个阶段，`$time_local` 变量更接近请求的结束而非开始。

<a id="english-anchor"></a>
##### 如何记录 HTTP 请求体？

NGINX 除非确实需要，否则不会解析客户端请求体，因此它通常不会填充 `$request_body` 变量。

例外情况是：

- 它将请求发送到代理
- 或 fastcgi 服务器

因此你确实需要在你的块中添加 `proxy_pass` 或 `fastcgi_pass` 指令。

```nginx
# 1) 设置日志格式：
log_format req_body_logging '$remote_addr - $remote_user [$time_local] '
                            '"$request" $status $body_bytes_sent '
                            '"$http_referer" "$http_user_agent" "$request_body"';

# 2) 限制请求体大小：
client_max_body_size 1k;
client_body_buffer_size 1k;
client_body_in_single_buffer on;

# 3) 放置日志格式：
server {

  ...

  location /api/v4 {

    access_log logs/access_req_body.log req_body_logging;
    proxy_pass http://127.0.0.1;

    ...

  }

  location = /post.php {

    access_log /var/log/nginx/postdata.log req_body_logging;
    fastcgi_pass php_cgi;

    ...

  }

}
```

为此，你也可以使用 [echo](https://github.com/openresty/echo-nginx-module) 模块。要记录请求体，我们需要使用 `echo_read_request_body` 指令和 `$request_body` 变量（包含 echo 模块的请求体）。

  > `echo_read_request_body` 显式读取请求体，以便 `$request_body` 变量始终具有非空值（除非请求体太大，NGINX 已将其保存到本地临时文件）。

```nginx
http {

  log_format req_body_logging '$request_body';
  access_log /var/log/nginx/access.log req_body_logging;

  ...

  server {

    location / {

      echo_read_request_body;

      ...

    }

    ...

  }

}
```

<a id="english-anchor"></a>
##### NGINX upstream 变量返回 2 个值

例如：

```
upstream_addr 192.168.50.201:8080 : 192.168.50.201:8080
upstream_bytes_received 427 : 341
upstream_connect_time 0.001 : 0.000
upstream_header_time 0.003 : 0.001
upstream_response_length 0 : 0
upstream_response_time 0.003 : 0.001
upstream_status 401 : 200
```

以下是每个变量的简短说明：

- `$upstream_addr` - 保留上游服务器的 IP 地址和端口，或 UNIX 域套接字的路径。如果在请求处理期间联系了多个服务器，它们的地址用逗号分隔，例如 `192.168.1.1:80, 192.168.1.2:80, unix:/tmp/sock`。如果从一个服务器组到另一个服务器组发生内部重定向（由 `X-Accel-Redirect` 或 `error_page` 发起），则来自不同组的服务器地址用冒号分隔，例如 `192.168.1.1:80, 192.168.1.2:80, unix:/tmp/sock : 192.168.10.1:80, 192.168.10.2:80`
- `$upstream_cache_status` - 保留访问响应缓存的状态 (0.8.3)。状态可以是 `MISS`、`BYPASS`、`EXPIRED`、`STALE`、`UPDATING`、`REVALIDATED` 或 `HIT`
- `$upstream_connect_time` - 与上游服务器建立连接所花费的时间
- `$upstream_cookie_` - 上游服务器在 `Set-Cookie` 响应头字段中发送的指定名称的 cookie (1.7.1)。仅保存来自最后一个服务器响应的 cookie
- `$upstream_header_time` - 从建立连接到从上游服务器接收响应头第一个字节之间的时间
- `$upstream_http_` - 保留服务器响应头字段。例如，`Server` 响应头字段可通过 `$upstream_http_server` 变量获取。将头字段名称转换为变量名称的规则与以 `$http_` 前缀开头的变量相同。仅保存来自最后一个服务器响应的头字段
- `$upstream_response_length` - 保留从上游服务器获得的响应长度 (0.7.27)；长度以字节为单位。多个响应的长度用逗号和冒号分隔，如同 `$upstream_addr` 变量中的地址
- `$upstream_response_time` - 从建立连接到从上游服务器接收响应体最后一个字节之间的时间
- `$upstream_status` - 保留从上游服务器获得的响应状态码。多个响应的状态码用逗号和冒号分隔，如同 `$upstream_addr` 变量中的地址

官方文档说明：

  > _[...] 如果在请求处理期间联系了多个服务器，它们的地址用逗号分隔。 [...] 如果从一个服务器组到另一个服务器组发生内部重定向（由 "X-Accel-Redirect" 或 error_page 发起），则来自不同组的服务器地址用冒号分隔_

这意味着它向后端发出了多个请求，很可能是因为你有一个裸的 `proxy_pass` 主机解析为不同的 IP（例如 Amazon ELB 作为源时经常出现这种情况），或者你配置了一个包含多个服务器的 upstream。除非禁用，否则代理模块将对所有健康后端进行轮询尝试。这可以通过 `proxy_next_upstream_*` 指令进行配置。

例如，如果这不是期望的行为，你可以这样做（指定在哪些情况下应将请求传递给下一个服务器）：

```nginx
# 应注意，只有在尚未向客户端发送任何内容时，
# 才能将请求传递给下一个服务器。也就是说，如果在传输响应的过程中
# 发生错误或超时，则无法修复此问题。
proxy_next_upstream off;
```

更多信息请参阅 [ngx_http_upstream_module](http://nginx.org/en/docs/http/ngx_http_upstream_module.html) 和 [proxy_next_upstream](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream)。

<a id="english-anchor"></a>
#### 反向代理

  > 阅读本章后，请参阅：[Rules: Reverse Proxy](RULES.md#reverse-proxy)。

这是 NGINX 最伟大的功能之一。简单来说，反向代理是位于内部应用程序和外部客户端之间的服务器，将客户端请求转发到相应的服务器。它接收客户端请求，将其传递给一个或多个服务器，然后将服务器的响应返回给客户端。

官方 NGINX 文档说明：

  > _代理通常用于在多个服务器之间分配负载，无缝显示来自不同网站的内容，或通过 HTTP 以外的协议将请求传递给应用程序服务器进行处理。_

你也可以阅读关于 [代理服务器和反向代理服务器有什么区别](https://stackoverflow.com/questions/224664/whats-the-difference-between-proxy-server-and-reverse-proxy-server/366212#366212) 的非常棒的解释。

反向代理可以分担高流量分布式 Web 应用程序的许多基础设施问题。

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/reverse-proxy/reverse-proxy_preview.png" alt="reverse-proxy_preview">
</p>

<sup><i>本图表来自 [Jenkins with NGINX - Reverse proxy with https](https://medium.com/@sportans300/nginx-reverse-proxy-with-https-466daa4da4fc)。</i></sup>

这允许你让 NGINX 反向代理请求到 unicorns、mongrels、webricks、thins 或任何你真正想要运行服务器的程序。

反向代理提供了许多高级功能，例如：

- 负载均衡、故障转移和后端服务器的透明维护
- 增强的安全性（例如 SSL 终止、隐藏上游配置）
- 提高性能（例如缓存、负载均衡）
- 简化访问控制职责（单一访问和维护点）
- 集中式日志记录和审计（单一维护点）
- 添加/删除/修改 HTTP 头

在我看来，与反向代理相关的两个最重要的事情是：

- 请求转发到后端的方式
- 转发到后端的头类型

如果谈到代理服务器的安全性，请查看关于 [Guidelines on Securing Public Web Servers](https://www.nist.gov/publications/guidelines-securing-public-web-servers) <sup>[NIST]</sup> 的建议。本文档是一个很好的起点。虽然有些陈旧，但仍有有趣的解决方案和建议。

有一个关于通过使用反向代理服务器提高安全性的好处的[精彩解释](https://serverfault.com/a/25095)。

  > 反向代理为你提供了一些可能使服务器更安全的功能：
  >
  > - 一个独立于 Web 服务器的监控和记录位置
  > - 一个独立于 Web 服务器的过滤位置，如果你知道系统的某个区域存在漏洞。根据代理的不同，你可能能够在应用程序级别进行过滤
  > - 另一个实现 ACL 和规则的位置，如果你的 Web 服务器因某种原因不够灵活
  > - 一个独立的网络栈，不会像你的 Web 服务器那样存在相同的漏洞。如果你的代理来自不同的供应商，这一点尤其正确
  > - 没有过滤的反向代理不会自动保护你免受所有威胁，但如果需要保护的系统价值很高，那么添加反向代理可能值得付出支持和性能成本

另一个关于反向代理实现最佳实践的[精彩回答](https://security.stackexchange.com/questions/48347/documented-best-practices-for-reverse-proxy-implementation)：

  > 根据我的经验，一些最重要的要求和缓解措施（按任意顺序）是：
  >
  > - 确保你的代理、后端 Web（和数据库）服务器不能建立直接的出站（互联网）连接（包括 DNS 和 SMTP，特别是 HTTP）。这意味着如果需要出站访问，则使用（正向）代理/中继
  > - 确保你的日志记录有用（§9.1 中的内容），并且连贯。你可能拥有来自多个设备（路由器、防火墙/IPS/WAF、代理、Web/应用服务器、数据库服务器）的日志。如果你不能快速、可靠且确定性地跨每个设备关联记录，那么你就做错了。这意味着需要 NTP，并记录以下部分或全部信息：PID、TID、会话 ID、端口、请求头、cookie、用户名、IP 地址等（这可能意味着某些日志包含机密信息）
  > - 了解协议，并做出审慎、知情的决定：包括密码/ TLS 版本选择、HTTP 头大小、URL 长度、cookie。限制应在反向代理上实施。如果你正在迁移到分层架构，请确保开发团队了解情况，以便尽早发现问题
  > - 从外部运行漏洞扫描，或请人代为执行。确保你知道你的攻击面，并且报告突出显示差异以及理论上的 TLS 问题
  > - 了解故障模式。当遇到负载或稳定性问题时，向用户发送一个裸的默认 "HTTP 500 - 系统崩溃了" 是不够的
  > - 监控、指标和图表：拥有正常和历史数据对于调查异常和容量规划至关重要
  > - 调优：从 TCP time-wait 到 listen backlog 再到 SYN-cookies，同样需要做出审慎、知情的决定
  > - 遵循基本操作系统加固指南，考虑使用 chroot/jail、基于主机的 IDS 和其他可用措施

<a id="english-anchor"></a>
##### 传递请求

  > **:bookmark: [使用与后端协议兼容的传递指令 - 反向代理 - P1](RULES.md#beginner-use-pass-directive-compatible-with-backend-protocol)**

当 NGINX 代理请求时，它会将请求发送到指定的代理服务器，获取响应，然后将其发送回客户端。

可以代理请求到：

- HTTP 服务器（例如 NGINX、Apache 或其他）使用 `proxy_pass` 指令：

  ```nginx
  upstream bk_front {

    server 192.168.252.20:8080 weight=5;
    server 192.168.252.21:8080

  }

  server {

    location / {

      proxy_pass http://bk_front;

    }

    location /api {

      proxy_pass http://192.168.21.20:8080;

    }

    location /info {

      proxy_pass http://localhost:3000;

    }

    location /ra-client {

      proxy_pass http://10.0.11.12:8080/guacamole/;

    }

    location /foo/bar/ {

      proxy_pass http://www.example.com/url/;

    }

    ...

  }
  ```

- 非 HTTP 服务器（例如 PHP、Node.js、Python、Java 或其他）使用 `proxy_pass` 指令（作为备用）或专门为此设计的指令：

  - `fastcgi_pass` - 将请求传递给 FastCGI 服务器（[PHP FastCGI Example](https://www.nginx.com/resources/wiki/start/topics/examples/phpfcgi/)）：

    ```nginx
    server {

      ...

      location ~ ^/.+\.php(/|$) {

        fastcgi_pass 127.0.0.1:9000;
        include /etc/nginx/fcgi_params;

      }

      ...

    }
    ```

  - `uwsgi_pass` - 将请求传递给 uWSGI 服务器（[Nginx support uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/Nginx.html)）：

    ```nginx
    server {

      location / {

        root html;
        uwsgi_pass django_cluster;
        uwsgi_param UWSGI_SCRIPT testapp;
        include /etc/nginx/uwsgi_params;

      }

      ...

    }
    ```

  - `scgi_pass` - 将请求传递给 SCGI 服务器：

    ```nginx
    server {

      location / {

        scgi_pass 127.0.0.1:4000;
        include /etc/nginx/scgi_params;

      }

      ...

    }
    ```

  - `memcached_pass` - 将请求传递给 Memcached 服务器：

    ```nginx
    server {

      location / {

        set $memcached_key "$uri?$args";
        memcached_pass memc_instance:4004;

        error_page 404 502 504 = @memc_fallback;

      }

      location @memc_fallback {

        proxy_pass http://backend;

      }

      ...

    }
    ```

  - `redis_pass` - 将请求传递给 Redis 服务器（[HTTP Redis](https://www.nginx.com/resources/wiki/modules/redis/)）：

    ```nginx
    server {

      location / {

        set $redis_key $uri;

        redis_pass redis_instance:6379;
        default_type text/html;
        error_page 404 = /fallback;

      }

      location @fallback {

        proxy_pass http://backend;

      }

      ...

    }
    ```

`proxy_pass` 和其他 `*_pass` 指令指定所有匹配该 location 块的请求都应转发到后端应用程序运行的特定套接字。

然而，更复杂的应用程序可能需要额外的指令：

  - `proxy_pass` - 参阅 [`ngx_http_proxy_module`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 指令说明
  - `fastcgi_pass` - 参阅 [`ngx_http_fastcgi_module`](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html) 指令说明
  - `uwsgi_pass` - 参阅 [`ngx_http_uwsgi_module`](http://nginx.org/en/docs/http/ngx_http_uwsgi_module.html) 指令说明
  - `scgi_pass` - 参阅 [`ngx_http_scgi_module`](http://nginx.org/en/docs/http/ngx_http_scgi_module.html) 指令说明
  - `memcached_pass` - 参阅 [`ngx_http_memcached_module`](http://nginx.org/en/docs/http/ngx_http_memcached_module.html) 指令说明
  - `redis_pass` - 参阅 [`ngx_http_redis_module`](https://www.nginx.com/resources/wiki/modules/redis/) 指令说明

<a id="english-anchor"></a>
##### 尾部斜杠

  > **:bookmark: [注意 proxy_pass 指令中的尾部斜杠 - 反向代理 - P3](RULES.md#beginner-be-careful-with-trailing-slashes-in-proxy_pass-directive)**

如果你有这样的配置：

```nginx
location /public/ {

  proxy_pass http://bck_testing_01;

}
```

然后访问 `http://example.com/public`，NGINX 会自动重定向你到 `http://example.com/public/`。

再看这个例子：

```nginx
location /foo/bar/ {

  # proxy_pass http://example.com/url/;
  proxy_pass http://192.168.100.20/url/;

}
```

如果 URI 与地址一起指定，它会替换请求 URI 中匹配 location 参数的部分。例如，这里对 `/foo/bar/page.html` URI 的请求将被代理到 `http://www.example.com/url/page.html`。

如果地址指定时没有 URI，或者无法确定要替换的 URI 部分，则传递完整的请求 URI（可能已修改）。

这是一个在 location 中有尾部斜杠但在 `proxy_pass` 中没有尾部斜杠的示例：

```nginx
location /foo/ {

  proxy_pass http://127.0.0.1:8080/bar;

}
```

注意 `bar` 和 `path` 是如何拼接的。如果访问 `http://yourserver.com/foo/path/id?param=1`，NGINX 将代理请求到 `http://127.0.0.1/barpath/id?param=1`。

正如 NGINX 文档所述，如果 `proxy_pass` 未使用 URI（即 `server:port` 之后没有路径），NGINX 将原样传递原始请求的 URI，包括双斜杠、`../` 等。

另请参阅配置片段：[Using trailing slashes](#using-trailing-slashes)。

以下是更多示例：

| <b>LOCATION</b> | <b>PROXY_PASS</b> | <b>请求</b> | <b>上游接收</b> |
| :---         | :---         | :---         | :---         |
| `/app/` | `http://localhost:5000/api/` | `/app/foo?bar=baz` | `/api/foo?bar=baz` |
| `/app/` | `http://localhost:5000/api` | `/app/foo?bar=baz` | `/apifoo?bar=baz` |
| `/app` | `http://localhost:5000/api/` | `/app/foo?bar=baz` | `/api//foo?bar=baz` |
| `/app` | `http://localhost:5000/api` | `/app/foo?bar=baz` | `/api/foo?bar=baz` |
| `/app` | `http://localhost:5000/api` | `/appfoo?bar=baz` | `/apifoo?bar=baz` |

换句话说：

  > 你通常总是需要尾部斜杠，永远不要混用带和不带尾部斜杠的情况，并且只在想要拼接特定路径组件时才不带尾部斜杠（我认为这种情况很少见）。请注意查询参数是如何被保留的。

<a id="english-anchor"></a>
##### 向后端传递请求头

  > **:bookmark: [正确设置 add_header 和 proxy_*_header 指令的 HTTP 头 - 基本规则 - P1](RULES.md#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)**<br>
  > **:bookmark: [移除对遗留和有风险的 HTTP 头的支持 - 加固 - P1](RULES.md#beginner-remove-support-for-legacy-and-risky-http-headers)**<br>
  > **:bookmark: [始终将 Host、X-Real-IP 和 X-Forwarded 头传递到后端 - 反向代理 - P2](RULES.md#beginner-always-pass-host-x-real-ip-and-x-forwarded-headers-to-the-backend)**<br>
  > **:bookmark: [使用不带 X- 前缀的自定义头 - 反向代理 - P3](RULES.md#beginner-use-reload-option-to-change-configurations-on-the-fly)**

默认情况下，NGINX 会重新定义代理请求中的两个头字段：

- `Host` 头被重写为 `$proxy_host` 变量定义的值。这将是在 `proxy_pass` 指令中直接定义的上游的 IP 地址或名称和端口号

- `Connection` 头被更改为 `close`。此头用于传递关于两个方之间特定连接的信息。在此情况下，NGINX 将其设置为 `close`，以向上游服务器指示此连接将在原始请求响应后关闭。上游不应期望此连接是持久连接

当 NGINX 代理请求时，它会自动对从客户端接收的请求头进行一些调整：

- NGINX 会丢弃空的头。将空值传递给另一个服务器毫无意义；只会使请求变得臃肿

- 默认情况下，NGINX 会将任何包含下划线的头视为无效，并将其从代理请求中删除。如果你希望 NGINX 将它们解释为有效，可以将 `underscores_in_headers` 指令设置为 `on`，否则你的头将永远不会到达后端服务器。头字段中的下划线是允许的（[RFC 7230, sec. 3.2.](https://tools.ietf.org/html/rfc7230#section-3.2)），但确实不常见

如果你希望上游服务器正确处理请求，仅传递 URI 是不够的。代表客户端来自 NGINX 的请求看起来与直接来自客户端的请求不同。

  > 请阅读官方 wiki 上的 [Managing request headers](https://www.nginx.com/resources/wiki/start/topics/examples/headers_management/)。

NGINX 支持任意请求头字段。变量名称的最后一部分是转换为小写、下划线替换破折号的字段名称：

```
$http_name_of_the_header_key
```

如果你在头中有 `X-Real-IP = 127.0.0.1`，可以使用 `$http_x_real_ip` 获取 `127.0.0.1`。

使用 `proxy_set_header` 指令设置发送到后端服务器的头。

  > HTTP 头用于在客户端和服务器之间传输附加信息。`add_header` 将头发送到客户端（浏览器），仅对成功的请求生效，除非你设置了 `always` 参数。`proxy_set_header` 将头发送到后端服务器。如果头字段的值为空字符串，则该字段不会传递给代理服务器。

区分请求头和响应头也很重要。请求头是发往 Web 服务器或后端应用的入站流量。响应头则相反（在你使用客户端（例如 curl 或浏览器）获得的 HTTP 响应中）。

好的，来看以下关于代理指令的简短说明（有关有效头值的更多信息，请参阅[此](RULES.md#beginner-always-pass-host-x-real-ip-and-x-forwarded-stack-headers-to-the-backend)规则）：

- `proxy_http_version` - 定义代理的 HTTP 协议版本，默认设置为 1.0。对于 WebSocket 和 keepalive 连接，你需要使用 1.1 版本：

  ```nginx
  proxy_http_version 1.1;
  ```

- `proxy_cache_bypass` - 设置响应不从缓存中获取的条件：

  ```nginx
  proxy_cache_bypass $http_upgrade;
  ```

- `proxy_intercept_errors` - 意味着任何 HTTP 代码 300 或更高的响应都由 `error_page` 指令处理，并确保如果代理的后端返回错误状态，NGINX 将显示错误页面（而不是后端一侧的错误页面）。如果你希望某些错误页面仍由上游服务器传递，只需不在反向代理上指定 `error_page <code>`（没有此设置，NGINX 会将来自上游服务器的错误页面转发给客户端）：

  ```nginx
  proxy_intercept_errors on;
  error_page 404 /404.html; # 来自代理

  # 要绕过错误拦截（如果你有 proxy_intercept_errors on）：
  # 1 - 不在反向代理上指定 error_page 404
  # 2 - 转到 @debug location
  error_page 500 503 504 @debug;
  location @debug {
    proxy_intercept_errors off;
    proxy_pass http://backend;
  }
  ```

- `proxy_set_header` - 允许重新定义或追加字段到传递给代理服务器的请求头

  - `Upgrade` 和 `Connection` - 如果你的应用程序使用 WebSocket，则需要这些头字段：

    ```nginx
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    ```

  - `Host` - 按以下优先顺序，`$host` 变量包含：请求行中的主机名，或 `Host` 请求头字段中的主机名，或匹配请求的服务器名称：NGINX 使用 `Host` 头进行 `server_name` 匹配。它不使用 TLS SNI。这意味着对于 SSL 服务器，NGINX 必须能够接受 SSL 连接，这归结为拥有证书/密钥。证书/密钥可以是任何类型，例如自签名：

    ```nginx
    proxy_set_header Host $host;
    ```

  - `X-Real-IP` - 将真实的访客远程 IP 地址转发到代理服务器：

    ```nginx
    proxy_set_header X-Real-IP $remote_addr;
    ```

  - `X-Forwarded-For` - 是从 HTTP 代理或负载均衡器连接到 Web 服务器的用户的原始 IP 地址的传统识别方式：

    ```nginx
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    ```

  - `X-Forwarded-Proto` - 识别客户端用于连接代理或负载均衡器的协议（HTTP 或 HTTPS）：

    ```nginx
    proxy_set_header X-Forwarded-Proto $scheme;
    ```

  - `X-Forwarded-Host` - 定义客户端请求的原始主机：

    ```nginx
    proxy_set_header X-Forwarded-Host $host;
    ```

  - `X-Forwarded-Port` - 定义客户端请求的原始端口：

    ```nginx
    proxy_set_header X-Forwarded-Port $server_port;
    ```

如果你想了解自定义头，请查看 [Why we need to deprecate x prefix for HTTP headers?](https://tonyxu.io/posts/2018/http-deprecate-x-prefix/) 和 [BalusC](https://stackoverflow.com/users/157882/balusc) 的[这个](https://stackoverflow.com/a/3561399)精彩回答。

<a id="english-anchor"></a>
###### `Host` 请求头的重要性

  > **:bookmark: [仅使用 $host 变量设置和传递 Host 头 - 反向代理 - P2](RULES.md#beginner-set-and-pass-host-header-only-with-host-variable)**

`Host` 头告诉 Web 服务器要使用哪个虚拟主机（如果设置了的话）。你甚至可以让同一个虚拟主机使用多个别名（域名和通配符域名）。这就是 host 头存在的原因。host 头指定哪个网站或 Web 应用程序应该处理传入的 HTTP 请求。

在 NGINX 中，`$host` 等于 `$http_host`，小写且不带端口号（如果存在），除非 `HTTP_HOST` 不存在或是空值。在这种情况下，`$host` 等于处理请求的服务器的 `server_name` 指令的值。

但请注意：

  > _未更改的 `Host` 请求头字段可以使用 `$http_host` 传递。但是，如果客户端请求头中不存在此字段，则不会传递任何内容。在这种情况下，最好使用 `$host` 变量——其值等于 `Host` 请求头字段中的服务器名称，如果此字段不存在，则等于主服务器名称。_

例如，如果你设置 `Host: MASTER:8080`，`$host` 将是 "master"（而 `$http_host` 将是 `MASTER:8080`，因为它只是反映整个头）。

另请参阅 [$10k host header](https://www.ezequiel.tech/p/10k-host-header.html) 和 [What is a Host Header Attack?](https://www.acunetix.com/blog/articles/automated-detection-of-host-header-attacks/)。

<a id="english-anchor"></a>
###### 重定向和 `X-Forwarded-Proto`

  > **:bookmark: [不要在反向代理后面使用带 $scheme 的 X-Forwarded-Proto - 反向代理 - P1](RULES.md#beginner-dont-use-x-forwarded-proto-with-scheme-behind-reverse-proxy)**

这个头非常重要，因为它可以防止重定向循环。当在 HTTPS 服务器块内使用时，来自代理服务器的每个 HTTP 响应都将被重写为 HTTPS。请看以下示例：

1. 客户端向代理发送 HTTP 请求
2. 代理向服务器发送 HTTP 请求
3. 服务器看到 URL 是 `http://`
4. 服务器发送回 3xx 重定向响应，告诉客户端连接到 `https://`
5. 客户端向代理发送 HTTPS 请求
6. 代理解密 HTTPS 流量并设置 `X-Forwarded-Proto: https`
7. 代理向服务器发送 HTTP 请求
8. 服务器看到 URL 是 `http://`，但也看到 `X-Forwarded-Proto` 是 https，并信任该请求是 HTTPS
9. 服务器发送回请求的网页或数据

<sup><i>此说明来自 [Purpose of the X-Forwarded-Proto HTTP Header](https://community.pivotal.io/s/article/Purpose-of-the-X-Forwarded-Proto-HTTP-Header)。</i></sup>

在上面的第 6 步中，代理设置 HTTP 头 `X-Forwarded-Proto: https` 以指定其接收的流量是 HTTPS。在第 8 步中，服务器随后使用 `X-Forwarded-Proto` 来确定请求是 HTTP 还是 HTTPS。

你可以在这里阅读如何正确设置：

- [Set correct scheme passed in X-Forwarded-Proto](HELPERS.md#set-correct-scheme-passed-in-x-forwarded-proto)
- [不要在反向代理后面使用带 $scheme 的 X-Forwarded-Proto - 反向代理 - P1](RULES.md#beginner-dont-use-x-forwarded-proto-with-scheme-behind-reverse-proxy)

<a id="english-anchor"></a>
###### 关于 `X-Forwarded-For` 的警告

  > **:bookmark: [正确设置 X-Forwarded-For 头的值 - 反向代理 - P1](RULES.md#beginner-set-properly-values-of-the-x-forwarded-for-header)**

我想我们应该先暂停一下。`X-Forwarded-For` 是最重要的头之一，具有安全隐患。

当连接通过一系列代理服务器时，`X-Forwarded-For` 可以提供以逗号分隔的 IP 地址列表，其中第一个是最下游（即用户）。

HTTP `X-Forwarded-For` 接受上述两个指令，如下所述：

- `<client>` - 是客户端的 IP 地址
- `<proxy>` - 是请求必须通过的代理。如果有多个代理，则列出每个后续代理的 IP 地址

语法：

```
X-Forwarded-For: <client>, <proxy1>, <proxy2>
```

`X-Forwarded-For` 不应被用于任何访问控制列表（ACL）检查，因为它可能被攻击者伪造。请使用真实的 IP 地址进行此类限制。诸如 `X-Forwarded-For`、`True-Client-IP` 和 `X-Real-IP` 之类的 HTTP 请求头不是构建任何安全措施（如访问控制）的可靠基础。

[正确设置 X-Forwarded-For 头的值（来自本手册）](RULES.md#beginner-set-properly-values-of-the-x-forwarded-for-header) - 请参阅此内容以获取有关如何正确设置 `X-Forwarded-For` 头值的更详细信息。

但这还不是全部。在反向代理后面，我们获取的用户 IP 通常是反向代理本身的 IP。如果你在代理和应用服务器之间使用其他 HTTP 服务器，你还应该设置正确的机制来解释此头的值。

我推荐阅读 [Nick M](https://serverfault.com/users/130923/nick-m) 的[这个](https://serverfault.com/questions/314574/nginx-real-ip-header-and-x-forwarded-for-seems-wrong/414166#414166)精彩解释。

1) 将头从代理传递到后端层：

    - [始终将 Host、X-Real-IP 和 X-Forwarded 头传递到后端](RULES.md#beginner-always-pass-host-x-real-ip-and-x-forwarded-headers-to-the-backend)
    - [正确设置 X-Forwarded-For 头的值（来自本手册）](RULES.md#beginner-set-properly-values-of-the-x-forwarded-for-header)

2) NGINX（后端） - 修改 `set_real_ip_from` 和 `real_ip_header` 指令：

    > 为此，必须安装 `http_realip_module`（`--with-http_realip_module`）。

    首先，你应该将以下行添加到配置中：

    ```nginx
    # 将这些添加到 set_real_ip.conf，这些是你的流量来源的
    # 真实 IP（前端代理/负载均衡器）：
    set_real_ip_from 192.168.20.10; # 主节点的 IP 地址
    set_real_ip_from 192.168.20.11; # 从节点的 IP 地址

    # 你也可以添加整个子网：
    set_real_ip_from 192.168.40.0/24;

    # 定义用于发送替换地址的请求头字段，
    # 在本例中我们使用 X-Forwarded-For：
    real_ip_header X-Forwarded-For;

    # 来自客户端、匹配受信地址之一的真实 IP
    # 将被替换为请求头字段中发送的最后一个非受信地址：
    real_ip_recursive on;

    # 将其包含到适当的上下文中：
    server {

      include /etc/nginx/set_real_ip.conf;

      ...

    }
    ```

3) NGINX - 添加/修改并设置日志格式：

    ```nginx
    log_format combined-1 '$remote_addr forwarded for $http_x_real_ip - $remote_user [$time_local]  '
                          '"$request" $status $body_bytes_sent '
                          '"$http_referer" "$http_user_agent"';

    # 或者：
    log_format combined-2 '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/example.com/access.log combined-1;
    ```

    这样，例如 PHP fastcgi 中的 `$_SERVER['REMOTE_ADDR']` 将被正确填充。你可以使用以下脚本进行测试：

    ```php
    # tls_check.php
    <?php

    echo '<pre>';
    print_r($_SERVER);
    echo '</pre>';
    exit;

    ?>
    ```

    并发送请求：

    ```bash
    curl -H Cache-Control: no-cache -ks https://example.com/tls-check.php?${RANDOM} | grep "HTTP_X_FORWARDED_FOR\|HTTP_X_REAL_IP\|SERVER_ADDR\|REMOTE_ADDR"
    [HTTP_X_FORWARDED_FOR] => 172.217.20.206
    [HTTP_X_REAL_IP] => 172.217.20.206
    [SERVER_ADDR] => 192.168.10.100
    [REMOTE_ADDR] => 192.168.10.10
    ```

<a id="english-anchor"></a>
###### 使用 `Forwarded` 提高可扩展性

自 2014 年以来，IETF 批准了代理的标准头定义，称为 `Forwarded`，记录在[此处](https://tools.ietf.org/html/rfc7239) <sup>[IETF]</sup> 和[此处](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Forwarded)，应该用于替代 `X-Forwarded` 头。如果你的请求由代理处理，你应该使用这个来可靠地获取原始 IP。官方 NGINX 文档也提供了 [Using the Forwarded header](https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/)。

总的来说，代理头（`Forwarded` 或 `X-Forwarded-For`）是获取客户端 IP 的正确方式，但前提是你确信它们是通过代理到达你的。如果没有代理头或其中没有可用值，则应默认使用 `REMOTE_ADDR` 服务器变量。

<a id="english-anchor"></a>
##### 响应头

  > **:bookmark: [正确设置 add_header 和 proxy_*_header 指令的 HTTP 头 - 基本规则 - P1](RULES.md#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)**

`add_header` 指令允许你定义一个任意响应头（主要用于信息/调试目的）和值，该值将包含在以下所有响应码中：

- 2xx 系列：200, 201, 204, 206
- 3xx 系列：301, 302, 303, 304, 307, 308

例如：

```nginx
add_header Custom-Header Value;
```

  > 要更改（添加或删除）现有头，你应该使用 [headers-more-nginx-module](https://github.com/openresty/headers-more-nginx-module) 模块。

如果你使用 `add_header` 指令（也适用于 `proxy_*_header` 指令），有一点你必须注意。请参阅以下解释：

- [Nginx add_header configuration pitfall](https://blog.g3rt.nl/nginx-add_header-pitfall.html)
- [Be very careful with your add_header in Nginx! You might make your site insecure](https://www.peterbe.com/plog/be-very-careful-with-your-add_header-in-nginx)

这种情况在官方文档中有描述：

  > _可以有多个 `add_header` 指令。这些指令从上一级继承，当且仅当当前级别没有定义 `add_header` 指令时。_

然而——这一点很重要——既然你在 `server` 上下文中定义了一个头，那么在 `http` 上下文中定义的所有其余头将不再被继承。这意味着你必须在 `server` 上下文中重新定义它们（或者如果它们对你的站点不重要，可以忽略它们）。

最后，关于操作头的指令总结：

- `proxy_set_header` 用于设置或移除请求头（并将其传递或不传递给后端）
- `add_header` 用于向响应添加头
- `proxy_hide_header` 用于隐藏响应头

我们还可以使用 [headers-more-nginx-module](https://github.com/openresty/headers-more-nginx-module) 模块操作请求和响应头：

- `more_set_headers` - 替换（如果有）或添加（如果没有）指定的输出头
- `more_clear_headers` - 清除指定的输出头
- `more_set_input_headers` - 与 `more_set_headers` 非常相似，但操作的是输入头（或请求头）
- `more_clear_input_headers` - 与 `more_clear_headers` 非常相似，但操作的是输入头（或请求头）

下图描述了负责操作 HTTP 请求和响应头的模块和指令：

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/reverse-proxy/headers_processing.png" alt="headers_processing">
</p>

<a id="english-anchor"></a>
#### 负载均衡算法

负载均衡本质上是一件非常好的事情。当你每秒处理数万（甚至更多）请求时，你会了解到它的重要性。当然，负载均衡不是唯一的原因——请同时考虑无停机维护任务。

通常，负载均衡是一种用于在多个计算资源和服务器之间分配工作负载的技术。我认为即使你有一个简单的应用或其他与其他人分享的东西，你也应该始终使用这种技术。

配置非常简单。NGINX 包含一个 `ngx_http_upstream_module` 来定义后端（服务器组或多个服务器实例）。更具体地说，`upstream` 指令负责此功能。

  > `upstream` 定义负载均衡池，仅提供服务器列表、某种权重以及与后端层相关的其他参数。

<a id="english-anchor"></a>
##### 后端参数

  > **:bookmark: [调整被动健康检查 - 负载均衡 - P3](RULES.md#beginner-tweak-passive-health-checks)**<br>
  > **:bookmark: [不要通过注释禁用后端，使用 down 参数 - 负载均衡 - P4](RULES.md#beginner-dont-disable-backends-by-comments-use-down-parameter)**

在开始讨论负载均衡技术之前，你应该了解一下 `server` 指令。它定义了后端服务器的地址和其他参数。

此指令接受以下选项：

- `weight=<num>` - 设置源服务器的权重，例如 `weight=10`

- `max_conns=<num>` - 限制从 NGINX 代理服务器到上游服务器的最大同时活动连接数（默认值：`0` = 无限制），例如 `max_conns=8`

  - 如果你设置 `max_conns=4`，第 5 个连接将被拒绝
  - 如果服务器组不驻留在共享内存中（`zone` 指令），则限制适用于每个工作进程

- `max_fails=<num>` - 与后端通信失败尝试的次数（默认值：`1`，`0` 禁用尝试计数），例如 `max_fails=3;`

- `fail_timeout=<time>` - 在这段时间内，发生指定次数的与服务器通信失败尝试后，该服务器将被视为不可用（默认值：`10 秒`），例如 `fail_timeout=30s;`

- `zone <name> <size>` - 定义共享内存区域，保存组的配置和运行时状态，这些状态在工作进程之间共享，例如 `zone backend 32k;`

- `backup` - 如果服务器标记为备份服务器，则它不会接收请求，除非其他两个服务器都不可用

- `down` - 将服务器标记为永久不可用

<a id="english-anchor"></a>
##### 使用 SSL 的上游服务器

在 NGINX 上设置 SSL 终止也非常简单，只需使用 SSL 模块即可。为此，你需要使用 upstream 模块和 proxy 模块。这里还有一个很好的[案例研究](https://www.nginx.com/resources/wiki/start/topics/examples/SSL-Offloader/)。

更多信息请阅读官方文档中的 [Securing HTTP Traffic to Upstream Servers](https://docs.nginx.com/nginx/admin-guide/security-controls/securing-http-traffic-upstream/)。

<a id="english-anchor"></a>
##### 轮询

这是最简单的负载均衡技术。轮询有一个服务器列表，并按顺序将每个请求转发到列表中的每个服务器。一旦到达最后一个服务器，循环再次跳转到第一个服务器并重新开始。

```nginx
upstream bck_testing_01 {

  # 所有服务器默认权重相同（weight=1）
  server 192.168.250.220:8080;
  server 192.168.250.221:8080;
  server 192.168.250.222:8080;

}
```

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/lb/nginx_lb_round-robin.png" alt="round-robin">
</p>

<a id="english-anchor"></a>
##### 加权轮询

在加权轮询负载均衡算法中，每个服务器根据其配置和处理请求的能力被分配一个权重。

此方法类似于轮询，因为请求分配给节点的方式仍然是循环的，但有所不同。配置较高的节点将被分配更多的请求。

```nginx
upstream bck_testing_01 {

  server 192.168.250.220:8080 weight=3;
  server 192.168.250.221:8080;           # 默认 weight=1
  server 192.168.250.222:8080;           # 默认 weight=1

}
```

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/lb/nginx_lb_weighted-round-robin.png" alt="weighted-round-robin">
</p>

<a id="english-anchor"></a>
##### 最少连接

此方法告诉负载均衡器查看每个服务器的连接数，并将下一个连接发送到连接数最少的服务器。

```nginx
upstream bck_testing_01 {

  least_conn;

  # 所有服务器默认权重相同（weight=1）
  server 192.168.250.220:8080;
  server 192.168.250.221:8080;
  server 192.168.250.222:8080;

}
```

例如：如果客户端 D10、D11 和 D12 在 A4、C2 和 C8 已断开但 A1、B3、B5、B6、C7 和 A9 仍保持连接时尝试连接，负载均衡器将分配客户端 D10 到服务器 2，而不是服务器 1 和服务器 3。之后，客户端 D11 将被分配到服务器 1，客户端 D12 将被分配到服务器 2。

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/lb/nginx_lb_least-conn.png" alt="least-conn">
</p>

<a id="english-anchor"></a>
##### 加权最少连接

这通常是一种非常公平的分配方法，因为它使用连接数与服务器权重的比率。集群中比率最低的服务器自动接收下一个请求。

```nginx
upstream bck_testing_01 {

  least_conn;

  server 192.168.250.220:8080 weight=3;
  server 192.168.250.221:8080;           # 默认 weight=1
  server 192.168.250.222:8080;           # 默认 weight=1

}
```

例如：如果客户端 D10、D11 和 D12 在 A4、C2 和 C8 已断开但 A1、B3、B5、B6、C7 和 A9 仍保持连接时尝试连接，负载均衡器将分配客户端 D10 到服务器 2 或 3（因为它们的活动连接数最少），而不是服务器 1。之后，客户端 D11 和 D12 将被分配到服务器 1，因为它具有最大的 `weight` 参数。

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/lb/nginx_lb_weighted-least-conn.png" alt="weighted-least-conn">
</p>

<a id="english-anchor"></a>
##### IP 哈希

IP 哈希方法使用客户端的 IP 创建唯一的哈希键，并将该哈希与其中一个服务器关联。这确保用户在未来会话中被发送到相同的服务器（一种基本的会话持久性），除非该服务器不可用。如果需要临时移除某个服务器，应使用 `down` 参数标记，以保留客户端 IP 地址的当前哈希。

当会话之间的操作需要保持活跃（例如购物车中的产品）或当会话状态涉及且未由应用程序的共享内存处理时，此技术特别有用。

```nginx
upstream bck_testing_01 {

  ip_hash;

  # 所有服务器默认权重相同（weight=1）
  server 192.168.250.220:8080;
  server 192.168.250.221:8080;
  server 192.168.250.222:8080;

}
```

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/lb/nginx_lb_ip-hash.png" alt="ip-hash">
</p>

<a id="english-anchor"></a>
##### 通用哈希

此技术与 IP 哈希非常相似，但针对每个请求，负载均衡器计算基于文本字符串、变量或你指定的组合的哈希，并将该哈希与其中一个服务器关联。

```nginx
upstream bck_testing_01 {

  hash $request_uri;

  # 所有服务器默认权重相同（weight=1）
  server 192.168.250.220:8080;
  server 192.168.250.221:8080;
  server 192.168.250.222:8080;

}
```

例如：负载均衡器根据完整的原始请求 URI（含参数）计算哈希。客户端 A4、C7、C8 和 A9 向 `/static` location 发送请求，将被分配到服务器 1。类似地，获取 `/sitemap.xml` 资源的客户端 A1、C2、B6 将被分配到服务器 2。客户端 B3 和 B5 向 `/api/v4` 发送请求，将被分配到服务器 3。

<p align="center">
  <img src="https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/lb/nginx_lb_generic-hash.png" alt="generic-hash">
</p>

<a id="english-anchor"></a>
##### 其他方法

这与通用哈希方法类似，因为你也可以指定唯一的哈希标识符，但分配到相应服务器由你控制。我认为这是一种有点原始的方法，不能说是一种完整的负载均衡技术，但在某些情况下非常有用。

  > 这主要有助于减少由许多具有类似配置的 `location` 块造成的配置混乱。

首先，创建一个 map：

```nginx
map $request_uri $bck_testing_01 {

  default       "192.168.250.220:8080";

  /api/v4       "192.168.250.220:8080";
  /api/v3       "192.168.250.221:8080";
  /static       "192.168.250.222:8080";
  /sitemap.xml  "192.168.250.222:8080";

}
```

并添加 `proxy_pass` 指令：

```nginx
server {

  ...

  location / {

    proxy_pass http://$bck_testing_01;

  }

  ...

}
```

<a id="english-anchor"></a>
#### 速率限制

  > **:bookmark: [限制并发连接 - 加固 - P1](RULES.md#beginner-limit-concurrent-connections)**
  > **:bookmark: [使用 limit_conn 改善下载速度限制 - 性能 - P3](RULES.md#beginner-use-limit_conn-to-improve-limiting-the-download-speed)

NGINX 有一个默认模块用于设置速率限制。对我来说，这是最有用的保护功能之一，但有时确实很难理解。

如果有疑问，我建议你阅读以下文档：

- [Rate Limiting with NGINX and NGINX Plus](https://www.nginx.com/blog/rate-limiting-nginx/)
- [NGINX rate-limiting in a nutshell](https://www.freecodecamp.org/news/nginx-rate-limiting-in-a-nutshell-128fe9e0126c/)
- [NGINX Rate Limiting](https://dzone.com/articles/nginx-rate-limiting)
- [How to protect your web site from HTTP request flood, DoS and brute-force attacks](https://www.ryadel.com/en/nginx-request-rate-limit-protect-web-site-http-request-flood-dos-brute-force/)

速率限制规则适用于：

- 流量整形
- 流量优化
- 减慢传入请求的速率
- 防止 HTTP 请求洪水
- 防止慢速 HTTP 攻击
- 防止消耗大量带宽
- 缓解 DDoS 攻击
- 防止暴力破解攻击

<a id="english-anchor"></a>
##### 变量

NGINX 有以下变量（唯一键）可用于速率限制规则。例如：

| <b>变量</b> | <b>描述</b> |
| :---         | :---         |
| `$remote_addr` | 客户端地址 |
| `$binary_remote_addr`| 二进制形式的客户端地址，比 `remote_addr` 更小，节省空间 |
| `$server_name` | 接受请求的服务器名称 |
| `$request_uri` | 完整的原始请求 URI（含参数） |
| `$query_string` | 请求行中的参数 |

<sup><i>请参阅[官方文档](https://nginx.org/en/docs/http/ngx_http_core_module.html#variables)以获取有关更多变量的信息。</i></sup>

<a id="english-anchor"></a>
##### 指令、键和区域

NGINX 还提供以下键：

| <b>键</b> | <b>描述</b> |
| :---         | :---         |
| `limit_req_zone` | 存储当前过量请求的数量 |
| `limit_conn_zone` | 存储允许的最大连接数 |

以及指令：

| <b>指令</b> | <b>描述</b> |
| :---         | :---         |
| `limit_req` | 与 `limit_conn_zone` 结合，设置共享内存区域和请求的最大突发大小 |
| `limit_conn` | 与 `limit_req_zone` 结合，设置共享内存区域和每个客户端 IP 允许的最大（同时）连接数 |

键用于存储每个 IP 地址的状态以及它访问受限对象的频率。这些信息存储在共享内存中，所有 NGINX 工作进程均可访问。

  > 你可以使用 `limit_req_dry_run on;` 启用干运行模式。在此模式下，请求处理速率不受限制，但在共享内存区域中，过量请求的数量照常计算。

两个键还提供响应状态参数，指示请求或连接过多，并指定 HTTP 状态码（默认为 **503**）。

- `limit_req_status <value>`
- `limit_conn_status <value>`

例如，如果你想设置服务器限制连接数量时的所需日志级别：

```nginx
# 添加到 http 上下文：
limit_req_status 429;

# 为 429 HTTP 状态码设置自定义错误页面：
error_page 429 /rate_limit.html;
location = /rate_limit.html {

  root /usr/share/www/http-error-pages/sites/other;
  internal;

}
```

并创建此文件：

```bash
cat > /usr/share/www/http-error-pages/sites/other/rate_limit.html << __EOF__
HTTP 429 Too Many Requests
__EOF__
```

速率限制规则还有区域，允许你定义用于计数传入请求或连接的共享空间。

  > 所有进入同一空间的请求或连接将计入相同的速率限制。这允许你按 URL、按 IP 或其他任何方式进行限制。在 HTTP/2 和 SPDY 中，每个并发请求被视为一个单独的连接。

区域有两个必需部分：

- `<name>` - 是区域标识符
- `<size>` - 是区域大小

示例：

```
<key> <variable> zone=<name>:<size>;
```

  > 大约 **16,000** 个 IP 地址的状态信息占用 **1 兆字节**。因此 **1 KB** 的区域可以容纳 **16** 个 IP 地址。

区域的范围如下：

- **http 上下文**

  ```nginx
  http {

    ... zone=<name>;

  ```

- **server 上下文**

  ```nginx
  server {

    ... zone=<name>;

  ```

- **location 指令**

  ```nginx
  location /api {

    ... zone=<name>;

  ```

  > 所有速率限制规则（定义）都应添加到 NGINX `http` 上下文中。

还要记住[这个](https://stackoverflow.com/questions/37438949/antiddos-protection-slowing-nginx-server/37439338#37439338)回答：

  > _如果你在加载网站，你不仅在加载该网站，还在加载其资源。NGINX 会将它们视为独立的连接。你定义了 10r/s 和突发大小为 5。因此，在 10 请求/秒之后，下一个请求将被延迟以进行速率限制。如果突发大小（5）被超过，后续请求将收到 503 错误。_

`limit_req_zone` 键允许你设置 `rate` 参数（可选）——它定义速率限制的 URL。

另请参阅示例（均来自本手册）：

- [Limiting the rate of requests with burst mode](HELPERS.md#limiting-the-rate-of-requests-with-burst-mode)
- [Limiting the rate of requests with burst mode and nodelay](HELPERS.md#limiting-the-rate-of-requests-with-burst-mode-and-nodelay)
- [Limiting the rate of requests per IP with geo and map](HELPERS.md#limiting-the-rate-of-requests-per-ip-with-geo-and-map)
- [Limiting the number of connections](HELPERS.md#limiting-the-number-of-connections)

<a id="english-anchor"></a>
##### Burst 和 nodelay 参数

要启用队列，你应该使用 `limit_req` 或 `limit_conn` 指令（见上文）。`limit_req` 还提供可选参数：

| <b>参数</b> | <b>描述</b> |
| :---         | :---         |
| `burst=<num>` | 设置等待及时处理的最大过量请求数；最大请求数为 `rate` * `burst`，在 `burst` 秒内 |
| `nodelay`| 强制执行速率限制，但不限制请求之间的允许间隔；默认情况下 NGINX 会返回 503 响应，不处理过量请求 |

  > `nodelay` 参数只有在同时设置 `burst` 时才有用。

没有 `nodelay`，NGINX 会等待（不返回 503 响应）并以一定延迟处理过量请求。

<a id="english-anchor"></a>
#### NAXSI Web 应用防火墙

- [NAXSI](https://github.com/nbs-system/naxsi)
- [NAXSI, a web application firewall for Nginx](https://www.nbs-system.com/en/blog/naxsi-web-application-firewall-for-nginx/)

NAXSI 是一个开源、高性能、低规则维护的 NGINX WAF，通常被称为 _正向模型应用防火墙_。它是一个开源 WAF（Web 应用防火墙），提供高性能和低规则维护的 Web 应用防火墙模块。

<a id="english-anchor"></a>
#### OWASP ModSecurity 核心规则集 (CRS)

- [OWASP Core Rule Set](https://coreruleset.org/)
- [OWASP Core Rule Set - Official documentation](https://coreruleset.org/documentation/)

OWASP ModSecurity 核心规则集 (CRS) 是一套用于 ModSecurity 或兼容 Web 应用防火墙的通用攻击检测规则。CRS 旨在以最少的误报保护 Web 应用程序免受广泛攻击，包括 OWASP Top Ten。

<a id="english-anchor"></a>
#### 核心模块

<a id="english-anchor"></a>
##### ngx_http_geo_module

文档：

- [`ngx_http_geo_module`](https://nginx.org/en/docs/http/ngx_http_geo_module.html)

此模块使变量可用，其值取决于客户端的 IP 地址。与 GeoIP 模块结合使用时，允许根据地理位置上下文制定非常精细的规则来提供内容。

默认情况下，用于进行查找的 IP 地址是 `$remote_addr`，但也可以指定其他变量。

  > 如果变量的值不代表有效的 IP 地址，则使用 `255.255.255.255` 地址。

<a id="english-anchor"></a>
###### 性能

请查看此内容（来自官方文档）：

  > _由于变量仅在使用时才被评估，因此即使存在大量声明的 `geo` 变量，也不会给请求处理带来额外成本。_

此模块（注意：不要将此模块与 GeoIP 混淆）在加载配置时构建内存中的基数树。这与路由中使用的数据结构相同，查找速度非常快。如果你每个网络有许多唯一值，则此长加载时间是由于在数组中搜索重复数据造成的。否则，可能是由于插入到基数树中造成的。

<a id="english-anchor"></a>
###### 示例

  > 请参阅本手册中的 [Use geo/map modules instead of allow/deny](RULES.md#beginner-use-geomap-modules-instead-of-allowdeny)。

```nginx
# 创建的变量是 $trusted_ips：
geo $trusted_ips {

  default       0;
  192.0.0.0/24  0;
  8.8.8.8       1;

}

server {

  if ( $trusted_ips = 1 ) {

    return 403;

  }

  ...

}
```

  > 如果变量的值不代表有效的 IP 地址，则使用 `255.255.255.255` 地址。

你还可以测试 IP 范围，例如：

```nginx
# 创建 geo-ranges.conf：
127.0.0.0-127.255.255.255   loopback;

# 添加 geo 定义：
geo $geo_ranges {

  ranges;
  default                   default;
  include                   geo-ranges.conf;
  10.255.0.0-10.255.255.255 internal;

}
```

<a id="english-anchor"></a>
#### 第三方模块

  > 并非所有外部模块都能与你当前的 NGINX 版本正常工作。在将每个模块添加到模块列表之前，你应该阅读其文档。你还应该检查模块的哪个版本与你的 NGINX 发行版兼容。此外，在生产环境中添加模块之前要小心。有些可能导致奇怪的行为、增加内存和 CPU 使用率，并降低 NGINX 的整体性能。

  > 在安装外部模块之前，请阅读[事件驱动架构](NGINX_BASICS.md#event-driven-architecture)部分，以了解为什么低质量的第三方模块可能会降低 NGINX 性能。

  > 如果你的服务器上已运行 NGINX，并且想要添加新模块，你需要根据当前安装的 NGINX 版本（`nginx -v`）编译它们，并且要使新模块与现有的 NGINX 二进制文件兼容，你需要使用相同的编译标志（`nginx -V`）。更多信息请参阅 [How to Compile Dynamic NGINX Modules](https://gorails.com/blog/how-to-compile-dynamic-nginx-modules)。

  > 如果你使用例如 `--with-stream=dynamic`，那么所有那些 `stream_xxx` 模块也必须构建为 NGINX 动态模块。否则你肯定会看到那些链接器错误。

<a id="english-anchor"></a>
##### ngx_set_misc

文档：

- [`ngx_set_misc`](https://github.com/openresty/set-misc-nginx-module)

<a id="english-anchor"></a>
##### ngx_http_geoip_module

文档：

- [`ngx_http_geoip_module`](http://nginx.org/en/docs/http/ngx_http_geoip_module.html)
- [`ngx_http_geoip2_module`](https://github.com/leev/ngx_http_geoip2_module)

此模块允许实时查询 Max Mind GeoIP 数据库。它使用旧版 API，在操作系统发行版中仍然非常常见。要使用新版 GeoIP API，请参阅 geoip2 模块。

Max Mind GeoIP 数据库是 IP 网络地址分配到地理位置的映射，虽然近似，但在相对精细的级别上识别 IP 主机地址关联的物理位置可能很有用。

<a id="english-anchor"></a>
###### 性能

GeoIP 模块设置多个变量，默认情况下 NGINX 仅在（重新）启动或 SIGHUP 时解析配置并将 geoip 数据加载到内存中。

  > GeoIP 查找来自分布式数据库，而非动态服务器，因此与 DNS 不同，最坏情况下的性能影响很小。此外，从性能角度来看，你无需担心，因为 geoip 数据库存储在内存中（在读取配置阶段），NGINX 的查找速度非常快。

GeoIP 模块根据请求客户端的 IP 地址和其中一个 Maxmind GeoIP 数据库创建（并赋值给）变量。常见用途之一是将最终用户的国家设置为 NGINX 变量。

NGINX 中的变量仅按需计算。如果在请求处理过程中未使用 `$geoip_*` 变量，则不会查找 geoip 数据库。因此，如果你在应用程序中不调用 geoip 变量，geoip 模块根本不会执行。使用非常大的地理数据库的唯一不便之处是配置读取时间。

<a id="english-anchor"></a>
###### 示例

  > 请参阅本手册中的 [Restricting access by geographical location](HELPERS.md#restricting-access-by-geographical-location)。
