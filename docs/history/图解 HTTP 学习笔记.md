> 前几个月看了《图解 HTTP》，简单易懂。现在整理一下笔记。



##Part 1. HTTP 的网络基础

Q1. 各层协议对一次 HTTP 请求的封装

答：见 p10 图。

![IMG_7910](http://47.94.165.69:3031/upload/aaa/d79aed2d2283f.jpg)       

<br />

Q2. 各种协议与 HTTP 协议的关系。

答： 图 p15 了解一下，IP 协议（网络层）、TCP 协议（传输层）和 DNS 服务（应用层）在使用 HTTP 协议的通信过程中各自发挥了哪些作用。

![IMG_5925](http://47.94.165.69:3031/upload/aaa/bdd97d521d9aa.jpg)  

<br />

##Part 2. 简单的 HTTP 协议

Q3. HTTP 是不保存状态的协议。

答： HTTP 为无状态（stateless）协议，即**不具备保存之前发送过的请求或响应的功能**。

但为了实现保持状态的功能，（比如，对于用户登陆，网站要掌握是谁送出的跳转等请求，）于是引入了 **Cookie 技术**。

<br />

Q4. HTTP/1.1 可用的方法

- **GET：获取资源**

```
# 请求
GET /index.html HTTP/1.1
Host: www.hacker.jp
# --------------------------------------- #
# 响应
返回 index.html 的页面资源
```

```
# 请求
GET /index.html HTTP/1.1
Host: www.hacker.jp
If-Modified-Since: Wed, 15 Aug 2018 08:30:11 GMT
# --------------------------------------- #
# 响应
返回 Wed, 15 Aug 2018 08:30:11 后有更新的 index.html 的页面资源；
否则，返回 304 Not Modified 的响应
```

- **POST：传输实体主体**

- **PUT： 传输文件**

  请求报文主体中包含文件内容，保存到请求 URI 指定的位置。

  响应码 204 No Content，表示该文件已在服务器上。

  HTTP/1.1 的 PUT 方法不带验证机制，存在安全问题，一般 Web 网站会禁用；可配合 Web 应用验证机制，或 REST 标准会开放该方法。

- **DELETE：删除文件**

  删除请求 URI 指定的资源。

  响应码 204 No Content，表示该文件从服务器上删除。

  与 PUT 一样，无验证机制。

- **HEAD：获得报文首部**

  只返回 GET 响应报文的头部。

- **OPTIONS：询问支持的方法**

  ```
  # 请求
  OPTIONS * HTTP/1.1
  Host: www.hacker.jp
  # --------------------------------------- #
  # 响应
  HTTP/1.1 200 OK
  Allow: GET,POST,HEAD,OPTIONS
  (返回服务器支持的方法)
  ```

- **TRACE：追踪路径**

  Max-Forwards 填入数值，每经过一个服务器就减1，当减到0时，停止传输；最后接到请求的服务器返回 200 OK 的响应。

  容易引发XST（Cross-Site Tracing，跨站追踪）攻击，通常不用。

  ```
  # 请求
  TRACE / HTTP/1.1
  Host: www.hacker.jp
  Max-Forwards: 2
  # --------------------------------------- #
  # 响应
  HTTP/1.1 200 OK
  ....
  TRACE / HTTP/1.1
  Host: www.hacker.jp
  Max-Forwards: 2 (返回响应包含请求内容)
  ```

- **CONNECT：要求使用隧道协议连接代理**

  要求在与代理服务器通信时建立隧道，实现隧道协议的 TCP 通信。主要使用 SSL 和 TLS 协议给通信加密，经网络隧道传输。

  ```
  # 请求
  CONNECT proxy.hacker.jp:8080 HTTP/1.1
  Host:  proxy.hacker.jp (代理服务器)
  # --------------------------------------- #
  # 响应
  HTTP/1.1 200 OK
  ```

<br />

Q5. 持久连接

答： 持久连接，即只要任意一端没有明确提出断开连接，则保持 TCP 连接状态。

在 HTTP/1.1 中，所有连接默认是持久连接，但在 HTTP/1.0 内未标准化。

<br />

Q6. 使用 Cookie 的状态管理

答：Cookie 会根据从**服务器端**发送的响应报文内的一个叫 Set-Cookie 的首部字段信息，通知**客户端**保存 Cookie。当下次**客户端**再往该**服务器**发送请求时，**客户端**会自动在请求报文种加入 Cookie 只后发送出去。

**服务器端**发现客户端发过来的 Cookie 后，回去检查究竟是从哪个**客户端**发来的请求，然后对比服务器上的记录，最后得到之前的状态信息

1）请求报文（没有 Cookie 信息的状态）

```
GET /reader/ HTTP/1.1
Host: hacker.jp
```

2）响应报文（服务器端生产 Cookie 信息）

```
HTTP/1.1 200 OK
Date: xxx
Server: Apache
<Set-Cookie: sid= 1234321; path=/; expires=Wed, 15 Aug 2018 08:30:11 GMT>
```

3）请求报文（自动发送保存着的 Cookie 信息）

```
GET /image/ HTTP/1.1
Host: hacker.jp
Cookie: sid= 1234321
```

<br />

##Part 3. HTTP 响应状态码

Q7. 常用状态码

- 200 OK 
- 204 No Content
- 206 Partial Content
- 301 Moved Permanently.
- 302 Found
- 303 See Other
- 304 Not Modified
- 307 Temporary Redirect
- 400 Bad Request
- 401 Unauthorized
- 404 Not Found
- 500 Internal Server Error
- 503 Service Unavailble

<br />

## Part 4. 与 HTTP 协作的 Web 服务器

Q8. 通信数据转发程序：代理、网关、隧道

1）代理

代理服务器的行为就是接受客户端的请求，转发发给其他服务器，不改变请求URI。转发时，需要附加 Via 首部字段以标记经过的主机信息。

2）网关

网关行为类似于代理，但能接发服务器提供的非 HTTP 协议服务，提高通信的安全性。

3）隧道

隧道通信中，可使用 SSL 等加密手段，保证和远距离服务器的安全通信。隧道本身透明，客户端不用在意隧道的存在。

<br />

## Part 5. HTTP 首部

Q9. HTTP 报文的首部构成

![IMG_7914](http://47.94.165.69:3031/upload/aaa/1a822f4977368.jpg)  

![IMG_7914](http://47.94.165.69:3031/upload/aaa/281af9ab92d24.jpg) 

1）**HTTP/1.1 通用首部字段**

请求和响应报文都会使用的首部。

- Cache-Control

  指令的参数可选，多个指令间 “,” 分隔。

  ```
  Cache-Control: private, max-age=0, no-cache
  ```

  可分为**缓存请求指令**和**缓存响应指令**。

  ```
  ## 缓存请求指令
  指令                　　参数        　　说明
  no-cache               无             强制向源服务器再次验证
  no-store               无             不缓存请求或响应的任何内容
  max-age=[秒]           必需        　　 响应的最大Age值
  max-stale(= [秒])      可省略          接收已过期的响应
  min-fresh=[秒]         必需            期望在指定时间内的响应仍有效
  no-transform           无             代理不可更改媒体类型
  only-if-cached         无             从缓存获取资源
  cache-extension     　　-             新指令标记(token)
  ```

  ```
  ## 缓存响应指令
  指令                　　 参数        　　说明
  public                 无              可向任意方提供响应的缓存
  private                可省略          仅向特定用户返回响应
  no-cache             　省略            缓存前必须先确认其有效性
  no-store               无             不缓存请求或响应的任何内容
  no-transform           无             代理不可更改媒体类型
  must-revalidate        无             可缓存但必须再向源服务器进行确认
  proxy-revalidate       无             要求中间缓存服务器对缓存的响应有效性再进行确认
  max-age=[秒]           必需            响应的最大Age值
  s-maxage=[秒]          必需           公共缓存服务器响应的最大Age值
  cache-extension        -             新指令标记(token)
  ```

- Connection

  作用：1. 控制不再转发给代理的首部字段；2. 管理持久连接，HTTP/1.1 默认 Keep-alive。

- Via

  追踪客户端与服务器之间请求和响应报文的传输路径，还可避免回环的发生。

2）**请求首部字段**

- Authorization

  告知服务器，用户代理的认证信息（证书值）。若认证失败，返回 401 。

- Expect

  告知服务器，期望出现的某种特定行为。若服务器无法理解客户端的期望，则返回 417 Expectation Failed。

  ```
  Expect: 100-continue
  # 客户端在等待状态吗100的响应
  ```

- If-xxx

  条件请求，服务器接受到条件请求，只有判断为真时，才执行请求。

  ```javascript
  If-Match: "123456"
  # 只有当 ETag（实体标记）为“123456”时，服务器才处理请求
  # 否则返回响应 412 Precondition Failed
  ```

  ```
  If-Modified-Since: 某时间
  # 若 Last-Modified 在某时间之后，则返回资源；
  # 否则返回 304 Not Modified
  ```

3）**响应首部字段**

- Age 

  资源在多久前创建，单位为秒。

- ETag

  服务器端分配的资源唯一标志，当资源更新时，ETag 值也需要更新。

- Loaction

  可将响应接受方引导至某个与请求URI位置不同的资源。配合 3xx： Redirection 的响应。

- WWW-Authenticate

  告知客户端适用于访问请求URI所指定的认证方案（Basic 或 Digest）和带参数提示的质询（challenge）。

  401 Unauthorized 响应中，肯定带有 WWW-Authenticate 字段。

  ```
  WWW-Authenticate: Basic ream="USagidesign Auth"
  ```

4）**实体首部字段**

- Allow

  通知客户端能够请求的 URI 资源的 HTTP 方法。若不支持，则返回 405 Method Not Allowed。

- Content-Range 

- Content-Length

- Content-Type

- Expires

- Last-Modified

5）**为 Cookie 服务的首部字段**

- Set-Cookie

  响应首部字段，开始管理客户端状态时，所使用的 Cookie 信息。

- Cookie

  请求首部字段。

<br />

## Part 6. 确保 Web 安全的 HTTPS

Q10. 什么是 HTTPS？

答：HTTP + 加密 + 认证 + 完整性保护 = HTTPS

HTTP 在通信接口部分用 SSL（Secure Socket Layer）和 TLS（Transport Layer Security）代替，便成了 HTTPS。

![IMG_7918](http://47.94.165.69:3031/upload/aaa/653177002ba5b.jpg)  

<br />

## Part 7. 访问者身份认证

Q11. HTTP 使用的认证方式

答：1）BASIC 认证（基本认证）

​	2）DIGEST 认证（摘要认证）

​	3）SSL 客户端认证

​	4）FormBase 认证（基于表单认证）

- **BASIC 认证**

![IMG_7922](http://47.94.165.69:3031/upload/aaa/fdd02adef686.jpg)  

- **DIGEST 认证**

![IMG_7924](http://47.94.165.69:3031/upload/aaa/577d54442595f.jpg)  

- **SSL 客户端认证**

![IMG_7926](http://47.94.165.69:3031/upload/aaa/0ac5123a4942f.jpg) 

- **FormBase 认证**

  基于表单认证标准尚无定论，一般用 Cookie 来管理 Session。

![IMG_7928](http://47.94.165.69:3031/upload/aaa/0ac5123a4942f.jpg) 

<br />