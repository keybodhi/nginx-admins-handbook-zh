<a id="base-rules"></a>
# 基础规则

返回 **[目录](https://github.com/trimstray/nginx-admins-handbook#table-of-contents)** 或 **[下一步？](https://github.com/trimstray/nginx-admins-handbook#whats-next)** 部分。

  > :pushpin:&nbsp; 以下是一组基础规则，用于保持 NGINX 的良好状态。

- **[≡ 基础规则 (16)](#base-rules)**
  * [组织 Nginx 配置](#beginner-organising-nginx-configuration)
  * [格式化、美化并缩进你的 Nginx 代码](#beginner-format-prettify-and-indent-your-nginx-code)
  * [使用 reload 选项动态更改配置](#beginner-use-reload-option-to-change-configurations-on-the-fly)
  * [为 80 和 443 端口分离 listen 指令](#beginner-separate-listen-directives-for-80-and-443-ports)
  * [使用 address:port 对定义 listen 指令](#beginner-define-the-listen-directives-with-addressport-pair)
  * [防止处理未定义服务器名称的请求](#beginner-prevent-processing-requests-with-undefined-server-names)
  * [绝不在 listen 或 upstream 指令中使用主机名](#beginner-never-use-a-hostname-in-a-listen-or-upstream-directives)
  * [正确使用 add_header 和 proxy_*_header 指令设置 HTTP 头](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)
  * [为 listen 指令只使用一个 SSL 配置](#beginner-use-only-one-ssl-config-for-the-listen-directive)
  * [使用 geo/map 模块代替 allow/deny](#beginner-use-geomap-modules-instead-of-allowdeny)
  * [映射一切...](#beginner-map-all-the-things)
  * [为未匹配的 location 设置全局 root 目录](#beginner-set-global-root-directory-for-unmatched-locations)
  * [使用 return 指令进行 URL 重定向 (301, 302)](#beginner-use-return-directive-for-url-redirection-301-302)
  * [配置日志轮转策略](#beginner-configure-log-rotation-policy)
  * [使用简单的自定义错误页面](#beginner-use-simple-custom-error-pages)
  * [不要重复使用 index 指令，只在 http 块中使用它](#beginner-dont-duplicate-index-directive-use-it-only-in-the-http-block)
- **[调试](#debugging)**
- **[性能](#performance)**
- **[加固](#hardening)**
- **[反向代理](#reverse-proxy)**
- **[负载均衡](#load-balancing)**
- **[其他](#others)**

<a id="beginner-organising-nginx-configuration"></a>
#### :beginner: 组织 Nginx 配置

###### 说明

  > 当 NGINX 配置增长时，组织配置的需求也会增长。组织良好的代码：
  >
  > - 更容易理解
  > - 更容易维护
  > - 更容易协作

  > 使用 `include` 指令将公共服务器设置移动并拆分到多个文件中，并将特定代码附加到全局配置或上下文中。这有助于将代码组织成逻辑组件。包含是递归处理的，即包含的文件可以进一步包含 include 语句。

  > 制定自己的目录结构（从顶级目录到最底层），并在使用 NGINX 时应用它。仔细思考，找出最适合你且最容易维护的方案。

  > 我总是尽量在配置树的根目录中保留多个目录。这些目录存储所有配置文件，这些文件被附加到主文件（如 `nginx.conf`），必要时也附加到包含 `server` 指令的文件中。

  > 我倾向于以下结构：
  >
  > - `html` - 用于默认静态文件，例如全局 5xx 错误页面
  > - `master` - 用于主要配置，例如 ACL、listen 指令和域名
  >   - `_acls` - 用于访问控制列表，例如 geo 或 map 模块
  >   - `_basic` - 用于速率限制规则、重定向映射或代理参数
  >   - `_listen` - 用于所有 listen 指令；也存储 SSL 配置
  >   - `_server` - 用于域名配置；也存储所有后端定义
  > - `modules` - 用于动态加载到 NGINX 的模块
  > - `snippets` - 用于 NGINX 别名、配置模板

###### 示例

```nginx
# 在 https.conf 中例如：
listen 10.240.20.2:443 ssl;

ssl_certificate /etc/nginx/master/_server/example.com/certs/nginx_example.com_bundle.crt;
ssl_certificate_key /etc/nginx/master/_server/example.com/certs/example.com.key;

...

# 将 'https.conf' 包含到 server 部分：
server {

  include /etc/nginx/master/_listen/10.240.20.2/https.conf;

  # 以及其他外部文件：
  include /etc/nginx/master/_static/errors.conf;
  include /etc/nginx/master/_server/_helpers/global.conf;

  server_name example.com www.example.com;

  ...
```

###### 外部资源

- [How I Manage Nginx Config](https://tylergaw.com/articles/how-i-manage-nginx-config/)
- [Organize your data and code](https://kbroman.org/steps2rr/pages/organize.html)
- [How to keep your R projects organized](https://richpauloo.github.io/2018-10-17-How-to-keep-your-R-projects-organized/)

<a id="beginner-format-prettify-and-indent-your-nginx-code"></a>
#### :beginner: 格式化、美化并缩进你的 Nginx 代码

###### 说明

  > 处理不可读的配置文件是件可怕的事。如果语法不清晰可读，会让眼睛酸痛，还会头疼。

  > 当代码格式化后，维护、调试和优化都会变得更加容易，可以在短时间内阅读和理解。你应该从 NGINX 配置文件中消除代码风格违规。

  > 空格、制表符和换行符不是 NGINX 配置的一部分。NGINX 引擎不会解释它们，但它们有助于使配置更具可读性。

  > 选择你的格式化工具风格并为其设置通用配置。有些规则是通用的，但在我看来，最重要的是在整个代码库中保持一致 NGINX 代码风格：
  >
  > - 使用空格和空行来排列和分隔代码块
  > - 制表符与空格 - 在整个代码中保持一致比使用任何特定类型更重要
  >   - 制表符一致、可自定义，并使错误更明显（除非你是一个 4 空格党）
  >   - 空格始终是一列，如果你希望你的漂亮作品对每个人都显示正确，请使用它
  > - 使用注释来解释为什么做某事，而不是做了什么
  > - 使用有意义的命名约定
  > - 简单优于复杂，但复杂优于晦涩

  > 有些人会说 NGINX 的文件是用它们自己的语言编写的，所以我们不应该过度应用上述规则。我认为，值得遵循通用的（编程）规则，让你和其他 NGINX 管理员的生活更轻松。

###### 示例

不推荐的代码风格：

`
ginx
http {
  include    nginx/proxy.conf;
  include    /etc/nginx/fastcgi.conf;
  index    index.html index.htm index.php;

  default_type application/octet-stream;
  log_format   main ' -  []   '
    '""  "" '
    '"" ""';
  access_log   logs/access.log    main;
  sendfile on;
  tcp_nopush   on;
  server_names_hash_bucket_size 128; # this seems to be required for some vhosts

  ...
`

推荐的代码风格：

`
ginx
http {

  # Attach global rules:
  include         /etc/nginx/proxy.conf;
  include         /etc/nginx/fastcgi.conf;

  index           index.html index.htm index.php;

  default_type    application/octet-stream;

  # Standard log format:
  log_format      main ' -  []   '
                       '""  "" '
                       '"" ""';

  access_log      /var/log/nginx/access.log main;

  sendfile        on;
  tcp_nopush      on;

  # This seems to be required for some vhosts:
  server_names_hash_bucket_size 128;

  ...
`

###### 外部资源

- [Programming style](https://en.wikipedia.org/wiki/Programming_style)
- [Toward Developing Good Programming Style](https://www2.cs.arizona.edu/~mccann/style_c.html)
- [Death to the Space Infidels!](https://blog.codinghorror.com/death-to-the-space-infidels/)
- [Tabs versus Spaces: An Eternal Holy War](https://www.jwz.org/doc/tabs-vs-spaces.html)
- [nginx-config-formatter](https://github.com/1connect/nginx-config-formatter)
- [Format and beautify nginx config files](https://github.com/vasilevich/nginxbeautifier)
<a id="beginner-use-reload-option-to-change-configurations-on-the-fly"></a>
#### :beginner: 使用 `reload` 选项动态更改配置

###### 说明

  > 使用 `reload` 选项可以在不停止服务器和不丢弃任何数据包的情况下优雅地重新加载配置。主进程的此功能允许回滚更改，并继续使用稳定的旧工作配置。

  > NGINX 的这一能力在需要高可用和动态环境中非常关键，可以保持负载均衡器或独立服务器在线。

  > 主进程检查新配置的语法有效性并尝试应用所有更改。如果此过程完成，主进程会创建新的工作进程并向旧的发送关闭消息。旧的工作进程在收到关闭信号后停止接受新连接，但仍在处理当前请求。之后，旧的工作进程退出。

  > 当你重启 NGINX 服务时，可能会遇到 NGINX 停止且无法再次启动的情况，原因是语法错误。Reload 方法比重启更安全，因为在旧进程终止之前，新配置文件会被解析，如果有任何问题，整个过程会被中止。

  > 要等待工作进程完成当前请求后再停止进程，请使用 `nginx -s quit` 命令。对于快速关闭，它比 `nginx -s stop` 更好。

  来自 NGINX 文档：

  > _为了使 NGINX 重新读取配置文件，应向主进程发送 `HUP` 信号。主进程首先检查语法有效性，然后尝试应用新配置，即打开日志文件和新的监听套接字。如果失败，它会回滚更改并继续使用旧配置。如果成功，它会启动新的工作进程，并向旧的工作进程发送消息，请求它们优雅地关闭。旧的工作进程关闭监听套接字并继续为旧客户端服务。所有客户端服务完毕后，旧的工作进程关闭。_

###### 示例

```bash
# 1)
systemctl reload nginx

# 2)
service nginx reload

# 3)
/etc/init.d/nginx reload

# 4)
/usr/sbin/nginx -s reload

# 5)
kill -HUP $(cat /var/run/nginx.pid)
# or
kill -HUP $(pgrep -f "nginx: master")

# 6)
/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
```

###### 外部资源

- [Changing Configuration](https://nginx.org/en/docs/control.html#reconfiguration)
- [Commands (from this handbook)](NGINX_BASICS.md#commands)

<a id="beginner-separate-listen-directives-for-80-and-443-ports"></a>
#### :beginner: 为 80 和 443 端口分离 `listen` 指令

###### 说明

  > 如果你使用完全相同的配置同时提供 HTTP 和 HTTPS 服务（一个服务器同时处理 HTTP 和 HTTPS 请求），NGINX 足够智能，可以在通过 80 端口加载时忽略 SSL 指令。

  > 我不喜欢重复规则，但分离 `listen` 指令确实有助于你维护和修改配置。如果我想从 HTTP 重定向到 HTTPS（或从 www 到非 www，反之亦然），我总是拆分配置。对我来说，在任何此类情况下，定义单独的 server 上下文是正确的方式。

  > 如果你将多个域名绑定到一个 IP 地址，这也很有用。这允许你将一个 `listen` 指令（例如，如果你将其保存在配置文件中）附加到多个域名配置。

  > 如果你使用 HTTPS，可能还需要硬编码域名，因为你必须事先知道将提供哪些证书。

  > 你还应该使用 `return` 指令进行从 HTTP 到 HTTPS 的重定向（以硬编码所有内容，完全不使用正则表达式）。

###### 示例

```nginx
# For HTTP:
server {

  listen 10.240.20.2:80;

  # If you need redirect to HTTPS:
  return 301 https://example.com$request_uri;

  ...

}

# For HTTPS:
server {

  listen 10.240.20.2:443 ssl;

  ...

}
```

###### 外部资源

- [Understanding the Nginx Configuration File Structure and Configuration Contexts](https://www.digitalocean.com/community/tutorials/understanding-the-nginx-configuration-file-structure-and-configuration-contexts)
- [Configuring HTTPS servers](http://nginx.org/en/docs/http/configuring_https_servers.html)
- [Force all connections over TLS - Hardening - P1 (from this handbook)](#beginner-force-all-connections-over-tls)

<a id="beginner-define-the-listen-directives-with-addressport-pair"></a>
#### :beginner: 使用 `address:port` 对定义 `listen` 指令

###### 说明

  > NGINX 通过用默认值替换缺失值来转换所有不完整的 `listen` 指令。

  > 而且，只有在需要区分匹配到 `listen` 指令相同级别的 server 块时，才会评估 `server_name` 指令。

  > 设置 IP 地址和端口号可以防止难以调试的软错误。此外，不指定 IP 意味着绑定到系统上的所有 IP，这可能会导致很多问题，并且是一种不良实践，因为建议只为服务配置最小的网络访问权限。

###### 示例

```nginx
# Client side:
$ curl -Iks http://api.random.com

# Server side:
server {

  # This block will be processed:
  listen 192.168.252.10; # --> 192.168.252.10:80

  ...

}

server {

  listen 80; # --> *:80 --> 0.0.0.0:80
  server_name api.random.com;

  ...

}
```

###### 外部资源

- [Nginx HTTP Core Module - Listen](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen)
- [Understanding different values for nginx 'listen' directive](https://serverfault.com/questions/875140/understanding-different-values-for-nginx-listen-directive)

<a id="beginner-prevent-processing-requests-with-undefined-server-names"></a>
#### :beginner: 防止处理未定义服务器名称的请求

###### 说明

  > 这可以防止配置错误，例如流量转发到错误的后端、绕过 ACL 或 WAF 等过滤器。通过创建一个默认的虚拟主机（使用 `default_server` 指令）来捕获所有带有无法识别的 `Host` 头的请求，可以轻松解决此问题。

  > 我们知道，`Host` 头告诉服务器要使用哪个虚拟主机（如果已设置）。你甚至可以拥有多个别名的同一虚拟主机（= 域名和通配符域名）。此头也可能被修改，因此出于安全和整洁的原因，良好的实践是拒绝没有主机头或主机头未在任何虚拟主机中配置的请求。据此，NGINX 应阻止处理具有未定义服务器名称（也包括 IP 地址）的请求。

  > 如果没有 `listen` 指令包含 `default_server` 参数，那么该 `address:port` 对的第一个服务器将成为该对的默认服务器（这意味着 NGINX 始终有一个默认服务器）。

  > 如果有人使用 IP 地址而不是服务器名称发出请求，`Host` 请求头字段将包含 IP 地址，该请求可以使用 IP 地址作为服务器名称来处理。

  > 在现代版本的 NGINX 中，服务器名称 `_` 不是必需的（所以你可以放任何东西）。实际上，`default_server` 不需要 `server_name` 语句，因为它会匹配其他 server 块未明确匹配的任何内容。

  > 如果找不到匹配的 `listen` 和 `server_name` 的服务器，NGINX 将使用默认服务器。如果你的配置分布在多个文件中，它们的评估顺序将是不确定的，因此你需要显式标记默认服务器。

  > NGINX 使用 `Host` 头进行 `server_name` 匹配，但不使用 TLS SNI。这意味着 NGINX 必须能够接受 SSL 连接，这归结为需要证书/密钥。证书/密钥可以是任何，例如自签名的。

  > 以下是所有未定义服务器名称的简单处理过程：
  >
  > - 一个 `server` 块，具有...
  > - 完整的 `listen` 指令，具有...
  > - `default_server` 参数，具有...
  > - 只有一个 `server_name`（但不必须）定义，并且...
  > - 我预防性地将其放在配置的开头（附加到文件 `nginx.conf`）

  > 对于默认服务器名称，一个好的做法是 `return 444;`（最常用于拒绝恶意或格式错误的请求），因为这会关闭连接（断开连接而不发送任何头，因此不返回任何内容）并在内部记录日志，适用于任何未在 NGINX 中定义的域名。此外，我还会实施速率限制规则。

###### 示例

```nginx
# Place it at the beginning of the configuration file to prevent mistakes:
server {

  # For ssl option remember about SSL parameters (private key, certs, cipher suites, etc.);
  # add default_server to your listen directive in the server that you want to act as the default:
  listen 10.240.20.2:443 default_server ssl;

  # We catch:
  #   - invalid domain names
  #   - requests without the "Host" header
  #   - and all others (also due to the above setting; like "--" or "!@#")
  #   - default_server in server_name directive is not required
  #     I add this for a better understanding and I think it's an unwritten standard
  # ...but you should know that it's irrelevant, really, you can put in everything there.
  server_name _ "" default_server;

  limit_req zone=per_ip_5r_s;

  ...

  # Close (hang up) connection without response:
  return 444;

  # We can also serve:
  # location / {
  #
  #   static file (error page):
  #     root /etc/nginx/error-pages/404;
  #   or redirect:
  #     return 301 https://badssl.com;
  #
  # }

  # Remember to log all actions (set up access and error log):
  access_log /var/log/nginx/default-access.log main;
  error_log /var/log/nginx/default-error.log warn;

}

server {

  listen 10.240.20.2:443 ssl;

  server_name example.com;

  ...

}

server {

  listen 10.240.20.2:443 ssl;

  server_name domain.org;

  ...

}
```

###### 外部资源

- [Server names](https://nginx.org/en/docs/http/server_names.html)
- [How processes a request](https://nginx.org/en/docs/http/request_processing.html)
- [nginx: how to specify a default server](https://blog.gahooa.com/2013/08/21/nginx-how-to-specify-a-default-server/)

<a id="beginner-never-use-a-hostname-in-a-listen-or-upstream-directives"></a>
#### :beginner: 绝不在 `listen` 或 `upstream` 指令中使用主机名

###### 说明

  > 通常，在 `listen` 或 `upstream` 指令中使用主机名是一种不良实践。在最坏的情况下，NGINX 将无法绑定到所需的 TCP 套接字，这将导致 NGINX 完全无法启动。

  > 最好且更安全的方式是知道需要绑定到的 IP 地址，并使用该地址而不是主机名。这也防止 NGINX 需要查找地址，并消除对外部和内部解析器的依赖。

  > 在 `server_name` 指令中使用 `$hostname`（机器的主机名）变量也是不良实践的示例（类似于使用主机名标签）。

  > 我相信设置 IP 地址和端口号对也是必要的，可以防止难以调试的软错误。

###### 示例

不推荐的配置：

```nginx
upstream bk_01 {

  server http://x-9s-web01-prod.int:8080;

}

server {

  listen rev-proxy-prod:80;

  ...

  location / {

    # It's OK, bk_01 is the internal name:
    proxy_pass http://bk_01;

    ...

  }

  location /api {

    proxy_pass http://x-9s-web01-prod-api.int:80;

    ...

  }

  ...

}
```

推荐的配置：

```nginx
upstream bk_01 {

  server http://192.168.252.200:8080;

}

server {

  listen 10.10.100.20:80;

  ...

  location / {

    # It's OK, bk_01 is the internal name:
    proxy_pass http://bk_01;

    ...

  }

  location /api {

    proxy_pass http://192.168.253.10:80;

    ...

  }

  ...

}
```

###### 外部资源

- [Using a Hostname to Resolve Addresses](https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/#using-a-hostname-to-resolve-addresses)
- [Define the listen directives with address:port pair - Base Rules - P1 (from this handbook)](#beginner-define-the-listen-directives-with-addressport-pair)

<a id="beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly"></a>
#### :beginner: 正确使用 `add_header` 和 `proxy_*_header` 指令设置 HTTP 头

###### 说明

  > `add_header` 指令适用于 `if`、`location`、`server` 和 `http` 作用域。`proxy_*_header` 指令适用于 `location`、`server` 和 `http` 作用域。当且仅当当前级别未定义任何 `add_header` 或 `proxy_*_header` 指令时，这些指令才会从上一级继承。

  > 如果在多个上下文中使用它们，只有最低层级的出现会被使用。因此，如果你在 `server` 和 `location` 上下文中都指定了它（即使你使用相同的指令和值设置不同的头），只有 `location` 块中的那个会被使用。为了防止这种情况，你应该定义一个公共配置片段，并仅在你希望发送这些头的每个单独 `location` 中包含它。这是最可预测的解决方案。

  > 在我看来，另一个有趣的解决方案是使用包含全局头部的包含文件，并将其添加到 `http` 上下文中（但是，你会不必要地重复规则）。接下来，你还应该设置另一个包含服务器/域名特定配置的包含文件（但始终包含你的全局头部！你必须在最低层级重复它），并将其添加到 `server/location` 上下文中。然而，这有点复杂，并且不能以任何方式保证一致性。

  > 还有其他解决方案，例如使用替代模块（[headers-more-nginx-module](https://github.com/openresty/headers-more-nginx-module)）在 `server` 或 `location` 块中定义特定头部。它不影响上述指令。

  这是问题的[精彩解释](https://www.keycdn.com/support/nginx-add_header)：

  > _因此，假设你有一个 http 块，并在该块内指定了 `add_header` 指令。然后，在 http 块内，你有 2 个 server 块 - 一个用于 HTTP，一个用于 HTTPS。_
  >
  > _假设我们不在 HTTP server 块中包含 `add_header` 指令，但在 HTTPS server 块中包含一个额外的 `add_header`。在这种情况下，http 块中定义的 `add_header` 指令将仅被 HTTP server 块继承，因为它在当前级别没有定义任何 `add_header` 指令。另一方面，HTTPS server 块将不会继承 http 块中定义的 `add_header` 指令。_

###### 示例

不推荐的配置：

```nginx
http {

  # In this context:
  # set:
  #   - 'FooX barX' (add_header)
  #   - 'Host $host' (proxy_set_header)
  #   - 'X-Real-IP $remote_addr' (proxy_set_header)
  #   - 'X-Forwarded-For $proxy_add_x_forwarded_for' (proxy_set_header)
  #   - 'X-Powered-By' (proxy_hide_header)

  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_hide_header X-Powered-By;

  add_header FooX barX;

  ...

  server {

    server_name example.com;

    # In this context:
    # set:
    #   - 'FooY barY' (add_header)
    #   - 'Host $host' (proxy_set_header)
    #   - 'X-Real-IP $remote_addr' (proxy_set_header)
    #   - 'X-Forwarded-For $proxy_add_x_forwarded_for' (proxy_set_header)
    #   - 'X-Powered-By' (proxy_hide_header)
    # not set:
    #   - 'FooX barX' (add_header)

    add_header FooY barY;

    ...

    location / {

      # In this context:
      # set:
      #   - 'Foo bar' (add_header)
      #   - 'Host $host' (proxy_set_header)
      #   - 'X-Real-IP $remote_addr' (proxy_set_header)
      #   - 'X-Forwarded-For $proxy_add_x_forwarded_for' (proxy_set_header)
      #   - 'X-Powered-By' (proxy_hide_header)
      #   - headers from ngx_headers_global.conf
      # not set:
      #   - 'FooX barX' (add_header)
      #   - 'FooY barY' (add_header)

      include /etc/nginx/ngx_headers_global.conf;
      add_header Foo bar;

      ...

    }

    location /api {

      # In this context:
      # set:
      #   - 'FooY barY' (add_header)
      #   - 'Host $host' (proxy_set_header)
      #   - 'X-Real-IP $remote_addr' (proxy_set_header)
      #   - 'X-Forwarded-For $proxy_add_x_forwarded_for' (proxy_set_header)
      #   - 'X-Powered-By' (proxy_hide_header)
      # not set:
      #   - 'FooX barX' (add_header)

      ...

    }

  }

  server {

    server_name a.example.com;

    # In this context:
    # set:
    #   - 'FooY barY' (add_header)
    #   - 'Host $host' (proxy_set_header)
    #   - 'X-Real-IP $remote_addr' (proxy_set_header)
    #   - 'X-Powered-By' (proxy_hide_header)
    # not set:
    #   - 'FooX barX' (add_header)
    #   - 'X-Forwarded-For $proxy_add_x_forwarded_for' (proxy_set_header)

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_hide_header X-Powered-By;

    add_header FooY barY;

    ...

    location / {

      # In this context:
      # set:
      #   - 'FooY barY' (add_header)
      #   - 'X-Powered-By' (proxy_hide_header)
      #   - 'Accept-Encoding """' (proxy_set_header)
      # not set:
      #   - 'FooX barX' (add_header)
      #   - 'Host $host' (proxy_set_header)
      #   - 'X-Real-IP $remote_addr' (proxy_set_header)
      #   - 'X-Forwarded-For $proxy_add_x_forwarded_for' (proxy_set_header)

      proxy_set_header Accept-Encoding "";

      ...

    }

  }

}
```

最推荐的配置：

```nginx
# Store it in a file, e.g. proxy_headers.conf:
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_hide_header X-Powered-By;

http {

  server {

    server_name example.com;

    ...

    location / {

      include /etc/nginx/proxy_headers.conf;
      include /etc/nginx/ngx_headers_global.conf;
      add_header Foo bar;

      ...

    }

    location /api {

      include /etc/nginx/proxy_headers.conf;
      include /etc/nginx/ngx_headers_global.conf;
      add_header Foo bar;

      more_set_headers 'FooY: barY';

      ...

    }

  }

  server {

    server_name a.example.com;

    ...

    location / {

      include /etc/nginx/proxy_headers.conf;
      include /etc/nginx/ngx_headers_global.conf;
      add_header Foo bar;
      add_header FooX barX;

      ...

    }

  }

  server {

    server_name b.example.com;

    ...

    location / {

      include /etc/nginx/proxy_headers.conf;
      include /etc/nginx/ngx_headers_global.conf;
      add_header Foo bar;

      ...

    }

  }

}
```

###### 外部资源

- [Module ngx_http_headers_module - add_header](http://nginx.org/en/docs/http/ngx_http_headers_module.html#add_header)
- [Managing request headers](https://www.nginx.com/resources/wiki/start/topics/examples/headers_management/)
- [Nginx add_header configuration pitfall](https://blog.g3rt.nl/nginx-add_header-pitfall.html)
- [Be very careful with your add_header in Nginx! You might make your site insecure](https://www.peterbe.com/plog/be-very-careful-with-your-add_header-in-nginx)

<a id="beginner-use-only-one-ssl-config-for-the-listen-directive"></a>
#### :beginner: 为 `listen` 指令只使用一个 SSL 配置

###### 说明

  > 对我来说，这条规则使调试和维护更容易。它还可以防止在同一监听地址上存在多个 TLS 配置。

  > 你应该使用一个 SSL 配置来在多个 HTTPS 配置（例如协议、密码、曲线）之间共享单个 IP 地址。这是为了防止错误和配置不一致。

  > 使用通用 TLS 配置（存储在一个文件中，使用 include 指令添加）用于所有 `server` 上下文可以防止奇怪的行为。我认为没有更好的方法来解决可能的配置混乱问题。

  > 请记住，无论 SSL 参数如何，你都可以在同一 `listen` 指令（IP 地址）上使用多个 SSL 证书。某些 TLS 参数也可能不同。

  > 还要记住默认服务器的配置。这一点很重要，因为如果没有 `listen` 指令包含 `default_server` 参数，那么配置中的第一个服务器将成为默认服务器。因此，你应该在同一 IP 地址上的多个服务器名称上只使用一个 SSL 设置。

###### 示例

```nginx
# Store it in a file, e.g. https.conf:
ssl_protocols TLSv1.2;
ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305";

ssl_prefer_server_ciphers off;

ssl_ecdh_curve secp521r1:secp384r1;

...

# Include this file to the server context (attach domain-a.com for specific listen directive):
server {

  listen 192.168.252.10:443 default_server ssl http2;

  include /etc/nginx/https.conf;

  server_name domain-a.com;

  ssl_certificate domain-a.com.crt;
  ssl_certificate_key domain-a.com.key;

  ...

}

# Include this file to the server context (attach domain-b.com for specific listen directive):
server {

  listen 192.168.252.10:443 ssl;

  include /etc/nginx/https.conf;

  server_name domain-b.com;

  ssl_certificate domain-b.com.crt;
  ssl_certificate_key domain-b.com.key;

  ...

}
```

###### 外部资源

- [Nginx one ip and multiple ssl certificates](https://serverfault.com/questions/766831/nginx-one-ip-and-multiple-ssl-certificates)
- [Configuring HTTPS servers](http://nginx.org/en/docs/http/configuring_https_servers.html)

<a id="beginner-use-geomap-modules-instead-of-allowdeny"></a>
#### :beginner: 使用 `geo/map` 模块代替 `allow/deny`

###### 说明

  > 使用 `map` 或 `geo` 模块（其中之一）来防止用户滥用你的服务器。这允许创建值取决于客户端 IP 地址的变量。

  > 由于变量仅在使用时才被评估，即使是大量声明的 `geo` 变量也不会给请求处理带来额外成本。

  > 这些指令提供了阻止无效访问者的完美方式，例如使用 `ngx_http_geoip_module`。例如，`geo` 模块非常适合有条件地允许或拒绝 IP。

  > `geo` 模块（注意：不要将此模块与 GeoIP 混淆）在加载配置时构建内存中的基数树。这与路由中使用的数据结构相同，查找速度非常快。

  > 我将两个模块都用于大型列表，但这些指令可能需要使用多个 `if` 条件。对我来说，`allow/deny` 指令对于简单列表是更好的解决方案（更直接）。

###### 示例

```nginx
# Map module:
map $remote_addr $globals_internal_map_acl {

  # Status code:
  #  - 0 = false
  #  - 1 = true
  default 0;

  ### INTERNAL ###
  10.255.10.0/24 1;
  10.255.20.0/24 1;
  10.255.30.0/24 1;
  192.168.0.0/16 1;

}

# Geo module:
geo $globals_internal_geo_acl {

  # Status code:
  #  - 0 = false
  #  - 1 = true
  default 0;

  ### INTERNAL ###
  10.255.10.0/24 1;
  10.255.20.0/24 1;
  10.255.30.0/24 1;
  192.168.0.0/16 1;

}
```

再看看下面的示例（`allow/deny` vs `geo + if` 语句）：

```nginx
# allow/deny:
location /internal {

  include acls/internal.conf;
  allow 192.168.240.0/24;
  deny all;

  ...

# vs geo/map:
location /internal {

  if ($globals_internal_map_acl) {
    set $pass 1;
  }

  if ($pass = 1) {
    proxy_pass http://localhost:80;
  }

  if ($pass != 1) {
    return 403;
  }

  ...

}
```

###### 外部资源

- [Nginx Basic Configuration (Geo Ban)](https://www.axivo.com/resources/nginx-basic-configuration.3/update?update=27)
- [What is the best way to redirect 57,000 URLs on nginx?](https://serverfault.com/questions/879534/what-is-the-best-way-to-redirect-57-000-urls-on-nginx)
- [How Radix trees made blocking IPs 5000 times faster](https://blog.sqreen.com/demystifying-radix-trees/)
- [Compressing Radix Trees Without (Too Many) Tears](https://medium.com/basecs/compressing-radix-trees-without-too-many-tears-a2e658adb9a0)
- [Blocking/allowing IP addresses (from this handbook)](HELPERS.md#blockingallowing-ip-addresses)
- [allow and deny (from this handbook)](NGINX_BASICS.md#allow-and-deny)
- [ngx_http_geoip_module (from this handbook)](NGINX_BASICS.md#ngx-http-geoip-module)

<a id="beginner-map-all-the-things"></a>
#### :beginner: 映射一切...

###### 说明

  > 使用 map 管理大量重定向，并自定义你的键值对。如果你在处理请求时遇到过需要使用 if 的情况，应该检查是否可以使用 `map` 代替。

  > `map` 指令映射字符串，因此可以将例如 `192.168.144.0/24` 表示为正则表达式，并继续使用 `map` 指令。

  > Map 模块为清晰解析大量正则表达式提供了更优雅的解决方案，例如 User-Agents、Referrers。

  > 你还可以使用 `include` 指令来管理你的 map，这样你的配置文件会看起来更整洁，并且可以在配置的许多地方使用。

###### 示例

```nginx
# Define in an external file (e.g. maps/http_user_agent.conf):
map $http_user_agent $device_redirect {

  default "desktop";

  ~(?i)ip(hone|od) "mobile";
  ~(?i)android.*(mobile|mini) "mobile";
  ~Mobile.+Firefox "mobile";
  ~^HTC "mobile";
  ~Fennec "mobile";
  ~IEMobile "mobile";
  ~BB10 "mobile";
  ~SymbianOS.*AppleWebKit "mobile";
  ~Opera\sMobi "mobile";

}

# Include to the server context:
include maps/http_user_agent.conf;

# And turn on in a specific context (e.g. location):
if ($device_redirect = "mobile") {

  return 301 https://m.example.com$request_uri;

}
```

###### 外部资源

- [Module ngx_http_map_module](http://nginx.org/en/docs/http/ngx_http_map_module.html)
- [Cool Nginx feature of the week](https://www.ignoredbydinosaurs.com/posts/236-cool-nginx-feature-of-the-week)

<a id="beginner-set-global-root-directory-for-unmatched-locations"></a>
#### :beginner: 为未匹配的 location 设置全局 root 目录

###### 说明

  > 在 server 指令中为请求设置全局 `root`。它指定了未定义 location 的根目录。

  > 如果在 `location` 块中定义 `root`，它将仅在该 `location` 中可用。这几乎总是导致 `root` 指令或文件路径的重复，两者都不好。

  > 如果在 `server` 块中定义，它总是被 `location` 块继承，因此在 `$document_root` 变量中始终可用，从而避免了文件路径的重复。

  来自官方文档：

  > _如果你在每个 location 块中添加 `root`，那么未匹配的 location 块将没有 `root`。因此，重要的是 `root` 指令应出现在你的 location 块之前，如果需要，location 块可以覆盖此指令。_

###### 示例

```nginx
server {

  server_name example.com;

  # It's important:
  root /var/www/example.com/public;

  location / {

    ...

  }

  location /api {

    ...

  }

  location /static {

    root /var/www/example.com/static;

    ...

  }

}
```

###### 外部资源

- [Nginx Pitfalls: Root inside location block](http://wiki.nginx.org/Pitfalls#Root_inside_Location_Block)

<a id="beginner-use-return-directive-for-url-redirection-301-302"></a>
#### :beginner: 使用 `return` 指令进行 URL 重定向 (301, 302)

###### 说明

  > 这是一个简单的规则。你应该使用 server 块和 `return` 语句，因为它们比评估正则表达式快得多。

  > 它更简单更快，因为 NGINX 停止处理请求（并且不必处理正则表达式）。此外，你可以指定 3xx 系列中的代码。

  > 如果你需要通过正则表达式验证 URL，或需要捕获原始 URL 中的元素（这些元素显然不在相应的 NGINX 变量中），那么你应该使用 `rewrite`。

###### 示例

```nginx
server {

  listen 192.168.252.10:80;

  ...

  server_name www.example.com;

  return 301 https://example.com$request_uri;

  # Other examples:
  # return 301 https://$host$request_uri;
  # return 301 $scheme://$host$request_uri;

}

server {

  ...

  server_name example.com;

  return 301 $scheme://www.example.com$request_uri;

}
```

###### 外部资源

- [Creating NGINX Rewrite Rules](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)
- [How to do an Nginx redirect](https://bjornjohansen.no/nginx-redirect)
- [rewrite vs return (from this handbook)](NGINX_BASICS.md#rewrite-vs-return)
- [Adding and removing the www prefix (from this handbook)](HELPERS.md#adding-and-removing-the-www-prefix)
- [Avoid checks server_name with if directive - Performance - P2 (from this handbook)](#beginner-avoid-checks-server_name-with-if-directive)
- [Use return directive instead of rewrite for redirects - Performance - P2 (from this handbook)](#beginner-use-return-directive-instead-of-rewrite-for-redirects)

<a id="beginner-configure-log-rotation-policy"></a>
#### :beginner: 配置日志轮转策略

###### 说明

  > 日志文件提供了关于服务器活动和性能以及可能发生的任何问题的反馈。它们记录了有关请求和 NGINX 内部信息的详细信息。不幸的是，日志会占用更多磁盘空间。

  > 你应该定义一个定期归档当前日志文件并启动新日志文件的过程，重命名并可选择压缩当前日志文件，删除旧日志文件，并强制日志系统开始使用新的日志文件。

  > 我认为最好的工具是 `logrotate`。如果我想自动管理日志，我到处使用它，也为了能睡个好觉。它是一个简单的日志轮转程序，使用 crontab 工作。它是计划任务，而不是守护进程，因此无需重新加载其配置。

###### 示例

- 手动轮转：

  ```bash
  # Check manually (all log files):
  logrotate -dv /etc/logrotate.conf

  # Check manually with force rotation (specific log file):
  logrotate -dv --force /etc/logrotate.d/nginx
  ```

- 自动轮转：

  ```bash
  # GNU/Linux distributions:
  cat > /etc/logrotate.d/nginx << __EOF__
  /var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 nginx nginx
    sharedscripts
    prerotate
      if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
        run-parts /etc/logrotate.d/httpd-prerotate; \
      fi \
    endscript
    postrotate
      # test ! -f /var/run/nginx.pid || kill -USR1 `cat /var/run/nginx.pid`
      invoke-rc.d nginx reload >/dev/null 2>&1
    endscript
  }

  /var/log/nginx/localhost/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 nginx nginx
    sharedscripts
    prerotate
      if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
        run-parts /etc/logrotate.d/httpd-prerotate; \
      fi \
    endscript
    postrotate
      # test ! -f /var/run/nginx.pid || kill -USR1 `cat /var/run/nginx.pid`
      invoke-rc.d nginx reload >/dev/null 2>&1
    endscript
  }

  /var/log/nginx/domains/example.com/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 nginx nginx
    sharedscripts
    prerotate
      if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
        run-parts /etc/logrotate.d/httpd-prerotate; \
      fi \
    endscript
    postrotate
      # test ! -f /var/run/nginx.pid || kill -USR1 `cat /var/run/nginx.pid`
      invoke-rc.d nginx reload >/dev/null 2>&1
    endscript
  }
  __EOF__
  ```

  ```bash
  # BSD systems:
  cat > /usr/local/etc/logrotate.d/nginx << __EOF__
  /var/log/nginx/*.log {
    daily
    rotate 14
    missingok
    sharedscripts
    compress
    postrotate
      kill -HUP `cat /var/run/nginx.pid`
    endscript
    dateext
  }
  /var/log/nginx/*/*.log {
    daily
    rotate 14
    missingok
    sharedscripts
    compress
    postrotate
      kill -HUP `cat /var/run/nginx.pid`
    endscript
    dateext
  }
  __EOF__
  ```

###### 外部资源

- [Understanding logrotate utility](https://support.rackspace.com/how-to/understanding-logrotate-utility/)
- [Rotating Linux Log Files - Part 2: Logrotate](http://www.ducea.com/2006/06/06/rotating-linux-log-files-part-2-logrotate/)
- [Managing Logs with Logrotate](https://serversforhackers.com/c/managing-logs-with-logrotate)
- [nginx and Logrotate](https://drumcoder.co.uk/blog/2012/feb/03/nginx-and-logrotate/)
- [nginx log rotation](https://wincent.com/wiki/nginx_log_rotation)

<a id="beginner-use-simple-custom-error-pages"></a>
#### :beginner: 使用简单的自定义错误页面

###### 说明

  > NGINX 中的默认错误页面很简单，但会显示版本信息并返回 "nginx" 字符串，这导致信息泄露漏洞。

  > 有关所用技术和软件版本的信息是极有价值的信息。这些细节允许识别和利用公开漏洞数据库中发布的已知软件漏洞。

  > 最好的选择是为每个 HTTP 代码生成页面，或使用 SSI 和 `map` 模块创建动态错误页面。

  > 你可以在 `nginx.conf` 中为每个 location 块设置自定义错误页面，或为整个站点设置全局错误页面。你还可以将标准错误代码组合在一起，为多种类型的错误使用单个页面。

  > 注意语法！你应该从 `error_page` 指令中去掉 `=`，因为使用 `error_page 404 = /404.html;` 会以状态码 200 返回 `404.html` 页面（`=` 将其转发到此页面），所以你应该设置 `error_page 404 /404.html;`，这样会返回原始错误代码。

  > 你还应该注意 HTTP 请求走私攻击（参见[更多](https://bertjwregeer.keybase.pub/2019-12-10%20-%20error_page%20request%20smuggling.pdf)）：
  > - `error_page 401 https://example.org/;` - 此处理程序易受攻击，允许攻击者走私请求并可能获取资源/信息
  > - `error_page 404 /404.html;` + `error_page 404 @404;` - 不易受攻击

  > 要生成自定义错误页面，你可以使用 [HTTP Static Error Pages Generator](https://github.com/trimstray/nginx-admins-handbook/tree/master/lib/nginx/snippets/http-error-pages#http-static-error-pages-generator)。

###### 示例

创建错误页面模板：

```bash
cat >> /usr/share/nginx/html/404.html << __EOF__
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
</body>
</html>
__EOF__

# Just an example, I know it is stupid...
cat >> /usr/share/nginx/html/50x.html << __EOF__
<html>
<head><title>server error</title></head>
<body>
<center><h1>server error</h1></center>
</body>
</html>
__EOF__
```

在 NGINX 侧设置它们：

```nginx
error_page 404 /404.html;
error_page 500 502 503 504 /50x.html;

location = /404.html {

  root /usr/share/nginx/html;
  internal;

}

location = /custom_50x.html {

  root /usr/share/nginx/html;
  internal;

}
```

###### 外部资源

- [error_page from ngx_http_core_module](http://nginx.org/en/docs/http/ngx_http_core_module.html#error_page)
- [src/http/ngx_http_special_response.c](https://github.com/nginx/nginx/blob/release-1.17.6/src/http/ngx_http_special_response.c)
- [HTTP Status Codes](https://httpstatuses.com/)
- [One NGINX error page to rule them all](https://blog.adriaan.io/one-nginx-error-page-to-rule-them-all.html)
- [NGINX - Custom Error Pages. A Decent Title Not Found](https://blog.swakes.co.uk/nginx-custom-error-pages/)
- [Dynamic error pages with SSI (from this handbook)](HELPERS.md#dynamic-error-pages-with-ssi)

<a id="beginner-dont-duplicate-index-directive-use-it-only-in-the-http-block"></a>
#### :beginner: 不要重复使用 `index` 指令，只在 http 块中使用它

###### 说明

  > 使用 `index` 指令一次。它只需要出现在你的 `http` 上下文中，就会被下面的层级继承。

  > 我认为我们应该小心重复相同的规则。但当然，规则重复有时是可以的，或者不一定是什么大问题。

###### 示例

不推荐的配置：

```nginx
http {

  ...

  index index.php index.htm index.html;

  server {

    server_name www.example.com;

    location / {

      index index.php index.html index.$geo.html;

      ...

    }

  }

  server {

    server_name www.example.com;

    location / {

      index index.php index.htm index.html;

      ...

    }

    location /data {

      index index.php;

      ...

    }

    ...

}
```

推荐的配置：

```nginx
http {

  ...

  index index.php index.htm index.html index.$geo.html;

  server {

    server_name www.example.com;

    location / {

      ...

    }

  }

  server {

    server_name www.example.com;

    location / {

      ...

    }

    location /data {

      ...

    }

    ...

}
```

###### 外部资源

- [Pitfalls and Common Mistakes - Multiple Index Directives](https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/#multiple-index-directives)

<a id="debugging"></a>
# 调试

返回 **[目录](https://github.com/trimstray/nginx-admins-handbook#table-of-contents)** 或 **[下一步？](https://github.com/trimstray/nginx-admins-handbook#whats-next)** 部分。

  > :pushpin:&nbsp; NGINX 有许多解决问题的方法。在本章中，我将介绍一些处理它们的方法。

- **[基础规则](#base-rules)**
- **[≡ 调试 (5)](#debugging)**
  * [使用自定义日志格式](#beginner-use-custom-log-formats)
  * [使用调试模式追踪意外行为](#beginner-use-debug-mode-to-track-down-unexpected-behaviour)
  * [通过禁用守护进程、主进程和除一个外的所有工作进程来改进调试](#beginner-improve-debugging-by-disable-daemon-master-process-and-all-workers-except-one)
  * [使用核心转储找出 NGINX 持续崩溃的原因](#beginner-use-core-dumps-to-figure-out-why-nginx-keep-crashing)
  * [使用 mirror 模块将请求复制到另一个后端](#beginner-use-mirror-module-to-copy-requests-to-another-backend)
- **[性能](#performance)**
- **[加固](#hardening)**
- **[反向代理](#reverse-proxy)**
- **[负载均衡](#load-balancing)**
- **[其他](#others)**

<a id="beginner-use-custom-log-formats"></a>
#### :beginner: 使用自定义日志格式

###### 说明

  > 你可以在 NGINX 配置中以变量形式访问的任何内容都可以记录，包括非标准 HTTP 头等。因此，为特定情况创建自己的日志格式是一种简单的方法。

  > 这对于调试特定的 `location` 指令非常有帮助。

  > 我还使用自定义日志格式来分析用户流量配置文件（例如 SSL/TLS 版本、密码等）。

###### 示例

```nginx
# Default main log format from nginx repository:
log_format main
                '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';

# Extended main log format:
log_format main-level-0
                '$remote_addr - $remote_user [$time_local] '
                '"$request_method $scheme://$host$request_uri '
                '$server_protocol" $status $body_bytes_sent '
                '"$http_referer" "$http_user_agent" '
                '$request_time';

# Debug log formats:
#   - level 0
#   - based on main-level-0 without "$http_referer" "$http_user_agent"
log_format debug-level-0
                '$remote_addr - $remote_user [$time_local] '
                '"$request_method $scheme://$host$request_uri '
                '$server_protocol" $status $body_bytes_sent '
                '$request_id $pid $msec $request_time '
                '$upstream_connect_time $upstream_header_time '
                '$upstream_response_time "$request_filename" '
                '$request_completion';
```

###### 外部资源

- [Module ngx_http_log_module](https://nginx.org/en/docs/http/ngx_http_log_module.html)
- [Nginx: Custom access log format and error levels](https://fabianlee.org/2017/02/14/nginx-custom-access-log-format-and-error-levels/)
- [nginx: Log complete request/response with all headers?](https://serverfault.com/questions/636790/nginx-log-complete-request-response-with-all-headers)
- [Custom log formats (from this handbook)](HELPERS.md#custom-log-formats)

<a id="beginner-use-debug-mode-to-track-down-unexpected-behaviour"></a>
#### :beginner: 使用调试模式追踪意外行为

###### 说明

  > 它可能会返回比你想要的更多详细信息，但这有时可以救命（但在流量非常高的站点上，日志文件会快速增长）。

  > 通常，`error_log` 指令在 `main` 上下文中指定，但你可以在特定的 `server` 或 `location` 块内指定，全局设置将被覆盖，这样的 `error_log` 指令将设置自己的日志文件路径和日志级别。

  > 可以为特定的 IP 地址或 IP 地址范围启用调试日志（参见示例）。

  > 存储调试日志的替代方法是将其保存在内存中（循环内存缓冲区）。即使在高负载下，调试级别的内存缓冲区也不会对性能产生显著影响。

  > 如果你想记录 `ngx_http_rewrite_module` 的日志（在 `notice` 级别），你应该在 `http`、`server` 或 `location` 上下文中启用 `rewrite_log on;`。

  > 注意事项：
  >
  >   - 切勿在生产环境中将调试日志保留到文件中
  >   - 不要忘记在流量非常高的站点上恢复 `error_log` 的调试级别
  >   - 绝对要使用日志轮转策略

  不久前，我发现了这个有趣的评论：

  > _`notice` 作为调试重写的错误级别比 `debug` 好得多，因为它会跳过大量低级别的无关调试信息（例如 SSL 或 gzip 细节；每个请求 50+ 行）。_

###### 示例

- 将调试日志写入文件：

  ```nginx
  # Turn on in a specific context, e.g.:
  #   - global    - for global logging
  #   - http      - for http and all locations logging
  #   - location  - for specific location
  error_log /var/log/nginx/error-debug.log debug;
  ```

- 将调试日志写入内存：

  ```nginx
  error_log memory:32m debug;
  ```

  > 你可以在[在内存中显示调试日志](HELPERS.md#show-debug-log-in-memory)章节了解更多。

- 为选定的客户端连接启用调试日志：

  ```nginx
  events {

    # Other connections will use logging level set by the error_log directive.
    debug_connection 192.168.252.15/32;
    debug_connection 10.10.10.0/24;

  }
  ```

- 为每个服务器启用调试日志：

  ```nginx
  error_log /var/log/nginx/debug.log debug;

  ...

  http {

    server {

      # To enable debugging:
      error_log /var/log/nginx/example.com/example.com-debug.log debug;
      # To disable debugging:
      error_log /var/log/nginx/example.com/example.com-debug.log;

      ...

    }

  }
  ```

###### 外部资源

- [Debugging NGINX](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/)
- [A debugging log](https://nginx.org/en/docs/debugging_log.html)
- [A little note to all nginx admins there - debug log](https://www.reddit.com/r/sysadmin/comments/7bofyp/a_little_note_to_all_nginx_admins_there/)
- [Error log severity levels (from this hadnbook)](NGINX_BASICS.md#error-log-severity-levels)

<a id="beginner-improve-debugging-by-disable-daemon-master-process-and-all-workers-except-one"></a>
#### :beginner: 通过禁用守护进程、主进程和除一个外的所有工作进程来改进调试

###### 说明

  > 这些具有以下值的指令主要用于开发和调试期间，例如在测试错误/功能时。

  > 例如，`daemon off;` 和 `master_process off;` 让我可以快速测试配置。

  > 对于正常的生产环境，NGINX 服务器将在后台启动（`daemon on;`）。在这种方式下，NGINX 和其他服务运行并相互通信。一台服务器运行多个服务。

  > 在开发或调试环境中（你绝不应在生产环境中以这种方式运行 NGINX），使用 `master_process off;`，我通常在前台运行 NGINX，没有主进程，并按 `^C`（`SIGINT`）简单地终止它。

  > `worker_processes 1;` 也很有用，因为它可以减少工作进程的数量和它们生成的数据，这对我们调试来说相当方便。

###### 示例

```nginx
# Update configuration file (in a global context):
daemon off
master_process off;
worker_processes 1;

# Or run NGINX from shell (oneliner):
/usr/sbin/nginx -t -g 'daemon off; master_process off; worker_processes 1;'
```

###### 外部资源

- [Core functionality](https://nginx.org/en/docs/ngx_core_module.html)

<a id="beginner-use-core-dumps-to-figure-out-why-nginx-keep-crashing"></a>
#### :beginner: 使用核心转储找出 NGINX 持续崩溃的原因

###### 说明

  > 核心转储基本上是程序崩溃时内存的快照。

  > NGINX 是一个非常稳定的守护进程，但有时运行的 NGINX 进程可能会意外终止。当你的 NGINX 实例收到意外错误或崩溃时，你应该始终启用核心转储。

  > 它确保两个重要的指令应被启用，以便保存内存转储，但为了正确处理内存转储，有几件事要做。有关完整信息，请参见[转储进程内存（来自本手册）](HELPERS.md#dump-a-processs-memory)章节。

  > 还要记住其他调试和故障排除工具，如 `eBPF`、`ftrace`、`perf trace` 或 `strace`（注意：`strace` 会为每个系统调用暂停目标进程，以便调试器可以读取状态，并且在系统调用开始和结束时执行两次，因此可能会拖垮你的生产环境），用于工作进程的系统调用跟踪，如 `read/readv/write/writev/close/shutdown`。

###### 示例

```nginx
worker_rlimit_core 500m;
worker_rlimit_nofile 65535;
working_directory /var/dump/nginx;
```

###### 外部资源

- [Debugging - Core dump](https://www.nginx.com/resources/wiki/start/topics/tutorials/debugging/#core-dump)
- [Debugging (from this handbook)](HELPERS.md#debugging)
- [Dump a process's memory (from this handbook)](HELPERS.md#dump-a-processs-memory)
- [Debugging socket leaks (from this handbook)](HELPERS.md#debugging-socket-leaks)
- [Debugging Symbols (from this handbook)](HELPERS.md#debugging-symbols)

<a id="beginner-use-mirror-module-to-copy-requests-to-another-backend"></a>
#### :beginner: 使用 mirror 模块将请求复制到另一个后端

###### 说明

  > 流量镜像非常有用：
  >
  >   - 分析和调试原始请求
  >   - 预生产测试（处理真实的生产流量）
  >   - 记录请求以进行安全分析和内容检查
  >   - 流量故障排除（诊断错误）
  >   - 将对生产系统进行最小更改的情况下将真实流量复制到测试环境

  > 镜像本身不会影响原始请求（只分析请求，不分析响应）。而且，镜像后端的错误不会影响主后端。

  如果你使用镜像，请记住：

  > _延迟处理下一个请求是 NGINX 中镜像实现的一个已知副作用，这不太可能改变。关键是要确保这确实是事实。_
  >
  > _通常镜像子请求不会影响主请求。但是镜像有两个问题：_
  >
  >   - _同一连接上的下一个请求将不会得到处理，直到所有镜像子请求完成。尝试为主 location 禁用 keepalive，看看是否有帮助_
  >
  >   - _如果你使用 `sendfile` 和 `tcp_nopush`，镜像子请求可能导致响应无法正确推送，从而可能导致延迟。关闭 `sendfile`，看看是否有帮助_

###### 示例

```nginx
location / {

  log_subrequest on;

  mirror /backend-mirror;
  mirror_request_body on;

  proxy_pass http://bk_web01;

  # Indicates whether the header fields of the original request
  # and original request body are passed to the proxied server:
  proxy_pass_request_headers on;
  proxy_pass_request_body on;

  # Uncomment if you have problems with latency:
  # keepalive_timeout 0;

}

location = /backend-mirror {

  internal;
  proxy_pass http://bk_web01_debug$request_uri;

  # Pass the headers that will be sent to the mirrored backend:
  proxy_set_header M-Server-Port $server_port;
  proxy_set_header M-Server-Addr $server_addr;
  proxy_set_header M-Host $host; # or $http_host for <host:port>
  proxy_set_header M-Real-IP $remote_addr;
  proxy_set_header M-Request-ID $request_id;
  proxy_set_header M-Original-URI $request_uri;

}
```

###### 外部资源

- [Module ngx_http_mirror_module](http://nginx.org/en/docs/http/ngx_http_mirror_module.html)
- [nginx mirroring tips and tricks](https://alex.dzyoba.com/blog/nginx-mirror/)

<a id="performance"></a>
# 性能

返回 **[目录](https://github.com/trimstray/nginx-admins-handbook#table-of-contents)** 或 **[下一步？](https://github.com/trimstray/nginx-admins-handbook#whats-next)** 部分。

  > :pushpin:&nbsp; NGINX 非常快，但你可以调整一些东西来确保它对你的用例尽可能快。

- **[基础规则](#base-rules)**
- **[调试](#debugging)**
- **[≡ 性能 (13)](#performance)**
  * [调整工作进程数](#beginner-adjust-worker-processes)
  * [使用 HTTP/2](#beginner-use-http2)
  * [维护 SSL 会话](#beginner-maintaining-ssl-sessions)
  * [启用 OCSP Stapling](#beginner-enable-ocsp-stapling)
  * [尽可能在 server_name 指令中使用精确名称](#beginner-use-exact-names-in-a-server_name-directive-if-possible)
  * [避免使用 if 指令检查 server_name](#beginner-avoid-checks-server_name-with-if-directive)
  * [使用 $request_uri 避免使用正则表达式](#beginner-use-request_uri-to-avoid-using-regular-expressions)
  * [使用 try_files 指令确保文件存在](#beginner-use-try_files-directive-to-ensure-a-file-exists)
  * [使用 return 指令代替 rewrite 进行重定向](#beginner-use-return-directive-instead-of-rewrite-for-redirects)
  * [启用 PCRE JIT 加速正则表达式处理](#beginner-enable-pcre-jit-to-speed-up-processing-of-regular-expressions)
  * [激活到上游服务器的连接缓存](#beginner-activate-the-cache-for-connections-to-upstream-servers)
  * [使用精确 location 匹配加速选择过程](#beginner-make-an-exact-location-match-to-speed-up-the-selection-process)
  * [使用 limit_conn 改进下载速度限制](#beginner-use-limit_conn-to-improve-limiting-the-download-speed)
- **[加固](#hardening)**
- **[反向代理](#reverse-proxy)**
- **[负载均衡](#load-balancing)**
- **[其他](#others)**

<a id="beginner-adjust-worker-processes"></a>
#### :beginner: 调整工作进程数

###### 说明

  > `worker_processes` 指令是 NGINX 的坚实脊梁。该指令负责让我们的虚拟服务器知道在绑定到正确的 IP 和端口后要生成多少个工作进程，其值在 CPU 密集型工作中很有帮助。

  > 最安全的设置是通过传递 `auto` 使用核心数。你可以调整此值以在高并发下获得最大吞吐量。该值应根据可用核心数、磁盘、网络子系统、服务器负载等更改为最佳值。

  > 你需要多少工作进程？进行多次负载测试。用一个工作进程高强度测试应用程序，观察结果。然后逐步增加并再次测试。在某个时候，你会达到真正饱和服务器资源的程度。那时你就知道找到了正确的平衡。

  > 在我看来，对于高负载代理服务器（也包括独立服务器），有趣的值是 `ALL_CORES - 1`（或更多），因为如果你在同一服务器上运行 NGINX 和其他关键服务，管理所有这些进程所需的上下文切换会占用 CPU。

  > 经验法则：如果大量时间花费在 I/O 阻塞上，应进一步增加工作进程数。

  > 增加工作进程数是克服单 CPU 核心瓶颈的好方法，但可能会带来一整套[新问题](https://blog.cloudflare.com/the-sad-state-of-linux-socket-balancing/)。

  官方 NGINX 文档说：

  > _当有疑问时，将其设置为可用 CPU 核心数是一个好的开始（值 "auto" 将尝试自动检测）。[...] 每个 CPU 核心运行一个工作进程 - 可以最有效地利用硬件资源。_

###### 示例

```nginx
# The safest and recommend way:
worker_processes auto;

# Alternative:
# VCPU = 4 , expr $(nproc --all) - 1, grep "processor" /proc/cpuinfo | wc -l
worker_processes 3;
```

###### 外部资源

- [Nginx Core Module - worker_processes](https://nginx.org/en/docs/ngx_core_module.html#worker_processes)
- [Processes (from this handbook)](NGINX_BASICS.md#processes)

<a id="beginner-use-http2"></a>
#### :beginner: 使用 HTTP/2

###### 说明

  > HTTP/2 将使我们的应用程序更快、更简单、更健壮。HTTP/2 的主要目标是通过实现完整的请求和响应多路复用来减少延迟，通过有效压缩 HTTP 头字段来最小化协议开销，并添加对请求优先级和服务器推送的支持。HTTP/2 还有一个非常大的旧和不安全密码的[黑名单](https://http2.github.io/http2-spec/#BadCipherSuites)。

  > `http2` 指令配置端口以接受 HTTP/2 连接。这并不意味着它只接受 HTTP/2 连接。HTTP/2 向后兼容 HTTP/1.1，因此完全忽略它也可以，一切都会像以前一样继续工作，因为不支持 HTTP/2 的客户端永远不会向服务器请求 HTTP/2 通信升级：它们之间的通信将完全是 HTTP1/1。

  > HTTP/2 在单个 TCP 连接内多路复用多个请求。通常，使用 HTTP/2 时，会与服务器建立单个 TCP 连接。

  > 你还应该启用 `ssl` 参数（但 NGINX 也可以配置为在不使用 SSL 的情况下接受 HTTP/2 连接），这是因为浏览器不支持没有加密的 HTTP/2（h2 规范允许在不安全的 `http://` 方案上使用 HTTP/2，但浏览器尚未实现这一点（大多数也不打算实现））。请注意，通过 TLS 接受 HTTP/2 连接需要 "Application-Layer Protocol Negotiation"（ALPN）TLS 扩展支持。

  > 显然，有得必有失。HTTP/2 比 HTTP/1.1 更安全，但在 HTTP/2 协议中发现了一些严重的漏洞。更多信息请参见 [HTTP/2 can shut you down!](https://www.secpod.com/blog/http2-dos-vulnerabilities/)、[On the recent HTTP/2 DoS attacks](https://blog.cloudflare.com/on-the-recent-http-2-dos-attacks/) 和 [HTTP/2: In-depth analysis of the top four flaws of the next generation web protocol](https://www.imperva.com/docs/Imperva_HII_HTTP2.pdf) <sup>[pdf]</sup>。

  > 别忘了向后兼容 HTTP/1.1，在安全方面也是如此。HTTP/1.1 的许多漏洞可能也存在于 HTTP/2 中。

  > 要使用 [RFC 7540](https://tools.ietf.org/html/rfc7540) <sup>[IETF]</sup> (HTTP/2) 和 [RFC 7541](https://tools.ietf.org/html/rfc7541) <sup>[IETF]</sup> (HPACK) 测试你的服务器，请使用 [h2spec](https://github.com/summerwind/h2spec) 工具。

###### 示例

```nginx
server {

  listen 10.240.20.2:443 ssl http2;

  ...
```

###### 外部资源

- [RFC 7540 - HTTP/2](https://tools.ietf.org/html/rfc7540) <sup>[IETF]</sup>
- [RFC 7540 - HTTP/2: Security Considerations](https://tools.ietf.org/html/rfc7540#section-10) <sup>[IETF]</sup>
- [Introduction to HTTP/2](https://developers.google.com/web/fundamentals/performance/http2/)
- [What is HTTP/2 - The Ultimate Guide](https://kinsta.com/learn/what-is-http2/)
- [The HTTP/2 Protocol: Its Pros & Cons and How to Start Using It](https://www.upwork.com/hiring/development/the-http2-protocol-its-pros-cons-and-how-to-start-using-it/)
- [HTTP/2 Compatibility with old Browsers and Servers](http://qnimate.com/http2-compatibility-with-old-browsers-and-servers/)
- [HTTP 2 protocol – it is faster, but is it also safer?](https://research.securitum.com/http-2-protocol-it-is-faster-but-is-it-also-safer/)
- [HTTP/2 Denial of Service Advisory](https://github.com/Netflix/security-bulletins/blob/master/advisories/third-party/2019-002.md)
- [HTTP/2, Brute! Then fall, server. Admin! Ops! The server is dead](https://www.theregister.co.uk/2019/08/14/http2_flaw_server/)
- [HTTP Headers (from this handbook)](HTTP_BASICS.md#http-headers)

<a id="beginner-maintaining-ssl-sessions"></a>
#### :beginner: 维护 SSL 会话

###### 说明

  > 默认的 "内置" 会话缓存并不理想，因为它只能被一个工作进程使用，并且可能导致内存碎片。

  > 使用 `ssl_session_cache` 指令启用会话缓存有助于减少 NGINX 服务器的 CPU 负载。这也从客户端的角度提高了性能，因为它消除了每次请求时进行新的（且耗时的）SSL 握手的需求。更重要的是，使用共享缓存要好得多。

  > 使用 `ssl_session_cache` 时，SSL 上的 keep-alive 连接性能可能会大幅提升。10M 是一个好的起始值（1MB 共享缓存可以容纳大约 4000 个会话）。使用 `shared` 时，缓存在所有工作进程之间共享（相同名称的缓存可以在多个虚拟服务器中使用）。

  > 对于 TLSv1.2，[RFC 5246 - 恢复会话](https://tools.ietf.org/html/rfc5246#appendix-F.1.4) 建议会话保持存活时间不超过 24 小时（这是最长时间）。通常，除非客户端和服务器都同意，否则无法恢复 TLS 会话，如果任何一方怀疑会话可能已被入侵，或者证书可能已过期或撤销，则应强制进行完整握手。但不久前，我发现 `ssl_session_timeout` 设置较短时间（例如 15 分钟）以防止被广告商（如 Google 和 Facebook）滥用，我不知道，我猜这有道理。

  另一方面，[RFC 5077 - 票据生命周期](https://tools.ietf.org/html/rfc5077#section-5.6) 说：

  > _票据生命周期可能长于 [RFC4346](https://tools.ietf.org/html/rfc4346) 中推荐的 24 小时生命周期。TLS 客户端可能会得到票据生命周期的提示。由于票据的生命周期可能未指定，客户端有自己的本地策略来确定何时丢弃票据。_

  > 大多数服务器不会清除会话或票据密钥，从而增加了服务器被入侵时会泄露以前（和未来）连接数据的风险。

  > [Vincent Bernat](https://vincent.bernat.ch/en) 编写了一个很好的[工具](https://github.com/vincentbernat/rfc5077/blob/master/rfc5077-client.c)，用于测试有无票据的会话恢复。

  [Ivan Ristić](https://twitter.com/ivanristic)（Hardenize 创始人）说：

  > _会话恢复要么创建一个可以被破解的大型服务器端缓存，要么使用票据会破坏前向保密。所以你需要在性能（不希望用户在每次连接时都进行完整握手）和安全（不希望过度妥协）之间取得平衡。不同的项目要求不同的设置。[...] 不使用非常大缓存（仅仅因为你可以）的一个原因是，流行的实现实际上不会从那里删除任何记录；即使过期的会话仍然在缓存中并且可以恢复。真正删除的唯一方法是使用新会话覆盖它们。[...] 如今我可能会将最大会话持续时间减少到 4 小时，而不是我书中目前的 24 小时。但这主要基于直觉，认为 4 小时足够你获得性能收益，而使用更短的生命周期总是更好。_

  [Ilya Grigorik](https://www.igvita.com/)（Google 网络性能工程师）关于 SSL 缓冲区说：

  > _1400 字节（实际上，可能应该更低一点）是交互式流量的推荐设置，你希望避免因 TLS 记录片段的数据包丢失/抖动而导致的任何不必要的延迟。然而，将每个 TLS 记录打包到专用数据包中确实会增加一些帧开销，如果你传输较大的（且对延迟不太敏感的）数据，可能需要更大的记录大小。4K 是一个 "合理" 但对任何一种情况都不是很好 的中间值。对于较小的记录，我们还应该为各种 TCP 选项（时间戳、SACK，最多 40 字节）预留空间，并考虑 TLS 记录开销（平均另外 20-60 字节，取决于协商的密码套件）。总之：1500 - 40（IP）- 20（TCP）- 40（TCP 选项）- TLS 开销（60-100）~= 1300 字节。如果你检查 Google 服务器发出的记录，你会看到由于上述数学计算，它们携带约 1300 字节的应用程序数据。_

  另一个推荐（在我看来作者是 Leif Hedstrom、Thomas Jackson、Brian Geffon）是使用以下值：

  > - 较小的 TLS 记录大小：MTU/MSS (1500) 减去 TCP (20 字节) 和 IP (40 字节) 开销：1500 - 40 - 20 = 1440 字节<br>
  > - 较大的 TLS 记录大小：最大 TLS 记录大小为 16383 (2^14 - 1)

###### 示例

```nginx
ssl_session_cache shared:NGX_SSL_CACHE:10m;
ssl_session_timeout 4h;
ssl_session_tickets off;
ssl_buffer_size 1400;
```

###### 外部资源

- [SSL Session (cache)](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_cache)
- [Speeding up TLS: enabling session reuse](https://vincent.bernat.ch/en/blog/2011-ssl-session-reuse-rfc5077)
- [SSL Session Caching (in nginx)](https://www.hezmatt.org/~mpalmer/blog/2011/06/28/ssl-session-caching-in-nginx.html)
- [ssl_session_cache in Nginx and the ab benchmark](https://www.peterbe.com/plog/ssl_session_cache-ab)
- [Improving OpenSSL Performance](https://software.intel.com/en-us/articles/improving-openssl-performance)

<a id="beginner-enable-ocsp-stapling"></a>
#### :beginner: 启用 OCSP Stapling

###### 说明

  > 与 OCSP 不同，在 OCSP Stapling 机制中，用户的浏览器不联系证书颁发者，而是由应用服务器定期执行。

  > 配置 OCSP Stapling 扩展以获得更好的性能（旨在降低 OCSP 验证的成本；提高浏览器与应用服务器的通信性能，并允许在访问应用时检索有关证书有效性的信息），并且用户隐私仍然得到维护。OCSP Stapling 是一种优化，如果不工作也不会破坏任何东西。

  > 使用 OCSP 而不实现 OCSP Stapling 扩展会增加用户隐私丢失的风险，以及由于无法验证证书有效性而对应用程序可用性产生负面影响的风险。

  > OCSP Stapling 在 TLS 证书状态请求（[RFC 6066 - 证书状态请求](https://tools.ietf.org/html/rfc6066#section-8)）扩展（"stapling"）中定义 OCSP 响应。在这种情况下，服务器将 OCSP 响应作为 TLS 扩展的一部分发送，因此客户端无需在 OCSP URL 上检查它（节省了客户端的撤销检查时间）。

  > NGINX 提供了几个需要记住的选项。例如：它从 `ssl_trusted_certificate` 指向的证书文件生成列表（这些证书的列表不会发送给客户端）。你需要发送此列表或关闭 `ssl_verify_client`。当完整的证书链（仅中间证书，不含根 CA，并且不得包含站点证书）已经通过 `ssl_certificate` 语句提供时，此步骤是可选的。如果只使用了证书（而不是 CA 的部分），那么需要使用 `ssl_trusted_certificate`。

  > 我在网上发现，两种类型的链（根 CA + 中间证书或仅中间证书）都可以作为 `ssl_trusted_certificate` 用于 OCSP 验证。不建议也不需要将根证书放在 `ssl_certificate` 中。如果你使用 Let's Encrypt，你不需要将根 CA 添加到 `ssl_trusted_certificate`，因为 OCSP 响应由中间证书本身签名。我认为，最安全的方式是在 `ssl_trusted_certificate` 中包含所有相应的根和中间 CA 证书。

  > 我总是使用最稳定的 DNS 解析器，如 Google 的 `8.8.8.8`、Quad9 的 `9.9.9.9`、CloudFlare 的 `1.1.1.1` 或 OpenDNS 的 `208.67.222.222`（当然你也可以使用 Bind9 或其他工具在内部和外部解析域名）。如果没有添加 `resolver` 行，或者你的 NGINX 没有外部访问权限，解析器将默认为服务器的 DNS 默认值。

  > 你应该知道，太短的解析器超时（默认 30 秒）可能是 OCSP Stapling 失败的另一个原因（暂时性的）。如果 NGINX `resolver_timeout` 指令设置得非常低（< 5 秒），可能会出现这样的日志消息：`"[...] ssl_stapling" ignored, host not found in OCSP responder [...]`。

  > 还要注意，NGINX 延迟加载 OCSP 响应。因此，第一个请求不会有 staple 响应，但后续请求会。这是因为 NGINX 不会在服务器启动时（或重新加载后）预取 OCSP 响应。

  来自 NGINX 文档的重要信息：

  > _要使 OCSP stapling 工作，服务器证书颁发者的证书应该是已知的。如果 `ssl_certificate` 文件不包含中间证书，服务器证书颁发者的证书应该出现在 `ssl_trusted_certificate` 文件中。_

  > _为了防止 DNS 欺骗（`resolver`），建议在适当保护的可信本地网络中配置 DNS 服务器。_

###### 示例

```nginx
# Turn on OCSP Stapling:
ssl_stapling on;

# Enable the server to check OCSP:
ssl_stapling_verify on;

# Point to a trusted CA (the company that signed our CSR) certificate chain
# (Intermediate certificates in that order from top to bottom) file, but only,
# if NGINX can not find the top level certificates from ssl_certificate:
ssl_trusted_certificate /etc/nginx/ssl/inter-CA-chain.pem

# For a resolution of the OCSP responder hostname, set resolvers and their cache time:
resolver 1.1.1.1 8.8.8.8 valid=300s;
resolver_timeout 5s;
```

要测试 OCSP Stapling：

```bash
openssl s_client -connect example.com:443 -servername example.com -tlsextdebug -status
echo | openssl s_client -connect example.com:443 -servername example.com -status 2> /dev/null | grep -A 17 'OCSP response:'
```

###### 外部资源

- [RFC 2560 - X.509 Internet Public Key Infrastructure Online Certificate Status Protocol - OCSP](https://tools.ietf.org/html/rfc2560)
- [OCSP Stapling on nginx](https://raymii.org/s/tutorials/OCSP_Stapling_on_nginx.html)
- [OCSP Stapling: Performance](https://www.tunetheweb.com/performance/ocsp-stapling/)
- [OCSP Stapling; SSL with added speed and privacy](https://scotthelme.co.uk/ocsp-stapling-speeding-up-ssl/)
- [High-reliability OCSP stapling and why it matters](https://blog.cloudflare.com/high-reliability-ocsp-stapling/)
- [OCSP Stapling: How CloudFlare Just Made SSL 30% Faster](https://blog.cloudflare.com/ocsp-stapling-how-cloudflare-just-made-ssl-30/)
- [Is the web ready for OCSP Must-Staple?](https://blog.apnic.net/2019/01/15/is-the-web-ready-for-ocsp-must-staple/)
- [The case for "OCSP Must-Staple"](https://www.grc.com/revocation/ocsp-must-staple.htm)
- [Page Load Optimization: OCSP Stapling](https://www.ssl.com/article/page-load-optimization-ocsp-stapling/)
- [ImperialViolet - No, don't enable revocation checking](https://www.imperialviolet.org/2014/04/19/revchecking.html)
- [The Problem with OCSP Stapling and Must Staple and why Certificate Revocation is still broken](https://blog.hboeck.de/archives/886-The-Problem-with-OCSP-Stapling-and-Must-Staple-and-why-Certificate-Revocation-is-still-broken.html)
- [Damn it, nginx! stapling is busted](https://blog.crashed.org/nginx-stapling-busted/)
- [Priming the OCSP cache in Nginx](https://unmitigatedrisk.com/?p=241)
- [How to make OCSP stapling on nginx work](https://matthiasadler.info/blog/ocsp-stapling-on-nginx-with-comodo-ssl/)
- [HAProxy OCSP stapling](https://icicimov.github.io/blog/server/HAProxy-OCSP-stapling/)
- [DNS Resolvers Performance compared: CloudFlare x Google x Quad9 x OpenDNS](https://medium.com/@nykolas.z/dns-resolvers-performance-compared-cloudflare-x-google-x-quad9-x-opendns-149e803734e5)
- [OCSP Validation with OpenSSL](https://akshayranganath.github.io/OCSP-Validation-With-Openssl/)

<a id="beginner-use-exact-names-in-a-server_name-directive-if-possible"></a>
#### :beginner: 尽可能在 `server_name` 指令中使用精确名称

###### 说明

  > 精确名称、以星号开头的通配符名称和以星号结尾的通配符名称存储在绑定到监听端口的三个哈希表中。

  > 首先搜索精确名称哈希表。因此，如果服务器最常请求的名称是 `example.com` 和 `www.example.com`，显式定义它们会更高效。

  > 如果未找到精确名称，则搜索以星号开头的通配符名称哈希表。如果仍未找到，则搜索以星号结尾的通配符名称哈希表。搜索通配符名称哈希表比搜索精确名称哈希表慢，因为名称是按域部分搜索的。

  > 按名称搜索虚拟服务器时，如果名称匹配多个指定变体，例如通配符名称和正则表达式都匹配，将按以下优先顺序选择第一个匹配的变体：
  >
  > - 精确名称
  > - 最长的以星号开头的通配符名称，例如 `*.example.org`
  > - 最长的以星号结尾的通配符名称，例如 `mail.*`
  > - 第一个匹配的正则表达式（按在配置文件中出现的顺序）

  > 正则表达式按顺序测试，因此是最慢的方法且不可扩展。因此，最好尽可能使用精确名称。

  来自官方文档：

  > _通配符名称只能在名称的开头或结尾包含星号，并且只能在点边界上。名称 `www.*.example.org` 和 `w*.example.org` 无效。[...] 特殊通配符名称 `.example.org` 可用于同时匹配精确名称 `example.org` 和通配符名称 `*.example.org`。_

  > _名称 `*.example.org` 不仅匹配 `www.example.org`，还匹配 `www.sub.example.org`。_

  > _要使用正则表达式，服务器名称必须以波浪号开头。[...] 否则将被视为精确名称，或者如果表达式包含星号，则被视为通配符名称（并且很可能被视为无效）。不要忘记设置 `^` 和 `$` 锚点。它们在语法上不是必需的，但在逻辑上是必需的。还要注意域名点应使用反斜杠转义。包含字符 `{` 和 `}` 的正则表达式应被引号括起来。_

###### 示例

不推荐的配置：

```nginx
server {

  listen 192.168.252.10:80;

  # From official documentation: "Searching wildcard names hash table is slower than searching exact names
  # hash table because names are searched by domain parts. Note that the special wildcard form
  # '.example.org' is stored in a wildcard names hash table and not in an exact names hash table.":
  server_name .example.org;

  ...

}
```

推荐的配置：

```nginx
# It is more efficient to define them explicitly:
server {

  listen 192.168.252.10:80;

  # .example.org = *.example.org
  server_name example.org www.example.org *.example.org;

  ...

}
```

###### 外部资源

- [Server names](https://nginx.org/en/docs/http/server_names.html)
- [Server Naming Conventions and Best Practices](https://blog.serverdensity.com/server-naming-conventions-and-best-practices/)
- [Server/Device Naming](https://www.vita.virginia.gov/media/vitavirginiagov/it-governance/ea/pdf/Server-Device-Naming-Technical-Brief.pdf) <sup>[pdf]</sup>
- [Handle incoming connections (from this handbook)](NGINX_BASICS.md#handle-incoming-connections)

<a id="beginner-avoid-checks-server_name-with-if-directive"></a>
#### :beginner: 避免使用 `if` 指令检查 `server_name`

###### 说明

  > 当 NGINX 收到请求时，无论请求的是哪个子域名，是 `www.example.com` 还是普通的 `example.com`，这个 `if` 指令总是被评估。因为你要求 NGINX 为每个请求检查 `Host` 头。这可能极其低效。

  > 相反，使用两个 server 指令，如下面的示例所示。这种方法减少了 NGINX 的处理需求。

  > 问题不仅仅在于 `$server_name` 指令。还要记住其他变量，例如 `$scheme`。在某些情况下（但并非总是），添加额外的块指令比使用 `if` 更好。

  另一方面，官方文档说：

  > _指令 if 在 location 上下文中使用时存在问题，在某些情况下它不会按预期工作，而是完全不同的东西。在某些情况下它甚至会 segfault。通常，如果可能的话，最好避免使用它。_

###### 示例

不推荐的配置：

```nginx
server {

  server_name example.com www.example.com;

  if ($host = www.example.com) {

    return 301 https://example.com$request_uri;

  }

  server_name example.com;

  ...

}
```

推荐的配置：

```nginx
server {

    listen 192.168.252.10:80;

    server_name www.example.com;

    return 301 $scheme://example.com$request_uri;

    # If you force your web traffic to use HTTPS:
    # return 301 https://example.com$request_uri;

    ...

}

server {

    listen 192.168.252.10:80;

    server_name example.com;

    ...

}
```

###### 外部资源

- [If Is Evil](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/)
- [if, break, and set (from this handbook)](NGINX_BASICS.md#if-break-and-set)

<a id="beginner-use-request_uri-to-avoid-using-regular-expressions"></a>
#### :beginner: 使用 `$request_uri` 避免使用正则表达式

###### 说明

  > 使用内置的 `$request_uri`，我们可以有效地避免任何捕获或匹配。默认情况下，正则表达式成本高昂，会降低性能。

  > 此规则涉及将 URL 原样传递到新主机，当然 return 更高效，只需传递现有 URI。

  > `$request_uri` 的值始终是从客户端接收的原始 URI（包含参数的完整原始请求 URI），不进行任何规范化处理，这与 `$uri` 指令不同。

  > 如果需要匹配 URI 及其查询字符串，请在 map 指令中使用 `$request_uri`。

  > 不加考虑地使用 `$request_uri` 可能会导致许多奇怪的行为。例如，在错误的地方使用 `$request_uri` 可能导致 URL 编码字符被双重编码。因此大多数时候你会使用 `$uri`，因为它被规范化了。

  我认为最好的解释来自官方文档：

  > _不要感到难过，正则表达式很容易让人困惑。事实上，很容易出错，所以我们应该努力保持它们整洁干净。_

###### 示例

不推荐的配置：

```nginx
# 1)
rewrite ^/(.*)$ https://example.com/$1 permanent;

# 2)
rewrite ^ https://example.com$request_uri permanent;
```

推荐的配置：

```nginx
return 301 https://example.com$request_uri;
```

###### 外部资源

- [Pitfalls and Common Mistakes - Taxing Rewrites](https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/#taxing-rewrites)
- [Module ngx_http_proxy_module - proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)
- [uri vs request_uri (from this handbook)](NGINX_BASICS.md#uri-vs-request_uri)

<a id="beginner-use-try_files-directive-to-ensure-a-file-exists"></a>
#### :beginner: 使用 `try_files` 指令确保文件存在

###### 说明

  > `try_files` 绝对是一个非常有用东西。你可以使用 `try_files` 指令按指定顺序检查文件是否存在。

  > 你应该使用 `try_files` 而不是 `if` 指令。这比使用 `if` 好得多，因为 `if` 指令极其低效，每次请求都会被评估。

  > 使用 `try_files` 的优点是可以用一个命令立即切换行为。我认为代码也更易读。

  > `try_files` 允许你：
  >
  > - 从预定义列表中检查文件是否存在
  > - 从指定目录检查文件是否存在
  > - 如果未找到任何文件，则使用内部重定向

###### 示例

不推荐的配置：

```nginx
server {

  ...

  root /var/www/example.com;

  location /images {

    if (-f $request_filename) {

      expires 30d;
      break;

    }

  ...

}
```

推荐的配置：

```nginx
server {

  ...

  root /var/www/example.com;

  location /images {

    try_files $uri =404;

  ...

}
```

###### 外部资源

- [Creating NGINX Rewrite Rules](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)
- [Pitfalls and Common Mistakes](https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/)
- [Serving Static Content](https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/)
- [Serve files with nginx conditionally](http://www.lazutkin.com/blog/2014/02/23/serve-files-with-nginx-conditionally/)
- [try_files directive (from this hadnbook)](NGINX_BASICS.md#try_files-directive)

<a id="beginner-use-return-directive-instead-of-rewrite-for-redirects"></a>
#### :beginner: 使用 `return` 指令代替 `rewrite` 进行重定向

###### 说明

  > 对我来说，在 NGINX 中重写 URL 是一个非常强大和重要的功能。从技术上讲，你可以使用这两种选项，但在我看来，你应该使用 `server` 块和 `return` 语句，因为它们比通过 `location` 块评估正则表达式更简单、更快。

  > NGINX 必须处理和启动搜索。`return` 指令停止处理（它直接停止执行）并将指定的代码返回给客户端。在任何上下文中，这都更可取。

  > 如果你需要通过正则表达式验证 URL，或需要捕获原始 URL 中的元素（这些元素显然不在相应的 NGINX 变量中），那么你应该使用 `rewrite`。

###### 示例

不推荐的配置：

```nginx
server {

  ...

  location / {

    try_files $uri $uri/ =404;

    rewrite ^/(.*)$ https://example.com/$1 permanent;

  }

  ...

}
```

推荐的配置：

```nginx
server {

  ...

  location / {

    try_files $uri $uri/ =404;

    return 301 https://example.com$request_uri;

  }

  ...

}
```

###### 外部资源

- [NGINX - rewrite vs redirect](http://think-devops.com/blogs/nginx-rewrite-redirect.html)
- [If Is Evil](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/)
- [rewrite vs return (from this handbook)](NGINX_BASICS.md#rewrite-vs-return)
- [Use return directive for URL redirection (301, 302) - Base Rules - P2 (from this handbook)](#beginner-use-return-directive-for-url-redirection-301-302)

<a id="beginner-enable-pcre-jit-to-speed-up-processing-of-regular-expressions"></a>
#### :beginner: 启用 PCRE JIT 加速正则表达式处理

###### 说明

  > 启用 JIT 用于正则表达式以加速其处理。具体来说，检查规则可能很耗时，尤其是复杂的正则表达式条件。

  > 通过使用 PCRE 库编译 NGINX，你可以使用 `location` 块执行复杂操作，并使用强大的 `rewrite` 指令。

  > PCRE JIT 规则匹配引擎可以显著加速正则表达式的处理。启用 `pcre_jit` 的 NGINX 比没有的快很多数量级。此选项可以提高性能，但在某些情况下，`pcre_jit` 可能产生负面影响。因此，在启用它之前，我建议你阅读这篇重要文档：[PCRE Performance Project](https://zherczeg.github.io/sljit/pcre.html)。

  > 如果你在没有 JIT 的情况下尝试使用 `pcre_jit on;`，或者 NGINX 编译时启用了 JIT，但当前加载的 PCRE 库不支持 JIT，将在配置解析期间发出警告。

  > `--with-pcre-jit` 仅在您使用 NGINX configure 编译 PCRE 库时需要（`./configure --with-pcre=`）。使用系统 PCRE 库时，是否支持 JIT 取决于库的编译方式。

  > 如果你不传递 `--with-pcre-jit`，NGINX configure 脚本足够智能，可以自动检测并启用它。参见[这里](http://hg.nginx.org/nginx/file/abd40ce603fa/auto/lib/pcre/conf)。因此，如果你的 PCRE 库足够新，简单的 `./configure` 不带任何开关将编译启用 `pcre_jit` 的 NGINX。

  来自 NGINX 文档：

  > _从版本 8.20 开始，使用 `--enable-jit` 配置参数构建的 PCRE 库中提供 JIT。当使用 nginx 构建 PCRE 库时（`--with-pcre=`），通过 `--with-pcre-jit` 配置参数启用 JIT 支持。_

###### 示例

```nginx
# In global context:
pcre_jit on;
```

###### 外部资源

- [Core functionality - pcre jit](https://nginx.org/en/docs/ngx_core_module.html#pcre_jit)
- [Performance comparison of regular expression engines](https://nasciiboy.land/raptorVSworld/)
- [Building OpenResty with PCRE JIT](https://www.cryptobells.com/building-openresty-with-pcre-jit/)

<a id="beginner-activate-the-cache-for-connections-to-upstream-servers"></a>
#### :beginner: 激活到上游服务器的连接缓存

###### 说明

  > keepalive 的思想是解决在高延迟网络上建立 TCP 连接的延迟问题。这种连接缓存在 NGINX 必须保持一定数量的到上游服务器的开放连接的情况下非常有用。

  > Keep-Alive 连接可以通过减少打开和关闭连接所需的 CPU 和网络开销来对性能产生重大影响。在 NGINX 上游服务器中启用 HTTP keepalive 可以降低延迟，从而提高性能，并减少 NGINX 耗尽临时端口的可能性。

  > 这可以大大减少新 TCP 连接的数量，因为 NGINX 现在可以重用每个上游的现有连接（`keepalive`）。

  > 如果你的上游服务器在其配置中支持 Keep-Alive，NGINX 将重用现有 TCP 连接，而不会创建新的。这可以大大减少繁忙服务器上 `TIME_WAIT` TCP 连接中的套接字数量（减少操作系统建立新连接的工作量，减少网络上的数据包）。

  > Keep-Alive 连接仅在 HTTP/1.1 及以上版本中支持。

###### 示例

```nginx
# Upstream context:
upstream backend {

  # Sets the maximum number of idle keepalive connections to upstream servers
  # that are preserved in the cache of each worker process.
  keepalive 16;

}

# Server/location contexts:
server {

  ...

  location / {

    # By default only talks HTTP/1 to the upstream,
    # keepalive is only enabled in HTTP/1.1:
    proxy_http_version 1.1;

    # Remove the Connection header if the client sends it,
    # it could be "close" to close a keepalive connection:
    proxy_set_header Connection "";

    ...

  }

}
```

###### 外部资源

- [NGINX keeps sending requests to offline upstream](https://serverfault.com/a/883019)
- [HTTP Keep-Alive connections (from this handbook)](NGINX_BASICS.md#http-keep-alive-connections)

<a id="beginner-make-an-exact-location-match-to-speed-up-the-selection-process"></a>
#### :beginner: 使用精确 location 匹配加速选择过程

###### 说明

  > 精确 location 匹配通常用于通过立即结束算法的执行来加速选择过程。

  > 存在时的正则表达式优先于简单的 URI 匹配，并且根据其复杂性，可能会增加显著的计算开销。

  > 使用 `=` 修饰符可以定义 URI 和 location 的精确匹配。处理速度非常快，可以节省大量的 CPU 周期。

  > 如果找到精确匹配，搜索终止。例如，如果 `/` 请求频繁发生，定义 `location = /` 将加速这些请求的处理，因为搜索在第一次比较后立即终止。这样的 location 显然不能包含嵌套 location。

###### 示例

```nginx
# Matches the query / only and stops searching:
location = / {

  ...

}

# Matches the query /v9 only and stops searching:
location = /v9 {

  ...

}

...

# Matches any query due to the fact that all queries begin at /,
# but regular expressions and any longer conventional blocks will be matched at first place:
location / {

  ...

}
```

###### 外部资源

- [Untangling the nginx location block matching algorithm](https://artfulrobot.uk/blog/untangling-nginx-location-block-matching-algorithm)

<a id="beginner-use-limit_conn-to-improve-limiting-the-download-speed"></a>
#### :beginner: 使用 `limit_conn` 改进下载速度限制

###### 说明

  > NGINX 提供了两个限制下载速度的指令：
  >   - `limit_rate_after` - 设置在 `limit_rate` 指令生效之前传输的数据量
  >   - `limit_rate` - 允许你限制单个客户端连接的传输速率（超过 `limit_rate_after` 后）

  > 此解决方案限制每个连接的 NGINX 下载速度，因此，如果一个用户打开多个视频文件，他将能够以 `X * 连接次数` 的速度下载视频文件。

  > 为了防止这种情况，请使用 `limit_conn_zone` 和 `limit_conn` 指令。

###### 示例

```nginx
# Create limit connection zone:
limit_conn_zone $binary_remote_addr zone=conn_for_remote_addr:1m;

# Add rules to limiting the download speed:
limit_rate_after 1m;  # run at maximum speed for the first 1 megabyte
limit_rate 250k;      # and set rate limit after 1 megabyte

# Enable queue:
location /videos {

  # Max amount of data by one client: 10 megabytes (limit_rate_after * 10)
  limit_conn conn_for_remote_addr 10;

  ...
```

###### 外部资源

- [How to Limit Nginx download Speed](https://www.scalescale.com/tips/nginx/how-to-limit-nginx-download-speed/)

<a id="hardening"></a>
# 加固

返回 **[目录](https://github.com/trimstray/nginx-admins-handbook#table-of-contents)** 或 **[下一步？](https://github.com/trimstray/nginx-admins-handbook#whats-next)** 部分。

  > :pushpin:&nbsp; 在本章中，我将讨论一些 NGINX 加固方法和安全标准。

- **[基础规则](#base-rules)**
- **[调试](#debugging)**
- **[性能](#performance)**
- **[≡ 加固 (31)](#hardening)**
  * [始终保持 NGINX 最新](#beginner-always-keep-nginx-up-to-date)
  * [以非特权用户运行](#beginner-run-as-an-unprivileged-user)
  * [禁用不必要的模块](#beginner-disable-unnecessary-modules)
  * [保护敏感资源](#beginner-protect-sensitive-resources)
  * [注意你的 ACL 规则](#beginner-take-care-about-your-acl-rules)
  * [隐藏 Nginx 版本号](#beginner-hide-nginx-version-number)
  * [隐藏 Nginx 服务器签名](#beginner-hide-nginx-server-signature)
  * [隐藏上游代理头](#beginner-hide-upstream-proxy-headers)
  * [移除对遗留和危险的 HTTP 请求头的支持](#beginner-remove-support-for-legacy-and-risky-http-request-headers)
  * [只使用最新支持的 OpenSSL 版本](#beginner-use-only-the-latest-supported-openssl-version)
  * [强制所有连接通过 TLS](#beginner-force-all-connections-over-tls)
  * [使用最少 2048 位 RSA 和 256 位 ECC](#beginner-use-min-2048-bit-for-rsa-and-256-bit-for-ecc)
  * [只保留 TLS 1.3 和 TLS 1.2](#beginner-keep-only-tls-13-and-tls-12)
  * [只使用强密码套件](#beginner-use-only-strong-ciphers)
  * [使用更安全的 ECDH 曲线](#beginner-use-more-secure-ecdh-curve)
  * [使用具有完美前向保密的强密钥交换](#beginner-use-strong-key-exchange-with-perfect-forward-secrecy)
  * [防止零往返时间的重放攻击](#beginner-prevent-replay-attacks-on-zero-round-trip-time)
  * [防御 BEAST 攻击](#beginner-defend-against-the-beast-attack)
  * [缓解 CRIME/BREACH 攻击](#beginner-mitigation-of-crimebreach-attacks)
  * [启用 HTTP 严格传输安全](#beginner-enable-http-strict-transport-security)
  * [降低 XSS 风险（内容安全策略）](#beginner-reduce-xss-risks-content-security-policy)
  * [控制 Referer 头的行为（Referrer-Policy）](#beginner-control-the-behaviour-of-the-referer-header-referrer-policy)
  * [提供点击劫持保护（X-Frame-Options）](#beginner-provide-clickjacking-protection-x-frame-options)
  * [防止某些类别的 XSS 攻击（X-XSS-Protection）](#beginner-prevent-some-categories-of-xss-attacks-x-xss-protection)
  * [防止 MIME 类型嗅探（X-Content-Type-Options）](#beginner-prevent-sniff-mimetype-middleware-x-content-type-options)
  * [拒绝使用浏览器功能（Feature-Policy）](#beginner-deny-the-use-of-browser-features-feature-policy)
  * [拒绝不安全的 HTTP 方法](#beginner-reject-unsafe-http-methods)
  * [防止缓存敏感数据](#beginner-prevent-caching-of-sensitive-data)
  * [限制并发连接](#beginner-limit-concurrent-connections)
  * [控制缓冲区溢出攻击](#beginner-control-buffer-overflow-attacks)
  * [缓解慢速 HTTP DoS 攻击（关闭慢速连接）](#beginner-mitigating-slow-http-dos-attacks-closing-slow-connections)
- **[反向代理](#reverse-proxy)**
- **[负载均衡](#load-balancing)**
- **[其他](#others)**

<a id="beginner-always-keep-nginx-up-to-date"></a>
#### :beginner: 始终保持 NGINX 最新

###### 说明

  > NGINX 非常安全和稳定，但主二进制文件本身确实会不时出现漏洞。这是尽可能保持 NGINX 最新的主要原因。

  > 在规划 NGINX 更新/升级过程时，最简单的方法是直接安装新发布的版本。但对我来说，处理 NGINX 更新的最常见方式是等待稳定版本发布几周后（并阅读社区关于新 NGINX 版本发布后所有可能和已识别问题的评论）。

  > 大多数现代 GNU/Linux 发行版不会将最新版本的 NGINX 推送到它们的默认软件包列表中，因此你可能需要考虑从源代码安装。

  > 在更新/升级 NGINX 之前，请记住：
  >   - 首先在测试环境中进行
  >   - 在更新之前确保备份当前配置

###### 示例

```bash
# RedHat/CentOS
yum install <pkgname>

# Debian/Ubuntu
apt-get install <pkgname>

# FreeBSD/OpenBSD
pkg -f install <pkgname>
```

###### 外部资源

- [Installing from prebuilt packages (from this handbook)](HELPERS.md#installing-from-prebuilt-packages)
- [Installing from source (from this handbook)](HELPERS.md#installing-from-source)

<a id="beginner-run-as-an-unprivileged-user"></a>
#### :beginner: 以非特权用户运行

###### 说明

  > 一个重要的通用原则是程序应具有完成其工作所需的最小权限。这样，如果程序被攻破，其损害是有限的。

  > 仅通过更改进程所有者名称在安全性上没有真正的区别。另一方面，在安全方面，最小权限原则指出，实体在给定系统内应被授予不超过完成其目标所需的权限。这样只有主进程以 root 身份运行。

  > NGINX 满足这些要求，这是默认行为，但请记住检查它。

  来自 [Secure Programming HOWTO - 7.4. Minimize Privileges](https://dwheeler.com/secure-programs/3.71/Secure-Programs-HOWTO/minimize-privileges.html) 文章：

  > _最极端的例子是简单地不编写安全程序 - 如果可能，通常应该这样做。例如，不要使你的程序 `setuid` 或 `setgid`；只需使其成为普通程序，并要求管理员在运行之前以该身份登录。_

###### 示例

```bash
# Edit/check nginx.conf:
user nginx;   # or 'www' for example; if group is omitted,
              # a group whose name equals that of user is used

# Check/set owner and group for root directory:
chown -R root:root /etc/nginx

# Set owner and group for app directory:
chown -R nginx:nginx /var/www/example.com
```

###### 外部资源

- [Why does nginx starts process as root?](https://unix.stackexchange.com/questions/134301/why-does-nginx-starts-process-as-root)
- [How and why Linux daemons drop privileges](https://linux-audit.com/how-and-why-linux-daemons-drop-privileges/)
- [POS36-C. Observe correct revocation order while relinquishing privileges](https://wiki.sei.cmu.edu/confluence/display/c/POS36-C.+Observe+correct+revocation+order+while+relinquishing+privileges)

<a id="beginner-disable-unnecessary-modules"></a>
#### :beginner: 禁用不必要的模块

###### 说明

  > 建议禁用任何不需要的模块，因为这将通过限制 Web 服务器允许的操作来最小化潜在攻击的风险。我还建议只编译和运行经过签名和测试的模块在你的生产环境中。

  > 禁用不需要的模块以减少内存使用并提高性能。不需要的模块只会使加载时间更长。

  > 卸载未使用模块的最佳方法是在安装期间使用 `configure` 选项。如果你静态链接了共享模块，应重新编译 NGINX。

  只使用高质量的模块，并记住：

  > _不幸的是，许多第三方模块使用阻塞调用，用户（有时甚至模块的开发人员）没有意识到缺点。阻塞操作可能会破坏 NGINX 性能，必须不惜一切代价避免。_

###### 示例

```nginx
# 1a) Check which modules can be turn on or off while compiling:
./configure --help | less

# 1b) Turn off during installation:
./configure --without-http_autoindex_module

# 2) Comment modules in the configuration file e.g. modules.conf:
# load_module   /usr/share/nginx/modules/ndk_http_module.so;
# load_module   /usr/share/nginx/modules/ngx_http_auth_pam_module.so;
# load_module   /usr/share/nginx/modules/ngx_http_cache_purge_module.so;
# load_module   /usr/share/nginx/modules/ngx_http_dav_ext_module.so;
load_module     /usr/share/nginx/modules/ngx_http_echo_module.so;
# load_module   /usr/share/nginx/modules/ngx_http_fancyindex_module.so;
load_module     /usr/share/nginx/modules/ngx_http_geoip_module.so;
load_module     /usr/share/nginx/modules/ngx_http_headers_more_filter_module.so;
# load_module   /usr/share/nginx/modules/ngx_http_image_filter_module.so;
# load_module   /usr/share/nginx/modules/ngx_http_lua_module.so;
load_module     /usr/share/nginx/modules/ngx_http_perl_module.so;
# load_module   /usr/share/nginx/modules/ngx_mail_module.so;
# load_module   /usr/share/nginx/modules/ngx_nchan_module.so;
# load_module   /usr/share/nginx/modules/ngx_stream_module.so;
```

###### 外部资源

- [NGINX 3rd Party Modules](https://www.nginx.com/resources/wiki/modules/)
- [nginx-modules](https://github.com/nginx-modules)
- [Emiller's Guide To Nginx Module Development](https://www.evanmiller.org/nginx-modules-guide.html)

<a id="beginner-protect-sensitive-resources"></a>
#### :beginner: 保护敏感资源

###### 说明

  > 隐藏的目录和文件永远不应通过 Web 访问 - 有时关键数据会在应用程序部署期间发布。如果你使用版本控制系统，绝对应该阻止访问（通过向攻击者提供更少信息）关键的隐藏目录/文件，如 `.git` 或 `.svn`，以防止暴露应用程序源代码。

  > 敏感资源包含滥用者可以用来完全重建站点源代码并查找错误、漏洞和暴露密码的内容。

  关于拒绝方法：

  > 在我看来，根据 [RFC 2616 - 403 Forbidden](https://tools.ietf.org/html/rfc2616#section-10.4.4) <sup>[IETF]</sup>，返回 403（或者甚至是 404，为了不泄露信息）如果你知道该资源在任何情况下都不应通过 http 访问，即使是在一般上下文中被 "授权"，也不容易出错。

  另外请注意：

  > 如果你使用带正则表达式的 location，NGINX 会按它们在配置文件中出现的顺序应用它们。你也可以使用 `^~` 修饰符，使前缀 location 块优先于同一级别的任何正则表达式 location 块。

  > NGINX 按阶段处理请求。`return` 指令来自重写模块，`deny` 来自访问模块。重写模块在 `NGX_HTTP_REWRITE_PHASE` 阶段（对于 `location` 上下文中的 `return`）处理，访问模块在 `NGX_HTTP_ACCESS_PHASE` 阶段处理，重写阶段（`return` 所属）在访问阶段（`deny` 所在）之前发生，因此 `return` 在重写阶段停止请求处理并返回 301。

  > `deny all` 将产生相同的后果，但留下出错的可能性。这个问题在[这个](https://serverfault.com/questions/748320/protecting-a-location-by-ip-while-applying-basic-auth-everywhere-else/748373#748373)答案中有所说明，建议不要在 `server { ... }` 级别使用 `satisfy` + `allow` + `deny`，因为继承问题。

  > 另一方面，根据 NGINX 文档：_`ngx_http_access_module` 模块允许限制对特定客户端地址的访问。_ 更具体地说，你无法限制对另一个模块的访问（`return` 更多用于当你想返回其他代码时，而不是阻止访问）。

###### 示例

不推荐的配置：

```nginx
if ($request_uri ~ "/\.git") {

  return 403;

}
```

推荐的配置：

```nginx
# 1) Catch only file names (without file extensions):
# Example: /foo/bar/.git but not /foo/bar/file.git
location ~ /\.git {

  return 403;

}

# 2) Catch file names and file extensions:
# Example: /foo/bar/.git and /foo/bar/file.git
location ~* ^.*(\.(?:git|svn|htaccess))$ {

  deny all;

}
```

最推荐的配置：

```nginx
# Catch all . directories/files excepted .well-known (without file extensions):
# Example: /foo/bar/.git but not /foo/bar/file.git
location ~ /\.(?!well-known\/) {

  deny all;
  access_log /var/log/nginx/hidden-files-access.log main;
  error_log /var/log/nginx/hidden-files-error.log warn;

}
```

再看看具有以下扩展名的文件：

```nginx
# Think also about the following rule (I haven't tested this but looks interesting). It comes from:
#   - https://github.com/h5bp/server-configs-nginx/blob/master/h5bp/location/security_file_access.conf
location ~* (?:#.*#|\.(?:bak|conf|dist|fla|in[ci]|log|orig|psd|sh|sql|sw[op])|~)$ {

  deny all;

}
#   - https://github.com/getgrav/grav/issues/1625
location ~ /(LICENSE\.txt|composer\.lock|composer\.json|nginx\.conf|web\.config|htaccess\.txt|\.htaccess) {

  deny all;

}

# Deny running scripts inside core system directories:
#   - https://github.com/getgrav/grav/issues/1625
location ~* /(system|vendor)/.*\.(txt|xml|md|html|yaml|yml|php|pl|py|cgi|twig|sh|bat)$ {

  return 418;

}

# Deny running scripts inside user directory:
#   - https://github.com/getgrav/grav/issues/1625
location ~* /user/.*\.(txt|md|yaml|yml|php|pl|py|cgi|twig|sh|bat)$ {

  return 418;

}
```

基于以上内容（已测试，我使用这个）：

```nginx
# Catch file names and file extensions:
# Example: /foo/bar/.git and /foo/bar/file.git
location ~* ^.*(\.(?:git|svn|hg|bak|bckp|save|old|orig|original|test|conf|cfg|dist|in[ci]|log|sql|mdb|sw[op]|htaccess|php#|php~|php_bak|aspx?|tpl|sh|bash|bin|exe|dll|jsp|out|cache|))$ {

  # Use also rate limiting:
  # in server context: limit_req_zone $binary_remote_addr zone=per_ip_5r_s:5m rate=5r/s;
  limit_req zone=per_ip_5r_s;

  deny all;
  access_log /var/log/nginx/restricted-files-access.log main;
  access_log /var/log/nginx/restricted-files-error.log main;

}
```

###### 外部资源

- [Hidden directories and files as a source of sensitive information about web application](https://github.com/bl4de/research/tree/master/hidden_directories_leaks)
- [1% of CMS-Powered Sites Expose Their Database Passwords](https://feross.org/cmsploit/)
- [RFC 5785 - Defining Well-Known Uniform Resource Identifiers (URIs)](https://tools.ietf.org/html/rfc5785) <sup>[IETF]</sup>

<a id="beginner-take-care-about-your-acl-rules"></a>
#### :beginner: 注意你的 ACL 规则

###### 说明

  > 在规划访问控制时，考虑几种访问选项。NGINX 提供了 `ngx_http_access_module`、`ngx_http_geo_module`、`ngx_http_map_module` 或 `ngx_http_auth_basic_module` 模块用于允许和拒绝权限。它们各自保护敏感文件和目录。

  > 你应该始终测试你的规则：
  >
  >   - 检查所有使用的指令及其在[请求处理的所有级别](NGINX_BASICS.md#request-processing-stages)的出现/优先级
  >   - 发送测试请求以验证允许或拒绝用户访问 Web 资源（也来自外部/黑名单 IP）
  >   - 发送测试请求以检查并验证所有受保护资源的 HTTP 响应码（参见：[响应码决策图](https://github.com/trimstray/nginx-admins-handbook/blob/master/static/img/http/http_decision_diagram.png)）
  >   - 少即是多，你应该最小化任何用户对关键资源的访问
  >   - 只添加真正需要的 IP 地址，并在 whois 数据库中检查其所有者
  >   - 定期审计你的访问控制规则，确保它们是最新的

  > 如果你使用 `*ACCESS_PHASE`（例如 `allow/deny` 指令），请记住 NGINX 按阶段处理请求，`rewrite` 阶段（`return` 所属）在 `access` 阶段（`deny` 所在）之前。参见[允许和拒绝](NGINX_BASICS.md#allow-and-deny)章节了解更多。这很重要，因为这可能会破坏你的安全层。

  > 然而，不建议使用 `if` 语句，但使用正则表达式可能更灵活一些（更多信息参见[这里](NGINX_BASICS.md#if-break-and-set)）。

###### 示例

- [使用基本身份验证限制访问](HELPERS.md#restricting-access-with-basic-authentication)
- [使用客户端证书限制访问](HELPERS.md#restricting-access-with-client-certificate)
- [按地理位置限制访问](HELPERS.md#restricting-access-by-geographical-location)
- [阻止/允许 IP 地址](HELPERS.md#blockingallowing-ip-addresses)
- [限制引用垃圾邮件](HELPERS.md#limiting-referrer-spam)
- [限制带突发模式的请求速率](HELPERS.md#limiting-the-rate-of-requests-with-burst-mode)
- [限制带突发模式和无延迟的请求速率](HELPERS.md#limiting-the-rate-of-requests-with-burst-mode-and-nodelay)
- [使用 geo 和 map 限制每个 IP 的请求速率](HELPERS.md#limiting-the-rate-of-requests-per-ip-with-geo-and-map)
- [限制连接数](HELPERS.md#limiting-the-number-of-connections)

###### 外部资源

- [Fastly - About ACLs](https://docs.fastly.com/en/guides/about-acls)
- [Restrict allowed HTTP methods in Nginx](https://bjornjohansen.no/restrict-allowed-http-methods-in-nginx)
- [Allow and deny (from this handbook)](NGINX_BASICS.md#allow-and-deny)
- [Protect sensitive resources - Hardening - P1 (from this handbook)](#beginner-protect-sensitive-resources)

<a id="beginner-hide-nginx-version-number"></a>
#### :beginner: 隐藏 Nginx 版本号

###### 说明

  > 披露正在运行的 NGINX 版本可能是不可取的，特别是在对信息披露敏感的环境中。NGINX 默认在错误页面和 HTTP 响应的头中显示版本号。

  > 此信息可以被知道特定版本存在特定漏洞的攻击者用作起点，并可能有助于更深入地了解正在使用的系统，并可能进一步开发针对特定 NGINX 版本的攻击。例如，Shodan 提供了广泛的此类信息数据库。尝试在所有随机服务器上使用漏洞比询问它们要高效得多。

  > 隐藏版本信息不会阻止攻击发生，但如果攻击者正在寻找特定版本的硬件或软件，它会让你不那么成为目标。我将 HTTP 服务器广播的数据视为个人信息。

  > 通过隐藏实现安全并不意味着你是安全的，但它确实有时会减慢人们的速度，而这正是零日漏洞所需要的。

  再看看关于这个问题的极好评论（来自 [specializt](https://serverfault.com/users/67666/specializt)）：

  > _完全忽视重要的安全因素，如 "无版本号"，甚至可能是 "无服务器供应商名称"，只是......初学者的错误。当然，通过隐藏实现安全本身并不会提高安全性，但它肯定至少能防范最普通、最简单的攻击向量 - 通过隐藏实现安全是必要的一步，可能是第一步，但绝不应是最后一项安全措施 - 完全跳过它是一个非常糟糕的错误，即使是最安全的 Web 服务器，如果知道特定版本的攻击向量，也可能被破解。_

###### 示例

```nginx
# This disables emitting NGINX version on error pages and in the "Server" response header field:
server_tokens off;
```

###### 外部资源

- [Remove Version from Server Header Banner in nginx](https://geekflare.com/remove-server-header-banner-nginx/)
- [Reduce or remove server headers](https://www.tunetheweb.com/security/http-security-headers/server-header/)
- [Fingerprint Web Server (OTG-INFO-002)](https://www.owasp.org/index.php/Fingerprint_Web_Server_(OTG-INFO-002))

<a id="beginner-hide-nginx-server-signature"></a>
#### :beginner: 隐藏 Nginx 服务器签名

###### 说明

  > `Server` 响应头字段包含有关用于处理请求的源服务器软件的信息。此字符串被 Alexa 和 Netcraft 等用于收集有关 Internet 上有多少种以及哪种类型的 Web 服务器在运行的统计信息。

  > 最简单的第一步之一是阻止 Web 服务器通过 `Server` 头显示其使用的软件和技术。当然，有几个原因会让你想要更改服务器头。可能是安全原因，可能是冗余系统、负载均衡器等。攻击者收集有关应用程序及其环境的所有可用信息。有关所用技术和软件版本的信息是极有价值的信息。

  > 在我看来，没有真正的理由或需要显示这么多关于你的服务器的信息。一旦你知道版本号，很容易查找特定的漏洞。然而，这不是你需要给出的信息，所以我通常赞成在可以用最少的工作完成的情况下删除它。

  > 你应该从源代码编译 NGINX，使用 `ngx_headers_more` 以使用 `more_set_headers` 指令，或使用 [nginx-remove-server-header.patch](https://gitlab.com/buik/nginx/blob/master/nginx-remove-server-header.patch)。

  也许这是一种非常严格的措施，但 [RFC 2616 - 个人信息](https://tools.ietf.org/html/rfc2616#section-15.1) 的指南对我来说总是非常有帮助：

  > _历史表明，这方面的错误常常造成严重的安全和/或隐私问题，并为实施者的公司带来高度不利的公众形象。[...] 像任何通用数据传输协议一样，HTTP 不能规范所传输数据的内容，也没有任何先验的方法来确定在给定请求的任何特定信息的敏感性。因此，应用程序应向该信息的提供者提供尽可能多的控制权。在此上下文中，有四个头字段值得特别提及：`Server`、`Via`、`Referer` 和 `From`。_

  官方 Apache 文档（是的，这不是玩笑，在我看来这是一个有趣的观点）说：

  > _将 ServerTokens 设置为小于最小值是不推荐的，因为这使得调试互操作性问题更加困难。还要注意，禁用 Server: 头对提高服务器安全性毫无作用。"通过隐藏实现安全" 的想法是一个神话，会导致虚假的安全感。_

###### 示例

推荐的配置：

```nginx
http {

  more_set_headers "Server: Unknown"; # or whatever else, e.g. 'WOULDN'T YOU LIKE TO KNOW! '

  ...
```

最推荐的配置：

```nginx
http {

  more_clear_headers 'Server';

  ...
```

你也可以使用 Lua 模块：

```nginx
http {

  header_filter_by_lua_block {
    ngx.header["Server"] = nil
  }

  ...
```

###### 外部资源

- [Shhh... don't let your response headers talk too loudly](https://www.troyhunt.com/shhh-dont-let-your-response-headers/)
- [How to change (hide) the Nginx Server Signature?](https://stackoverflow.com/questions/24594971/how-to-changehide-the-nginx-server-signature)
- [Configuring Your Web Server to Not Disclose Its Identity](https://www.acunetix.com/blog/articles/configure-web-server-disclose-identity/)

<a id="beginner-hide-upstream-proxy-headers"></a>
#### :beginner: 隐藏上游代理头

###### 说明

  > 保护服务器的安全性远不止不显示运行的内容，但我认为少即是多更好。

  > 当 NGINX 用作代理服务器将请求转发到上游服务器（例如 PHP-FPM 实例）时，隐藏上游响应中发送的某些头（例如运行的 PHP 版本）可能是有益的。

  > 你应该使用 `proxy_hide_header`（或 Lua 模块）来隐藏/移除从上游服务器返回到 NGINX 反向代理（进而返回到客户端）的头。

###### 示例

```nginx
# Hide some standard response headers:
proxy_hide_header X-Powered-By;
proxy_hide_header X-AspNetMvc-Version;
proxy_hide_header X-AspNet-Version;
proxy_hide_header X-Drupal-Cache;

# Hide some Amazon S3 specific response headers:
proxy_hide_header X-Amz-Id-2;
proxy_hide_header X-Amz-Request-Id;

# Hide other risky response headers:
proxy_hide_header X-Runtime;
```

###### 外部资源

- [Remove insecure http headers](https://veggiespam.com/headers/)
- [CRLF Injection and HTTP Response Splitting Vulnerability](https://www.netsparker.com/blog/web-security/crlf-http-header/)
- [HTTP Response Splitting](https://owasp.org/www-community/attacks/HTTP_Response_Splitting)
- [HTTP response header injection](https://portswigger.net/kb/issues/00200200_http-response-header-injection)
- [X-Runtime header related attacks](https://stackoverflow.com/questions/38584331/x-runtime-header-related-attacks)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-remove-support-for-legacy-and-risky-http-request-headers"></a>
#### :beginner: 移除对遗留和危险的 HTTP 请求头的支持

###### 说明

  > 在我看来，支持这些头本身不是漏洞，但更像是配置错误，在某些情况下可能导致漏洞。

  > 良好的做法是绝对移除（或剥离/规范化其值）对有风险的 HTTP 请求头的支持。如果不考虑其内容，它们绝不应到达你的应用程序或通过代理服务器。

  > 使用 `X-Original-URL` 或 `X-Rewrite-URL` 的能力可能会产生严重后果。这些头允许用户访问一个 URL，但让你的应用（例如使用 PHP/Symfony）返回不同的 URL，这可以绕过更高级别缓存和 Web 服务器的限制，例如，如果你在代理上为 `/admin` 等 location 设置了拒绝规则（`deny all; return 403;`）。

  > 如果你的一个或多个后端使用 `X-Host`、`X-Forwarded-Host`、`X-Forwarded-Server`、`X-Rewrite-Url` 或 `X-Original-Url` HTTP 请求头的内容来决定向哪个用户（或安全域）发送 HTTP 响应，你可能受到此类漏洞的影响。如果你将这些头传递到后端，攻击者可能导致将包含任意内容的响应存储到受害者的缓存中。

  看看以下来自 [PortSwigger Research - Practical Web Cache Poisoning](https://portswigger.net/research/practical-web-cache-poisoning) 的解释：

  > _这揭示了头 `X-Original-URL` 和 `X-Rewrite-URL`，它们覆盖请求的路径。我第一次注意到它们影响运行 Drupal 的目标，深入研究 Drupal 的代码后发现，对此头的支持来自流行的 PHP 框架 Symfony，而 Symfony 又从 Zend 获取了代码。最终结果是大量 PHP 应用在不知情的情况下支持这些头。在尝试将这些头用于缓存投毒之前，我应该指出它们也非常适合绕过 WAF 和安全规则 [...]_

###### 示例

```nginx
# Remove risky request headers (the safest method):
proxy_set_header X-Original-URL "";
proxy_set_header X-Rewrite-URL "";
proxy_set_header X-Forwarded-Server "";
proxy_set_header X-Forwarded-Host "";
proxy_set_header X-Host "";

# Or consider setting the vulnerable headers to a known-safe value:
proxy_set_header X-Original-URL $request_uri;
proxy_set_header X-Rewrite-URL $original_uri;
proxy_set_header X-Forwarded-Host $host;
```

###### 外部资源

- [CVE-2018-14773: Remove support for legacy and risky HTTP request headers](https://symfony.com/blog/cve-2018-14773-remove-support-for-legacy-and-risky-http-headers)
- [Local File Inclusion Vulnerability in Concrete5 version 5.7.3.1](https://hackerone.com/reports/59665)
- [PortSwigger Research - Practical Web Cache Poisoning](https://portswigger.net/research/practical-web-cache-poisoning)
- [Passing headers to the backend (from this handbook)](NGINX_BASICS.md#passing-headers-to-the-backend)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-use-only-the-latest-supported-openssl-version"></a>
#### :beginner: 只使用最新支持的 OpenSSL 版本

###### 说明

  > 在开始之前，请参阅 OpenSSL 网站上的[发布策略政策](https://www.openssl.org/policies/releasestrat.html)和[更新日志](https://www.openssl.org/news/changelog.html)。选择 OpenSSL 版本的标准可能不同，完全取决于你的使用情况。

  > OpenSSL 库的最新主要版本是（可能已更改）：
  >
  >   - OpenSSL 的下一个版本将是 3.0.0
  >   - 版本 1.1.1 将支持到 2023-09-11 (LTS)
  >     - 最后小版本：1.1.1d (2019-09-10)
  >   - 版本 1.1.0 将支持到 2019-09-11
  >     - 最后小版本：1.1.0k (2018-05-28)
  >   - 版本 1.0.2 将支持到 2019-12-31 (LTS)
  >     - 最后小版本：1.0.2s (2018-05-28)
  >   - 任何其他版本不再受支持

  > 在我看来，唯一安全的方式是基于最新的、仍然受支持且生产就绪的 OpenSSL 版本。而且，我建议坚持使用最新版本（例如目前是 1.1.1 或 1.1.1d）。所以，确保你的 OpenSSL 库更新到最新的可用版本，并鼓励你的客户端也使用更新的 OpenSSL 及其相关软件。

  > 在开始使用 OpenSSL 1.1.1 之前，你应该知道一件事：它的 API 与当前的 1.0.2 不同，所以这不是简单的开关切换。NGINX 从 1.13.0 版本开始支持 TLS 1.3，但当 OpenSSL 开发人员发布 OpenSSL 1.1.1 时，NGINX 已经支持这个全新的协议版本。

  > 如果你的软件仓库系统没有最新的 OpenSSL，你可以执行[编译](https://github.com/trimstray/nginx-admins-handbook#installing-from-source)过程（参见 OpenSSL 子部分）。

  > 我还建议跟踪 [OpenSSL 漏洞](https://www.openssl.org/news/vulnerabilities.html)官方通讯，如果你想了解 OpenSSL 中修复的安全错误和问题。

###### 外部资源

- [OpenSSL Official Website](https://www.openssl.org/)
- [OpenSSL Official Blog](https://www.openssl.org/blog/)
- [OpenSSL Official Newslog](https://www.openssl.org/news/newslog.html)

<a id="beginner-force-all-connections-over-tls"></a>
#### :beginner: 强制所有连接通过 TLS

###### 说明

  > TLS 提供两个主要服务。首先，它为用户验证用户正在连接的服务器身份。其次，它保护从用户到服务器的敏感信息传输。

  > 在我看来，你应该始终使用 HTTPS 而不是 HTTP（仅将 HTTP 用于重定向到 HTTPS）来保护你的网站，即使它不处理敏感通信并且没有混合内容。应用程序可能有许多敏感的地方需要保护。

  > 始终将登录页面、注册表单、所有后续身份验证页面、联系表单和付款详细信息表单放在 HTTPS 中，以防止嗅探和注入（攻击者可以向未加密的 HTTP 传输中注入代码，因此即使有人只阅读非关键内容，也会增加篡改内容的风险。参见[浏览器中间人攻击](https://owasp.org/www-community/attacks/Man-in-the-browser_attack)）。它们必须仅通过 TLS 访问，以确保你的流量安全。

  > 如果页面通过 TLS 可用，则必须完全由通过 TLS 传输的内容组成。使用不安全的 HTTP 协议请求子资源会削弱整个页面和 HTTPS 协议的安全性。现代浏览器应默认阻止或报告所有通过 HTTP 在 HTTPS 页面上提供的活动混合内容。

  > 还要记住实施 [HTTP 严格传输安全 (HSTS)](#beginner-enable-http-strict-transport-security) 并确保正确配置 TLS（协议版本、密码套件、正确的证书链等）。

  > 目前我们有第一个免费和开放的 CA - [Let's Encrypt](https://letsencrypt.org/) - 因此生成和实现证书从未如此简单。它旨在提供免费且易于使用的 TLS 和 SSL 证书。

###### 示例

- 强制所有流量使用 TLS：

  ```nginx
  server {

    listen 10.240.20.2:80;

    server_name example.com;

    return 301 https://$host$request_uri;

  }

  server {

    listen 10.240.20.2:443 ssl;

    server_name example.com;

    ...

  }
  ```

- 强制登录页面使用 TLS：

  ```nginx
  server {

    listen 10.240.20.2:80;

    server_name example.com;

    ...

    location ^~ /login {

      return 301 https://example.com$request_uri;

    }

  }
  ```

###### 外部资源

- [Does My Site Need HTTPS?](https://doesmysiteneedhttps.com/)
- [HTTP vs HTTPS Test](https://www.httpvshttps.com/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [Should we force user to HTTPS on website?](https://security.stackexchange.com/questions/23646/should-we-force-user-to-https-on-website)
- [Force a user to HTTPS](https://security.stackexchange.com/questions/137542/force-a-user-to-https)
- [The Security Impact of HTTPS Interception](https://jhalderm.com/pub/papers/interception-ndss17.pdf) <sup>[pdf]</sup>
- [HTTPS with self-signed certificate vs HTTP (from this handbook)](#https-with-self-signed-certificate-vs-http)
- [Enable HTTP Strict Transport Security - Hardening - P1 (from this handbook)](#beginner-enable-http-strict-transport-security)

<a id="beginner-use-min-2048-bit-for-rsa-and-256-bit-for-ecc"></a>
#### :beginner: 使用最少 2048 位 `RSA` 和 256 位 `ECC`

###### 说明

  > SSL 证书最常使用 `RSA` 密钥，这些密钥的推荐大小不断增加以保持足够的密码强度。`RSA` 的替代方案是 `ECC`。`ECC`（和 `ECDSA`）可能对大多数用途更好，但不是对所有用途都好。两种密钥类型共享相同的重要属性，即它们是非对称算法（一个密钥用于加密，一个密钥用于解密）。NGINX 支持双证书，因此你可以获得更轻巧、更强大的 `ECC` 证书，但仍然让访问者使用标准证书浏览你的网站。

  > 事实是（如果谈论 `RSA`），业界/社区在这个问题上存在分歧。我自己属于 "_使用 2048，因为 4096 几乎没什么用，而成本却相当高_" 的阵营。

  > 目前建议使用 2048 位 `RSA`（或 256 位 `ECC`）密钥。安全专家预计 2048 位将足够商业用途直到大约 2030 年（根据 [NIST](https://www.keylength.com/en/4/)）。美国国家安全局（NSA）要求所有绝密文件和文档使用 384 位 `ECC` 密钥（7680 位 `RSA` 密钥）加密。此外，出于安全原因，最新的 [CA/Browser forum - Baseline Requirements](https://cabforum.org/wp-content/uploads/CA-Browser-Forum-BR-1.6.7.pdf) <sup>[pdf]</sup> 论坛和 IST 建议订阅者证书/密钥使用 2048 位 RSA 密钥。接下来，当前的建议（[NIST SP 800-57-2](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-57pt2r1.pdf) <sup>[pdf]</sup>）现在是 2048 或 3072 位，取决于互操作性要求。另一方面，最新版本的 [FIPS-186-5 (草案)](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-5-draft.pdf) <sup>[pdf]</sup> 指定使用位长度为偶数且大于或等于 2048 位的模数（旧的 FIPS-186-4 说美国政府生成（和使用）1024、2048 或 3072 位密钥长度的数字签名）。

  > 接下来，OpenSSL 默认使用 [2048 位密钥](https://github.com/openssl/openssl/commit/44e0c2bae4bfd87d770480902618dbccde84fd81)。欧洲支付委员会（[EPC342-08 v8.0](https://www.europeanpaymentscouncil.eu/sites/default/files/kb/file/2019-01/EPC342-08%20v8.0%20Guidelines%20on%20cryptographic%20algorithms%20usage%20and%20key%20management.pdf) <sup>[pdf]</sup>）的建议说，你应该避免在新的应用中使用 1024 位 RSA 密钥和 160 位 ECC 密钥，除非用于短期低价值保护（例如单设备的临时身份验证）。EPC 还建议至少使用 2048 位 RSA 或 224 位 ECC 用于中期（例如 10 年）保护。他们将 `SHA-1`、1024 位的 `RSA` 模数、160 位的 `ECC` 密钥分类为适合遗留使用（但我认为 `SHA-1` 不再适合作为遗留使用）。

  > 一般来说，只要你使用合理的过期间隔（例如 2048 位密钥和证书不超过 6-12 个月），选择 4096 位密钥而不是 2048 位没有令人信服的理由。这给攻击者更少的时间破解密钥，并减少如果密钥被泄露时利用任何漏洞的可能性，但这对于证书的安全性本身目前来说并不是必要的。RSA 的安全级别基于已知的最强 RSA 攻击与破解对称加密算法所需的处理量相比。对我来说，我们更应该关注私钥在服务器被入侵时被盗，以及技术进步使我们的密钥易受攻击的情况。

  > 256 位 `ECC` 密钥可能比 2048 位经典密钥更强。如果你使用 `ECDSA`，建议的密钥大小会根据使用情况而变化，参见 [NIST 800-57-3 - 特定应用密钥管理指南（第 12 页，表 2-1）](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-57pt3r1.pdf) <sup>[pdf]</sup>。虽然更长的密钥确实提供更好的安全性，但将 `RSA` 密钥长度从 2048 加倍到 4096，安全位数的增加只有 18，仅增加 16%（签名消息的时间增加了 7 倍，验证签名的时间在某些情况下增加了 3 倍以上）。此外，除了需要更多存储空间外，更长的密钥也会增加 CPU 使用率。

  > `ECC` 在密钥长度方面比 `RSA` 更好。但主要问题在于实现。我认为 `RSA` 比 `ECC` 更容易实现。`ECDSA` 密钥（包含 `ECC` 公钥）比 `RSA` 更受推荐，因为它以更小的密钥提供相同的安全级别。`ECC` 密钥比 `RSA & DSA` 密钥更好，因为 `ECC` 算法更难破解（更不易受攻击）。在我看来，`ECC` 适用于资源受限的环境（有限的存储或数据处理能力），例如手机、PDA 和嵌入式系统。当然，`RSA` 密钥非常快，提供非常简单的加密和验证，并且比 `ECC` 更容易实现。

  > 更长的 `RSA` 密钥生成需要更多时间，在加密和解密时需要使用更多的 CPU 和电力，每个连接开始的 SSL 握手也会更慢。它对客户端（例如浏览器）也有微小的影响。使用 `curve25519` 时，`ECC` 被认为更安全。它速度快，并且设计上免疫各种侧信道攻击。`RSA` 在实际应用中也同样安全，并且也被认为现代技术无法攻破。

  > 如今使用 4096 位密钥的真正优势是面向未来。如果你想在 SSL Lab 中获得 **A+ 和 100% 评分**（针对密钥交换），你绝对应该使用 4096 位私钥。这是你应该使用它们的主要原因（对我来说也是唯一的原因）。

  > 使用 OpenSSL 的 `speed` 命令来基准测试两种类型并比较结果，例如 `openssl speed rsa2048 rsa4096`、`openssl speed rsa` 或 `openssl speed ecdsa`。但请记住，在 OpenSSL 速度测试中，你看到的是块密码速度的差异，而在实际生活中，大部分 CPU 时间花在 SSL 握手中的非对称算法上。另一方面，现代处理器能够在单个核心上每秒执行至少 1k 次 RSA 1024 位签名，所以这通常不是问题。

  "SSL/TLS Deployment Best Practices" 一书说：

  > _用于建立安全连接的密码学握手是一项成本受私钥大小影响很大的操作。使用太短的密钥不安全，但使用太长的密钥会导致 "过多" 的安全性和缓慢的操作。对于大多数网站，使用超过 2048 位的 RSA 密钥和超过 256 位的 ECDSA 密钥是 CPU 能力的浪费，并可能损害用户体验。同样，将临时密钥交换的强度提高到 DHE 的 2048 位以上和 ECDHE 的 256 位以上收益甚微。_

  Konstantin Ryabitsev（Reddit）：

  > _一般来说，如果我们发现 2048 位密钥不再足够，那不是因为当前计算机暴力破解能力的改进，而是因为 RSA 作为一种技术被革命性的计算进步所淘汰。如果真的发生这种情况，3072 或 4096 位也不会有太大区别。这就是为什么超过 2048 位的任何东西通常被视为一种感觉良好的对冲策略。_

  **我的建议：**

  > 目前使用 256 位 `ECDSA` 或 2048 位密钥而不是 4096 位 `RSA`。

###### 示例

```bash
### Example (RSA):
( _fd="example.com.key" ; _len="2048" ; openssl genrsa -out ${_fd} ${_len} )

# Let's Encrypt:
certbot certonly -d example.com -d www.example.com --rsa-key-size 2048

### Example (ECC):
# _curve: prime256v1, secp521r1, secp384r1
( _fd="example.com.key" ; _fd_csr="example.com.csr" ; _curve="prime256v1" ; \
openssl ecparam -out ${_fd} -name ${_curve} -genkey ; \
openssl req -new -key ${_fd} -out ${_fd_csr} -sha256 )

# Let's Encrypt (from above):
certbot --csr ${_fd_csr} -[other-args]
```

对于 `x25519`：

```bash
( _fd="private.key" ; _curve="x25519" ; \
openssl genpkey -algorithm ${_curve} -out ${_fd} )
```

&nbsp;&nbsp;:arrow_right: ssllabs score: <b>100%</b>

```bash
( _fd="example.com.key" ; _len="2048" ; openssl genrsa -out ${_fd} ${_len} )

# Let's Encrypt:
certbot certonly -d example.com -d www.example.com
```

&nbsp;&nbsp;:arrow_right: ssllabs score: <b>90%</b>

###### 外部资源

- [Key Management Guidelines by NIST](https://csrc.nist.gov/Projects/Key-Management/Key-Management-Guidelines) <sup>[NIST]</sup>
- [Recommendation for Transitioning the Use of Cryptographic Algorithms and Key Lengths](https://csrc.nist.gov/publications/detail/sp/800-131a/archive/2011-01-13) <sup>[NIST]</sup>
- [NIST SP 800-52 Rev. 2](https://csrc.nist.gov/publications/detail/sp/800-52/rev-2/final) <sup>[NIST]</sup>
- [NIST SP 800-57 Part 1 Rev. 3](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-3/archive/2012-07-10) <sup>[NIST]</sup>
- [FIPS PUB 186-4 - Digital Signature Standard (DSS)](http://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf) <sup>[NIST, pdf]</sup>
- [Cryptographic Key Length Recommendations](https://www.keylength.com/)
- [Key Lengths - Contribution to The Handbook of Information Security](https://infoscience.epfl.ch/record/164539/files/NPDF-32.pdf) <sup>[pdf]</sup>
- [NIST - Key Management](https://csrc.nist.gov/Projects/Key-Management/publications) <sup>[NIST]</sup>
- [CA/Browser Forum Baseline Requirements](https://cabforum.org/baseline-requirements-documents/)
- [Mozilla Guidelines - Key Management](https://infosec.mozilla.org/guidelines/key_management.html)
- [So you're making an RSA key for an HTTPS certificate. What key size do you use?](https://certsimple.com/blog/measuring-ssl-rsa-keys)
- [RSA Key Sizes: 2048 or 4096 bits?](https://danielpocock.com/rsa-key-sizes-2048-or-4096-bits/)
- [Create a self-signed ECC certificate](https://msol.io/blog/tech/create-a-self-signed-ecc-certificate/)
- [ECDSA: Elliptic Curve Signatures](https://cryptobook.nakov.com/digital-signatures/ecdsa-sign-verify-messages)
- [Elliptic Curve Cryptography Explained](https://fangpenlin.com/posts/2019/10/07/elliptic-curve-cryptography-explained/)
- [You should be using ECC for your SSL/TLS certificates](https://www.thesslstore.com/blog/you-should-be-using-ecc-for-your-ssl-tls-certificates/)
- [Comparing ECC vs RSA](https://www.linkedin.com/pulse/comparing-ecc-vs-rsa-ott-sarv)
- [Comparison And Evaluation Of Digital Signature Schemes Employed In Ndn Network](https://arxiv.org/pdf/1508.00184.pdf) <sup>[pdf]</sup>
- [HTTPS Performance, 2048-bit vs 4096-bit](https://blog.nytsoi.net/2015/11/02/nginx-https-performance)
- [RSA and ECDSA hybrid Nginx setup with LetsEncrypt certificates](https://hackernoon.com/rsa-and-ecdsa-hybrid-nginx-setup-with-letsencrypt-certificates-ee422695d7d3)
- [Why ninety-day lifetimes for certificates?](https://letsencrypt.org/2015/11/09/why-90-days.html)
- [SSL Certificate Validity Will Be Limited to One Year by Apple's Safari Browser](https://www.thesslstore.com/blog/ssl-certificate-validity-will-be-limited-to-one-year-by-apples-safari-browser/)
- [Certificate lifetime capped to 1 year from Sep 2020](https://scotthelme.co.uk/certificate-lifetime-capped-to-1-year-from-sep-2020/)
- [Why some cryptographic keys are much smaller than others](https://blog.cloudflare.com/why-are-some-keys-small/)
- [Bit security level](https://xtendo.org/bit_security_level)
- [RSA key lengths](https://www.javamex.com/tutorials/cryptography/rsa_key_length.shtml)

<a id="beginner-keep-only-tls-13-and-tls-12"></a>
#### :beginner: 只保留 TLS 1.3 和 TLS 1.2

###### 说明

  > 建议启用 TLS 1.2/1.3 并完全禁用 SSLv2、SSLv3、TLS 1.0 和 TLS 1.1，因为它们存在协议弱点并使用较旧的密码套件（不提供任何现代密码模式），我们真的不应该再使用它们了。TLS 1.2 是目前使用最广泛的 TLS 版本，在安全性方面相比 TLS 1.1 进行了多项改进。绝大多数站点确实支持 TLSv1.2，但仍然有一些不支持（而且，仍然并非所有客户端都兼容每个 TLS 版本）。TLS 1.3 是最新且更健壮的 TLS 协议版本，应尽可能使用（并且在不需要向后兼容的地方）。放弃 TLS 1.0 和 1.1 的最大好处是，现代 AEAD 密码仅由 TLS 1.2 及以上版本支持。

  > TLS 1.0 和 TLS 1.1 不应再使用（参见[弃用 TLSv1.0 和 TLSv1.1](https://tools.ietf.org/id/draft-moriarty-tls-oldversions-diediedie-00.html) <sup>[IETF]</sup>），它们已被 TLS 1.2 取代，而 TLS 1.2 现在又被 TLS 1.3 取代（必须在 2024 年 1 月 1 日前包含）。根据政府机构（例如 [NIST Special Publication (SP) 800-52 Revision 2](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-52r2.pdf) <sup>[pdf]</sup>）和行业联盟如支付卡行业协会（[PCI-TLS - 迁移从 SSL 和早期 TLS (信息补充)](https://www.pcisecuritystandards.org/documents/Migrating-from-SSL-Early-TLS-Info-Supp-v1_1.pdf) <sup>[pdf]</sup>）的指导，它们也在被积极弃用。例如，2020 年 3 月，[Firefox 将禁用对 TLS 1.0 和 TLS 1.1 的支持](https://blog.mozilla.org/security/2018/10/15/removing-old-versions-of-tls/)。

  > 坚持使用 TLS 1.0 是一个非常糟糕的主意，而且相当不安全。可能被 [POODLE](https://en.wikipedia.org/wiki/POODLE)、[BEAST](https://en.wikipedia.org/wiki/Transport_Layer_Security#BEAST_attack) 或其他[填充预言攻击](https://en.wikipedia.org/wiki/Padding_oracle_attack)攻击。还有许多其他 CVE（参见 [TLS Security 6: TLS 漏洞和攻击示例](https://www.acunetix.com/blog/articles/tls-vulnerabilities-attacks-final-part/)）弱点仍然适用，除非关闭 TLS 1.0 否则无法修复。坚持使用 TLS 1.1 只是一个糟糕的妥协，尽管它部分摆脱了 TLS 1.0 的问题。另一方面，有时仍然需要在实际中使用它们（以支持较旧的客户端）。坚持使用 TLS 1.0 或 1.1 有许多其他安全风险，因此我强烈建议所有人更新其客户端、服务和设备以支持至少 TLS 1.2。

  > 移除向后兼容的 SSL/TLS 版本通常是防止降级攻击的唯一方法。Google 提出了一个名为 `TLS_FALLBACK_SCSV` 的 SSL/TLS 扩展（应由你的 OpenSSL 库支持），旨在防止强制 SSL/TLS 降级（该扩展已于 2015 年 4 月被采纳为 [RFC 7507](https://tools.ietf.org/html/rfc7507)）。仅升级是不够的。你必须禁用 SSLv2 和 SSLv3 - 因此，如果你的服务器不允许 SSLv3（或 v2）连接，则不需要它（因为这些降级连接将无法工作）。从技术上讲，即使禁用了 SSL，`TLS_FALLBACK_SCSV` 仍然有用，因为它有助于避免连接降级到低于 TLS 1.2。要测试此扩展，请阅读[这篇](https://dwradcliffe.com/2014/10/16/testing-tls-fallback.html)优秀教程。

  > TLS 1.2 和 TLS 1.3 都没有安全问题（TLSv1.2 只有在满足某些条件时才安全，例如禁用 `CBC` 密码）。只有这些版本提供现代密码算法，并添加 TLS 扩展和密码套件。TLS 1.2 改进了密码套件，减少了对已被 BEAST 和前述 POODLE 等攻击利用的块密码的依赖。TLS 1.3 是一个新的 TLS 版本，将在未来几年推动更快、更安全的网络。而且，TLS 1.3 去掉了大量内容（已移除）：重新协商、压缩和许多遗留算法：`DSA`、`RC4`、`SHA1`、`MD5` 和 `CBC` 密码。此外，如前所述，TLS 1.0 和 TLS 1.1 协议将于 2020 年初从浏览器中移除。

  > TLS 1.2 确实需要仔细配置，以确保不会与它一起使用已识别漏洞的过时密码套件。TLS 1.3 消除了做出这些决策的需要，不需要任何特定配置，因为所有密码都是安全的，默认情况下 OpenSSL 仅为 TLSv1.3 启用 `GCM` 和 `Chacha20/Poly1305`，而不启用 `CCM`。TLS 1.3 版本还改进了 TLS 1.2 的安全、隐私和性能问题。

  > 在启用特定协议版本之前，你应该检查该协议支持哪些密码。因此，如果你启用 TLS 1.2，请记住使用[正确（且强）](#beginner-use-only-strong-ciphers)的密码来处理它们。否则，没有支持的密码它们根本无法工作（TLS 握手将失败）。

  > 我认为部署安全配置的最佳方式是：启用 TLS 1.2（作为最低版本；足够安全）不带任何 `CBC` 密码（`ChaCha20+Poly1305` 或 `AES/GCM` 应优先于 `CBC`（参考 BEAST），然而，对我来说使用 `CBC` 密码本身不是漏洞，Zombie POODLE 等是漏洞）和/或 TLS 1.3，由于其处理改进和排除了自 TLS 1.2 出现以来所有过时的东西，TLS 1.3 更安全。因此，将 TLS 1.2 设为你的 "最低协议级别" 是可靠的选择和行业最佳实践（所有行业标准如 PCI-DSS、HIPAA、NIST 都强烈建议使用 TLS 1.2 而不是 TLS 1.1/1.0）。

  > TLS 1.2 可能不足以支持遗留客户端。NIST 指南并不适用于所有用例，你应该在决定支持或放弃哪些协议之前始终分析你的用户群（例如，通过添加用于 TLS 版本和密码的变量到日志格式）。重要的是要记住，并非所有客户端都支持 TLS 提供的最新和最好的功能。

  > 如果你告诉 NGINX 使用 TLS 1.3，它将在可用时仅使用 TLS 1.3。NGINX 从 1.13.0 版本（2017 年 4 月发布）开始支持 TLS 1.3，当针对 OpenSSL 1.1.1 或更高版本构建时。

  > 对于 TLS 1.3，考虑使用 [`ssl_early_data`](#beginner-prevent-replay-attacks-on-zero-round-trip-time) 以允许 TLS 1.3 0-RTT 握手。

  **我的建议：**

  > 只使用 [TLSv1.3 和 TLSv1.2](#keep-only-tls1.2-tls13)。

###### 示例

TLS 1.3 + 1.2：

```nginx
ssl_protocols TLSv1.3 TLSv1.2;
```

TLS 1.2：

```nginx
ssl_protocols TLSv1.2;
```

&nbsp;&nbsp;:arrow_right: ssllabs score: <b>100%</b>

TLS 1.3 + 1.2 + 1.1：

```nginx
ssl_protocols TLSv1.3 TLSv1.2 TLSv1.1;
```

TLS 1.2 + 1.1：

```nginx
ssl_protocols TLSv1.2 TLSv1.1;
```

&nbsp;&nbsp;:arrow_right: ssllabs score: <b>95%</b>

###### 外部资源

- [The Transport Layer Security (TLS) Protocol Version 1.2](https://www.ietf.org/rfc/rfc5246.txt) <sup>[IETF]</sup>
- [The Transport Layer Security (TLS) Protocol Version 1.3](https://tools.ietf.org/html/draft-ietf-tls-tls13-18) <sup>[IETF]</sup>
- [Transport Layer Security Protocol: Documentation & Implementations](https://tlswg.org/)
- [TLS1.2 - Every byte explained and reproduced](https://tls12.ulfheim.net/)
- [TLS1.3 - Every byte explained and reproduced](https://tls13.ulfheim.net/)
- [TLS1.3 - OpenSSLWiki](https://wiki.openssl.org/index.php/TLS1.3)
- [TLS v1.2 handshake overview](https://medium.com/@ethicalevil/tls-handshake-protocol-overview-a39e8eee2cf5)
- [An Overview of TLS 1.3 - Faster and More Secure](https://kinsta.com/blog/tls-1-3/)
- [A Detailed Look at RFC 8446 (a.k.a. TLS 1.3)](https://blog.cloudflare.com/rfc-8446-aka-tls-1-3/)
- [Differences between TLS 1.2 and TLS 1.3](https://www.wolfssl.com/differences-between-tls-1-2-and-tls-1-3/)
- [TLS 1.3 in a nutshell](https://assured.se/2018/08/29/tls-1-3-in-a-nut-shell/)
- [TLS 1.3 is here to stay](https://www.ssl.com/article/tls-1-3-is-here-to-stay/)
- [TLS 1.3: Everything you need to know](https://securityboulevard.com/2019/07/tls-1-3-everything-you-need-to-know/)
- [TLS 1.3: better for individuals - harder for enterprises](https://www.ncsc.gov.uk/blog-post/tls-13-better-individuals-harder-enterprises)
- [How to enable TLS 1.3 on Nginx](https://ma.ttias.be/enable-tls-1-3-nginx/)
- [How to deploy modern TLS in 2019?](https://blog.probely.com/how-to-deploy-modern-tls-in-2018-1b9a9cafc454)
- [Deploying TLS 1.3: the great, the good and the bad](https://media.ccc.de/v/33c3-8348-deploying_tls_1_3_the_great_the_good_and_the_bad)
- [Why TLS 1.3 isn't in browsers yet](https://blog.cloudflare.com/why-tls-1-3-isnt-in-browsers-yet/)
- [Downgrade Attack on TLS 1.3 and Vulnerabilities in Major TLS Libraries](https://www.nccgroup.trust/us/about-us/newsroom-and-events/blog/2019/february/downgrade-attack-on-tls-1.3-and-vulnerabilities-in-major-tls-libraries/)
- [How does TLS 1.3 protect against downgrade attacks?](https://blog.gypsyengineer.com/en/security/how-does-tls-1-3-protect-against-downgrade-attacks.html)
- [Phase two of our TLS 1.0 and 1.1 deprecation plan](https://www.fastly.com/blog/phase-two-our-tls-10-and-11-deprecation-plan)
- [Deprecating TLSv1.0 and TLSv1.1 (IETF)](https://tools.ietf.org/id/draft-moriarty-tls-oldversions-diediedie-00.html) <sup>[IETF]</sup>
- [Deprecating TLS 1.0 and 1.1 - Enhancing Security for Everyone](https://www.keycdn.com/blog/deprecating-tls-1-0-and-1-1)
- [End of Life for TLS 1.0/1.1](https://support.umbrella.com/hc/en-us/articles/360033350851-End-of-Life-for-TLS-1-0-1-1-)
- [Legacy TLS is on the way out: Start deprecating TLSv1.0 and TLSv1.1 now](https://scotthelme.co.uk/legacy-tls-is-on-the-way-out/)
- [TLS/SSL Explained – Examples of a TLS Vulnerability and Attack, Final Part](https://www.acunetix.com/blog/articles/tls-vulnerabilities-attacks-final-part/)
- [A Challenging but Feasible Blockwise-Adaptive Chosen-Plaintext Attack on SSL](https://eprint.iacr.org/2006/136)
- [TLS/SSL hardening and compatibility Report 2011](http://www.g-sec.lu/sslharden/SSL_comp_report2011.pdf) <sup>[pdf]</sup>
- [This POODLE bites: exploiting the SSL 3.0 fallback](https://security.googleblog.com/2014/10/this-poodle-bites-exploiting-ssl-30.html)
- [New Tricks For Defeating SSL In Practice](https://www.blackhat.com/presentations/bh-dc-09/Marlinspike/BlackHat-DC-09-Marlinspike-Defeating-SSL.pdf) <sup>[pdf]</sup>
- [Are You Ready for 30 June 2018? Saying Goodbye to SSL/early TLS](https://blog.pcisecuritystandards.org/are-you-ready-for-30-june-2018-sayin-goodbye-to-ssl-early-tls)
- [What Happens After 30 June 2018? New Guidance on Use of SSL/Early TLS](https://blog.pcisecuritystandards.org/what-happens-after-30-june-2018-new-guidance-on-use-of-ssl/early-tls-)
- [Mozilla Security Blog - Removing Old Versions of TLS](https://blog.mozilla.org/security/2018/10/15/removing-old-versions-of-tls/)
- [Google - Modernizing Transport Security](https://security.googleblog.com/2018/10/modernizing-transport-security.html)
- [These truly are the end times for TLS 1.0, 1.1](https://www.theregister.co.uk/2020/02/10/tls_10_11_firefox_complete_eradication/)
- [Who's quit TLS 1.0?](https://who-quit-tls10.com/)
- [Recommended Cloudflare SSL configurations for PCI compliance](https://support.cloudflare.com/hc/en-us/articles/205043158-PCI-compliance-and-Cloudflare-SSL#h_8d214b26-c3e5-4632-8056-d2ccd08790dd)
- [Cloudflare SSL cipher, browser, and protocol support](https://support.cloudflare.com/hc/en-us/articles/203041594-Cloudflare-SSL-cipher-browser-and-protocol-support)
- [SSL and TLS Deployment Best Practices](https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices)
- [What level of SSL or TLS is required for HIPAA compliance?](https://luxsci.com/blog/level-ssl-tls-required-hipaa.html)
- [AEAD Ciphers - shadowsocks](https://shadowsocks.org/en/spec/AEAD-Ciphers.html)
- [Building a faster and more secure web with TCP Fast Open, TLS False Start, and TLS 1.3](https://blogs.windows.com/msedgedev/2016/06/15/building-a-faster-and-more-secure-web-with-tcp-fast-open-tls-false-start-and-tls-1-3/)
- [SSL Labs Grade Change for TLS 1.0 and TLS 1.1 Protocols](https://blog.qualys.com/ssllabs/2018/11/19/grade-change-for-tls-1-0-and-tls-1-1-protocols)
- [ImperialViolet - TLS 1.3 and Proxies](https://www.imperialviolet.org/2018/03/10/tls13.html)
- [How Netflix brings safer and faster streaming experiences to the living room on crowded networks using TLS 1.3](https://netflixtechblog.com/how-netflix-brings-safer-and-faster-streaming-experience-to-the-living-room-on-crowded-networks-78b8de7f758c)
- [TLS versions (from this handbook)](#tls-versions)
- [Defend against the BEAST attack - Hardening - P1 (from this handbook)](#beginner-defend-against-the-beast-attack)

<a id="beginner-use-only-strong-ciphers"></a>
#### :beginner: 只使用强密码套件

###### 说明

  > 此参数的变化比其他参数更频繁，今天推荐的配置明天可能就过时了。在我看来，拥有一个经过深思熟虑且最新的高安全性密码套件列表对于高安全性的 SSL/TLS 通信非常重要。如有疑问，应遵循 [Mozilla Security/Server Side TLS](https://wiki.mozilla.org/Security/Server_Side_TLS)（这是很好的资源；所有 Mozilla 网站和部署都应遵循本文档的建议）。

  > 检查服务器上 OpenSSL 支持的密码：`openssl ciphers -s -v`、`openssl ciphers -s -v ECDHE` 或 `openssl ciphers -s -v DHE`。

  > 如果不仔细选择密码套件（TLS 1.3 为你处理了这一点！），你可能会协商到一个弱的（安全性较低且无法应对最新漏洞的；参见[此](https://ciphersuite.info/page/faq/)）密码套件，可能被入侵。如果另一方不支持符合你标准的密码套件，并且你高度重视该连接的安全性，你不应该允许系统使用低质量的密码套件。

  > 为了更高的安全性，只使用强且不易受攻击的密码套件。将 `ECDHE+AESGCM`（根据 [Alexa Top 1 Million Security Analysis](https://crawler.ninja/)，超过 92.8% 使用加密的网站更喜欢使用基于 `ECDHE` 的密码）和 `DHE` 套件放在列表顶部（如果你关心性能，优先选择 `ECDHE-ECDSA` 和 `ECDHE-RSA` 而不是 `DHE`；Chrome 将优先选择基于 `ECDHE` 的密码而不是基于 `DHE` 的密码）。`DHE` 通常很慢，在 TLS 1.2 及以下版本中容易受到弱组（目前小于 2048 位）的影响。而且，没有指定对使用组的任何限制。这些问题不影响 `ECDHE`，这就是为什么它目前普遍更受欢迎。

  > 顺序很重要，因为 `ECDHE` 套件更快，只要客户端支持，你就想使用它们。临时 `DHE/ECDHE` 是推荐的，并支持完美前向保密（一种方法，不像其他解决方案那样在不支持高安全性密码套件时引入重放攻击的漏洞）。`ECDHE-ECDSA` 的性能与 `RSA` 大致相同，但安全得多。`ECDHE` 配合 `RSA` 更慢，但仍比单独的 `RSA` 安全得多。

  > 对于向后兼容的软件组件，考虑使用限制较少的密码。你不仅需要至少启用一个特殊的 `AES128` 密码以支持 HTTP/2（根据 [RFC 7540 - TLS 1.2 密码套件](https://tools.ietf.org/html/rfc7540#section-9.2.2) <sup>[IETF]</sup>），还必须允许 `prime256` 椭圆曲线，这会使密钥交换的评分再降低 10%，即使设置了安全的服务器优先顺序。

  > 服务器要么使用客户端最偏好的密码套件，要么使用自己的。大多数服务器使用自己的偏好。禁用 `DHE` 消除了前向保密，但导致握手时间大幅缩短。我认为，只要你只控制对话的一方，将系统限制为仅支持一个密码套件是荒谬的（这会使太多客户端和太多流量被切断）。另一方面，看看 [David Benjamin](https://davidben.net/)（来自 Chrome 网络团队）对此的看法：_服务器还应禁用 `DHE` 密码。即使首选 `ECDHE`，仅仅支持一个弱组也会使支持 `DHE` 的客户端容易受到攻击。_

  > 现代密码套件（例如来自 Mozilla 推荐的）也存在兼容性问题，主要是因为放弃了 `SHA-1`（参见 Google 在 2014 年对此的说法：[逐步弃用 SHA-1](https://security.googleblog.com/2014/09/gradually-sunsetting-sha-1.html)）。但如果你要使用带有 `HMAC-SHA-1` 的密码，要小心，因为它们自 2017 年以来已被证明容易受到碰撞攻击（参见[此](https://shattered.io/)）。虽然这不影响其作为 `MAC` 的使用，但应考虑更安全的替代方案，如 `SHA-256` 或 `SHA-3`。有一个很好的[解释](https://crypto.stackexchange.com/a/26518)说明了原因。

  > 如果你想在 SSL Lab 中获得 **A+ 和 100% 评分**（针对密码强度），你应该绝对禁用 `128-bit`（这是不应使用它们的主要原因）和 `CBC` 密码套件，后者有许多弱点。

  > 在我看来，`128-bit` 对称加密并不更不安全。而且，它们大约快 30%，仍然安全。例如 TLS 1.3 使用 `TLS_AES_128_GCM_SHA256 (0x1301)`（用于符合 TLS 的应用程序）。

  > 你应该禁用 `CHACHA20_POLY1305`（例如 `TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256` 和 `TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256`）以符合 HIPAA 和 [NIST SP 800-38D](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf) <sup>[pdf]</sup>（Mozilla 和 Cloudflare 使用它们，IETF 也推荐使用这些密码套件）指南以及 `CBC` 密码套件以符合 PCI DSS、HIPAA 和 NIST 指南。但对我来说，放弃 `CHACHA20_POLY1305` 很奇怪，我没有找到合理的解释为什么要这样做。`ChaCha20` 比 `AES` 更简单，如果没有 `AES` 硬件加速，目前可能是相当快的加密算法（实际上 `AES` 通常在硬件中实现，这给了它优势）。而且，速度和安全性可能是 Google 已经支持 Chrome 中 `ChaCha20 + Poly1305/AES` 的原因。

  > Mozilla 建议保留 TLSv1.3 的默认密码，不要在配置中显式启用它们（TLSv1.3 不需要任何特定的更改）。这是其中一个变化：我们需要知道的是，密码套件是固定的，除非应用程序显式定义 TLS 1.3 密码套件。因此，你所有的 TLSv1.3 连接将使用 `AES-256-GCM`、`ChaCha20`，然后是 `AES-128-GCM`，按此顺序。我还建议依赖 OpenSSL，因为对于 TLS 1.3，密码套件是固定的，设置它们不会产生影响（你将自动使用这三个密码）。

  > 默认情况下，OpenSSL 1.1.1* 配合 TLSv1.3 禁用 `TLS_AES_128_CCM_SHA256` 和 `TLS_AES_128_CCM_8_SHA256` 密码。在我看来，`ChaCha20+Poly1305` 或 `AES/GCM` 在大多数情况下都非常高效。在现代处理器上，常见的 `AES-GCM` 密码和模式由专用硬件加速，使得该算法的实现速度远超其他算法。但在缺乏该功能的较旧或较便宜的处理器上，`ChaCha20` 密码的运行速度比 `AES-GCM` 快，这正是 `ChaCha20` 设计者的意图。

  > 对于 TLS 1.2，你应该考虑禁用没有前向保密的弱密码，如使用 `CBC` 算法的密码。`CBC` 模式在 TLS 1.0、SSL 3.0 及更低版本中容易受到明文攻击。然而，真正的修复在 TLS 1.2 中实现，其中引入了 `GCM` 模式，该模式不易受到 BEAST 攻击。使用它们也会降低最终评分，因为它们不使用临时密钥。在我看来，你应该使用带有 `AEAD`（TLS 1.3 只支持这些套件）加密的密码，因为它们没有已知的弱点。

  > 存在如 Zombie POODLE、GOLDENDOODLE、0-Length OpenSSL 和 Sleeping POODLE 等漏洞，它们针对使用 `CBC`（密码块链）块密码模式的网站发布。这些漏洞仅适用于服务器使用 TLS 1.0、TLS 1.1 或带有 `CBC` 密码模式的 TLS 1.2。请参阅 Black Hat Asia 2019 上的演讲 [Zombie POODLE, GOLDENDOODLE, & How TLSv1.3 Can Save Us All](https://i.blackhat.com/asia-19/Fri-March-29/bh-asia-Young-Zombie-Poodle-Goldendoodle-and-How-TLSv13-Can-Save-Us-All.pdf) <sup>[pdf]</sup>。TLS 1.0 和 TLS 1.1 可能受到 [FREAK、POODLE、BEAST 和 CRIME](https://www.acunetix.com/blog/articles/tls-vulnerabilities-attacks-final-part/) 等漏洞的影响。

  > 有趣的是，Tripwire 的漏洞和暴露研究团队的安全研究员 Craig Young 发现 SSL 3.0 的后继者 TLS 1.2 中存在漏洞，由于 TLS 1.2 持续支持一种早已过时的密码方法：密码块链接（`CBC`），可能导致类似 POODLE 的攻击。这些缺陷允许中间人攻击用户在加密 Web 会话上的攻击。

  > 我建议禁用使用 `RSA` 加密的 TLS 密码模式（所有以 `TLS_RSA_WITH_*` 开头的密码），因为它们容易受到 [ROBOT](https://robotattack.org/) 攻击。相反，你应该添加对使用 `ECDHE` 或 `DHE`（以符合 [NIST SP 800-56B](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-56Br2.pdf) <sup>[pdf]</sup>）进行密钥传输的密码套件的支持。如果你的服务器配置为支持称为静态密钥密码的密码，你应该知道这些密码不支持 "前向保密"。在 HTTP/2 的新规范中，这些密码已被列入黑名单。并非所有支持 `RSA` 密钥交换的服务器都容易受到攻击，但建议禁用 `RSA` 密钥交换密码，因为它不支持前向保密。另一方面，`TLS_ECDHE_RSA` 密码可能没问题，因为在这种情况下 `RSA` 不参与密钥传输。TLS 1.3 不使用 `RSA` 密钥交换，因为它们没有前向保密。

  > 你无论如何都应该绝对禁用弱密码，无论你使用什么 TLS 版本，比如名称中带有 `DSS`、`DSA`、`DES/3DES`、`RC4`、`MD5`、`SHA1`、`null`、anon 的密码。

  > 我们有一个很好的在线工具用于测试与用户代理的兼容性密码套件：[CryptCheck](https://tls.imirhil.fr/suite)。我认为这将对你有很大帮助。

  > 如果怀疑，使用推荐的 Mozilla 套件之一（见下文），也请查看[支持的密码套件](https://tls.imirhil.fr/ciphers)和[用户代理兼容性](https://tls.imirhil.fr/suite)。

  看看 [Keith Shaw](https://github.com/keithws) 关于弱密码的精彩解释：

  > _弱并不意味着不安全。[...] 密码通常被标记为弱，因为存在一些基本的设计缺陷使其难以安全实现。_

  最后，一些有趣的统计数据来自 [Logjam: the latest TLS vulnerability explained](https://blog.cloudflare.com/logjam-the-latest-tls-vulnerability-explained/)：

  > _94% 的 CloudFlare 客户站点的 TLS 连接使用 `ECDHE`（更准确地说，其中 90% 是某种 `ECDHE-RSA-AES`，10% 是 `ECDHE-RSA-CHACHA20-POLY1305`），并提供前向保密。其余使用静态 `RSA`（5.5% 使用 `AES`，0.6% 使用 `3DES`）。_

  **我的建议：**

  > 只使用 [TLSv1.3 和 TLSv1.2](#keep-only-tls1.2-tls13) 配合以下密码套件（记住对于使用 TLSv1.2 的 `DHE`，需要至少 `2048-bit` DH 参数）：
  ```nginx
  ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256";
  ```

###### 示例

TLSv1.3 的密码套件：

```nginx
# - it's only example because for TLS 1.3 the cipher suites are fixed so setting them will not affect
# - if you have no explicit cipher suites configuration then you will automatically use those three and will be able to negotiate TLSv1.3
# - I recommend not setting ciphers for TLSv1.3 in NGINX
ssl_ciphers "TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-256-GCM-SHA384";
```

TLSv1.2 的密码套件：

```nginx
# Without DHE, only ECDHE:
ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-SHA384";
```

&nbsp;&nbsp;:arrow_right: ssllabs score: <b>100%</b>

TLSv1.3 的密码套件：

```nginx
# - it's only example because for TLS 1.3 the cipher suites are fixed so setting them will not affect
# - if you have no explicit cipher suites configuration then you will automatically use those three and will be able to negotiate TLSv1.3
# - I recommend not setting ciphers for TLSv1.3 in NGINX
ssl_ciphers "TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-256-GCM-SHA384:TLS13-AES-128-GCM-SHA256";
```

TLSv1.2 的密码套件：

```nginx
# 1)
# With DHE (remember about min. 2048-bit DH params for DHE!):
ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256";

# 2)
# Without DHE, only ECDHE (DH params are not required):
ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256";

# 3)
# With DHE (remember about min. 2048-bit DH params for DHE!):
ssl_ciphers "EECDH+CHACHA20:EDH+AESGCM:AES256+EECDH:AES256+EDH";
```

&nbsp;&nbsp;:arrow_right: ssllabs score: <b>90%</b>

这也可以作为与 [Mozilla SSL Configuration Generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/) 比较的基准：

- Modern profile, OpenSSL 1.1.1 (and variants) for TLSv1.3

```nginx
# However, Mozilla does not enable them in the configuration:
#   - for TLS 1.3 the cipher suites are fixed unless an application explicitly defines them
# ssl_ciphers "TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-256-GCM-SHA384:TLS13-AES-128-GCM-SHA256";
```

- Modern profile, OpenSSL 1.1.1 (and variants) for TLSv1.2 + TLSv1.3

```nginx
# However, Mozilla does not enable them in the configuration:
#   - for TLS 1.3 the cipher suites are fixed unless an application explicitly defines them
# ssl_ciphers "TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-256-GCM-SHA384:TLS13-AES-128-GCM-SHA256";
ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
```

- Intermediate profile, OpenSSL 1.1.0b + 1.1.1 (and variants) for TLSv1.2

```nginx
ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
```

还有推荐用于 HIPAA 和 TLS v1.2+ 的密码：

```nginx
ssl_ciphers "TLS13-AES-256-GCM-SHA384:TLS13-AES-128-GCM-SHA256:TLS13-AES-128-CCM-8-SHA256:TLS13-AES-128-CCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-CCM:DHE-RSA-AES128-CCM:DHE-RSA-AES256-CCM8:DHE-RSA-AES128-CCM8:DH-RSA-AES256-GCM-SHA384:DH-RSA-AES128-GCM-SHA256:ECDH-RSA-AES256-GCM-SHA384:ECDH-RSA-AES128-GCM-SHA256";
```

<details>
<summary><b>各密码套件的扫描结果（提供 TLSv1.2）</b></summary>

###### 我的推荐

- 密码套件：

```nginx
ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256";
```

- DH: **2048-bit**

- SSL Labs 评分：

  - 证书: **100%**
  - 协议支持: **100%**
  - 密钥交换: **90%**
  - 密码强度: **90%**

- SSLLabs 服务器优先顺序：

```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca8)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)   ECDH x25519 (eq. 3072 bits RSA)   FS 128
TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 (0x9f)   DH 2048 bits   FS  256
TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xccaa)   DH 2048 bits   FS  256
TLS_DHE_RSA_WITH_AES_128_GCM_SHA256 (0x9e)   DH 2048 bits   FS  128
```

- SSLLabs 'Handshake Simulation' 错误：

```
IE 11 / Win Phone 8.1  R  Server sent fatal alert: handshake_failure
Safari 6 / iOS 6.0.1  Server sent fatal alert: handshake_failure
Safari 7 / iOS 7.1  R Server sent fatal alert: handshake_failure
Safari 7 / OS X 10.9  R Server sent fatal alert: handshake_failure
Safari 8 / iOS 8.4  R Server sent fatal alert: handshake_failure
Safari 8 / OS X 10.10  R  Server sent fatal alert: handshake_failure
```

- testssl.sh：

```
' SSLv2
' SSLv3
' TLS 1
' TLS 1.1
' TLS 1.2
'  xc030   ECDHE-RSA-AES256-GCM-SHA384       ECDH 521   AESGCM      256      TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
'  x9f     DHE-RSA-AES256-GCM-SHA384         DH 2048    AESGCM      256      TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
'  xcca8   ECDHE-RSA-CHACHA20-POLY1305       ECDH 253   ChaCha20    256      TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
'  xccaa   DHE-RSA-CHACHA20-POLY1305         DH 2048    ChaCha20    256      TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256
'  xc02f   ECDHE-RSA-AES128-GCM-SHA256       ECDH 521   AESGCM      128      TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
'  x9e     DHE-RSA-AES128-GCM-SHA256         DH 2048    AESGCM      128      TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
```

###### SSLLabs 100%

- 密码套件：

```nginx
ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-SHA384";
```

- DH: **not used**

- SSL Labs 评分：

  - 证书: **100%**
  - 协议支持: **100%**
  - 密钥交换: **90%**
  - 密码强度: **100%**

- SSLLabs 服务器优先顺序：

```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca8)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
```

- SSLLabs 'Handshake Simulation' 错误：

```
Android 5.0.0 Server sent fatal alert: handshake_failure
Android 6.0 Server sent fatal alert: handshake_failure
Firefox 31.3.0 ESR / Win 7  Server sent fatal alert: handshake_failure
IE 11 / Win 7  R  Server sent fatal alert: handshake_failure
IE 11 / Win 8.1  R  Server sent fatal alert: handshake_failure
IE 11 / Win Phone 8.1  R  Server sent fatal alert: handshake_failure
IE 11 / Win Phone 8.1 Update  R Server sent fatal alert: handshake_failure
Safari 6 / iOS 6.0.1  Server sent fatal alert: handshake_failure
Safari 7 / iOS 7.1  R Server sent fatal alert: handshake_failure
Safari 7 / OS X 10.9  R Server sent fatal alert: handshake_failure
Safari 8 / iOS 8.4  R Server sent fatal alert: handshake_failure
Safari 8 / OS X 10.10  R  Server sent fatal alert: handshake_failure
```

- testssl.sh：

```
' SSLv2
' SSLv3
' TLS 1
' TLS 1.1
' TLS 1.2
'  xc030   ECDHE-RSA-AES256-GCM-SHA384       ECDH 521   AESGCM      256      TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
'  xcca8   ECDHE-RSA-CHACHA20-POLY1305       ECDH 253   ChaCha20    256      TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
```

###### SSLLabs 90% (1)

- 密码套件：

```nginx
ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA256";
```

- DH: **2048-bit**

- SSL Labs 评分：

  - 证书: **100%**
  - 协议支持: **100%**
  - 密钥交换: **90%**
  - 密码强度: **90%**

- SSLLabs 服务器优先顺序：

```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 (0x9f)   DH 2048 bits   FS  256
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca8)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xccaa)   DH 2048 bits   FS  256
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)   ECDH x25519 (eq. 3072 bits RSA)   FS 128
TLS_DHE_RSA_WITH_AES_128_GCM_SHA256 (0x9e)   DH 2048 bits   FS  128
```

- SSLLabs 'Handshake Simulation' 错误：

```
IE 11 / Win Phone 8.1  R  Server sent fatal alert: handshake_failure
Safari 6 / iOS 6.0.1  Server sent fatal alert: handshake_failure
Safari 7 / iOS 7.1  R Server sent fatal alert: handshake_failure
Safari 7 / OS X 10.9  R Server sent fatal alert: handshake_failure
Safari 8 / iOS 8.4  R Server sent fatal alert: handshake_failure
Safari 8 / OS X 10.10  R  Server sent fatal alert: handshake_failure
```

- testssl.sh：

```
' SSLv2
' SSLv3
' TLS 1
' TLS 1.1
' TLS 1.2
'  xc030   ECDHE-RSA-AES256-GCM-SHA384       ECDH 521   AESGCM      256      TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
'  x9f     DHE-RSA-AES256-GCM-SHA384         DH 2048    AESGCM      256      TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
'  xcca8   ECDHE-RSA-CHACHA20-POLY1305       ECDH 253   ChaCha20    256      TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
'  xccaa   DHE-RSA-CHACHA20-POLY1305         DH 2048    ChaCha20    256      TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256
'  xc02f   ECDHE-RSA-AES128-GCM-SHA256       ECDH 521   AESGCM      128      TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
'  x9e     DHE-RSA-AES128-GCM-SHA256         DH 2048    AESGCM      128      TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
```

###### SSLLabs 90% (2)

- 密码套件：

```nginx
ssl_ciphers "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256";
```

- DH: **not used**

- SSL Labs 评分：

  - 证书: **100%**
  - 协议支持: **100%**
  - 密钥交换: **90%**
  - 密码强度: **90%**

- SSLLabs 服务器优先顺序：

```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca8)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)   ECDH x25519 (eq. 3072 bits RSA)   FS 128
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 (0xc028)   ECDH x25519 (eq. 3072 bits RSA)   FS   WEAK  256
TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 (0xc027)   ECDH x25519 (eq. 3072 bits RSA)   FS   WEAK  128
```

- SSLLabs 'Handshake Simulation' 错误：

```
No errors
```

- testssl.sh：

```
' SSLv2
' SSLv3
' TLS 1
' TLS 1.1
' TLS 1.2
'  xc030   ECDHE-RSA-AES256-GCM-SHA384       ECDH 521   AESGCM      256      TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
'  xc028   ECDHE-RSA-AES256-SHA384           ECDH 521   AES         256      TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
'  xcca8   ECDHE-RSA-CHACHA20-POLY1305       ECDH 253   ChaCha20    256      TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
'  xc02f   ECDHE-RSA-AES128-GCM-SHA256       ECDH 521   AESGCM      128      TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
'  xc027   ECDHE-RSA-AES128-SHA256           ECDH 521   AES         128      TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
```

###### SSLLabs 90% (3)

- 密码套件：

```nginx
ssl_ciphers "EECDH+CHACHA20:EDH+AESGCM:AES256+EECDH:AES256+EDH";
```

- DH: **2048-bit**

- SSL Labs 评分：

  - 证书: **100%**
  - 协议支持: **100%**
  - 密钥交换: **90%**
  - 密码强度: **90%**

- SSLLabs 服务器优先顺序：

```
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca8)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 (0x9f)   DH 2048 bits   FS  256
TLS_DHE_RSA_WITH_AES_128_GCM_SHA256 (0x9e)   DH 2048 bits   FS  128
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 (0xc028)   ECDH x25519 (eq. 3072 bits RSA)   FS   WEAK  256
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (0xc014)   ECDH x25519 (eq. 3072 bits RSA)   FS   WEAK 256
TLS_DHE_RSA_WITH_AES_256_CCM_8 (0xc0a3)   DH 2048 bits   FS 256
TLS_DHE_RSA_WITH_AES_256_CCM (0xc09f)   DH 2048 bits   FS 256
TLS_DHE_RSA_WITH_AES_256_CBC_SHA256 (0x6b)   DH 2048 bits   FS   WEAK 256
TLS_DHE_RSA_WITH_AES_256_CBC_SHA (0x39)   DH 2048 bits   FS   WEAK  256
```

- SSLLabs 'Handshake Simulation' 错误：

```
No errors.
```

- testssl.sh：

```
' SSLv2
' SSLv3
' TLS 1
' TLS 1.1
' TLS 1.2
'  xc030   ECDHE-RSA-AES256-GCM-SHA384       ECDH 521   AESGCM      256      TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
'  xc028   ECDHE-RSA-AES256-SHA384           ECDH 521   AES         256      TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
'  xc014   ECDHE-RSA-AES256-SHA              ECDH 521   AES         256      TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
'  x9f     DHE-RSA-AES256-GCM-SHA384         DH 2048    AESGCM      256      TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
'  xcca8   ECDHE-RSA-CHACHA20-POLY1305       ECDH 253   ChaCha20    256      TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
'  xc0a3   DHE-RSA-AES256-CCM8               DH 2048    AESCCM8     256      TLS_DHE_RSA_WITH_AES_256_CCM_8
'  xc09f   DHE-RSA-AES256-CCM                DH 2048    AESCCM      256      TLS_DHE_RSA_WITH_AES_256_CCM
'  x6b     DHE-RSA-AES256-SHA256             DH 2048    AES         256      TLS_DHE_RSA_WITH_AES_256_CBC_SHA256
'  x39     DHE-RSA-AES256-SHA                DH 2048    AES         256      TLS_DHE_RSA_WITH_AES_256_CBC_SHA
'  x9e     DHE-RSA-AES128-GCM-SHA256         DH 2048    AESGCM      128      TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
```

###### Mozilla modern profile

- 密码套件：

```nginx
ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
```

- DH: **2048-bit**

- SSL Labs 评分：

  - 证书: **100%**
  - 协议支持: **100%**
  - 密钥交换: **90%**
  - 密码强度: **90%**

- SSLLabs 服务器优先顺序：

```
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)   ECDH x25519 (eq. 3072 bits RSA)   FS 128
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca8)   ECDH x25519 (eq. 3072 bits RSA)   FS 256
TLS_DHE_RSA_WITH_AES_128_GCM_SHA256 (0x9e)   DH 2048 bits   FS  128
TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 (0x9f)   DH 2048 bits   FS  256
```

- SSLLabs 'Handshake Simulation' 错误：

```
IE 11 / Win Phone 8.1  R  Server sent fatal alert: handshake_failure
Safari 6 / iOS 6.0.1  Server sent fatal alert: handshake_failure
Safari 7 / iOS 7.1  R Server sent fatal alert: handshake_failure
Safari 7 / OS X 10.9  R Server sent fatal alert: handshake_failure
Safari 8 / iOS 8.4  R Server sent fatal alert: handshake_failure
Safari 8 / OS X 10.10  R  Server sent fatal alert: handshake_failure
```

- testssl.sh：

```
' SSLv2
' SSLv3
' TLS 1
' TLS 1.1
' TLS 1.2
'  xc030   ECDHE-RSA-AES256-GCM-SHA384       ECDH 521   AESGCM      256      TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
'  x9f     DHE-RSA-AES256-GCM-SHA384         DH 2048    AESGCM      256      TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
'  xcca8   ECDHE-RSA-CHACHA20-POLY1305       ECDH 253   ChaCha20    256      TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
'  xc02f   ECDHE-RSA-AES128-GCM-SHA256       ECDH 521   AESGCM      128      TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
'  x9e     DHE-RSA-AES128-GCM-SHA256         DH 2048    AESGCM      128      TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
```

</details>

<details>
<summary><b>各密码套件的扫描结果（提供 TLSv1.3）</b></summary>

###### Mozilla modern profile（我的推荐）

- 密码套件：**not set**

- DH: **2048-bit**

- SSL Labs 评分：

  - 证书: **100%**
  - 协议支持: **100%**
  - 密钥交换: **90%**
  - 密码强度: **90%**

- SSLLabs 服务器优先顺序：

```
TLS_AES_256_GCM_SHA384 (0x1302)   ECDH x25519 (eq. 3072 bits RSA)   FS  256
TLS_CHACHA20_POLY1305_SHA256 (0x1303)   ECDH x25519 (eq. 3072 bits RSA)   FS  256
TLS_AES_128_GCM_SHA256 (0x1301)   ECDH x25519 (eq. 3072 bits RSA)   FS  128
```

- SSLLabs 'Handshake Simulation' 错误：

```
Chrome 69 / Win 7  R  Server sent fatal alert: protocol_version
Firefox 62 / Win 7  R Server sent fatal alert: protocol_version
OpenSSL 1.1.0k  R Server sent fatal alert: protocol_version
```

- testssl.sh：

```
' SSLv2
' SSLv3
' TLS 1
' TLS 1.1
' TLS 1.2
' TLS 1.3
'  x1302   TLS_AES_256_GCM_SHA384            ECDH 253   AESGCM      256      TLS_AES_256_GCM_SHA384
'  x1303   TLS_CHACHA20_POLY1305_SHA256      ECDH 253   ChaCha20    256      TLS_CHACHA20_POLY1305_SHA256
'  x1301   TLS_AES_128_GCM_SHA256            ECDH 253   AESGCM      128      TLS_AES_128_GCM_SHA256
```

</details>

###### 外部资源

- [RFC 7525 - TLS Recommendations](https://tools.ietf.org/html/rfc7525) <sup>[IETF]</sup>
- [TLS Cipher Suites](https://www.iana.org/assignments/tls-parameters/tls-parameters.xhtml#tls-parameters-4) <sup>[IANA]</sup>
- [SEC 1: Elliptic Curve Cryptography](http://www.secg.org/sec1-v2.pdf) <sup>[pdf]</sup>
- [TLS Cipher Suite Search](https://ciphersuite.info/)
- [Elliptic Curve Cryptography: a gentle introduction](https://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction/)
- [SSL/TLS: How to choose your cipher suite](https://technology.amis.nl/2017/07/04/ssltls-choose-cipher-suite/)
- [HTTP/2 and ECDSA Cipher Suites](https://sparanoid.com/note/http2-and-ecdsa-cipher-suites/)
- [TLS 1.3 (with AEAD) and TLS 1.2 cipher suites demystified: how to pick your ciphers wisely](https://www.cloudinsidr.com/content/tls-1-3-and-tls-1-2-cipher-suites-demystified-how-to-pick-your-ciphers-wisely/)
- [Which SSL/TLS Protocol Versions and Cipher Suites Should I Use?](https://www.securityevaluators.com/ssl-tls-protocol-versions-cipher-suites-use/)
- [Recommendations for a cipher string by OWASP](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/TLS_Cipher_String_Cheat_Sheet.md)
- [Recommendations for TLS/SSL Cipher Hardening by Acunetix](https://www.acunetix.com/blog/articles/tls-ssl-cipher-hardening/)
- [Mozilla's Modern compatibility suite](https://wiki.mozilla.org/Security/Server_Side_TLS#Modern_compatibility)
- [Cloudflare SSL cipher, browser, and protocol support](https://support.cloudflare.com/hc/en-us/articles/203041594-Cloudflare-SSL-cipher-browser-and-protocol-support)
- [TLS & Perfect Forward Secrecy](https://vincent.bernat.ch/en/blog/2011-ssl-perfect-forward-secrecy)
- [Why use Ephemeral Diffie-Hellman](https://tls.mbed.org/kb/cryptography/ephemeral-diffie-hellman)
- [Cipher Suite Breakdown](https://blogs.technet.microsoft.com/askpfeplat/2017/12/26/cipher-suite-breakdown/)
- [Zombie POODLE and GOLDENDOODLE Vulnerabilities](https://blog.qualys.com/technology/2019/04/22/zombie-poodle-and-goldendoodle-vulnerabilities)
- [SSL Labs Grading Update: Forward Secrecy, Authenticated Encryption and ROBOT](https://blog.qualys.com/ssllabs/2018/02/02/forward-secrecy-authenticated-encryption-and-robot-grading-update)
- [Logjam: the latest TLS vulnerability explained](https://blog.cloudflare.com/logjam-the-latest-tls-vulnerability-explained/)
- [The CBC Padding Oracle Problem](https://eklitzke.org/the-cbc-padding-oracle-problem)
- [Goodbye TLS_RSA](https://lightshipsec.com/goodbye-tls_rsa/)
- [ImperialViolet - TLS Symmetric Crypto](https://www.imperialviolet.org/2014/02/27/tlssymmetriccrypto.html)
- [IETF drops RSA key transport from TLS 1.3](https://www.theinquirer.net/inquirer/news/2343117/ietf-drops-rsa-key-transport-from-ssl)
- [Why TLS 1.3 is a Huge Improvement](https://securityboulevard.com/2018/12/why-tls-1-3-is-a-huge-improvement/)
- [Overview of TLS v1.3 - What's new, what's removed and what's changed?](https://owasp.org/www-chapter-london/assets/slides/OWASPLondon20180125_TLSv1.3_Andy_Brodie.pdf) <sup>[pdf]</sup>
- [OpenSSL IANA Mapping](https://testssl.sh/openssl-iana.mapping.html)
- [Testing for Weak SSL/TLS Ciphers, Insufficient Transport Layer Protection (OTG-CRYPST-001)](https://www.owasp.org/index.php/Testing_for_Weak_SSL/TLS_Ciphers,_Insufficient_Transport_Layer_Protection_(OTG-CRYPST-001))
- [Bypassing Web-Application Firewalls by abusing SSL/TLS](https://0x09al.github.io/waf/bypass/ssl/2018/07/02/web-application-firewall-bypass.html)
- [What level of SSL or TLS is required for HIPAA compliance?](https://luxsci.com/blog/level-ssl-tls-required-hipaa.html)
- [Cryptographic Right Answers](https://latacora.micro.blog/2018/04/03/cryptographic-right-answers.html)
- [ImperialViolet - ChaCha20 and Poly1305 for TLS](https://www.imperialviolet.org/2013/10/07/chacha20.html)
- [Do the ChaCha: better mobile performance with cryptography](https://blog.cloudflare.com/do-the-chacha-better-mobile-performance-with-cryptography/)
- [AES Is Great ... But We Need A Fall-back: Meet ChaCha and Poly1305](https://medium.com/asecuritysite-when-bob-met-alice/aes-is-great-but-we-need-a-fall-back-meet-chacha-and-poly1305-76ee0ee61895)
- [There's never magic, but plenty of butterfly effects](https://docs.microsoft.com/en-us/archive/blogs/ieinternals/theres-never-magic-but-plenty-of-butterfly-effects)
- [Cipher suites (from this handbook)](SSL_TLS_BASICS.md#cipher-suites)

<a id="beginner-use-more-secure-ecdh-curve"></a>
#### :beginner: 使用更安全的 ECDH 曲线

###### 说明

  > 也要注意这一点：_标准曲线的安全实现理论上是可能的，但非常困难。_

  > 在我看来，你的主要知识来源应该是 [SafeCurves 网站](https://safecurves.cr.yp.to/)。该网站报告了各种特定曲线的安全评估。

  > 对于 SSL 服务器证书，"椭圆曲线" 证书将仅用于数字签名（`ECDSA` 算法）。NGINX 提供了指定用于 `ECDHE` 密码的曲线的指令（`ssl_ecdh_curve`）。

  > `x25519` 是一个更安全（也符合 SafeCurves 要求）但稍微不太兼容的选择。我认为为了最大化与现有浏览器和服务器的互操作性，坚持使用 `P-256`（`prime256v1`）和 `P-384`（`secp384r1`）曲线。当然，关于它们有大量不同的观点。

  > NSA Suite B 说 NSA 使用曲线 `P-256` 和 `P-384`（在 OpenSSL 中，它们分别被指定为 `prime256v1` 和 `secp384r1`）。`P-521` 没有什么问题，只是它在实践中无用。可以说，`P-384` 也是无用的，因为更高效的 `P-256` 曲线已经提供了无法通过计算能力积累来破解的安全性。

  > Bernstein 和 Lange 认为 NIST 曲线不是最优的，存在同样快速但更好（更安全）的曲线，例如 `x25519`。

  > SafeCurves 说：
  >   - `NIST P-224`、`NIST P-256` 和 `NIST P-384` 是 **不安全的**

  > 在此描述的曲线中，只有 `x25519` 是符合所有 SafeCurves 要求的曲线。

  > 我认为你可以使用 `P-256` 来最小化麻烦。如果你觉得在有 384 位曲线可用的情况下使用 256 位曲线会威胁到你的男子气概，那么使用 `P-384`：它会增加你的计算和网络成本。

  > 如果你使用 TLS 1.3，你应该启用 `prime256v1` 签名算法。否则 SSL Lab 会报告 `TLS_AES_128_GCM_SHA256 (0x1301)` 签名为弱。

  > 如果你不设置 `ssl_ecdh_curve`，那么 NGINX 将使用其默认设置，例如 Chrome 将优先选择 `x25519`，但这**不推荐**，因为你无法控制 NGINX 的默认设置（似乎是 `P-256`）。

  > 显式设置 `ssl_ecdh_curve X25519:prime256v1:secp521r1:secp384r1;` **会降低 SSL Labs 的密钥交换评分**。

  > 绝对不要使用 `secp112r1`、`secp112r2`、`secp128r1`、`secp128r2`、`secp160k1`、`secp160r1`、`secp160r2`、`secp192k1` 曲线。根据 NIST 建议，它们的大小太小，不适合安全应用。

  **我的建议：**

  > 只使用 [TLSv1.3 和 TLSv1.2](#keep-only-tls1.2-tls13) 和[只使用强密码](#use-only-strong-ciphers)配合以下曲线：
  ```nginx
  ssl_ecdh_curve X25519:secp521r1:secp384r1:prime256v1;
  ```

###### 示例

TLS 1.2 的曲线：

```nginx
ssl_ecdh_curve secp521r1:secp384r1:prime256v1;
```

&nbsp;&nbsp;:arrow_right: ssllabs score: <b>100%</b>

```nginx
# Alternative (this one doesn't affect compatibility, by the way; it's just a question of the preferred order).

# This setup downgrade Key Exchange score but is recommended for TLS 1.2 + TLS 1.3:
ssl_ecdh_curve X25519:secp521r1:secp384r1:prime256v1;
```

###### 外部资源

- [Elliptic Curves for Security](https://tools.ietf.org/html/rfc7748) <sup>[IETF]</sup>
- [Standards for Efficient Cryptography Group](http://www.secg.org/)
- [SafeCurves: choosing safe curves for elliptic-curve cryptography](https://safecurves.cr.yp.to/)
- [A note on high-security general-purpose elliptic curves](https://eprint.iacr.org/2013/647)
- [P-521 is pretty nice prime](https://blog.cr.yp.to/20140323-ecdsa.html)
- [Safe ECC curves for HTTPS are coming sooner than you think](https://certsimple.com/blog/safe-curves-and-openssl)
- [Cryptographic Key Length Recommendations](https://www.keylength.com/)
- [Testing for Weak SSL/TLS Ciphers, Insufficient Transport Layer Protection (OTG-CRYPST-001)](https://www.owasp.org/index.php/Testing_for_Weak_SSL/TLS_Ciphers,_Insufficient_Transport_Layer_Protection_(OTG-CRYPST-001))
- [Elliptic Curve performance: NIST vs Brainpool](https://tls.mbed.org/kb/cryptography/elliptic-curve-performance-nist-vs-brainpool)
- [Which elliptic curve should I use?](https://security.stackexchange.com/questions/78621/which-elliptic-curve-should-i-use/91562)
- [Elliptic Curve Cryptography for those who are afraid of maths](http://www.lapsedordinary.net/files/ECC_BSidesLDN_2015.pdf) <sup>[pdf]</sup>
- [Security dangers of the NIST curves](http://cr.yp.to/talks/2013.05.31/slides-dan+tanja-20130531-4x3.pdf) <sup>[pdf]</sup>
- [How to design an elliptic-curve signature system](http://blog.cr.yp.to/20140323-ecdsa.html)
- [Win10 Crypto Vulnerability: Cheating in Elliptic Curve Billiards 2](https://medium.com/zengo/win10-crypto-vulnerability-cheating-in-elliptic-curve-billiards-2-69b45f2dcab6)

<a id="beginner-use-strong-key-exchange-with-perfect-forward-secrecy"></a>
#### :beginner: 使用具有完美前向保密的强密钥交换

###### 说明

  > 这些参数决定了 OpenSSL 库如何执行 Diffie-Hellman (DH) 密钥交换（DH 需要一些初始设置参数，使用 `openssl dhparam ...` 生成，并在 `ssl_dhparam` 指令中设置）。从数学角度来看，它们包括一个域素数 `p` 和一个生成元 `g`。更大的 `p` 将使得找到共同的密钥 `K` 更加困难，从而防御被动攻击。

  > 要使用基于签名的身份验证，你需要某种 DH 交换（固定或临时/短暂）来交换会话密钥。如果你使用 `DHE` 密码但不指定这些参数，NGINX 将使用默认的临时 Diffie-Hellman 参数来定义如何执行 Diffie-Hellman (DH) 密钥交换。在较老的版本中，NGINX 使用弱密钥（默认为 `1024 bit`），评分较低。

  > 你应该始终使用椭圆曲线 Diffie Hellman 临时密钥交换（`ECDHE`），如果你想保留对旧客户端的支持，也使用 `DHE`。由于对普遍监视的担忧日益增加，推荐使用提供前向保密的密钥交换，参见例如 [RFC 7525 - 前向保密](https://tools.ietf.org/html/rfc7525#section-6.3) <sup>[IETF]</sup>。

  > 确保你的 OpenSSL 库更新到最新的可用版本，并鼓励你的客户端也使用更新的软件。更新的浏览器会丢弃低和弱的 DH 参数（低于 768/1024 位）。

  > 为了更好的兼容性同时保证密钥交换的安全性，你应该优先选择 E（临时）而不是 E（EC）。推荐的配置是：`ECDHE` > `DHE`（使用至少 2048 位的唯一密钥）> `ECDH`。这样，如果初始握手失败，将使用 `DHE` 启动另一个握手。

  > `DHE` 比 `ECDHE` 慢。如果你关心性能，优先选择 `ECDHE-ECDSA` 或 `ECDHE-RSA` 而不是 `DHE`。OWASP 估计，带 `DHE` 的 TLS 握手比 `ECDHE` 多消耗 2.4 倍的 CPU。

  > Diffie-Hellman 需要一些初始设置参数。`ssl_dhparam` 中的参数（使用 `openssl dhparam ...` 生成）定义了 OpenSSL 如何执行 Diffie-Hellman (DH) 密钥交换。它们包括一个域素数 `p` 和一个生成元 `g`。

  > 自定义这些参数的可用性的目的是允许每个人使用自己的参数，最重要的是，找到这样的素数在计算上非常密集，你无法在每次连接时都承担这样的开销，所以它们是预先计算好的（从 HTTP 服务器设置）。在 NGINX 的情况下，我们使用 ssl_dhparam 指令设置它们。然而，使用自定义参数将使服务器不符合 [FIPS](https://csrc.nist.gov/News/2018/NIST-Publishes-Updates-to-SP-800-56A-and-800-56C) 要求：_该出版物批准使用特定的安全素数域参数组用于有限域 DH 和 MQV 方案，以及以前批准的域参数集。_ 另请参见批准的 [FFC 密钥协商的 TLS 组](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-56Ar3.pdf)（表 26，第 133 页）<sup>[NIST, pdf]</sup>。

  > 你可以使用自定义参数来防止受到 Logjam 攻击的影响（客户端和服务器都需要存在漏洞才能使攻击成功，因为服务器必须接受签署小的 `DHE_EXPORT` 参数，客户端必须接受它们作为有效的 `DHE` 参数）。

  > 现代客户端更喜欢 `ECDHE` 而不是其他变体，如果你的 NGINX 接受这种偏好，那么握手将根本不使用 DH 参数，因为它不会执行 `DHE` 密钥交换，而是执行 `ECDHE` 密钥交换。因此，如果你的服务器没有配置普通的 `DH/DHE` 密码，而只有椭圆曲线 DH（例如 `ECDHE`），那么你不需要设置自己的 `ssl_dhparam` 指令。启用 `DHE` 需要我们关注 DH 素数（也称为 `dhparams`）并信任 `DHE` - 在较新版本中，NGINX 为我们处理了这一点。

  > 椭圆曲线 Diffie-Hellman 是一种改进的 Diffie-Hellman 交换，它使用椭圆曲线密码学代替传统的 RSA 风格的大素数。因此，虽然我不确定它需要什么参数（如果需要的话），但我认为它不需要你正在生成的那种参数（`ECDH` 基于曲线，而不是素数，所以我认为传统的 DH 参数对你没有帮助）。

  > OpenSSL 中使用 `DHE` 密钥交换的密码套件需要 `tmp_DH` 参数，由 `ssl_dhparam` 指令提供。`DH_anon` 密钥交换也是如此，但实际上没有人使用它们。OpenSSL 关于 Diffie Hellman 参数的 wiki 页面说：_要使用完美前向保密密码套件，你必须设置 Diffie-Hellman 参数（在服务器端）。_ 另请参见 [SSL_CTX_set_tmp_dh_callback](https://www.openssl.org/docs/man1.0.2/man3/SSL_CTX_set_tmp_dh.html)。

  > 如果你使用 `ECDH/ECDHE` 密钥交换，请参见[使用更安全的 ECDH 曲线](#beginner-use-more-secure-ecdh-curve)规则。

  > 在较老版本的 OpenSSL 中，如果没有指定密钥大小，默认密钥大小是 `512/1024 bits` - 它很脆弱且可被破解。为了最佳安全配置，请使用你自己的 DH 组（至少 `2048 bit`）或使用已知安全的预定义 DH 组（推荐；由 IETF 在 [RFC 7919 - 支持的组注册表](https://tools.ietf.org/html/rfc7919#appendix-A) <sup>[IETF]</sup> 中推荐的预定义 DH 组 `ffdhe2048`、`ffdhe3072` 或 `ffdhe4096`，符合 NIST 和 FIPS 标准。它们经过审计，可能比随机生成的更能抵抗攻击。预定义组示例：
  >
  >  - [ffdhe2048](https://ssl-config.mozilla.org/ffdhe2048.txt)
  >  - [ffdhe4096](https://ssl-config.mozilla.org/ffdhe4096.txt)

  > `2048 bit` 通常被认为是安全的，并且已经远远进入了 "无法破解" 的区域。然而，多年前人们认为 1024 位是安全的，所以如果你追求长期抵抗能力，你会使用 `4096 bit`（对于 RSA 密钥和 DH 参数都是如此）。如果你想在 SSL Labs 测试的密钥交换中获得 100% 评分，这一点也很重要。

  > TLS 客户端也应拒绝静态 Diffie-Hellman - 在[此](https://tools.ietf.org/id/draft-dkg-tls-reject-static-dh-00.html)草案中有所描述。

  > 你应该记住，`4096 bit` 模数会使 DH 计算变慢，并且实际上不会提高安全性。

  关于 DH 参数推荐大小的[好解释](https://security.stackexchange.com/questions/47204/dh-parameters-recommended-size/47207#47207)：

  > _当前各个机构（包括 NIST）的建议要求 DH 使用 `2048-bit` 模数。已知的 DH 破解算法的成本高得离谱，无法使用已知的地球技术完成。请参阅此站点了解相关主题。_
  >
  > _你不想过度追求大小，因为计算使用成本随素数大小急剧上升（介于二次和三次之间，取决于一些实现细节），但 `2048-bit` DH 应该是足够的（一台基本的低端 PC 每秒可以处理数百个 `2048-bit` DH 操作）。_

  再看看 [Matt Palmer](https://www.hezmatt.org/~mpalmer/blog/) 的回答：

  > _确实，将 `2048 bit` DH 参数描述为 "弱得可怜" 是非常误导人的。没有已知的可行密码攻击可以破解任意的强 2048 位 DH 组。为了保护免受因破解 DH 而导致的会话密钥未来泄露，当然，你希望你的 DH 参数尽可能长，但由于 `1024 bit` DH 只是刚刚变得可行，`2048 bits` 在大多数用途中应该还可以持续一段时间。_

  看看来自 [Guide to Deploying Diffie-Hellman for TLS](https://weakdh.org/sysadmin.html) 的有趣答案：

  > _2. 部署（临时）椭圆曲线 Diffie-Hellman（ECDHE）。椭圆曲线 Diffie-Hellman（ECDH）密钥交换避免了所有已知可行的密码分析攻击，现代 Web 浏览器现在更喜欢 ECDHE 而不是原始的有限域 Diffie-Hellman。我们用来攻击标准 Diffie-Hellman 组的离散对数算法并没有从预计算中获得同样强的优势，并且各个服务器不需要生成唯一的椭圆曲线。_

  **我的建议：**

  > 如果你只使用 TLS 1.3 - 不需要 `ssl_dhparam`（不使用）。同样，如果你使用 `ECDHE/ECDH` - 不需要 `ssl_dhparam`（不使用）。如果你使用 `DHE/DH` - 需要 `ssl_dhparam` 配合 DH 参数（至少 `2048 bit`）。默认情况下没有设置参数，因此 `DHE` 密码将不会被使用。

###### 示例

设置 DH 参数：

```nginx
# curl https://ssl-config.mozilla.org/ffdhe2048.txt --output ffdhe2048.pem
ssl_dhparam ffdhe2048.pem;
```

生成 DH 参数：

```bash
# To generate a DH parameters:
openssl dhparam -out /etc/nginx/ssl/dhparam_4096.pem 4096

# To produce "DSA-like" DH parameters:
openssl dhparam -dsaparam -out /etc/nginx/ssl/dhparam_4096.pem 4096

# Use the pre-defined DH groups:
curl https://ssl-config.mozilla.org/ffdhe4096.txt > /etc/nginx/ssl/ffdhe4096.pem

# NGINX configuration only for DH/DHE:
ssl_dhparam /etc/nginx/ssl/dhparams_4096.pem;
```

&nbsp;&nbsp;:arrow_right: ssllabs score: <b>100%</b>

```bash
# To generate a DH parameters:
openssl dhparam -out /etc/nginx/ssl/dhparam_2048.pem 2048

# To produce "DSA-like" DH parameters:
openssl dhparam -dsaparam -out /etc/nginx/ssl/dhparam_2048.pem 2048

# Use the pre-defined DH groups:
curl https://ssl-config.mozilla.org/ffdhe2048.txt > /etc/nginx/ssl/ffdhe2048.pem

# NGINX configuration only for DH/DHE:
ssl_dhparam /etc/nginx/ssl/dhparam_2048.pem;
```

&nbsp;&nbsp;:arrow_right: ssllabs score: <b>90%</b>

###### 外部资源

- [Guide to Deploying Diffie-Hellman for TLS](https://weakdh.org/sysadmin.html)
- [Imperfect Forward Secrecy: How Diffie-Hellman Fails in Practice](https://weakdh.org/imperfect-forward-secrecy-ccs15.pdf) <sup>[pdf]</sup>
- [Weak Diffie-Hellman and the Logjam Attack](https://weakdh.org/)
- [Logjam: the latest TLS vulnerability explained](https://blog.cloudflare.com/logjam-the-latest-tls-vulnerability-explained/)
- [Pre-defined DHE groups](https://github.com/mozilla/ssl-config-generator/tree/master/docs)
- [Why is Mozilla recommending predefined DHE groups?](https://security.stackexchange.com/questions/149811/why-is-mozilla-recommending-predefined-dhe-groups)
- [Instructs OpenSSL to produce "DSA-like" DH parameters](https://security.stackexchange.com/questions/95178/diffie-hellman-parameters-still-calculating-after-24-hours/95184#95184)
- [OpenSSL generate different types of self signed certificate](https://security.stackexchange.com/questions/44251/openssl-generate-different-types-of-self-signed-certificate)
- [Public Diffie-Hellman Parameter Service/Tool](https://2ton.com.au/dhtool/)
- [Vincent Bernat's SSL/TLS & Perfect Forward Secrecy](http://vincent.bernat.im/en/blog/2011-ssl-perfect-forward-secrecy.html)
- [What's the purpose of DH Parameters?](https://security.stackexchange.com/questions/94390/whats-the-purpose-of-dh-parameters)
- [RSA and ECDSA performance](https://securitypitfalls.wordpress.com/2014/10/06/rsa-and-ecdsa-performance/)
- [SSL/TLS: How to choose your cipher suite](https://technology.amis.nl/2017/07/04/ssltls-choose-cipher-suite/)
- [Diffie-Hellman and its TLS/SSL usage](https://security.stackexchange.com/questions/41205/diffie-hellman-and-its-tls-ssl-usage)
- [Google Plans to Deprecate DHE Cipher Suites](https://www.digicert.com/blog/google-plans-to-deprecate-dhe-cipher-suites/)
- [Downgrade Attacks](https://tlseminar.github.io/downgrade-attacks/)
- [Diffie-Hellman key exchange (from this handbook)](SSL_TLS_BASICS.md#diffie-hellman-key-exchange)

<a id="beginner-prevent-replay-attacks-on-zero-round-trip-time"></a>
#### :beginner: 防止零往返时间的重放攻击

###### 说明

  > 此规则仅对 TLS 1.3 重要。默认情况下启用 TLS 1.3 不会启用 0-RTT 支持。毕竟，你应该充分了解使用此选项的所有潜在暴露因素和相关风险。

  > 0-RTT 握手是 TLS 会话恢复的替代方案的一部分，灵感来自 QUIC 协议。

  > 0-RTT 带来了显著的安全风险。使用 0-RTT，威胁行为者可以拦截加密的客户端消息并将其重新发送到服务器，欺骗服务器不适当地扩展对威胁行为者的信任，从而可能授予威胁行为者访问敏感数据的权限。

  > 另一方面，包含 0-RTT（零往返时间恢复）可以显著提高效率和连接时间。TLS 1.3 具有更快的握手，在 1-RTT 内完成。此外，它有一种特殊的会话恢复模式，在某些条件下，可以在第一次传输（0-RTT）中将数据发送到服务器。

  > 例如，Cloudflare 仅支持 [GET 请求且无查询参数](https://new.blog.cloudflare.com/introducing-0-rtt/) 的 0-RTT，以限制攻击面。此外，为了改进识别连接恢复尝试，他们通过向 0-RTT 请求添加额外的头来将此信息中继到源服务器。此头唯一标识请求，因此如果请求被重复，源服务器将知道这是重放攻击（应用程序需要跟踪从该头接收的值，并在非幂等端点上拒绝重复值）。

  > 为了在应用层防御此类攻击，应使用 `$ssl_early_data` 变量。你还需要确保 `Early-Data` 头被传递到你的应用程序。`$ssl_early_data` 如果使用了 TLS 1.3 早期数据且握手未完成，则返回 1。

  > 然而，作为升级的一部分，你应该禁用 0-RTT，直到你可以审计你的应用程序是否存在此类漏洞。

  > 为了发送早期数据，客户端和服务器[必须支持 PSK 交换模式](https://tools.ietf.org/html/rfc8446#section-2.3) <sup>[IETF]</sup>（会话 cookies）。

  > 此外，我想推荐[这篇](https://news.ycombinator.com/item?id=16667036)关于 TLS 1.3 和 0-RTT 的精彩讨论。

  如果你不确定是否启用 0-RTT，看看 Cloudflare 对此的看法：

  > _一般来说，0-RTT 对大多数网站和应用程序是安全的。如果你的 Web 应用程序做了奇怪的事情，并且你担心其重放安全性，请考虑不使用 0-RTT，直到你可以确定没有负面影响。[...] TLS 1.3 是 Web 性能和安全性的一大进步。通过将 TLS 1.3 与 0-RTT 结合，性能增益甚至更加显著。_

###### 示例

使用 OpenSSL 测试 0-RTT：

```bash
# 1)
_host="example.com"

cat > req.in << __EOF__
HEAD / HTTP/1.1
Host: $_host
Connection: close
__EOF__
# or:
# echo -e "GET / HTTP/1.1\r\nHost: $_host\r\nConnection: close\r\n\r\n" > req.in

openssl s_client -connect ${_host}:443 -tls1_3 -sess_out session.pem -ign_eof < req.in
openssl s_client -connect ${_host}:443 -tls1_3 -sess_in session.pem -early_data req.in

# 2)
python -m sslyze --early_data "$_host"
```

使用 `$ssl_early_data` 变量启用 0-RTT：

```nginx
server {

  ...

  ssl_protocols TLSv1.2 TLSv1.3;
  # To enable 0-RTT (TLS 1.3):
  ssl_early_data on;

  location / {

    proxy_pass http://backend_x20;
    # It protect against such attacks at the application layer:
    proxy_set_header Early-Data $ssl_early_data;

  }

  ...

}
```

###### 外部资源

- [Security Review of TLS1.3 0-RTT](https://github.com/tlswg/tls13-spec/issues/1001)
- [Introducing Zero Round Trip Time Resumption (0-RTT)](https://new.blog.cloudflare.com/introducing-0-rtt/)
- [What Application Developers Need To Know About TLS Early Data (0RTT)](https://blog.trailofbits.com/2019/03/25/what-application-developers-need-to-know-about-tls-early-data-0rtt/)
- [Zero round trip time resumption (0-RTT)](https://www.riklewis.com/2019/08/zero-round-trip-time-resumption-0-rtt/)
- [Session Resumption Protocols and Efficient Forward Security for TLS 1.3 0-RTT](https://eprint.iacr.org/2019/228)
- [Replay Attacks on Zero Round-Trip Time: The Case of the TLS 1.3 Handshake Candidates](https://eprint.iacr.org/2017/082.pdf) <sup>[pdf]</sup>
- [0-RTT and Anti-Replay](https://tools.ietf.org/html/rfc8446#section-8) <sup>[IETF]</sup>
- [Using Early Data in HTTP (2017)](https://tools.ietf.org/id/draft-thomson-http-replay-00.html_) <sup>[IETF]</sup>
- [Using Early Data in HTTP (2018)](https://tools.ietf.org/html/draft-ietf-httpbis-replay-04) <sup>[IETF]</sup>
- [0-RTT Handshakes](https://ldapwiki.com/wiki/0-RTT%20Handshakes)

<a id="beginner-defend-against-the-beast-attack"></a>
#### :beginner: 防御 BEAST 攻击

###### 说明

  > 通常 BEAST 攻击依赖于 SSL/TLS（TLSv1.0 及更早版本）中 `CBC` 模式使用的弱点。

  > 更具体地说，要成功执行 BEAST 攻击，需要满足一些条件：
  >
  >   - 必须使用使用块密码（特别是 `CBC`）的易受攻击的 SSL 版本
  >   - JavaScript 或 Java applet 注入 - 必须与网站在同一来源
  >   - 必须能够嗅探网络连接数据

  > 为了防止可能的 BEAST 攻击，你应该启用服务器端保护，这会使服务器密码优先于客户端密码，并完全从协议栈中排除 TLS 1.0。

  > 当 `ssl_prefer_server_ciphers` 设置为 on 时，Web 服务器所有者可以控制哪些密码可用。

  > 控制之所以更受青睐，是因为 SSL 以及 TLS v1.0 和 TLS v1.1 中存在旧的和不安全的密码，当服务器支持旧 TLS 版本且 `ssl_prefer_server_ciphers` 关闭时，攻击者可以干扰握手并强制连接使用弱密码，从而允许解密连接。

  > 在现代设置中，首选设置是 `ssl_prefer_server_ciphers off`，因为客户端设备可以根据客户端设备的硬件能力选择其偏好的加密方法。因此，我们让客户端为其硬件配置选择最高性能的密码套件。

###### 示例

```nginx
# In TLSv1.0 and TLSv1.1
ssl_prefer_server_ciphers on;

# In TLSv1.2 and TLSv1.3
ssl_prefer_server_ciphers off;
```

###### 外部资源

- [Here Come The ⊕ Ninjas](https://bug665814.bmoattachments.org/attachment.cgi?id=540839) <sup>[pdf]</sup>
- [An Illustrated Guide to the BEAST Attack](https://commandlinefanatic.com/cgi-bin/showarticle.cgi?article=art027)
- [How the BEAST Attack Works](https://www.netsparker.com/blog/web-security/how-the-beast-attack-works/)
- [Is BEAST still a threat?](https://blog.ivanristic.com/2013/09/is-beast-still-a-threat.html)
- [SSL/TLS attacks: Part 1 – BEAST Attack](https://niiconsulting.com/checkmate/2013/12/ssltls-attacks-part-1-beast-attack/)
- [Beat the BEAST with TLS 1.1/1.2 and More](https://blogs.cisco.com/security/beat-the-beast-with-tls) <sup>[not found]</sup>
- [Duong and Rizzo's paper on the BEAST attack)](https://images.techhive.com/downloads/idge/imported/article/ifw/2011/09/26/beast_duong_rizzo.pdf) <sup>[pdf]</sup>
- [ImperialViolet - Real World Crypto 2013](https://www.imperialviolet.org/2013/01/13/rwc03.html)
- [Use only strong ciphers - Hardening - P1 (from this handbook)](#beginner-use-only-strong-ciphers)

<a id="beginner-mitigation-of-crimebreach-attacks"></a>
#### :beginner: 缓解 CRIME/BREACH 攻击

###### 说明

  > 禁用 HTTP 压缩或仅压缩零敏感内容。此外，在使用 TLS 时，不应在私有响应上使用 HTTP 压缩。

  > 默认情况下，`gzip` 压缩模块已安装但未在 NGINX 中启用。

  > 你可能永远不应使用 TLS 压缩。某些用户代理（至少 Chrome 无论如何都会禁用它）。禁用 SSL/TLS 压缩可以非常有效地阻止攻击（像 OpenSSL 这样的库在构建时使用 `no-comp` 配置选项禁用压缩）。通过 TLS 1.2 部署 HTTP/2 必须禁用 TLS 压缩（请参见 [RFC 7540 - TLS 功能的使用](https://tools.ietf.org/html/rfc7540#section-9.2) <sup>[IETF]</sup>）。

  > CRIME 利用 SSL/TLS 压缩，自 NGINX 1.3.2 起已禁用。BREACH 仅利用 HTTP 压缩。

  > 由于在 SSL 请求上启用了 gzip（HTTP 压缩，而非 TLS 压缩），某些攻击是可能的（例如真正的 BREACH 攻击很复杂，并且仅当特定信息在 HTTP 响应中返回时才适用）。

  > 压缩不是攻击成功的唯一条件，因此使用压缩并不意味着攻击会成功。一般来说，你应该考虑 HTTPS 站点意外性能下降是否比 HTTPS 站点意外易受攻击更好。

  > 在大多数情况下，最好的操作是迁移到 TLS 1.3 或禁用 SSL 的 gzip（在比 1.3 更旧的 TLS 版本中），但一些资源解释说这不是解决问题的合适方案。缓解主要是在应用程序级别，然而常见的缓解方法是在包含敏感数据的任何响应中添加随机长度数据（这是 TLSv1.3 的默认行为 - [5.4. 记录填充](https://tools.ietf.org/html/draft-ietf-tls-tls13-21#section-5.4) <sup>[IETF]</sup>）。更多信息请参阅 [nginx-length-hiding-filter-module](https://github.com/nulab/nginx-length-hiding-filter-module)。此过滤模块提供功能，可在响应体末尾追加随机生成的 HTML 注释，以隐藏正确的响应长度，使攻击者难以猜测安全令牌。

  > 我倾向于安全优先于性能，但压缩可以（我认为）用于对公开可用的静态内容（如 css 或 js）和零敏感信息的 HTML 内容（如 "关于我们" 页面）进行 HTTP 压缩。

###### 示例

```nginx
# Disable dynamic HTTP compression:
gzip off;

# Enable dynamic HTTP compression for specific location context:
location ^~ /assets/ {

  gzip on;

  ...

}
```

###### 外部资源

- [Is HTTP compression safe?](https://security.stackexchange.com/questions/20406/is-http-compression-safe)
- [HTTP compression continues to put encrypted communications at risk](https://www.pcworld.com/article/3051675/http-compression-continues-to-put-encrypted-communications-at-risk.html)
- [SSL/TLS attacks: Part 2 – CRIME Attack](http://niiconsulting.com/checkmate/2013/12/ssltls-attacks-part-2-crime-attack/)
- [How BREACH works (as I understand it)](https://security.stackexchange.com/questions/172581/to-avoid-breach-can-we-use-gzip-on-non-token-responses/172646#172646)
- [Defending against the BREACH Attack](https://blog.qualys.com/ssllabs/2013/08/07/defending-against-the-breach-attack)
- [To avoid BREACH, can we use gzip on non-token responses?](https://security.stackexchange.com/questions/172581/to-avoid-breach-can-we-use-gzip-on-non-token-responses)
- [Brotli compression algorithm and BREACH attack](https://security.stackexchange.com/questions/172188/brotli-compression-algorithm-and-breach-attack/197535#197535)
- [Don't Worry About BREACH](https://blog.ircmaxell.com/2013/08/dont-worry-about-breach.html)
- [The current state of the BREACH attack](https://www.sjoerdlangkemper.nl/2016/11/07/current-state-of-breach-attack/)
- [Module ngx_http_gzip_static_module](http://nginx.org/en/docs/http/ngx_http_gzip_static_module.html)
- [Offline Compression with Nginx](https://theartofmachinery.com/2016/06/06/nginx_gzip_static.html)
- [ImperialViolet - Real World Crypto 2013](https://www.imperialviolet.org/2013/01/13/rwc03.html)

<a id="beginner-enable-http-strict-transport-security"></a>
#### :beginner: 启用 HTTP 严格传输安全

###### 说明

  > 通常 HSTS 是网站告诉浏览器连接应仅加密的一种方式。这可以防止 MITM 攻击、降级攻击、发送明文 cookies 和会话 ID。正确实施 HSTS 是符合多层安全原则（纵深防御）的额外安全机制。

  > 此头对性能很有好处，因为它指示浏览器在客户端进行 HTTP 到 HTTPS 的重定向，而无需触及网络。

  > 该头指示浏览器在特定域名的特定时间内应无条件拒绝参与不安全的 HTTP 连接。

  > 当浏览器知道某个域名已启用 HSTS 时，它会做两件事：
  >
  > - 始终使用 `https://` 连接，即使点击 `http://` 链接或在地址栏中输入域名而不指定协议
  > - 移除用户点击无效证书警告的能力

  > HSTS 头需要在带有 `ssl` listen 语句的 HTTP 块内设置，否则你可能冒着在服务器上可能配置的 HTTP 站点上发送 Strict-Transport-Security 头的风险。此外，你应该对 HTTP 服务器块使用 `return 301` 以重定向到 HTTPS。

  > 理想情况下，你应该始终将 `includeSubdomains` 与 HSTS 一起使用。这将为主机名以及所有子域提供强健的安全保护。这里的问题是（不带 `includeSubdomains`），中间人攻击者可以创建任意子域并使用它们将 cookie 注入到你的应用程序中。在某些情况下，甚至可能发生泄露。当然，`includeSubdomains` 的缺点是，你将必须在 SSL 上部署所有子域。

  > 有几个简单的 HSTS 最佳实践（来自 [The Importance of a Proper HTTP Strict Transport Security Implementation on Your Web Server](https://blog.qualys.com/securitylabs/2016/03/28/the-importance-of-a-proper-http-strict-transport-security-implementation-on-your-web-server)）：
  >
  > - 最强的保护是确保所有请求的资源仅使用带有格式良好的 HSTS 头的 TLS。Qualys 建议在目标域的所有 HTTPS 资源上提供 HSTS 头
  >
  > - 建议将 `max-age` 指令的值设置为大于 `10368000` 秒（120 天），理想情况下设置为 `31536000`（一年）。网站应逐步提高 `max-age` 值，以确保当前域名和/或子域名的长期安全性
  >
  > - [RFC 6797 - includeSubDomains 的必要性](https://tools.ietf.org/html/rfc6797) <sup>[IETF]</sup> 主张 Web 应用程序在可能的情况下务必在策略定义中添加 `includeSubDomain` 指令。该指令的存在确保 HSTS 策略应用于发证主机的域名及其所有子域，例如 `example.com` 和 `www.example.com`
  >
  > - 应用程序绝不应通过明文 HTTP 头发送 HSTS 头，因为这样做会使连接容易受到 SSL 剥离攻击
  >
  > - 不推荐通过 meta 标签的 `http-equiv` 属性提供 HSTS 策略。根据 [RFC 6797](https://tools.ietf.org/html/rfc6797) <sup>[IETF]</sup>，用户代理不会关注接收内容上 `<meta>` 元素的 `http-equiv="Strict-Transport-Security"` 属性

  > 要满足 [HSTS 预加载列表](https://hstspreload.org/) 标准，根域名需要返回一个包含 `includeSubDomains` 和 `preload` 指令且最小 `max-age` 为一年的 `strict-transport-security` 头。你的站点还必须在根域名和所有子域名上提供有效的 SSL 证书，并在同一主机上将所有 HTTP 请求重定向到 HTTPS。

  > 在启用 HSTS 之前，你最好非常确定你的网站确实全部使用 HTTPS，因为 HSTS 会增加回滚策略的复杂性。Google 建议以这种方式启用 HSTS：
  >
  >   1) 首先在没有 HSTS 的情况下推出 HTTPS 页面
  >   2) 开始使用较短的 `max-age` 发送 HSTS 头。监控来自用户和其他客户端的流量，以及依赖方的性能（如广告）
  >   3) 缓慢增加 HSTS `max-age`
  >   4) 如果 HSTS 没有对你的用户和搜索引擎产生负面影响，如果需要，你可以要求将你的站点添加到大多数主要浏览器使用的 HSTS 预加载列表中

  **我的建议：**

  > 将 `max-age` 设置为较大的值，如 `31536000`（12 个月）或 `63072000`（24 个月），并包含 `includeSubdomains` 参数。

###### 示例

```nginx
add_header Strict-Transport-Security "max-age=63072000; includeSubdomains" always;
```

&nbsp;&nbsp;:arrow_right: ssllabs score: <b>A+</b>

###### 外部资源

- [OWASP Secure Headers Project - HSTS](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#hsts)
- [Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)
- [Security HTTP Headers - Strict-Transport-Security](https://zinoui.com/blog/security-http-headers#strict-transport-security)
- [HTTP Strict Transport Security](https://https.cio.gov/hsts/)
- [HTTP Strict Transport Security Cheat Sheet](https://www.owasp.org/index.php/HTTP_Strict_Transport_Security_Cheat_Sheet)
- [HSTS Cheat Sheet](https://scotthelme.co.uk/hsts-cheat-sheet/)
- [HSTS Preload and Subdomains](https://textslashplain.com/2018/04/09/hsts-preload-and-subdomains/)
- [Check HSTS preload status and eligibility](https://hstspreload.org/)
- [HTTP Strict Transport Security (HSTS) and NGINX](https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/)
- [Is HSTS as a proper substitute for HTTP-to-HTTPS redirects?](https://www.reddit.com/r/bigseo/comments/8zw45d/is_hsts_as_a_proper_substitute_for_httptohttps/)
- [How to configure HSTS on www and other subdomains](https://www.danielmorell.com/blog/how-to-configure-hsts-on-www-and-other-subdomains)
- [HSTS: Is includeSubDomains on main domain sufficient?](https://serverfault.com/questions/927336/hsts-is-includesubdomains-on-main-domain-sufficient)
- [The HSTS preload list eligibility](https://www.danielmorell.com/blog/how-to-configure-hsts-on-www-and-other-subdomains)
- [HSTS Deployment Recommendations](https://hstspreload.org/#deployment-recommendations)
- [How does HSTS handle mixed content?](https://serverfault.com/questions/927145/how-does-hsts-handle-mixed-content)
- [Broadening HSTS to secure more of the Web](https://security.googleblog.com/2017/09/broadening-hsts-to-secure-more-of-web.html)
- [The Road To HSTS](https://engineeringblog.yelp.com/2017/09/the-road-to-hsts.html)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-reduce-xss-risks-content-security-policy"></a>
#### :beginner: 降低 XSS 风险（内容安全策略）

###### 说明

  > CSP 降低了一系列攻击的风险和影响，包括现代浏览器中的跨站脚本和其他跨站注入（跨站脚本漏洞允许你进入显示页面的代码中，插入浏览器在显示页面时将执行的元素（特别是攻击者浏览器执行的未授权脚本））。它是一个很好的纵深防御措施，使得利用偶然的失误变得更加困难。

  > CSP 策略的加入显著阻碍了成功的 XSS 攻击、UI 重绘（点击劫持）、恶意使用框架或 CSS 注入。

  > 将已知良好的资源源列入白名单、拒绝执行潜在危险的內联脚本以及禁止使用 eval 都是缓解跨站脚本攻击的有效机制。

  > 开始构建头的默认策略是：阻止所有。通过修改 CSP 值，管理员/程序员可以针对特定资源组（例如分别针对脚本、图像等）放宽限制。

  > 你应该非常个性化地处理，绝不设置从互联网或其他地方找到的 CSP 示例值。盲目部署 "标准/推荐" 版本的 CSP 头会破坏大多数 Web 应用程序。请注意，配置错误的 Content Security Policy 可能使应用程序面临客户端威胁，包括跨站脚本、跨框架脚本和跨站请求伪造。

  > 在启用此头之前，你应该与开发人员和应用程序架构师讨论 CSP 参数。他们可能必须更新 Web 应用程序以移除任何內联脚本和样式，并在此进行一些额外的修改（实施用户引入的内容验证机制、使用允许字符列表限制用户可输入到应用程序各个字段的内容，或对应用程序传输到浏览器的用户数据进行编码）。

  > 严格的策略将显著提高安全性，更高的代码质量将减少总体错误数量。CSP 永远不能取代安全代码 - 新限制有助于减少攻击（如 XSS）的影响，但它们不是预防机制！

  > 在实施之前，你应该始终验证 CSP：
  >
  > - [CSP Evaluator](https://csp-evaluator.withgoogle.com/)
  > - [Content Security Policy (CSP) Validator](https://cspvalidator.org/)

  > 用于生成策略（但请注意，这些类型的工具可能过期或存在错误）：
  >
  > - [https://report-uri.com/home/generate](https://report-uri.com/home/generate)

###### 示例

```nginx
# This policy allows images, scripts, AJAX, and CSS from the same origin, and does not allow any other resources to load:
add_header Content-Security-Policy "default-src 'none'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self';" always;
```

###### 外部资源

- [OWASP Secure Headers Project - Content-Security-Policy](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#csp)
- [Content Security Policy (CSP) Quick Reference Guide](https://content-security-policy.com/)
- [Content Security Policy Cheat Sheet – OWASP](https://www.owasp.org/index.php/Content_Security_Policy_Cheat_Sheet)
- [Content Security Policy – OWASP](https://www.owasp.org/index.php/Content_Security_Policy)
- [Content Security Policy - An Introduction - Scott Helme](https://scotthelme.co.uk/content-security-policy-an-introduction/)
- [CSP Cheat Sheet - Scott Helme](https://scotthelme.co.uk/csp-cheat-sheet/)
- [Security HTTP Headers - Content-Security-Policy](https://zinoui.com/blog/security-http-headers#content-security-policy)
- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [Content Security Policy (CSP) Validator](https://cspvalidator.org/)
- [Can I Use CSP](https://caniuse.com/#search=CSP)
- [CSP Is Dead, Long Live CSP!](https://ai.google/research/pubs/pub45542)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-control-the-behaviour-of-the-referer-header-referrer-policy"></a>
#### :beginner: 控制 Referer 头的行为（Referrer-Policy）

###### 说明

  > 引荐策略处理浏览器向服务器传输哪些信息（与 url 相关）以检索外部资源。

  > 基本上这是一种隐私增强，当你想要隐藏链接被点击时用户来自你网站的信息，对链接目标的域名所有者隐藏信息。

  > 我认为最安全的值是 `no-referrer`，它指定不将任何引荐信息与从特定请求客户端到任何源发出的请求一起发送。该头将被完全省略。

  > 使用 `no-referrer` 有其优势，因为它允许你隐藏 HTTP 头，这增加了在线隐私和用户自身的安全性。另一方面，它主要影响分析（理论上，不应该对 SEO 有任何影响），因为 `no-referrer` 指定隐藏这类信息。

  > Mozilla 有一个很好的表格，解释了每种引荐策略选项的工作原理。来自 [Mozilla 关于 Referer Policy 的参考文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy)。

###### 示例

```nginx
# This policy does not send information about the referring site after clicking the link:
add_header Referrer-Policy "no-referrer";
```

###### 外部资源

- [OWASP Secure Headers Project - Referrer-Policy](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#rp)
- [A new security header: Referrer Policy](https://scotthelme.co.uk/a-new-security-header-referrer-policy/)
- [Security HTTP Headers - Referrer-Policy](https://zinoui.com/blog/security-http-headers#referrer-policy)
- [What you need to know about Referrer Policy](https://searchengineland.com/need-know-referrer-policy-276185)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-provide-clickjacking-protection-x-frame-options"></a>
#### :beginner: 提供点击劫持保护（X-Frame-Options）

###### 说明

  > 通过声明你的应用程序是否可以使用框架嵌入到其他（外部）页面，帮助保护访问者免受点击劫持攻击。

  > 建议在不应允许在框架中渲染页面的页面上使用 `x-frame-options` 头。

  > 此头允许 3 个参数，但在我看来，你应该只考虑两个：`deny` 参数用于通常禁止嵌入资源，或 `sameorigin` 参数用于允许在同一主机/源上嵌入资源。

  > 它的优先级低于 CSP，但我认为它值得作为后备使用。

###### 示例

```nginx
# Only pages from the same domain can "frame" this URL:
add_header X-Frame-Options "SAMEORIGIN" always;
```

###### 外部资源

- [OWASP Secure Headers Project - X-Frame-Options](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#xfo)
- [HTTP Header Field X-Frame-Options](https://tools.ietf.org/html/rfc7034) <sup>[IETF]</sup>
- [Clickjacking Defense Cheat Sheet](https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet)
- [Security HTTP Headers - X-Frame-Options](https://zinoui.com/blog/security-http-headers#x-frame-options)
- [X-Frame-Options - Scott Helme](https://scotthelme.co.uk/hardening-your-http-response-headers/#x-frame-options)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-prevent-some-categories-of-xss-attacks-x-xss-protection"></a>
#### :beginner: 防止某些类别的 XSS 攻击（X-XSS-Protection）

###### 说明

  > 启用现代 Web 浏览器内置的跨站脚本（XSS）过滤器。

  > 它通常默认已启用，因此此头的作用是在用户禁用了过滤器的情况下为这个特定网站重新启用它。

  > 我认为你可以设置此头而无需咨询 Web 应用架构师，但所有编写良好的应用都应发出头 `X-XSS-Protection: 0` 并忘记此功能。如果你想要更好的用户代理可以提供的额外安全性，请使用严格的 `Content-Security-Policy` 头。[Mikko Rantalainen](https://stackoverflow.com/users/334451/mikko-rantalainen) 有一个[准确的答案](https://stackoverflow.com/a/57802070)。

###### 示例

```nginx
add_header X-XSS-Protection "1; mode=block" always;
```

###### 外部资源

- [OWASP Secure Headers Project - X-XSS-Protection](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#xxxsp)
- [XSS (Cross Site Scripting) Prevention Cheat Sheet](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet)
- [DOM based XSS Prevention Cheat Sheet](https://www.owasp.org/index.php/DOM_based_XSS_Prevention_Cheat_Sheet)
- [X-XSS-Protection HTTP Header](https://www.tunetheweb.com/security/http-security-headers/x-xss-protection/)
- [Security HTTP Headers - X-XSS-Protection](https://zinoui.com/blog/security-http-headers#x-xss-protection)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-prevent-sniff-mimetype-middleware-x-content-type-options"></a>
#### :beginner: 防止 MIME 类型嗅探（X-Content-Type-Options）

###### 说明

  > 防止浏览器进行 MIME 类型嗅探。

  > 设置此头将阻止浏览器将文件解释为 HTTP 头中声明的 content type 以外的其他内容。

###### 示例

```nginx
# Disallow content sniffing:
add_header X-Content-Type-Options "nosniff" always;
```

###### 外部资源

- [OWASP Secure Headers Project - X-Content-Type-Options](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#xcto)
- [X-Content-Type-Options HTTP Header](https://www.keycdn.com/support/x-content-type-options)
- [Security HTTP Headers - X-Content-Type-Options](https://zinoui.com/blog/security-http-headers#x-content-type-options)
- [X-Content-Type-Options - Scott Helme](https://scotthelme.co.uk/hardening-your-http-response-headers/#x-content-type-options)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-deny-the-use-of-browser-features-feature-policy"></a>
#### :beginner: 拒绝使用浏览器功能（Feature-Policy）

###### 说明

  > 此头保护你的站点免受第三方使用具有安全和隐私影响的 API，也防止你自己的团队添加过时的 API 或优化不佳的图像。

###### 示例

```nginx
add_header Feature-Policy "geolocation 'none'; midi 'none'; notifications 'none'; push 'none'; sync-xhr 'none'; microphone 'none'; camera 'none'; magnetometer 'none'; gyroscope 'none'; speaker 'none'; vibrate 'none'; fullscreen 'none'; payment 'none'; usb 'none';";
```

###### 外部资源

- [Feature Policy Explainer](https://docs.google.com/document/d/1k0Ua-ZWlM_PsFCFdLMa8kaVTo32PeNZ4G7FFHqpFx4E/edit)
- [Policy Controlled Features](https://github.com/w3c/webappsec-feature-policy/blob/master/features.md)
- [Security HTTP Headers - Feature-Policy](https://zinoui.com/blog/security-http-headers#feature-policy)
- [Feature policy playground](https://featurepolicy.info/)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-reject-unsafe-http-methods"></a>
#### :beginner: 拒绝不安全的 HTTP 方法

###### 说明

  > 普通的 Web 服务器支持 `GET`、`HEAD` 和 `POST` 方法以获取静态和动态内容。其他方法（例如 `OPTIONS`、`TRACE`）不应在公共 Web 服务器上支持，因为它们会增加攻击面。

  > 其中一些方法通常暴露是危险的，有些在生产环境中是多余的，可以被视为额外的攻击面。尽管如此，关闭它们也值得，因为你可能不需要它们。

  > 某些 API（例如 RESTful API）也使用其他方法。除了以下保护之外，应用程序架构师还应验证传入请求。

  > 支持 `TRACE` 方法可能导致跨站跟踪攻击，这有助于捕获其他应用程序用户的会话 ID。此外，此方法可用于尝试识别有关应用程序运行环境的额外信息（例如到达应用程序路径上的缓存服务器的存在）。

  > 支持 `OPTIONS` 方法不是直接威胁，但它是攻击者的额外信息来源，可能有助于有效攻击。

  > 支持 `HEAD` 方法也有风险（真的！） - 它不被认为是危险的，但可以通过模仿 `GET` 请求来攻击 Web 应用程序。其次，使用 `HEAD` 可以通过限制服务器发送的数据量来加速攻击过程。如果授权机制基于 `GET` 和 `POST`，`HEAD` 方法可能允许绕过这些保护。

  > 我认为 `HEAD` 请求通常被代理或 CDN 用于有效地确定页面是否已更改，而无需下载整个正文（这对检索响应头中写入的元信息很有用）。而且，如果你禁用它，只会增加吞吐量成本。

  > 不推荐使用 `if` 语句来阻止不安全的 HTTP 方法，相反你可以使用 `limit_except` 指令（应该比正则表达式评估更快），但请记住，它的使用有限制：仅在 `location` 内部。我认为在这种情况下使用正则表达式更灵活一些。

  > 在选择配置哪种方法之前，请注意关于 [401、403 和 405 HTTP 响应码之间的区别](https://serverfault.com/questions/905708/using-limit-except-to-deny-all-except-get-head-and-post/905922#905922) 的出色解释。HTTP 方法差异的简要描述：
  >
  >   - 0: 请求到来...
  >   - 1: `405 Method Not Allowed` 表示服务器不允许该 URI 上的该方法
  >   - 2: `401 Unauthorized` 表示用户未通过身份验证
  >   - 3: `403 Forbidden` 表示访问客户端未被授权执行该请求

  > 在我看来，如果 HTTP 资源无法处理给定 HTTP 方法的请求，它应该发送一个 `Allow` 头来列出允许的 HTTP 方法。为此，你可以使用 `add_header`，但记住[潜在问题](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)。

###### 示例

推荐的配置：

```nginx
# If we are in server context, it's good to use construction like this:
add_header Allow "GET, HEAD, POST" always;

if ($request_method !~ ^(GET|HEAD|POST)$) {

  # You can also use 'add_header' inside 'if' context:
  # add_header Allow "GET, HEAD, POST" always;
  return 405;

}
```

替代配置（仅在 `location` 上下文内）：

```nginx
# Note: allowing the GET method makes the HEAD method also allowed.
location /api {

  limit_except GET POST {

    allow 192.168.1.0/24;
    deny  all;  # always return 403 error code

    # or:
    # auth_basic "Restricted access";
    # auth_basic_user_file /etc/nginx/htpasswd;

    ...

  }

}
```

但绝不要这样做（非常不推荐！）混合使用 `allow/deny` 和 `return` 指令：

```nginx
location /api {

  limit_except GET POST {

    allow 192.168.1.0/24;
    # It's only example (return is not allowed in limit_except),
    # all clients (also from 192.168.1.0/24) might get 405 error code if it worked:
    return 405;

    ...

  }

}
```

###### 外部资源

- [Hypertext Transfer Protocol (HTTP) Method Registry](https://www.iana.org/assignments/http-methods/http-methods.xhtml) <sup>[IANA]</sup>
- [Vulnerability name: Unsafe HTTP methods](https://www.onwebsecurity.com/security/unsafe-http-methods.html)
- [Cross Site Tracing](https://owasp.org/www-community/attacks/Cross_Site_Tracing)
- [Cross-Site Tracing (XST): The misunderstood vulnerability](https://deadliestwebattacks.com/2010/05/18/cross-site-tracing-xst-the-misunderstood-vulnerability/)
- [Penetration Testing Of A Web Application Using Dangerous HTTP Methods](https://www.sans.org/reading-room/whitepapers/testing/penetration-testing-web-application-dangerous-http-methods-33945) <sup>[pdf]</sup>
- [Blocking/allowing IP addresses (from this handbook)](HELPERS.md#blockingallowing-ip-addresses)
- [allow and deny (from this handbook)](NGINX_BASICS.md#allow-and-deny)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-prevent-caching-of-sensitive-data"></a>
#### :beginner: 防止缓存敏感数据

###### 说明

  > 此策略应由应用程序架构师实施，但根据经验，我知道这并不总是发生。

  > 不要缓存或持久化敏感数据。由于浏览器对缓存 HTTPS 内容有不同的默认行为，包含敏感信息的页面应包括 `Cache-Control` 头以确保内容不被缓存。

  > 一种选择是向相关的 HTTP/1.1 和 HTTP/2 响应添加反缓存头，例如 `Cache-Control: no-cache, no-store` 和 `Expires: 0`。

  > 为了覆盖各种浏览器实现，防止内容被缓存的完整头集应为：
  >
  > 1 - `Cache-Control: no-cache, no-store, private, must-revalidate, max-age=0, no-transform`<br>
  > 2 - `Pragma: no-cache`<br>
  > 3 - `Expires: 0`

###### 示例

```nginx
location /api {

  expires 0;
  add_header Cache-Control "no-cache, no-store";

}
```

###### 外部资源

- [RFC 2616 - HTTP/1.1: Standards Track](https://tools.ietf.org/html/rfc2616) <sup>[IETF]</sup>
- [RFC 7234 - HTTP/1.1: Caching](https://tools.ietf.org/html/rfc7234) <sup>[IETF]</sup>
- [HTTP Cache Headers - A Complete Guide](https://www.keycdn.com/blog/http-cache-headers)
- [Caching best practices & max-age gotchas](https://jakearchibald.com/2016/caching-best-practices/)
- [Increasing Application Performance with HTTP Cache Headers](https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers)
- [HTTP Caching](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-limit-concurrent-connections"></a>
#### :beginner: 限制并发连接

###### 说明

  > NGINX 提供了基本且易于启用的拒绝服务攻击（如 DoS）保护。默认情况下，用户可拥有的活动连接数没有限制。

  > 在我看来，全局（在 `http` 上下文中）切断冗余/不必要的连接也是好的，但要小心并监控你的访问和错误日志。你也可以在每个 NGINX `location` 匹配上下文中设置它，例如为搜索页面、在线用户显示、成员列表等设置。

  > 你应该限制单个客户端 IP 地址可以打开的会话数，再次设置为适合真实用户的值。因为大多数情况下，多余的流量是由机器人生成的，旨在压倒服务器，流量速率远高于人类用户能生成的速率。并发连接限制必须激活以启用最大会话限制。

  > 然而，请注意，虽然 NGINX 是 Cloudflare 风格保护的关键元素，但在你的 Web 服务器上设置 NGINX 并希望它能保护你是远远不够的。

  > IP 连接限制在一定程度上有所帮助，但对于足够大的第 7 层 DDoS 攻击，它仍然可能压倒你的服务器。对我来说，第一道防线应该是硬件防火墙（但这还不够）或 DDoS 缓解设备，具有无状态保护机制，可以处理数百万个连接尝试（提供深度检查以过滤不良流量并允许良好流量通过），而无需连接表条目或耗尽其他系统资源。

  > 特别是，最好在网络提供商一侧启用缓解措施，并在流量到达你之前通过外部公司提供的第 7 层 DDoS 缓解过滤器路由流量。我认为这是最好的解决方案。

###### 示例

```nginx
http {

  limit_conn_zone $binary_remote_addr zone=slimit:10m;

  # Set globally:
  limit_conn slimit 10;

  ...

  server {

    # Or in the server context:
    limit_conn slimit 10;

    ...

  }

}
```

###### 外部资源

- [Module ngx_http_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)
- [Mitigating DDoS Attacks with NGINX and NGINX Plus](https://www.nginx.com/blog/mitigating-ddos-attacks-with-nginx-and-nginx-plus/)
- [What is a DDoS Attack?](https://www.cloudflare.com/learning/ddos/what-is-a-ddos-attack/)
- [Nginx-Lua-Anti-DDoS](https://github.com/C0nw0nk/Nginx-Lua-Anti-DDoS)
- [Extend NGINX with Lua — DDOS Mitigation using Cookie validation](https://medium.com/@satrobit/extend-nginx-with-lua-ddos-mitigation-using-cookie-validation-8b27ffc1322a)
- [Blocking/allowing IP addresses (from this handbook)](HELPERS.md#blockingallowing-ip-addresses)
- [allow and deny (from this handbook)](NGINX_BASICS.md#allow-and-deny)
- [ngx_http_geoip_module (from this handbook)](NGINX_BASICS.md#ngx-http-geoip-module)
- [Control Buffer Overflow attacks - Hardening - P2 (from this handbook)](#beginner-control-buffer-overflow-attacks)
- [Mitigating Slow HTTP DoS attacks (Closing Slow Connections) - Hardening - P2 (from this handbook)](#beginner-mitigating-slow-http-dos-attacks-closing-slow-connections)
- [Use limit_conn to improve limiting the download speed - Performance - P3 (from this handbook)](#beginner-use-limit_conn-to-improve-limiting-the-download-speed)

<a id="beginner-control-buffer-overflow-attacks"></a>
#### :beginner: 控制缓冲区溢出攻击

###### 说明

  > 缓冲区溢出攻击通过将数据写入缓冲区并超过该缓冲区的边界，覆盖进程的内存片段来实现。为了防止在 NGINX 中出现这种情况，我们可以为所有客户端设置缓冲区大小限制。

  > 如果使用了所有服务器内存，大的 `POST` 请求大小可能有效地导致 DoS 攻击。允许大文件上传到服务器可能会使攻击者更容易利用系统资源并成功执行拒绝服务攻击。

  > 相应的值取决于你的服务器内存和流量大小。很久以前我找到了一个有趣的公式：
  >
  > &nbsp;&nbsp;`MAX_MEMORY = client_body_buffer_size x CONCURRENT_TRAFFIC - OS_RAM - FS_CACHE`
  >
  > 我认为关键是要监控所有内容（内存/CPU/流量），并根据你的使用情况更改设置，从小开始，然后逐步增加。

  > 在我看来，使用较小的 `client_body_buffer_size`（略大于 10k，但不要太大）肯定更好，因为更大的缓冲区可能会加剧 DoS 攻击，因为你会为它分配更多内存。

  > 提示：如果请求体大于 `client_body_buffer_size`，它会被写入磁盘，不在内存中可用，因此 `$request_body` 不可用。此外，将 `client_body_buffer_size` 设置得太高可能会影响日志文件大小（如果你记录了 `$request_body`）。

###### 示例

```nginx
client_body_buffer_size 16k;      # default: 8k (32-bit) | 16k (64-bit)
client_header_buffer_size 1k;     # default: 1k
client_max_body_size 100k;        # default: 1m
large_client_header_buffers 2 1k; # default: 4 8k
```

###### 外部资源

- [Module ngx_http_core_module](http://nginx.org/en/docs/http/ngx_http_core_module.html)
- [SCG WS nginx](https://www.owasp.org/index.php/SCG_WS_nginx)

<a id="beginner-mitigating-slow-http-dos-attacks-closing-slow-connections"></a>
#### :beginner: 缓解慢速 HTTP DoS 攻击（关闭慢速连接）

###### 说明

  > 你可以关闭写入数据过于频繁的连接，这可能代表试图尽可能长时间保持连接开放（从而降低服务器接受新连接的能力）。

  > 在我看来，2-3 秒的 `keepalive_timeout` 通常足以让大多数人解析 HTML/CSS 并检索所需的图像、图标或框架，NGINX 中的连接成本很低，因此增加这个值通常是安全的。然而，设置过高会导致资源浪费（主要是内存），因为即使没有流量，连接也会保持打开状态，可能显著影响性能。我认为这应尽可能接近你的平均响应时间。

  > 我还建议，如果你将 `send_timeout` 设置得较小，那么你的 Web 服务器将快速关闭连接，这将为连接主机提供更多可用连接。

  > 这些参数可能仅在高流量 Web 服务器中才相关。两者的目标是相同的：减少连接数并更高效地处理请求。要么将所有请求放入一个连接（keep alive），要么快速关闭连接以处理更多请求（send timeout）。

###### 示例

```nginx
client_body_timeout 10s;    # default: 60s
client_header_timeout 10s;  # default: 60s
keepalive_timeout 5s 5s;    # default: 75s
send_timeout 10s;           # default: 60s
```

###### 外部资源

- [Module ngx_http_core_module](http://nginx.org/en/docs/http/ngx_http_core_module.html)
- [Module ngx_http_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)
- [Mitigating DDoS Attacks with NGINX and NGINX Plus](https://www.nginx.com/blog/mitigating-ddos-attacks-with-nginx-and-nginx-plus/)
- [What is a DDoS Attack?](https://www.cloudflare.com/learning/ddos/what-is-a-ddos-attack/)
- [SCG WS nginx](https://www.owasp.org/index.php/SCG_WS_nginx)
- [How to Protect Against Slow HTTP Attacks](https://blog.qualys.com/securitylabs/2011/11/02/how-to-protect-against-slow-http-attacks)
- [Effectively Using and Detecting The Slowloris HTTP DoS Tool](https://ma.ttias.be/effectively-using-detecting-the-slowloris-http-dos-tool/)
- [Limit concurrent connections - Hardening - P1 (from this handbook)](#beginner-limit-concurrent-connections)

<a id="reverse-proxy"></a>
# 反向代理

返回 **[目录](https://github.com/trimstray/nginx-admins-handbook#table-of-contents)** 或 **[下一步？](https://github.com/trimstray/nginx-admins-handbook#whats-next)** 部分。

  > :pushpin:&nbsp; NGINX 的常见用途之一是将其设置为代理服务器，可以卸载高容量分布式 Web 应用程序的许多基础设施问题。

- **[基础规则](#base-rules)**
- **[调试](#debugging)**
- **[性能](#performance)**
- **[加固](#hardening)**
- **[≡ 反向代理 (8)](#reverse-proxy)**
  * [使用与后端协议兼容的 pass 指令](#beginner-use-pass-directive-compatible-with-backend-protocol)
  * [注意 proxy_pass 指令中的尾部斜杠](#beginner-be-careful-with-trailing-slashes-in-proxy_pass-directive)
  * [仅使用 $host 变量设置和传递 Host 头](#beginner-set-and-pass-host-header-only-with-host-variable)
  * [正确设置 X-Forwarded-For 头的值](#beginner-set-properly-values-of-the-x-forwarded-for-header)
  * [在反向代理后面不要使用带 $scheme 的 X-Forwarded-Proto](#beginner-dont-use-x-forwarded-proto-with-scheme-behind-reverse-proxy)
  * [始终将 Host、X-Real-IP 和 X-Forwarded 头传递到后端](#beginner-always-pass-host-x-real-ip-and-x-forwarded-headers-to-the-backend)
  * [使用不带 X- 前缀的自定义头](#beginner-use-custom-headers-without-x--prefix)
  * [在 proxy_pass 中始终使用 $request_uri 而不是 $uri](#beginner-always-use-request_uri-instead-of-uri-in-proxy_pass)
- **[负载均衡](#load-balancing)**
- **[其他](#others)**

<a id="beginner-use-pass-directive-compatible-with-backend-protocol"></a>
#### :beginner: 使用与后端协议兼容的 pass 指令

###### 说明

  > 所有 `proxy_*` 指令都与使用特定后端协议的后端相关。

  > 你应该始终仅对在后端层工作的 HTTP 服务器使用 `proxy_pass`（在引用 HTTP 后端之前也设置 `http://` 协议），和其他 `*_pass` 指令仅用于非 HTTP 后端服务器（如 uWSGI 或 FastCGI）。

  > 诸如 `uwsgi_pass`、`fastcgi_pass` 或 `scgi_pass` 等指令专门为非 HTTP 应用程序设计，你应该使用它们而不是 `proxy_pass`（非 HTTP 通信）。

  > 例如：`uwsgi_pass` 使用 uwsgi 协议。`proxy_pass` 使用普通 HTTP 与 uWSGI 服务器通信。uWSGI 文档声称 uwsgi 协议更好、更快，并且可以从 uWSGI 的所有特殊功能中受益。你可以向 uWSGI 发送数据类型信息以及应调用哪个 uWSGI 插件来生成响应。使用 HTTP（`proxy_pass`）则无法获得这些。

###### 示例

不推荐的配置：

```nginx
server {

  location /app/ {

    # For this, you should use uwsgi_pass directive.
    # backend layer: uWSGI Python app
    proxy_pass 192.168.154.102:4000;

  }

  ...

}
```

推荐的配置：

```nginx
server {

  location /app/ {

    # backend layer: OpenResty as a front for app
    proxy_pass http://192.168.154.102:80;

  }

  location /app/v3 {

    # backend layer: uWSGI Python app
    uwsgi_pass 192.168.154.102:8080;

  }

  location /app/v4 {

    # backend layer: php-fpm app
    fastcgi_pass 192.168.154.102:8081;

  }
  ...

}
```

###### 外部资源

- [Passing a Request to a Proxied Server](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/#passing-a-request-to-a-proxied-server)
- [Reverse proxy (from this handbook)](NGINX_BASICS.md#reverse-proxy)

<a id="beginner-be-careful-with-trailing-slashes-in-proxy_pass-directive"></a>
#### :beginner: 注意 `proxy_pass` 指令中的尾部斜杠

###### 说明

  > NGINX 按字面替换部分内容，你可能最终得到一些奇怪的 URL。

  > 如果 `proxy_pass` 没有使用 URI（即没有 `server:port` 之后的路径），NGINX 将按照原始请求的方式放置 URI，包括所有双斜杠、`../` 等。

  > `proxy_pass` 中的 URI 类似于 alias 指令，意味着 NGINX 将用 `proxy_pass` 指令中的 URI 替换匹配 location 前缀的部分（我故意将其设置为与 location 前缀相同），因此 URI 将与请求的相同但经过规范化（没有双斜杠和所有那些东西）。

###### 示例

```nginx
location = /a {

  proxy_pass http://127.0.0.1:8080/a;

  ...

}

location ^~ /a/ {

  proxy_pass http://127.0.0.1:8080/a/;

  ...

}
```

###### 外部资源

- [Module ngx_http_proxy_module - proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)
- [Trailing slashes (from this handbook)](NGINX_BASICS.md#trailing-slashes)

<a id="beginner-set-and-pass-host-header-only-with-host-variable"></a>
#### :beginner: 仅使用 `$host` 变量设置和传递 `Host` 头

###### 说明

  > 你应该几乎始终使用 `$host` 作为传入主机变量，因为它是唯一保证无论用户代理如何行为都能提供合理内容的变量，除非你特别需要其他变量之一的语义。

  > 修改 `Host` 头以确保下游服务器上的虚拟主机解析按预期工作总是一个好主意。

  > `$host` 只是 `$http_host` 经过一些处理（去除端口号并转换为小写）并带有默认值（来自 `server_name`），因此使用 `$http_host` 时并不会减少对客户端发送的 `Host` 头的 "暴露"。但这并没有什么危险。

  > 变量 `$host` 是来自请求行或 http 头的主机名。变量 `$server_name` 是我们当前所在 server 块的名称。

  > 区别在 NGINX 文档中有解释：
  >
  >   - `$host` 按此优先顺序包含：来自请求行的主机名、或来自 `Host` 请求头字段的主机名、或匹配请求的服务器名称
  >   - `$http_host` 包含 HTTP `Host` 头字段的内容（如果请求中存在）（始终等于 `HTTP_HOST` 请求头）
  >   - `$server_name` 包含处理请求的虚拟主机的 `server_name`，如 NGINX 配置中定义的那样。如果一个服务器包含多个服务器名称，只有第一个会出现在此变量中

  > `$http_host` 优于 `$host:$server_port`，因为它使用 URL 中存在的端口，而 `$server_port` 使用 NGINX 监听的端口。

  > 另一方面，使用 `$host` 有其自身的漏洞；你必须通过定义默认 server 块来正确处理 `Host` 字段缺失的情况，以捕获这些请求。关键在于，将 `proxy_set_header Host $host` 更改为其他值根本不会改变这种行为，因为当 `Host` 请求字段存在时，`$host` 值将等于 `$http_host` 值。

  > 在 NGINX 服务器中，我们将通过使用 catch-all 虚拟主机来实现这一点。如果客户端请求中出现无法识别/未定义的 `Host` 头，Web 服务器会引用这些虚拟主机。指定精确（而非通配符）的 `server_name` 值也是一个好主意。

  > 当然，最重要的防线是在应用程序端正确实现解析机制，例如使用 `Host` 头的允许值列表。你的 Web 应用程序应完全符合 [RFC 7230](https://tools.ietf.org/html/rfc7230)，以避免因不一致解释与 HTTP 事务关联的主机而导致的问题。根据上述 RFC，正确的解决方案是将多个 `Host` 头和字段名周围的空格视为错误。

###### 示例

```nginx
proxy_set_header Host $host;
```

###### 外部资源

- [RFC 2616 - The Resource Identified by a Request](http://tools.ietf.org/html/rfc2616#section-5.2) <sup>[IETF]</sup>
- [RFC 2616 - Host](http://tools.ietf.org/html/rfc2616#section-14.23) <sup>[IETF]</sup>
- [Module ngx_http_proxy_module - proxy_set_header](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header)
- [What is the difference between Nginx variables $host, $http_host, and $server_name?](https://serverfault.com/questions/706438/what-is-the-difference-between-nginx-variables-host-http-host-and-server-na/706439#706439)
- [HTTP_HOST and SERVER_NAME Security Issues](https://expressionengine.com/blog/http-host-and-server-name-security-issues)
- [Reasons to use '$http_host' instead of '$host' with 'proxy_set_header Host' in template?](https://github.com/jwilder/nginx-proxy/issues/763#issuecomment-286481168)
- [Tip: keep the Host header via nginx proxy_pass](https://www.simplicidade.org/notes/2011/02/15/tip-keep-the-host-header-via-nginx-proxy_pass/)
- [What is a Host Header Attack?](https://www.acunetix.com/blog/articles/automated-detection-of-host-header-attacks/)
- [Practical HTTP Host header attacks](https://www.skeletonscribe.net/2013/05/practical-http-host-header-attacks.html)
- [Host of Troubles Vulnerabilities](https://hostoftroubles.com/)
- [$10k host header](https://www.ezequiel.tech/p/10k-host-header.html)

<a id="beginner-set-properly-values-of-the-x-forwarded-for-header"></a>
#### :beginner: 正确设置 `X-Forwarded-For` 头的值

###### 说明

  > `X-Forwarded-For` (XFF) 是一个自定义 HTTP 头，用于携带客户端的原始 IP 地址（标识通过代理或负载均衡器服务的原始请求的客户端 IP 地址），以便另一端的应用程序知道它是什么。否则它只会看到代理 IP 地址，这会让一些应用程序不高兴。

  > `X-Forwarded-For` 依赖于代理服务器，它实际上应该传递连接到它的客户端的 IP 地址。当连接通过一系列代理服务器时，`X-Forwarded-For` 可以提供一个逗号分隔的 IP 地址列表，第一个是最下游（即用户）。因此，代理服务器后面的服务器需要知道哪些是可信任的。

  > 在大多数情况下，大多数代理或负载均衡器会自动包含一个 `X-Forwarded-For` 头，用于调试、统计和生成位置相关内容，基于原始请求。

  > XFF 的有用性取决于代理服务器是否真实报告原始主机的 IP 地址；因此，有效使用 XFF 需要知道哪些代理是可信任的，例如通过在其维护者可以信任的服务器白名单中查找。

  > 使用的代理可以将其设置为任何想要的值，因此你不能信任其值。然而，大多数代理确实设置了正确的值。此头主要被缓存代理使用，在这些情况下，你控制着代理，因此可以验证它是否提供了正确的信息。在所有其他情况下，其值应被视为不可信。

  > 一些系统还使用 `X-Forwarded-For` 来实施访问控制。许多应用程序依赖于了解客户端的实际 IP 地址来帮助防止欺诈和实现访问。

  > `X-Forwarded-For` 头字段的值可以在客户端设置 - 这也可以称为 `X-Forwarded-For` 欺骗。然而，当 Web 请求通过代理服务器时，代理服务器会通过附加客户端的 IP 地址来修改 `X-Forwarded-For` 字段。这将导致 `X-Forwarded-For` 字段中出现两个逗号分隔的 IP 地址。

  > 反向代理不会透明地传递源 IP 地址。当你在后端服务器的日志中需要正确的客户端源 IP 地址时，这会很麻烦。我认为这个问题的最佳解决方案是配置负载均衡器，添加/修改一个包含客户端源 IP 的 `X-Forwarded-For` 头，并以正确的格式将其转发到后端。

  > 不幸的是，在代理端我们无法解决这个问题（所有解决方案都可能被欺骗），重要的是这个头能被应用程序服务器正确解释。这样做可以确保应用程序或下游服务拥有准确的信息来做出决策，包括那些关于访问和授权的决策。

  > 鉴于最新的 httpoxy 漏洞，确实需要一个完整的示例来说明如何正确使用 `HTTP_X_FORWARDED_FOR`。简而言之，负载均衡器设置头的 "最新" 部分。在我看来，出于安全原因，代理服务器必须由管理员手动指定。

  还有一个关于此情况的有趣想法：

  > _为了防止这种情况，我们必须默认不信任该头，并从我们的服务器向后追溯 IP 地址面包屑。首先我们需要确保 `REMOTE_ADDR` 是我们信任的、已将正确值追加到 `X-Forwarded-For` 末尾的人。如果是，那么我们需要确保我们信任 `X-Forwarded-For` IP 已经追加了之前的正确 IP，依此类推。直到，最终我们到达一个我们不信任的 IP，此时我们必须假设这是我们的用户的 IP。_ - 来自 [Xiao Yu](https://github.com/xyu) 的 [Proxies & IP Spoofing](https://xyu.io/2013/07/04/proxies-ip-spoofing/)。

###### 示例

```nginx
# The whole purpose that it exists is to do the appending behavior:
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

# Above is equivalent for this:
proxy_set_header X-Forwarded-For $http_x_forwarded_for,$remote_addr;

# The following is also equivalent for above but in this example we use http_realip_module:
proxy_set_header X-Forwarded-For "$http_x_forwarded_for, $realip_remote_addr";
```

###### 外部资源

- [Prevent X-Forwarded-For Spoofing or Manipulation](https://totaluptime.com/kb/prevent-x-forwarded-for-spoofing-or-manipulation/)
- [Bypass IP blocks with the X-Forwarded-For header](https://www.sjoerdlangkemper.nl/2017/03/01/bypass-ip-block-with-x-forwarded-for-header/)
- [Forwarded header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Forwarded)

<a id="beginner-dont-use-x-forwarded-proto-with-scheme-behind-reverse-proxy"></a>
#### :beginner: 在反向代理后面不要使用带 `$scheme` 的 `X-Forwarded-Proto`

###### 说明

  > `X-Forwarded-Proto` 可以由反向代理设置，以告诉应用程序是 HTTPS 还是 HTTP，甚至是无效的名称。

  > scheme（即 HTTP、HTTPS）变量仅在需要时评估（仅用于当前请求）。

  > 设置 `$scheme` 变量会导致在使用多个代理时出现扭曲。例如：如果客户端访问 `https://example.com`，代理将 scheme 值存储为 HTTPS。如果代理和下一级代理之间的通信通过 HTTP 进行，那么后端看到的 scheme 是 HTTP。因此，如果你在下一级代理上为 `X-Forwarded-Proto` 设置 `$scheme`，应用程序将看到与客户端传入时不同的值。

  > 为了解决这个问题，你也可以使用[这个](HELPERS.md#set-correct-scheme-passed-in-x-forwarded-proto)配置片段。

###### 示例

```nginx
# 1) client <-> proxy <-> backend
proxy_set_header X-Forwarded-Proto $scheme;

# 2) client <-> proxy <-> proxy <-> backend
# proxy_set_header X-Forwarded-Proto https;
proxy_set_header X-Forwarded-Proto $proxy_x_forwarded_proto;
```

###### 外部资源

- [Reverse Proxy - Passing headers (from this handbook)](NGINX_BASICS.md#passing-headers)

<a id="beginner-always-pass-host-x-real-ip-and-x-forwarded-headers-to-the-backend"></a>
#### :beginner: 始终将 `Host`、`X-Real-IP` 和 `X-Forwarded` 头传递到后端

###### 说明

  > 当使用 NGINX 作为反向代理时，你可能希望将远程客户端的某些信息传递到你的后端 Web 服务器。我认为这是好的实践，因为它让你对转发的头有更多的控制。

  > 对于代理后面的服务器来说，这非常重要，因为它允许正确解释客户端。代理是这些服务器的 "眼睛"，它们不应允许对现实的扭曲感知。如果不是所有请求都通过代理传递，结果直接从客户端接收的请求可能包含例如头中不准确的 IP 地址。

  > `X-Forwarded` 头对于统计或过滤也很重要。另一个例子可以是应用程序上的访问控制规则，因为没有这些头，过滤机制可能无法正常工作。

  > 如果你使用像 Apache 之类的前端服务作为 API 的前端，你将需要这些头来理解用于连接到 API 的 IP 或主机名。

  > 如果你使用 HTTPS 协议（如今已成为标准），转发这些头也很重要。

  > 然而，我不会依赖所有 `X-Forwarded` 头的存在或它们数据的有效性。

###### 示例

```nginx
location / {

  proxy_pass http://bk_upstream_01;

  # The following headers also should pass to the backend:
  #   - Host - host name from the request line, or host name from the Host request header field, or the server name matching a request
  # proxy_set_header Host $host:$server_port;
  # proxy_set_header Host $http_host;
  proxy_set_header Host $host;

  #   - X-Real-IP - forwards the real visitor remote IP address to the proxied server
  proxy_set_header X-Real-IP $remote_addr;

  # X-Forwarded headers stack:
  #   - X-Forwarded-For - mark origin IP of client connecting to server through proxy
  # proxy_set_header X-Forwarded-For $remote_addr;
  # proxy_set_header X-Forwarded-For $http_x_forwarded_for,$remote_addr;
  # proxy_set_header X-Forwarded-For "$http_x_forwarded_for, $realip_remote_addr";
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

  #   - X-Forwarded-Host - mark origin host of client connecting to server through proxy
  # proxy_set_header X-Forwarded-Host $host:443;
  proxy_set_header X-Forwarded-Host $host:$server_port;

  #   - X-Forwarded-Server - the hostname of the proxy server
  proxy_set_header X-Forwarded-Server $host;

  #   - X-Forwarded-Port - defines the original port requested by the client
  # proxy_set_header X-Forwarded-Port 443;
  proxy_set_header X-Forwarded-Port $server_port;

  #   - X-Forwarded-Proto - mark protocol of client connecting to server through proxy
  # proxy_set_header X-Forwarded-Proto https;
  # proxy_set_header X-Forwarded-Proto $proxy_x_forwarded_proto;
  proxy_set_header X-Forwarded-Proto $scheme;

}
```

###### 外部资源

- [Forwarding Visitor's Real-IP + Nginx Proxy/Fastcgi backend correctly](https://easyengine.io/tutorials/nginx/forwarding-visitors-real-ip/)
- [Using the Forwarded header](https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/)
- [Reverse Proxy - Passing headers (from this handbook)](NGINX_BASICS.md#passing-headers)
- [Set properly values of the X-Forwarded-For header - Reverse Proxy - P1 (from this handbook)](#beginner-set-properly-values-of-the-x-forwarded-for-header)
- [Don't use X-Forwarded-Proto with $scheme behind reverse proxy - Reverse Proxy - P1 (from this handbook)](#beginner-dont-use-x-forwarded-proto-with-scheme-behind-reverse-proxy)

<a id="beginner-use-custom-headers-without-x--prefix"></a>
#### :beginner: 使用不带 `X-` 前缀的自定义头

###### 说明

  > 使用带有 `X-` 前缀的自定义头不是被禁止的，但不鼓励。换句话说，你可以继续使用 `X-` 前缀的头，但不推荐，你也不应将它们记录为公共标准。

  > [IETF 草案](https://tools.ietf.org/html/draft-saintandre-xdash-00) 被提出以弃用推荐的非标准头使用 `X-` 前缀。原因在于，当带有 `X-` 前缀的非标准头成为标准时，移除 `X-` 前缀会破坏向后兼容性，迫使应用协议支持两个名称（例如，`x-gzip` 和 `gzip` 现在等同）。因此，官方建议是直接使用有意义的名称，不带 `X-` 前缀。

  > Internet 工程任务组在 [RFC 6648](https://tools.ietf.org/html/rfc6648) <sup>[IETF]</sup> 中发布了官方弃用 `X-` 前缀的建议：
  >
  > - _3. 对新参数创建者的建议 [...] 不应在其参数名称前加上 "X-" 或类似结构。_
  > - _4. 对协议设计者的建议 [...] 不应禁止带有 "X-" 前缀或类似结构的参数被注册。[...] 绝对不能规定带有 "X-" 前缀或类似结构的参数需要被理解为非标准化的。[...] 绝对不能规定不带 "X-" 前缀或类似结构的参数需要被理解为标准化的。_

  > 自定义头名称前面的 `X-` 习惯上表示它是实验性/非标准/供应商特定的。一旦成为 HTTP 的标准部分，它就会失去前缀。

  > 如果可能的话，新的自定义头应使用非未使用且有意义的头名称。

###### 示例

不推荐的配置：

```nginx
add_header X-Backend-Server $hostname;
```

推荐的配置：

```nginx
add_header Backend-Server $hostname;
```

###### 外部资源

- [Use of the "X-" Prefix in Application Protocols](https://tools.ietf.org/html/draft-saintandre-xdash-00) <sup>[IETF]</sup>
- [Why we need to deprecate x prefix for HTTP headers?](https://tonyxu.io/posts/2018/http-deprecate-x-prefix/)
- [Custom HTTP headers : naming conventions](https://stackoverflow.com/questions/3561381/custom-http-headers-naming-conventions/3561399#3561399)
- [Set the HTTP headers with add_header and proxy_*_header directives properly - Base Rules - P1 (from this handbook)](#beginner-set-the-http-headers-with-add_header-and-proxy__header-directives-properly)

<a id="beginner-always-use-request_uri-instead-of-uri-in-proxy_pass"></a>
#### :beginner: 在 `proxy_pass` 中始终使用 `$request_uri` 而不是 `$uri`

###### 说明

  > 当然，对于我要说的内容总有例外。我认为将未更改的 URI 传递到后端层的最佳规则是使用不带任何参数的 `proxy_pass http://<backend>`。

  > 如果你在 `proxy_pass` 指令末尾添加更多内容，除非你知道自己在做什么，否则应始终将未更改的 URI 传递到后端层。例如，在 `proxy_pass` 指令中意外使用 `$uri` 会使你面临 HTTP 头注入漏洞，因为 URL 编码的字符会被解码（这有时很重要，与 `$request_uri` 不等效）。而且，`$uri` 的值可能在请求处理过程中发生变化，例如在进行内部重定向或使用索引文件时。

  > `$request_uri` 等于从客户端接收的原始请求 URI，包括参数。在这种情况下（传递如 `$request_uri` 这样的变量），如果指令中指定了 URI，它将按原样传递给服务器，替换原始请求 URI。

  > 还要注意，使用带变量的 `proxy_pass` 意味着各种其他副作用，特别是使用解析器进行动态名称解析，通常比在配置中使用名称效率低。

  看看 NGINX 文档对此的描述：

  > _如果 proxy_pass 未指定 URI，请求 URI 会以客户端发送时的相同形式传递给服务器，当处理原始请求时 [...]_

###### 示例

不推荐的配置：

```nginx
location /foo {

  proxy_pass http://django_app_server$uri;

}
```

推荐的配置：

```nginx
location /foo {

  proxy_pass http://django_app_server$request_uri;

}
```

最推荐的配置：

```nginx
location /foo {

  proxy_pass http://django_app_server;

}
```

<a id="load-balancing"></a>
# 负载均衡

返回 **[目录](https://github.com/trimstray/nginx-admins-handbook#table-of-contents)** 或 **[下一步？](https://github.com/trimstray/nginx-admins-handbook#whats-next)** 部分。

  > :pushpin:&nbsp; 负载均衡是一种有用的机制，可以将传入流量分配到多个有能力的服务器。我们可以改进一些关于 NGINX 作为负载均衡器工作的规则。

- **[基础规则](#base-rules)**
- **[调试](#debugging)**
- **[性能](#performance)**
- **[加固](#hardening)**
- **[反向代理](#reverse-proxy)**
- **[≡ 负载均衡 (2)](#load-balancing)**
  * [调整被动健康检查](#beginner-tweak-passive-health-checks)
  * [不要通过注释禁用后端，使用 down 参数](#beginner-dont-disable-backends-by-comments-use-down-parameter)
- **[其他](#others)**

<a id="beginner-tweak-passive-health-checks"></a>
#### :beginner: 调整被动健康检查

###### 说明

  > 健康监控对所有类型的负载均衡都很重要，主要是为了业务连续性。被动检查监视当请求通过 NGINX 时出现的失败或超时连接。

  > 此功能默认启用，但此处提到的参数允许你调整其行为。默认值为：`max_fails=1` 和 `fail_timeout=10s`。

###### 示例

```nginx
upstream backend {

  server bk01_node:80 max_fails=3 fail_timeout=5s;
  server bk02_node:80 max_fails=3 fail_timeout=5s;

}
```

###### 外部资源

- [Module ngx_http_upstream_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html)

<a id="beginner-dont-disable-backends-by-comments-use-down-parameter"></a>
#### :beginner: 不要通过注释禁用后端，使用 `down` 参数

###### 说明

  > 有时我们需要关闭后端，例如在维护期间。我认为好的解决方案是使用 `down` 参数将服务器标记为永久不可用，即使停机时间很短。

  > 如果你使用 IP Hash 负载均衡技术，这一点也很重要。如果其中一个服务器需要临时移除，应使用此参数标记它，以保留客户端 IP 地址的当前哈希。

  > 注释适用于真正永久禁用服务器，或者如果你想保留信息用于历史目的。

  > NGINX 还提供了 `backup` 参数，将服务器标记为备用服务器。当主服务器不可用时，它会接收请求。我很少出于上述目的使用此选项，只有在确定后端在维护期间可以工作时才使用。

  > 你还可以使用 `ngx_dynamic_upstream` 通过 HTTP API 动态操作 upstream。

###### 示例

```nginx
upstream backend {

  server bk01_node:80 max_fails=3 fail_timeout=5s down;
  server bk02_node:80 max_fails=3 fail_timeout=5s;

}
```

###### 外部资源

- [Module ngx_http_upstream_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html)
- [Module ngx_dynamic_upstream](https://github.com/cubicdaiya/ngx_dynamic_upstream)

<a id="others"></a>
# 其他

返回 **[目录](https://github.com/trimstray/nginx-admins-handbook#table-of-contents)** 或 **[下一步？](https://github.com/trimstray/nginx-admins-handbook#whats-next)** 部分。

  > :pushpin:&nbsp; 这些规则并非严格与 NGINX 相关，但在我看来它们也是安全中非常重要的方面。

- **[基础规则](#base-rules)**
- **[调试](#debugging)**
- **[性能](#performance)**
- **[加固](#hardening)**
- **[反向代理](#reverse-proxy)**
- **[负载均衡](#load-balancing)**
- **[≡ 其他 (4)](#others)**
  * [正确设置证书链](#beginner-set-the-certificate-chain-correctly)
  * [启用 DNS CAA 策略](#beginner-enable-dns-caa-policy)
  * [使用 security.txt 定义安全策略](#beginner-define-security-policies-with-securitytxt)
  * [使用 tcpdump 诊断和排查 HTTP 问题](#beginner-use-tcpdump-to-diagnose-and-troubleshoot-the-http-issues)

<a id="beginner-set-the-certificate-chain-correctly"></a>
#### :beginner: 正确设置证书链

###### 说明

  > 信任链是从终端实体数字证书到作为信任锚的根证书颁发机构（CA）的验证和确认的链接路径。

  > 你的浏览器（可能还有你的操作系统）附带一组受信任的 CA 列表。这些预安装的证书作为信任锚，从中派生出所有进一步的信任。在访问 HTTPS 网站时，你的浏览器会验证服务器在 TLS 握手期间提供的信任链是否结束于一个本地受信任的根证书。

  > 证书链的验证是任何基于证书的身份验证过程中的关键部分。如果系统不遵循证书到根服务器的信任链，证书作为信任度量就失去了所有用处。

  > 服务器始终发送链，但绝不应呈现包含作为根 CA 证书的信任锚的证书链，因为根对验证目的无用。根据 TLS 标准，链可能包含也可能不包含根证书本身；客户端不需要该根，因为它已经有了。实际上，如果客户端还没有根，那么从服务器接收它也没有帮助，因为根只能通过已经存在而被信任。

  > 此外，信任锚在认证路径中的存在可能对使用 SSL/TLS 建立连接时的性能产生负面影响，因为根在每个握手中被 "下载"（这允许你每个连接节省约 1 kB 的带宽，并减少服务器端用于 TLS 会话参数的内存消耗）。

  > 作为最佳实践，从服务器中移除自签名根。证书捆绑包应仅包括证书的公钥和任何中间证书颁发机构的公钥。浏览器只信任解析到其信任存储中已有的根的证书，它们会忽略证书捆绑包中发送的根证书（否则，任何人都可以发送任何根）。

  > 如果链断开，就无法验证托管数据的服务器是否正确（预期的）服务器 - 无法确定服务器是否如其所说（你失去了验证连接安全性或信任它的能力）。连接仍然安全（仍然加密），但主要问题应该是修复该证书链。你应该通过将所有证书从证书连接到受信任的根证书（不包括根，按此顺序）连接起来，手动解决不完整的证书链问题，以防止此类问题。

  > 不完整链示例：[incomplete-chain.badssl.com](https://incomplete-chain.badssl.com/)。

  来自 "SSL Labs: SSL and TLS Deployment Best Practices - 2.1 Use Complete Certificate Chains"：

  > _无效的证书链实际上会使服务器证书失效，并导致浏览器警告。在实践中，这个问题有时难以诊断，因为某些浏览器可以重建不完整的链，而某些则不能。所有浏览器倾向于缓存和重用中间证书。_

###### 示例

在 OpenSSL 侧：

```bash
$ ls
root_ca.crt inter_ca.crt example.com.crt

# Build manually. Server certificate comes first in the chain file, then the intermediates:
$ cat example.com.crt inter_ca.crt > certs/example.com/example.com-chain.crt
```

要构建有效的 SSL 证书链，你可以使用 [mkchain](https://github.com/trimstray/mkchain) 工具。它还可以帮助你修复不完整的链并下载所有缺失的 CA 证书。你也可以从远程服务器下载所有证书并正确设置证书链。

```bash
# If you have all certificates:
$ ls /data/certs/example.com
root.crt inter01.crt inter02.crt certificate.crt
$ mkchain -i /data/certs/example.com -o /data/certs/example.com-chain.crt

# If you have only server certificate (downloads all missing CA certificates automatically):
$ ls /data/certs/example.com
certificate.crt
$ mkchain -i certificate.crt -o /data/certs/example.com-chain.crt

# If you want to download certificate chain from existing domain:
$ mkchain -i https://incomplete-chain.badssl.com/ -o /data/certs/example.com-chain.crt
```

在 NGINX 侧：

```nginx
server {

  listen 192.168.10.2:443 ssl http2;

  ssl_certificate certs/example.com/example.com-chain.crt;
  ssl_certificate_key certs/example.com/example.com.key;

  ...
```

###### 外部资源

- [What is the SSL Certificate Chain?](https://support.dnsimple.com/articles/what-is-ssl-certificate-chain/)
- [What is a chain of trust?](https://www.ssl.com/faqs/what-is-a-chain-of-trust/)
- [The Difference Between Root Certificates and Intermediate Certificates](https://www.thesslstore.com/blog/root-certificates-intermediate/)
- [Get your certificate chain right](https://medium.com/@superseb/get-your-certificate-chain-right-4b117a9c0fce)
- [Verify certificate chain with OpenSSL](https://www.itsfullofstars.de/2016/02/verify-certificate-chain-with-openssl/)
- [Chain of Trust (from this handbook)](SSL_TLS_BASICS.md#chain-of-trust)

<a id="beginner-enable-dns-caa-policy"></a>
#### :beginner: 启用 DNS CAA 策略

###### 说明

  > DNS CAA 策略帮助你控制哪些证书颁发机构被允许为你的域名颁发证书，因为如果没有 CAA 记录，任何 CA 都允许为域名颁发证书。

  > CAA 记录的目的是允许域名所有者声明哪些证书颁发机构被允许为域名颁发证书。它们还提供了指示在有人从未经授权的证书颁发机构请求证书时的通知规则的手段。

  > 如果没有 CAA 记录，任何 CA 都允许为域名颁发证书。如果存在 CAA 记录，则只有记录中列出的 CA 被允许为该主机名颁发证书。

###### 示例

针对 Let's Encrypt 的通用配置（Google Cloud DNS、Route 53、OVH 和其他托管服务）：

```bash
example.com. CAA 0 issue "letsencrypt.org"
```

针对 Let's Encrypt 的标准区域文件（BIND、PowerDNS 和 Knot DNS）：

```bash
example.com. IN CAA 0 issue "letsencrypt.org"
```

###### 外部资源

- [DNS Certification Authority Authorization (CAA) Resource Record](https://tools.ietf.org/html/rfc6844) <sup>[IETF]</sup>
- [CAA Records](https://support.dnsimple.com/articles/caa-record/)
- [CAA Record Helper](https://sslmate.com/caa/)

<a id="beginner-define-security-policies-with-securitytxt"></a>
#### :beginner: 使用 `security.txt` 定义安全策略

###### 说明

  > 将 `security.txt` 添加到你的站点，文件中包含正确的联系方式，这样报告安全问题的人就不必猜测将报告发送到哪里。

  > `security.txt` 的主要目的是帮助公司和安全研究人员在努力保护平台时更容易。它还提供了帮助披露安全漏洞的信息。

  > 当安全研究人员检测到页面或应用程序中的潜在漏洞时，他们会尝试联系 "合适" 的人以 "负责任地" 揭示问题。确保信息到达正确的地址是值得的。

  > 此文件应放置在 `/.well-known/` 路径下，例如 `/.well-known/security.txt`（[RFC 5785](https://tools.ietf.org/html/rfc5785) <sup>[IETF]</sup>），适用于域名或 IP 地址的 Web 属性。

###### 示例

```bash
$ curl -ks https://example.com/.well-known/security.txt

Contact: security@example.com
Contact: +1-209-123-0123
Encryption: https://example.com/pgp.txt
Preferred-Languages: en
Canonical: https://example.com/.well-known/security.txt
Policy: https://example.com/security-policy.html
```

以及来自 Google：

```bash
$ curl -ks https://www.google.com/.well-known/security.txt

Contact: https://g.co/vulnz
Contact: mailto:security@google.com
Encryption: https://services.google.com/corporate/publickey.txt
Acknowledgements: https://bughunter.withgoogle.com/
Policy: https://g.co/vrp
Hiring: https://g.co/SecurityPrivacyEngJobs
# Flag: BountyCon{075e1e5eef2bc8d49bfe4a27cd17f0bf4b2b85cf}
```

###### 外部资源

- [A Method for Web Security Policies](https://tools.ietf.org/html/draft-foudil-securitytxt-05) <sup>[IETF]</sup>
- [security.txt](https://securitytxt.org/)
- [Say hello to security.txt](https://scotthelme.co.uk/say-hello-to-security-txt/)

<a id="beginner-use-tcpdump-to-diagnose-and-troubleshoot-the-http-issues"></a>
#### :beginner: 使用 tcpdump 诊断和排查 HTTP 问题

###### 说明

  > Tcpdump 是所有管理员和开发人员在故障排除时的瑞士军刀（是著名的命令行数据包分析器/协议解码工具）。你可以使用它来监控代理（或客户端）和后端之间的 HTTP 流量。

  > 我使用 tcpdump 进行快速检查。如果需要更深入的检查，我会捕获所有内容到文件中，并使用强大的嗅探器（如 Wireshark）打开它，它可以解码大量协议和过滤器。

  > 运行 `man tcpdump | less -Ip examples` 查看一些示例。

###### 示例

捕获所有内容并写入文件：

```bash
tcpdump -X -s0 -w dump.pcap <tcpdump_params>
```

监控入口（接口上）流量，按 `<ip:port>` 过滤：

```bash
tcpdump -ne -i eth0 -Q in host 192.168.252.1 and port 443
```

  * `-n` - 不转换地址（`-nn` 将不解析主机名或端口）
  * `-e` - 打印链路级头
  * `-i [iface|any]` - 设置接口
  * `-Q|-D [in|out|inout]` - 选择发送/接收方向（`-D` - 用于旧版 tcpdump）
  * `host [ip|hostname]` - 设置主机，也支持 `[host not]`
  * `[and|or]` - 设置逻辑
  * `port [1-65535]` - 设置端口号，也支持 `[port not]`

监控入口（接口上）流量，按 `<ip:port>` 过滤并写入文件：

```bash
tcpdump -ne -i eth0 -Q in host 192.168.252.10 and port 80 -c 5 -w dump.pcap
```

  * `-c [num]` - 仅捕获指定数量的数据包
  * `-w [filename]` - 将数据包写入文件，`-r [filename]` - 从文件读取

监控所有 HTTP GET 流量/请求：

```bash
tcpdump -i eth0 -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'
```

  * `tcp[((tcp[12:1] & 0xf0) >> 2):4]` - 首先确定我们感兴趣的字节位置（在 TCP 头之后），然后选择要匹配的 4 个字节
  * `0x47455420` - 代表字符 'G', 'E', 'T', ' ' 的 ASCII 值

监控所有 HTTP POST 流量/请求，按目标端口过滤：

```bash
tcpdump -i eth0 -s 0 -A -vv 'tcp dst port 80 and (tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354)'
```

  * `0x504F5354` - 代表字符 'P', 'O', 'S', 'T', ' ' 的 ASCII 值

监控包括请求和响应头以及消息体在内的 HTTP 流量，按端口过滤：

```bash
tcpdump -A -s 0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

监控包括请求和响应头以及消息体在内的 HTTP 流量，按 `<src-ip:port>` 过滤：

```bash
tcpdump -A -s 0 'src 192.168.252.10 and tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

###### 外部资源

- [TCPDump Capture HTTP GET/POST requests – Apache, Weblogic & Websphere](https://www.middlewareinventory.com/blog/tcpdump-capture-http-get-post-requests-apache-weblogic-websphere/)
- [Wireshark tcp filter: `tcp[((tcp[12:1] & 0xf0) >> 2):4]`](https://security.stackexchange.com/questions/121011/wireshark-tcp-filter-tcptcp121-0xf0-24)
- [How to Use tcpdump Commands with Examples](https://linoxide.com/linux-how-to/14-tcpdump-commands-capture-network-traffic-linux/)
- [Helpers - Debugging (from this handbook)](https://github.com/trimstray/nginx-admins-handbook/blob/master/doc/HELPERS.md#debugging)
