<a id="http-static-error-pages-generator"></a>
# HTTP 静态错误页面生成器

<a id="description"></a>
## 描述

一个易于使用的生成器，用于生成**静态错误页面**，以替代任何 Web 服务器（如 **Nginx** 或 **Apache**）自带的默认错误页面。

<a id="use-example"></a>
## 使用示例

以下是启动工具的示例：

```bash
./httpgen
```

命令执行结果位于 **sites/** 目录中。

<a id="error-examples"></a>
## 错误示例

<a id="404-not-found"></a>
### 404 Not Found
![alt text](doc/img/404_not_found.png)

<a id="503-service-unavailable"></a>
### 503 Service Unavailable
![alt text](doc/img/503_service_unavailable.png)

<a id="temporary-maintenance"></a>
### 临时维护
![alt text](doc/img/temporary_maintenance.png)

<a id="rate-limit"></a>
### 速率限制
![alt text](doc/img/rate_limit.png)

<a id="your-own-error-pages"></a>
## 自定义错误页面

如果你想添加自己的静态页面进行生成，请编辑 **src/other.json** 文件并添加：

```bash
  {
    "code" : "903",
    "title": "HTTP Error Code",
    "desc" : "This is a example http error code description.",
    "icon" : "fas fa-info-circle blue"
  }
```

说明：

- `code` - 指定响应状态码（例如 400, 404, 501）
- `title` - 指定状态码的简短标题，与 `code` 键关联（例如 "Not Found", "Bad Gateway"）
- `desc` - 确定错误可能的原因（例如 "The web server is currently undergoing some maintenance"）
- `icon` - 为错误码设置一个 font-awesome 小图标（例如 "fas fa-info-circle green", "fas fa-info-circle red"）
