<a id="configuration-examples"></a>
# 配置示例

返回 **[⬆ 目录](https://github.com/trimstray/nginx-admins-handbook#table-of-contents)** 或 **[⬆ 下一步](https://github.com/trimstray/nginx-admins-handbook#whats-next)** 章节。

- **[≡ 配置示例](#examples)**
  * [反向代理](#reverse-proxy)
    * [安装](#installation)
    * [配置](#configuration)
    * [导入配置](#import-configuration)
    * [设置绑定 IP 地址](#set-bind-ip-address)
    * [设置域名](#set-your-domain-name)
    * [重新生成私钥和证书](#regenerate-private-keys-and-certs)
    * [更新模块列表](#update-modules-list)
    * [生成必要的错误页面](#generating-the-necessary-error-pages)
    * [添加新域名](#add-new-domain)
    * [测试你的配置](#test-your-configuration)

  > 请记得备份当前配置以及所有文件/目录。

本章仍在编写中。

<a id="installation"></a>
## 安装

我使用了本手册中的逐步教程：[从源码安装](HELPERS.md#installing-from-source)。

<a id="configuration"></a>
## 配置

我使用了 Google Cloud 实例，参数如下：

| <b>项目</b> | <b>值</b> | <b>备注</b> |
| :---         | :---         | :---         |
| VM | Google Cloud Platform | |
| vCPU | 2x | |
| 内存 | 4096MB | |
| HTTP | Varnish 端口 80 | |
| HTTPS | NGINX 端口 443 | |

<a id="reverse-proxy"></a>
## 反向代理

本章描述了我的代理服务器的基本配置（针对 [blkcipher.info](https://blkcipher.info) 域名）。

  > 配置基于[从源码安装](HELPERS.md#installing-from-source)章节。如果你逐步完成安装过程，可以使用以下配置（可能需要进行小幅调整）。

<a id="import-configuration"></a>
#### 导入配置

非常简单——克隆仓库、备份当前配置并执行完整的目录同步：

```bash
git clone https://github.com/trimstray/nginx-admins-handbook

tar czvfp ~/nginx.etc.tgz /etc/nginx && mv /etc/nginx /etc/nginx.old

rsync -avur lib/nginx/ /etc/nginx/
```

  > 如果你从源码编译 NGINX，还应更新/刷新模块。所有编译好的模块存储在 `/usr/local/src/nginx-${ngx_version}/master/objs`，并根据 `--modules-path` 变量的值进行安装。

<a id="set-bind-ip-address"></a>
#### 设置绑定 IP 地址

<a id="find-and-replace-1921682522-string-in-directory-and-file-names"></a>
###### 查找并替换目录和文件名中的 192.168.252.2 字符串

```bash
cd /etc/nginx
find . -depth -not -path '*/\.git*' -name '*192.168.252.2*' -execdir bash -c 'mv -v "$1" "${1//192.168.252.2/xxx.xxx.xxx.xxx}"' _ {} \;
```

<a id="find-and-replace-1921682522-string-in-configuration-files"></a>
###### 查找并替换配置文件中的 192.168.252.2 字符串

```bash
cd /etc/nginx
find . -not -path '*/\.git*' -type f -print0 | xargs -0 sed -i 's/192.168.252.2/xxx.xxx.xxx.xxx/g'
```

<a id="set-your-domain-name"></a>
#### 设置域名

<a id="find-and-replace-blkcipherinfo-string-in-directory-and-file-names"></a>
###### 查找并替换目录和文件名中的 blkcipher.info 字符串

```bash
cd /etc/nginx
find . -not -path '*/\.git*' -depth -name '*blkcipher.info*' -execdir bash -c 'mv -v "$1" "${1//blkcipher.info/example.com}"' _ {} \;
```

<a id="find-and-replace-blkcipherinfo-string-in-configuration-files"></a>
###### 查找并替换配置文件中的 blkcipher.info 字符串

```bash
cd /etc/nginx
find . -not -path '*/\.git*' -type f -print0 | xargs -0 sed -i 's/blkcipher_info/example_com/g'
find . -not -path '*/\.git*' -type f -print0 | xargs -0 sed -i 's/blkcipher.info/example.com/g'
```

<a id="regenerate-private-keys-and-certs"></a>
#### 重新生成私钥和证书

<a id="for-localhost"></a>
###### 针对 localhost

```bash
cd /etc/nginx/master/_server/localhost/certs

# Private key + Self-signed certificate:
( _fd="localhost.key" ; _fd_crt="nginx_localhost_bundle.crt" ; \
openssl req -x509 -newkey rsa:2048 -keyout ${_fd} -out ${_fd_crt} -days 365 -nodes \
-subj "/C=X0/ST=localhost/L=localhost/O=localhost/OU=X00/CN=localhost" )
```

<a id="for-default_server"></a>
###### 针对 `default_server`

```bash
cd /etc/nginx/master/_server/defaults/certs

# Private key + Self-signed certificate:
( _fd="defaults.key" ; _fd_crt="nginx_defaults_bundle.crt" ; \
openssl req -x509 -newkey rsa:2048 -keyout ${_fd} -out ${_fd_crt} -days 365 -nodes \
-subj "/C=X1/ST=default/L=default/O=default/OU=X11/CN=default_server" )
```

<a id="for-your-domain"></a>
###### 针对你的域名（例如 Let's Encrypt）

```bash
cd /etc/nginx/master/_server/example.com/certs

# For multidomain:
certbot certonly -d example.com -d www.example.com --rsa-key-size 2048

# For wildcard:
certbot certonly --manual --preferred-challenges=dns -d example.com -d *.example.com --rsa-key-size 2048

# Copy private key and chain:
cp /etc/letsencrypt/live/example.com/fullchain.pem nginx_example.com_bundle.crt
cp /etc/letsencrypt/live/example.com/privkey.pem example.com.key
```

<a id="update-modules-list"></a>
#### 更新模块列表

更新模块列表并将 `modules.conf` 引入到你的配置中：

```bash
_mod_dir="/etc/nginx/modules"

:>"${_mod_dir}.conf"

for _module in $(ls "${_mod_dir}/") ; do echo -en "load_module\t\t${_mod_dir}/$_module;\n" >> "${_mod_dir}.conf" ; done
```

<a id="generating-the-necessary-error-pages"></a>
#### 生成必要的错误页面

  > 在示例（`lib/nginx`）中，错误页面从 `lib/nginx/master/_static/errors.conf` 文件引入。

- 默认位置：`/etc/nginx/html`：
  ```
  50x.html  index.html
  ```
- 自定义位置：`/usr/share/www`：
  ```bash
  cd /etc/nginx/snippets/http-error-pages

  ./httpgen

  # 你也可以将 sites/ 目录同步到 /etc/nginx/html：
  #   rsync -var sites/ /etc/nginx/html/
  rsync -var sites/ /usr/share/www/
  ```

<a id="add-new-domain"></a>
#### 添加新域名

<a id="updated-nginxconf"></a>
###### 更新 `nginx.conf`

```nginx
# At the end of the file (in 'IPS/DOMAINS' section):
include /etc/nginx/master/_server/domain.com/servers.conf;
include /etc/nginx/master/_server/domain.com/backends.conf;
```

<a id="init-domain-directory"></a>
###### 初始化域名目录

```bash
cd /etc/nginx/master/_server
cp -R example.com domain.com

cd domain.com
find . -not -path '*/\.git*' -depth -name '*example.com*' -execdir bash -c 'mv -v "$1" "${1//example.com/domain.com}"' _ {} \;
find . -not -path '*/\.git*' -type f -print0 | xargs -0 sed -i 's/example_com/domain_com/g'
find . -not -path '*/\.git*' -type f -print0 | xargs -0 sed -i 's/example.com/domain.com/g'
```

<a id="create-log-directories"></a>
#### 创建日志目录

```bash
mkdir -p /var/log/nginx/localhost
mkdir -p /var/log/nginx/defaults
mkdir -p /var/log/nginx/others
mkdir -p /var/log/nginx/domains/blkcipher.info

chown -R nginx:nginx /var/log/nginx
```

<a id="logrotate-configuration"></a>
#### Logrotate 配置

```bash
cp /etc/nginx/snippets/logrotate.d/nginx /etc/logrotate.d/
```

<a id="test-your-configuration"></a>
#### 测试你的配置

```bash
nginx -t -c /etc/nginx/nginx.conf
```
