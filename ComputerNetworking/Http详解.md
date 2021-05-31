# Http详解

[TOC]

## 组成

请求报文：请求行、请求头、空行、请求体

响应报文：状态行、响应头、空行、响应体



## 概念

- HTTP协议是一种应用层协议
  - HTTP是HyperText Transfer Protocol(超文本传输协议)的英文缩写。HTTP可以通过传输层的TCP协议在客户端和服务器之间传输数据。HTTP协议主要用于Web浏览器和 Web服务器之间的数据交换。我们在使用IE或Firefox浏览网页或下载Web资源时，通过在地址栏中输入，开头的4个字母http就相当于通知浏览 器使用HTTP协议来和host所确定的服务器进行通讯。
- http连接的特点（五大特点）
  - **无连接**：客户端每次发送的请求，都需要服务器响应，请求结束后，会主动释放连接。从建立连接到关闭连接的过程，成为”一次连接”。
  - **无状态**：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它**必须重传**，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。
  - 支持客户/服务器模式
  - 简单快速：客户向服务器请求服务时，只需传送请求方法和路径。每种方法规定了客户与服务器联系的类型不同。
  - 灵活：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记。





## 方法

- Get  （获取资源）

- Head （和Get方法类似，但是不返回报文实体主体部分，主要用于确认URL的有效性以及资源更新的日期时间等）

- Post （传输实体主体）

- Put （上传文件，存在安全性问题）

- Patch （对资源进行部分修改，put也可以用于修改资源，但是只能完全替代原始资源，patch允许部分修改）

- Delete （与put功能相反，同样不带验证机制，存在安全性问题）

- Options （查询指定的URL能够支持的方法，会返回 `Allow: GET, POST, HEAD, OPTIONS` 这样的内容）

- Connect （使用 SSL（Secure Sockets Layer，安全套接层）和 TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输）

- Trace （服务器会将通信路径返回给客户端，容易受到 XST 攻击（Cross-Site Tracing，跨站追踪））



## Get和Post比较

GET 用于获取资源，而 POST 用于传输实体主体

|          |                                                             |                                                        |
| -------- | :---------------------------------------------------------: | :----------------------------------------------------: |
|          |                             GET                             |                          POST                          |
| 可见性   |                   数据在URL中对所有人可见                   |                  数据不会显示在URL中                   |
| 安全性   | 与post相比，get的安全性较差，因为所 发送的数据是URL的一部分 | 安全，因为参数不会被保存在浏览器 历史或web服务器日志中 |
| 数据长度 |                       受限制，最长2kb                       |                         无限制                         |
| 编码类型 |              application/x-www-form-urlencoded              |                  multipart/form-data                   |
| 缓存     |                          能被缓存                           |                       不能被缓存                       |

### 参数

GET 和 POST 的请求都能使用额外的参数，但是 GET 的参数是以查询字符串出现在 **URL 中**，而 POST 的参数存储在**实体主体中**。不能因为 POST 参数存储在实体主体中就认为它的安全性更高，因为照样可以通过一些抓包工具（Fiddler）查看。

因为 URL 只支持 ASCII 码，因此 GET 的参数中如果存在中文等字符就需要先进行编码。例如 `中文` 会转换为 `%E4%B8%AD%E6%96%87`，而空格会转换为 `%20`。POST 参数支持标准字符集。

~~~markdown
GET /test/demo_form.asp?name1=value1&name2=value2 HTTP/1.1

POST /test/demo_form.asp HTTP/1.1
Host: w3schools.com
name1=value1&name2=value2
~~~



### 安全

Get是不安全的，因为在传输过程，数据被放在请求的URL中；

Post的所有操作对用户来说都是不可见的。 但是这种做法也不时绝对的，大部分人的做法也是按照上面的说法来的，但是也可以在get请求加上 request body，给 post请求带上 URL 参数。



### 幂等性

幂等的 HTTP 方法，同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。换句话说就是，幂等方法不应该具有副作用（统计用途除外）。

所有的安全方法也都是幂等的。

在正确实现的条件下，GET，HEAD，PUT 和 DELETE 等方法都是幂等的，而 POST 方法不是。

GET /pageX HTTP/1.1 是幂等的，连续调用多次，客户端接收到的结果都是一样的：

~~~markdown
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
~~~

POST /add_row HTTP/1.1 不是幂等的，如果调用多次，就会增加多行记录：

~~~markdown
POST /add_row HTTP/1.1   -> Adds a 1nd row
POST /add_row HTTP/1.1   -> Adds a 2nd row
POST /add_row HTTP/1.1   -> Adds a 3rd row
~~~



### 可缓存

如果要对响应进行缓存，需要满足以下条件：

- 请求报文的 HTTP 方法本身是可缓存的，包括 GET 和 HEAD，但是 PUT 和 DELETE 不可缓存，POST 在多数情况下不可缓存的。
- 响应报文的状态码是可缓存的，包括：200, 203, 204, 206, 300, 301, 404, 405, 410, 414, and 501。
- 响应报文的 Cache-Control 首部字段没有指定不进行缓存。

- 在使用 XMLHttpRequest 的 POST 方法时，浏览器会先发送 Header 再发送 Data。但并不是所有浏览器会这么做，例如火狐就不会。
- 而 GET 方法 Header 和 Data 会一起发送



## Http状态码

| 状态码 |               类别               |            含义            |
| :----: | :------------------------------: | :------------------------: |
|  1XX   |  Informational（信息性状态码）   |     接收的请求正在处理     |
|  2XX   |      Success（成功状态码）       |      请求正常处理完毕      |
|  3XX   |   Redirection（重定向状态码）    | 需要进行附加操作以完成请求 |
|  4XX   | Client Error（客户端错误状态码） |     服务器无法处理请求     |
|  5XX   | Server Error（服务器错误状态码） |     服务器处理请求出错     |



### 1XX 信息

- **100 Continue** ：表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应。

- **101 Switching Protocal**:  针对请求头的Upgrade返回的信息。表明服务器正在切换到指定的协议。

- **103 Early Hints**:  此状态代码主要用于与Link链接头一起使用，以允许用户代理在服务器仍在准备响应时开始预加载资源



### 2XX 成功

- **200 OK**： 请求成功

- **201 Created**:  常用于POST，PUT 请求，表明请求已经成功，并新建了一个资源。并在响应体中返回路径。

- **202 Accepted**:  请求已经接收到，但没有响应，稍后也不会返回一个异步请求结果。 该状态码适用于等待其他进程处理或者批处理的场景。

- **203 No-Authoritative Information**:  表明响应返回的元信息（meta-infomation）和最初的服务器不同，而是从本地或者第三方获取的

- **204 No Content** ：请求已经成功处理，但是返回的响应报文不包含实体的主体部分。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用。

- **205 Reset Content**:  告诉用户代理（浏览器）重置发送该请求的文档。

- **206 Partial Content** ：表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容。



### 3XX重定向

- **301 Moved Permanently** ：永久性重定向

- **302 Found** ：暂时重定向

- **303 See Other** ：和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源。

- 注：虽然 HTTP 协议规定 301、302 状态下重定向时不允许把 POST 方法改成 GET 方法，但是大多数浏览器都会在 301、302 和 303 状态下的重定向把 POST 方法改成 GET 方法。

- **304 Not Modified** ：如果请求报文首部包含一些条件，例如：If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码。

- **307 Temporary Redirect** ：临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法。



### 4XX客户端错误

- **400 Bad Request** ：客户端请求的语法错误，服务器无法理解。

- **401 Unauthorized** ：该状态码表示发送的请求需要有认证信息（BASIC 认证、DIGEST 认证）。如果之前已进行过一次请求，则表示用户认证失败。

- **403 Forbidden** ： 服务器理解请求客户端的请求，但是拒绝执行此请求。

- **404 Not Found**：服务器无法根据客户端的请求找到资源（网页）。



### 5XX服务器错误

- **500 Internal Server Error** ：服务器内部错误，无法完成请求。

- **502：Bad Gateway**：  作为网关或者代理服务器尝试执行请求时，从远程服务器接收到了无效的响应。

- **503 Service Unavailable** ：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。



