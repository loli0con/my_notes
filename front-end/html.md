# html
Hyper Text Markup Language（超文本标记语言）

## 标签
![html+20210715200735](https://raw.githubusercontent.com/loli0con/picgo/master/images/html%2B20210715200735.png%2B2021-07-15-20-07-36)

1. 标签的格式：`<标签名>封装的数据</标签名>`
2. 标签名大小写不敏感
3. 标签拥有自己的属性：`<标签名 属性1="属性值1" 属性2="属性值2">封装的数据</标签名>`
   * 基本属性：修改简单的样式效果
   * 事件属性：设置事件响应后的代码
4. 标签又分为单标签和双标签
   * 单标签：`<“自结束标签” />`
   * 双标签：`<“开始标签”>封装的数据</“结束标签”>`

## 属性
下面列出了适用于大多数 HTML 元素的属性（通用属性）：
|属性|描述|
|---|---|
|class|为html元素定义一个或多个类名（多个类名以空格分隔）|
|id|定义元素的唯一id|
|style|规定元素的行内样式|
|title|描述了元素的额外信息|

## 常用标签
### 标题
```html
<!-- 标题标签
h1 - h6 都是标题标签 
h1 最大
h6 最小

align属性是对齐属性:
    left   左对齐(默认)
    center 居中
    right  右对齐
-->
<h1 align="left">标题 1</h1>
<h2 align="center">标题 2</h2>
<h3 align="right">标题 3</h3>
<h4>标题 4</h4>
<h5>标题 5</h5>
<h6>标题 6</h6>
```

### 超链接
在网页中所有点击之后可以跳转的内容都是超链接
```html
<!--
a标签是超链接
href 属性设置连接的地址
target 属性设置哪个目标进行跳转
  _self  表示当前页面(默认值)
  _blank 表示打开新页面来进行跳转 

-->

<a href="baidu.com">百度1</a>
<a href="baidu.com" target="_self">百度2</a>
<a href="baidu.com" target="_blank">百度3</a>
```
### 图片
```html
<!-- 
img标签是图片标签,用来显示图片

src 属性可以设置图片的路径
width 属性设置图片的宽度
height 属性设置图片的高度
border 属性设置图片边框大小
alt 属性设置当指定路径找不到图片时,用来代替显示的文本内容
 -->
<img src="图片.jpg" width="666" height="666" border="1" alt="图片找不到"/>
```

### 列表
```html
<!-- 无序列表
无序列表是一个项目的列表，列表项目使用圆点进行标记。
无序列表始于 <ul> 标签，每个列表项始于 <li> 标签。
-->
<ul>
<li>Coffee</li>
<li>Milk</li>
</ul>

<!-- 有序列表
有序列表是一个项目的列表，列表项目使用数字进行标记。
有序列表始于 <ol> 标签，每个列表项始于 <li> 标签。
-->
<ol>
<li>Coffee</li>
<li>Milk</li>
</ol>

<!-- 自定义列表
自定义列表不仅仅是一列项目，而是项目及其注释的组合。
自定义列表以 <dl> 标签开始，每个自定义列表项以 <dt> 开始，每个自定义列表项的定义以 <dd> 开始。
-->
<dl>
<dt>Coffee</dt>
<dd>- black hot drink</dd>
<dt>Milk</dt>
<dd>- white cold drink</dd>
</dl>
```

### 表格
![html+20210715201359](https://raw.githubusercontent.com/loli0con/picgo/master/images/html%2B20210715201359.png%2B2021-07-15-20-14-00)
table标签是表格标签，有如下属性：
* border 设置表格标签
* width 设置表格宽度
* height 设置表格高度
* align 设置表格相对于页面的对齐方式
* cellspacing 设置单元格间距

表格内存在如下标签：
* caption 是表格的标题
* thead 定义表格的页眉
* tbody 定义表格的主体
* tfoot 定义表格的页脚
* tr 是行标签，一行里面可以有以下内容：
  * th 是表头标签
  * td 是单元格标签，有如下属性：
    * align 属性设置单元格文本对齐方式
    * colspan 属性设置跨列
    * rowspan 属性设置跨行

### 表单
表单就是 html 页面中，用来收集用户信息的所有标签集合，然后把这些信息发送给服务器。

form 标签是表单标签，有如下属性：
* action 属性设置提交的服务器地址
* method 属性设置提交的方式GET(默认值)或POST
  * GET 请求的特点是：
    1. 浏览器地址栏中的地址是:action 属性\[+?+请求参数]  
       请求参数的格式是:name=value&name=value
    2. 不安全
    3. 它有数据长度的限制
  * POST 请求的特点是：
    1. 浏览器地址栏中只有 action 属性值
    2. 相对于 GET 请求要安全
    3. 理论上没有数据长度的限制

在发送信息给服务器的时候，表单内的子标签（用于收集不同类型信息的标签）的name属性会作为数据的*key*，在服务端可以根据该*key*来获取对应的值。

#### input标签
多数情况下被用到的表单标签是输入标签\<input>。
\<input>标签在\<form>标签中使用，用来声明允许用户输入数据的 input 控件。

输入字段可通过多种方式改变，取决于 type 属性。大多数经常被用到的输入类型如下：
* type="text"是文件输入框，value设置默认显示内容
* type="password"是密码输入框，value设置默认显示内容
* type="radio"是单选框，name属性可以对其进行分组，checked="checked"表示默认选中，value属性定义送往服务器的选项值
* type=checkbox是复选框，checked="checked"表示默认选中
* type=reset是重置按钮，value属性修改按钮上的文本
* type=submit是提交按钮，value属性修改按钮上的文本
* type=button是按钮，value属性修改按钮上的文本
* type=file是文件上传域
* type=hidden是隐藏域

#### select标签
\<select>标签是一种表单控件没，用来创建下拉列表，用于在表单中接受用户输入：
* name属性定义下拉列表的名称
* multiple属性为true时，可选择多个选项

\<option>标签定义了列表中的可用选项：
* 属性selected="selected"时表示该选项默认选中。
* 属性value定义送往服务器的选项值

#### 多行文本输入框
textarea 表示多行文本输入框 (起始标签和结束标签中的内容是默认值)：
* rows 属性设置可以显示几行的高度
* cols 属性设置每行可以显示几个字符宽度

### 分割标签
* \<div> 标签大分割，多行数据的分割布局，用于控制多行数据的样式
* \<span> 标签小分割，一行内数据的分割布局，用于一行内局部位置数据的样式

### 其他标签
p标签：默认会在段落的上方或下方各空出一行来(如果已有就不再空)


## 实体字符
实体字符（ASCII Encoding Reference）是用来在代码中以实体代替与HTML语法相同的字符，避免浏览解析错误。它的两种表示方式，第一种为 `&` 外加实体字符名称，例如 `&nbsp;`，第二种为 & 加实体字符序号，例如 `&#160;`。

### 常用HTML字符实体
![html+20210712214535](https://raw.githubusercontent.com/loli0con/picgo/master/images/html%2B20210712214535.png%2B2021-07-12-21-45-36)
