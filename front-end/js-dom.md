# DOM
DOM 全称是 Document Object Model 文档对象模型，它是 HTML 和 XML 文档的编程接口，它把文档中的*标签、属性和文本*转换成为对象来管理。

document 对象的理解:
1. document 它管理了所有的 HTML 文档内容。
2. document 它是一种树结构的文档，有层级关系。
3. 它让我们把所有的标签都对象化。
4. 我们可以通过 document 访问所有的标签对象。

![js-dom+20210715210022](https://raw.githubusercontent.com/loli0con/picgo/master/images/js-dom%2B20210715210022.png%2B2021-07-15-21-00-23)

## 百科全书
[参考手册-菜鸟版](https://www.runoob.com/jsref/jsref-tutorial.html)

## 节点
严格地讲，我们这里的DOM节点是指`Element`，但是DOM节点实际上是`Node`，在HTML中，Node包括Element、Comment、CDATA_SECTION等很多种，以及根节点Document类型，但是，绝大多数时候我们只关心Element，也就是实际控制页面结构的Node，其他类型的Node忽略即可。根节点Document已经自动绑定为全局变量document。    

## 方法
### 获取节点对象
* document.getElementById(elementId)：通过id得到唯一的元素，如果有同名的只会得到第一个
* document.getElementByName(elementName)：通过name属性得到一组元素
* document.getElementsByTagName(tagName)：通过标签名得到一组元素
* document.getElementsByClassName(className)：通过class类名得到一组元素
* document.querySelector(CSS选择器)：通过css选择器得到符合的标签中第一个元素对象
* document.querySelectorAll(CSS选择器)：通过css选择器得到符合的所有元素对象
* document.createElement(tagName)

### 节点方法
* parentNode.appendChild(childNode)
* parentNode.removeChild(childNode)
* parentNode.replaceChild(newNode, oldNode)
* parentNode.insertBefore(newItem,existingItem)
* element.getAttribute(attributeName)
* element.setAttribute(attributeName)
* element.remove()：删除自己

### 节点属性
* childNodes：获取当前节点的所有子节点
* firstElementChild：获取当前节点的第一个Element子节点
* lastElementChild：获取当前节点的最后一个Element子节点
* parentNode：获取当前节点的父节点
* nextElementSibling：获取当前节点的下一个Element节点
* previousElementSibling：获取当前节点的上一个Element节点
***
* innerHTML：表示获取/设置起始标签和结束标签中的元素(html代码)
* innerText：表示获取/设置起始标签和结束标签中的纯文本
***
* element.属性名：表示获取/设置element的属性值
* element.className：用于获取或设置标签的 class 属性值
* element.style.样式名：表示获取/设置element的样式（因为CSS允许font-size这样的名称，但它并非JavaScript有效的属性名，所以需要在JavaScript中改写为驼峰式命名fontSize）



## 事件
### 官方解释
事件是您在编程时系统内发生的动作或者发生的事情，系统响应事件后，如果需要，您可以某种方式对事件做出回应。

[官方文档-事件](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Building_blocks/Events)

### 常用的事件
* onload-加载完成事件：页面加载完成之后。
* onunload-卸载完成事件：页面被卸载(关闭)完成之后。
***
* onmousedown-鼠标按钮被按下
* onmouseup-鼠标按钮被松开
* onclick-鼠标单击事件：常用于按钮的点击响应操作。
* ondblclick-鼠标双击事件
* onmouseover-鼠标指针移动到元素事件
* onmouseout-鼠标指针离开元素
***
* onkeydown-某个键盘的键被按下
* onkeypress-某个键盘的键被按下或按住
* onkeyup-某个键盘的键被松开
***
* onfocus-获得焦点事件
* onblur-失去焦点事件：常用用于输入框失去焦点后验证其输入内容是否合法。
* onchange-内容发生改变事件：常用于下拉列表和输入框内容发生改变后操作。
* onsubmit-表单提交事件：常用于表单提交前，验证所有表单项是否合法。


### 绑定
告诉浏览器，当事件响应后要执行哪些操作代码，叫事件注册或事件绑定。

#### 静态绑定
通过 html 标签的事件属性直接赋于事件响应后的代码，这种方式我们叫静态注册。
```html
<input type="button" onclick="func(this)">
<!-- this代表当前事件源对象 -->
```

#### 动态绑定
是指先通过 js 代码得到标签的 dom 对象，然后再通过 dom 对象.事件名 = function(){} 这种形式赋于事件 响应后的代码，叫动态注册。
```js
按钮对象.onclick = function() { /* 代码 */ };
```
