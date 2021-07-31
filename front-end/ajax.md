# ajax
AJAX = Asynchronous JavaScript and XML（异步的 JavaScript 和 XML），意思就是用JavaScript执行异步网络请求。

AJAX 最大的优点是在不重新加载整个页面的情况下，可以与服务器交换数据并更新部分网页内容。

![ajax+20210718162023](https://raw.githubusercontent.com/loli0con/picgo/master/images/ajax%2B20210718162023.png%2B2021-07-18-16-20-23)

## XMLHttpRequest
XMLHttpRequest 是 AJAX 的基础，XMLHttpRequest 对象用于和服务器交换数据。

### 创建对象
```js
let xmlhttp=new XMLHttpRequest();
```

### 发送请求
如需将请求发送到服务器，我们使用 XMLHttpRequest 对象的 open() 和 send() 方法：
* open(method,url,async)：规定请求的类型、URL 以及是否异步处理请求
  * method：请求的类型；GET 或 POST
  * url：文件在服务器上的位置
  * async：true（异步）或 false（同步：浏览器将停止响应，直到AJAX请求完成——即等待xmlhttp.send()执行完成）
* send(string)：将请求发送到服务器
  * string：仅用于 POST 请求


如果需要像 HTML 表单那样 POST 数据，请使用 setRequestHeader() 来添加 HTTP 头，然后在 send() 方法中规定您希望发送的数据。
* setRequestHeader(header,value)：向请求添加 HTTP 头
  * header: 规定头的名称
  * value: 规定头的值

```js
// GET
xmlhttp.open("GET","/try/ajax/demo_get2.php?fname=Henry&lname=Ford",true);
xmlhttp.send();

// POST
xmlhttp.open("POST","/try/ajax/demo_post2.php",true);
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
xmlhttp.send("fname=Henry&lname=Ford");
```


### 设置回调
XMLHttpRequest 对象请求过程中的状态变更会赋值给 readyState 属性，同时触发 onreadystatechange 事件。

下面是 XMLHttpRequest 对象的三个重要的属性：
#### onreadystatechange
存储函数（或函数名），每当 readyState 属性改变时，就会调用该函数。
#### readyState
存有 XMLHttpRequest 的状态。  
从 0 到 4 发生变化：
* 0：UNSENT，XMLHttpRequest 代理已被创建， 但尚未调用 open() 方法。
* 1：OPENED，open() 方法已经被触发。在这个状态中，可以通过  setRequestHeader() 方法来设置请求的头部， 可以调用 send() 方法来发起请求。
* 2：HEADERS_RECEIVED，send() 方法已经被调用，响应头也已经被接收。
* 3：LOADING，响应体部分正在被接收。如果 responseType 属性是“text”或空字符串， responseText 将会在载入的过程中拥有部分响应数据。
* 4：DONE，请求操作已经完成。这意味着数据传输已经彻底完成或失败。

##### 状态示例
```js
var xhr = new XMLHttpRequest();
console.log('UNSENT', xhr.readyState); // readyState 为 0

xhr.open('GET', '/api', true);
console.log('OPENED', xhr.readyState); // readyState 为 1

xhr.onprogress = function () {
    console.log('LOADING', xhr.readyState); // readyState 为 3
};

xhr.onload = function () {
    console.log('DONE', xhr.readyState); // readyState 为 4
};

xhr.send(null);
```
#### status
HTTP 请求的返回的状态：
* 200: "OK"
* 404: 未找到页面

***
#### 使用回调函数
```js
var xhttp = new XMLHttpRequest();
xhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
    myFunction(this); // 使用回调函数
    }
};
xhttp.open("GET", "api", true);
xhttp.send();

// 创建回调函数
function myFunction(xhttp) {
    // do something
}
```

### 获得响应
如需获得来自服务器的响应，请使用 XMLHttpRequest 对象的 responseText（字符串形式的响应数据） 或 responseXML（XML 形式的响应数据） 属性。

## 安全限制
上面代码的URL使用的是相对路径。如果你把它改为 http://www.sina.com.cn/ ，再运行，肯定报错。在Chrome的控制台里，还可以看到错误信息。

这是因为浏览器的**同源策略**导致的。默认情况下，JavaScript在发送AJAX请求时，URL的域名必须和当前页面完全一致。

完全一致的意思是，域名要相同（ www.example.com 和 example.com 不同），协议要相同（http和https不同），端口号要相同（默认是:80端口，它和:8080就不同）。有的浏览器口子松一点，允许端口不同，大多数**浏览器都会严格遵守这个限制**。

### 绕过浏览器的安全限制
#### Flash
已过时
#### 代理服务器
通过在同源域名下架设一个代理服务器来转发，JavaScript负责把请求发送到代理服务器：`'/proxy?url=http://www.sina.com.cn'`。代理服务器再把结果返回，这样就遵守了浏览器的同源策略。这种方式麻烦之处在于需要服务器端额外做开发。
#### JSONP
JSONP，它有个限制，只能用GET请求，并且要求返回JavaScript。这种方式跨域实际上是利用了浏览器允许跨域引用JavaScript资源：
```html
<!-- 引用JavaScript资源 -->
<script src="http://example.com/abc.js"></script>
```
JSONP通常以函数调用的形式返回，例如，返回JavaScript内容如下：`callback('data');`。  
这样一来，我们如果在页面中先准备好foo()函数，然后给页面动态加一个\<script>节点，相当于动态读取外域的JavaScript资源，最后就等着接收回调了。

##### 例子🌰
```html
<!-- 第一步：事先准备好callback函数 -->
<script type="text/javascript">
function callback(info){
    doSomethingOnInfo(info); // 自定义的处理info数据的函数，下面是一个例子
    // doSomethingOnInfo(info){
    //     alert(info);
    // }
}
</script>

<!-- 第二步：给页面添加一个<script>节点 -->
<script type="text/javascript">
function request(url){
    document.script = document.createElement("script");
    document.script.setAttribute("type", "text/javascript");
    document.script.setAttribute("src", url);
    document.body.appendChild(document.script);
}

request("服务端的url地址");
</script>

<!-- 第三步：页面加入该节点后，调用callback函数 -->
<script src="http://example.com/abc.js">
// 服务端返回的内容：callback('hello world!');
// 这意味着页面生成了一个 <script>callback('hello world!');< /script>
// 即调用实现定义好的callback函数
</script>
```


## CORS
CORS全称Cross-Origin Resource Sharing（跨域资源共享），是HTML5规范定义的如何跨域访问资源。它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。

[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)  
[CORS-官方文档](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)



## JQuery
jquery是js的框架，可以提高异步开发效率。

### 三种常用的ajax方法
![ajax+20210725150147](https://raw.githubusercontent.com/loli0con/picgo/master/images/ajax%2B20210725150147.png%2B2021-07-25-15-01-48)

#### get
jq的get方法异步请求语法：`$.get(url,[data],[callback],[type])`
* url：设置服务器处理请求资源地址
* data：设置传递的请求数据，支持2种格式
  * 格式1：键值对字符串形式，eg:"key1=value1&key2=value2..."
  * 格式2：json对象格式，eg:{key1:value1,key2:value2,...}
* callback：回调函数，用于接收服务器返回的数据
  * 格式：`function(result){...}`，result就是服务器返回的数据(内部已经判断ajax状态为4，http通信状态为200)
* type：设置服务器返回的数据类型
  * 类型1：type=text,代表返回文本字符串
  * 类型2：type=html,代表返回html文本字符串
  * 类型3：type=script,代表返回javascript文本字符串
  * 类型4：type=xml,代表返回xml文本字符串
  * 类型5：type=json,代表返回json文本字符串
  * 如果不设置默认值：type为服务器响应的正文类型

#### post
jq的get方法异步请求语法：`$.post(url,[data],[callback],[type])`
* url：设置服务器处理请求资源地址
* data：设置传递的请求数据，支持2种格式
  * 格式1：键值对字符串形式，eg:"key1=value1&key2=value2..."
  * 格式2：json对象格式，eg:{key1:value1,key2:value2,...}
* callback：回调函数，用于接收服务器返回的数据
  * 格式：`function(result){...}`，result就是服务器返回的数据(内部已经判断ajax状态为4，http通信状态为200)
* type：设置服务器返回的数据类型
  * 类型1：type=text,代表返回文本字符串
  * 类型2：type=html,代表返回html文本字符串
  * 类型3：type=script,代表返回javascript文本字符串
  * 类型4：type=xml,代表返回xml文本字符串
  * 类型5：type=json,代表返回json文本字符串
  * 如果不设置默认值：type为服务器响应的正文类型

#### ajax
jq的get方法语法：`$.ajax(json对象)`或`$.ajax([settings])`  
json对象常用属性如下：
* url：服务器访问地址
* async：设置后台发送类型，是异步或同步发送。
  * 默认值：true，异步发送请求
* data：发送给服务器的数据的格式
  * 格式1：键=值
  * 格式2：{键:值} 里面可以有多个键值对
* method：发送请求的方式
  * 默认的请求方式，GET方式
* contentType：即将发送信息至服务器的内容编码类型(默认: "application/x-www-form-urlencoded; charset=UTF-8")
* dataType：服务器返回的数据类型（将服务器端返回的数据转换成指定类型）
  * "xml": 将服务器端返回的内容转换成xml格式
  * "text": 将服务器端返回的内容转换成普通文本格式
  * "html": 将服务器端返回的内容转换成普通文本格式，在插入DOM中时，如果包含JavaScript标签，则会尝试去执行。
  * "script": 尝试将返回值当作JavaScript去执行，然后再将服务器端返回的内容转换成普通文本格式
  * "json": 将服务器端返回的内容转换成相应的JavaScript对象
  * "jsonp": JSONP 格式使用 JSONP 形式调用函数时，如 "myurl?callback=?" jQuery 将自动替换 ? 为正确的函数名，以执行回调函数
* success	服务器响应成功以后调用的回调函数（回调函数的参数是服务器返回的数据）
* error	如果服务器出现异常调用这个函数
* timeout：设置请求超时时间（毫秒）
* beforeSend：发送请求前执行的函数（全局）
* complete：完成之后执行的回调函数（全局）
* accepts：通过请求头发送给服务器，告诉服务器当前客户端可接受的数据类型

##### demo
```js
$.ajax({
    url:"LoginServlet",
    method:"post",
    data:{username:name,password:pwd},
    success:function (result){
        //result返回值："true" 或 "false"
        if(result=="true"){
            alert(name+"登录成功了");
        }else{
            alert("登录失败");
        }
    },
    error:function (){
        alert("服务器忙，请明天再来");
    }
})
```