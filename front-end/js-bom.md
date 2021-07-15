# BOM
浏览器对象模型 (BOM, Browser Object Mode) 使 JavaScript 有能力与浏览器"对话"——取浏览器提供的很多对象，并进行操作。

## window对象
所有浏览器都支持 window 对象，它表示浏览器窗口，也充当全局作用域。

所有 JavaScript 全局对象、函数以及变量均自动成为 window 对象的成员：
* 全局变量是 window 对象的属性。
* 全局函数是 window 对象的方法。

HTML DOM 的 document 也是 window 对象的属性之一。

## window尺寸
window对象有innerWidth和innerHeight属性，可以获取浏览器窗口的内部宽度和高度（包括滚动条）。  
内部宽高是指除去菜单栏、工具栏、边框等占位元素后，用于显示网页的净宽高。

对应的，还有一个outerWidth和outerHeight属性，可以获取浏览器窗口的整个宽高。

## window方法
* window.open() - 打开新窗口
* window.close() - 关闭当前窗口
* window.moveTo() - 移动当前窗口
* window.resizeTo() - 调整当前窗口的尺寸

## navigator
navigator对象表示浏览器的信息（不一定正确），最常用的属性包括：
* 浏览器代号: navigator.appCodeName
* 浏览器名称: navigator.appName
* 浏览器版本: navigator.appVersion
* 启用Cookies: navigator.cookieEnabled
* 硬件平台: navigator.platform
* 用户代理: navigator.userAgent
* 用户代理语言: navigator.language

## screen
screen对象表示屏幕的信息，常用的属性有：
* screen.width：屏幕宽度，以像素为单位
* screen.height：屏幕高度，以像素为单位
* screen.colorDepth：返回颜色位数，如8、16、24
* screen.availWidth：可用的屏幕宽度
* screen.availHeight：可用的屏幕高度

## location
location 对象用于获得当前页面的地址 (URL)，并把浏览器重定向到新的页面
```js
// http://www.example.com:8080/path/index.html?a=1&b=2#TOP

location.protocol; // 'http'
location.host; // 'www.example.com'
location.port; // '8080'
location.pathname; // '/path/index.html'
location.search; // '?a=1&b=2'
location.hash; // 'TOP'
```

要加载一个新页面，可以调用location.assign()方法，传入url参数。   
如果要重新加载当前页面，调用location.reload()方法。

## history
history对象保存了浏览器的历史记录，JavaScript可以调用history对象的back()或forward ()，相当于用户点击了浏览器的“后退”或“前进”按钮。

除此之外可以用 history.go() 这个方法来实现向前，后退的功能。该方法传入一个整数：向前(正数)/向后(负数)/刷新(零)。

## 弹窗
可以在 JavaScript 中创建三种消息框：警告框、确认框、提示框。

### 警告框
alert("sometext")

### 确认框
confirm("sometext")，返回值true(确认)/false(取消)  
用户如果点击 "确认"，确认框返回 true ；如果点击 "取消"，确认框返回 false。

### 提示框
prompt("sometext","defaultvalue")  
如果用户点击确认，那么返回值为输入的值；如果用户点击取消，那么返回值为 null。