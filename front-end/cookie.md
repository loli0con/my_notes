# cookie
HTTP Cookie（也叫 Web Cookie 或浏览器 Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。

通常，它用于告知服务端两个请求是否来自同一浏览器。Web服务器之后可以利用这些信息来标识用户。Cookie 使基于无状态的HTTP协议记录稳定的状态信息成为了可能。


## 创建cookie
当服务器收到 HTTP 请求时，服务器可以在响应头里面添加一个 Set-Cookie 选项。浏览器收到响应后通常会保存下 Cookie，之后对该服务器每一次请求中都通过 Cookie 请求头部将 Cookie 信息发送给服务器。另外，Cookie 的过期时间、域、路径、有效期、适用站点都可以根据需要来指定。

### 示例
服务器使用 Set-Cookie 响应头部向用户代理（一般是浏览器）发送 Cookie信息。  
一个简单的 Cookie 可能像这样：`Set-Cookie: <cookie名>=<cookie值>`
一个完整的 Cookie （带有选项：任意数量的选项都可以在单一的cookie中指定，并且这些选项可以以任何顺序存在）：`Set-Cookie: value[; expires=date][; domain=domain][; path=path][; secure]`

服务器通过该头部告知客户端保存 Cookie 信息。
```http
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry

[页面内容]
```

现在，对该服务器发起的每一次新请求，浏览器都会将之前保存的Cookie信息通过 Cookie 请求头部再发送给服务器（无选项）。
```http
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
```


## 生命周期
Cookie 的生命周期可以通过两种方式定义：
* 会话期 Cookie 是最简单的 Cookie：浏览器关闭之后它会被自动删除，也就是说它仅在会话期内有效。会话期Cookie不需要指定过期时间（Expires）或者有效期（Max-Age）。需要注意的是，有些浏览器提供了会话恢复功能，这种情况下即使关闭了浏览器，会话期Cookie 也会被保留下来，就好像浏览器从来没有关闭一样，这会导致 Cookie 的生命周期无限期延长。
* 持久性 Cookie 的生命周期取决于过期时间（Expires）或有效期（Max-Age）指定的一段时间。

### 示例
```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;
```


## 限制访问
有两种方法可以确保 Cookie 被安全发送，并且不会被意外的参与者或脚本访问：Secure 属性和 HttpOnly 属性：
* 标记为 Secure 的 Cookie 只应通过被 HTTPS 协议加密过的请求发送给服务端
* JavaScript Document.cookie API 无法访问带有 HttpOnly 属性的cookie

### 示例
```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
```


## 作用域
Domain 和 Path 标识定义了Cookie的作用域：即允许 Cookie 应该发送给哪些URL。
### Domain 属性
Domain 指定了哪些主机可以接受 Cookie。如果不指定，默认为 origin，不包含子域名。如果指定了Domain，则一般包含子域名。

例如，如果设置 Domain=mozilla.org，则 Cookie 也包含在子域名中（如developer.mozilla.org）。

domain设置的值必须是发送Set-Cookie消息头的域名。
### Path 属性
Path 标识指定了主机下的哪些路径可以接受 Cookie（该 URL 路径必须存在于请求 URL 中）。以字符 %x2F ("/") 作为路径分隔符，子路径也会被匹配。

例如，设置 Path=/docs，则以下地址都会匹配：
* /docs
* /docs/Web/
* /docs/Web/HTTP


## SameSite 属性
[SameSite 属性](https://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html) 允许服务器要求某个 cookie 在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击。

SameSite 可以有下面三种值：
* None。浏览器会在同站请求、跨站请求下继续发送 cookies，不区分大小写。
* Strict。浏览器将只在访问相同站点时发送 cookie。
* Lax。与 Strict 类似，但用户从外部站点导航至URL时（例如通过链接）除外。 在新版本浏览器中，为默认选项，Same-site cookies 将会为一些跨站子请求保留，如图片加载或者 frames 的调用，但只有当用户从外部站点导航到URL时才会发送。

### 示例
```
Set-Cookie: key=value; SameSite=Strict
```

## 修改cookie
假定存在一个cookie，这个cooke有四个标识符：cookie的name、domain、path、secure标记。
```
Set-Cooke:name=Greg; domain=nczonline.net; path=/blog
```

要想在将来改变这个cookie的值，需要发送另一个具有相同cookie的name、domain、path的Set-Cookie消息头，这将以一个新的值来覆盖原来cookie的值。

然而，仅仅只是改变这些选项的某一个也会创建一个完全不同的cookie，例如：
```
Set-Cookie:name=Nicholas; domain=nczonline.net; path=/
```

在返回这个消息头后，会存在两个同时拥有“name”的不同的cookie。如果你访问在www.nczonline.net/blog下的一个页面，以下的消息头将被包含进来：
```
Cookie: name=Greg; name=Nicholas
```
在这个消息头中存在了两个名为“name”的cookie，path值越详细则cookie越靠前，domain-path越详细则cookie字符串越靠前。

## JavaScript 通过 Document.cookie 访问 Cookie
通过 Document.cookie 属性可创建新的 Cookie，也可通过该属性访问非HttpOnly标记的Cookie。
```js
document.cookie = "yummy_cookie=choco";
document.cookie = "tasty_cookie=strawberry";
console.log(document.cookie);
// logs "yummy_cookie=choco; tasty_cookie=strawberry"
```

## 参考
* https://www.cnblogs.com/ajianbeyourself/p/4900140.html
* https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies
* https://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html