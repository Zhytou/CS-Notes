# HTTP

- [HTTP](#http)
  - [方法](#方法)
    - [GET](#get)
    - [HEAD](#head)
    - [POST](#post)
    - [PUT](#put)
    - [PATCH](#patch)
    - [DELETE](#delete)
    - [OPTIONS](#options)
    - [CONNECT](#connect)
    - [TRACE](#trace)
  - [状态码](#状态码)
    - [1XX 信息](#1xx-信息)
    - [2XX 成功](#2xx-成功)
    - [3XX 重定向](#3xx-重定向)
    - [4XX 客户端错误](#4xx-客户端错误)
    - [5XX 服务器错误](#5xx-服务器错误)
  - [HTTP1.0 HTTP1.x HTTP2.0](#http10-http1x-http20)
    - [HTTP1.0](#http10)
      - [缺陷](#缺陷)
      - [解决方案](#解决方案)
    - [HTTP1.x](#http1x)
    - [HTTP2.0](#http20)
  - [GET 和 POST 的比较](#get-和-post-的比较)
    - [作用](#作用)
    - [参数](#参数)
    - [幂等性](#幂等性)

## 方法

客户端发送的**请求报文**第一行为请求行，包含了方法字段。

### GET

> 获取资源

当前网络请求中，绝大部分使用的是 GET 方法。

### HEAD

> 获取报文首部

和 GET 方法类似，但是不返回报文实体主体部分。

主要用于确认 URL 的有效性以及资源更新的日期时间等。

### POST

> 传输实体主体

POST 主要用来传输数据，而 GET 主要用来获取资源。

更多 POST 与 GET 的比较请见第九章。

### PUT

> 上传文件

由于自身不带验证机制，任何人都可以上传文件，因此存在安全性问题，一般不使用该方法。

```html
PUT /new.html HTTP/1.1
Host: example.com
Content-type: text/html
Content-length: 16

<p>New File</p>
```

### PATCH

> 对资源进行部分修改

PUT 也可以用于修改资源，但是只能完全替代原始资源，PATCH 允许部分修改。

```html
PATCH /file.txt HTTP/1.1
Host: www.example.com
Content-Type: application/example
If-Match: "e0023aa4e"
Content-Length: 100

[description of changes]
```

### DELETE

> 删除文件

与 PUT 功能相反，并且同样不带验证机制。

```html
DELETE /file.html HTTP/1.1
```

### OPTIONS

> 查询支持的方法

查询指定的 URL 能够支持的方法。

会返回 `Allow: GET, POST, HEAD, OPTIONS` 这样的内容。

### CONNECT

> 要求在与代理服务器通信时建立隧道

使用 SSL（Secure Sockets Layer，安全套接层）和 TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输。

```html
CONNECT www.example.com:443 HTTP/1.1
```

![http](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/dc00f70e-c5c8-4d20-baf1-2d70014a97e3.jpg)

### TRACE

> 追踪路径

服务器会将通信路径返回给客户端。

发送请求时，在 Max-Forwards 首部字段中填入数值，每经过一个服务器就会减 1，当数值为 0 时就停止传输。

通常不会使用 TRACE，并且它容易受到 XST 攻击（Cross-Site Tracing，跨站追踪）。

## 状态码

### 1XX 信息

- **100 Continue**  ：表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应。

### 2XX 成功

- **200 OK**  

- **204 No Content**  ：请求已经成功处理，但是返回的响应报文不包含实体的主体部分。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用。

- **206 Partial Content**  ：表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容。

### 3XX 重定向

- **301 Moved Permanently**  ：永久性重定向

- **302 Found**  ：临时性重定向

- **303 See Other**  ：和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源。

- 注：虽然 HTTP 协议规定 301、302 状态下重定向时不允许把 POST 方法改成 GET 方法，但是大多数浏览器都会在 301、302 和 303 状态下的重定向把 POST 方法改成 GET 方法。

- **304 Not Modified**  ：如果请求报文首部包含一些条件，例如：If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码。

- **307 Temporary Redirect**  ：临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法。

### 4XX 客户端错误

- **400 Bad Request**  ：请求报文中存在语法错误。

- **401 Unauthorized**  ：该状态码表示发送的请求需要有认证信息（BASIC 认证、DIGEST 认证）。如果之前已进行过一次请求，则表示用户认证失败。

- **403 Forbidden**  ：请求被拒绝。

- **404 Not Found**  

### 5XX 服务器错误

- **500 Internal Server Error**  ：服务器正在执行请求时发生错误。

- **503 Service Unavailable**  ：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

## HTTP1.0 HTTP1.x HTTP2.0

### HTTP1.0

#### 缺陷

- 浏览器与服务器只保持短暂的连接

- 每次请求都需要与服务器建立一个TCP连接
  - TCP连接的新建成本很高，因为需要客户端和服务器三次握手

#### 解决方案

- 添加头信息——非标准的Connection字段
  - 当Connection为keep-alive时，网络连接就是持久的，不会关

### HTTP1.x

**改进**：

- 持久连接：引入了持久连接，即TCP连接默认不关闭，可以被多个请求复用，不用声明`Connection: keep-alive`
- 管道机制：即在同一个TCP连接里面，客户端可以同时发送多个请求
- 分块传输编码：即服务端每产生一块数据，就发送一块，采用”流模式”而取代”缓存模式”

**缺陷**：

- 虽然允许复用TCP连接，但是同一个TCP连接里面，所有的数据通信是按次序进行的。
  - 服务器只有处理完一个请求，才会接着处理下一个请求
  - 如果前面的处理特别慢，后面就会有许多请求排队等着

### HTTP2.0

**改进**：

- 采用二进制格式而非文本格式；
- 完全多路复用，而非有序并阻塞的、只需一个连接即可实现并行；
  - 路复用允许同时通过单一的http/2 连接发起多重的请求-响应消息。有了新的分帧机制后，http/2 不再依赖多个TCP连接去实现多流并行了。
  - 每个数据流都拆分成很多互不依赖的帧，而这些帧可以交错（乱序发送），还可以分优先级，最后再在另一端把它们重新组合起来

- 使用报头压缩，降低开销
  - 一方面，头信息压缩后再发送；
  - 另一方面，客户端和服务器同时维护一张头信息表，所有字段都会存入这个表，产生一个索引号，之后就不发送同样字段了，只需发送索引号
- 服务器推送

## GET 和 POST 的比较

### 作用

GET 用于获取资源，而 POST 用于传输实体主体。

### 参数

GET 和 POST 的请求都能使用额外的参数，但是 GET 的参数是以查询字符串出现在 URL 中，而 POST 的参数存储在实体主体中。不能因为 POST 参数存储在实体主体中就认为它的安全性更高，因为照样可以通过一些抓包工具（Fiddler）查看。

### 幂等性

GET，HEAD，PUT 和 DELETE 等方法都是幂等的，而 POST 方法不是。
