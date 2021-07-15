# Javascript
[参考手册-菜鸟版](https://www.runoob.com/jsref/jsref-tutorial.html)
## 结合方式
在html页面中使用script标签来书写JavaScript代码（简单）
```html
<script type="text/javascript">
/* JavaScript代码 */
</script>
```

使用script标签引入单独的JavaScript代码文件（正规）
```html
<script type="text/javascript" src="code.js"></script>
```

## Hello World
```js
window.alert('hello world!'); // 弹框提示
console.log('hello world'); // 浏览器的控制台输出
```

## 变量
### 数据类型
JavaScript的数据类型：
* *数值*类型：number
* *布尔*类型：boolean
* *未定义*类型：undefined
* *字符串*类型：string
* *函数*类型：function
* *对象*类型：object
  * iterable
    * Array
    * Map
    * Set
  * Date
  * RegExp
  * JSON

### 特殊值
JavaScript里的特殊值：
* undefined：未定义，所有js变量未赋值的时候的默认值
* null：空值
* NaN：Not a Number，非数值

```js
> typeof undefined;
'undefined'
> typeof null;
'object'
> typeof NaN;
'number'
```

#### undefined 与 null 的区别
表面上 undefined 与 null 都是什么都没有的意思，但是实际上 undefined 是未定义（就是变量没有初始化），null 是一个变量初始化了，但是什么值都没给，只给了一个空对象；进一步说，undefined 与 null是值相等，类型不相等。

### 定义变量
```JavaScript
let var; // 声明
var = 1; // 赋值

// 推荐使用let
let a = 1; // 声明并赋值
var b = '2'; 
const c = true; // 常量
```

### 作用域
作用域是可访问变量的集合。

#### 全局变量
在函数外声明的变量是全局变量，网页上的所有脚本和函数都能访问它。

#### 生命周期
局部变量会在函数运行以后被删除，全局变量会在页面关闭后被删除。

#### 全局属性
如果直接给一个未声明的变量赋值，该变量将被自动作为 window 的一个属性。


### 原则
闲的蛋疼也不要使用包装对象！！！

不要问为什么，这就是JavaScript代码的乐趣！

#### 类型转换
* 用parseInt()或parseFloat()来转换任意类型到number。
* 全局方法 Number() 可以将字符串、布尔值、日期转换为number。
* 用String()来转换任意类型到string。
* 通常不必把任意类型转换为boolean再判断，因为可以直接写if (myVar) {...}；  
所有的变量都可以做为一个 boolean 类型的变量去使用：0 、null、 undefined、""(空串) 都认为是 false。
#### 类型判断
* typeof操作符可以判断出number、boolean、string、function、undefined和object。
* 可通过 instanceof 操作符来判断对象的具体类型。
* 判断Array要使用Array.isArray(arr)。
* 判断null请使用myVar === null。
* NaN这与所有其他值都不相等（包括它自己），只能通过isNaN()函数判断。
* 通过对象的constructor属性判断对象的类型

```js
/* 类型判断 */

// constructor 是一个属性，构造函数属性，不同类型的数据，会得到相应不同的值。
"John".constructor                 // 返回函数 String()  { [native code] }
(3.14).constructor                 // 返回函数 Number()  { [native code] }
false.constructor                  // 返回函数 Boolean() { [native code] }
[1,2,3,4].constructor              // 返回函数 Array()   { [native code] }
{name:'John', age:34}.constructor  // 返回函数 Object()  { [native code] }
new Date().constructor             // 返回函数 Date()    { [native code] }
function () {}.constructor         // 返回函数 Function(){ [native code] }

// 判断是否为Array类型
function isArray(myArray) {
    return myArray.constructor.toString().indexOf("Array") > -1;
}

// 判断是否为Data类型
function isDate(myDate) {
    return myDate.constructor.toString().indexOf("Date") > -1;
}
```



## 运算符
### 关系运算
* 等于： ==，自动转换数据类型再比较字面值
* 全等于： ===，比较数据类型和字面值

### 逻辑运算
* 与：&&（短路）
* 或：||（短路）
* 非：!



## 流程控制
### 条件判断
```js
// if
var age = 3;
if (age >= 18) {
  alert('adult');
} else if (age >= 6) {
    alert('teenager');
} else {
    alert('kid');
}

// switch
// switch 中 case 的判断是 “===” 
switch(n)
{
  case 1:
    // 执行代码块 1
    break;
  case 2:
    // 执行代码块 2
    break;
  default:
    // 与 case 1 和 case 2 不同时执行的代码
}
/* 
如果 case 后面有多行语句，也不需要 { }，带不带都行，
因为 case 是一直执行到下面第一个 break
*/
```

### 循环
* while
* do … while
* for
  * for(… ; … ; …)
  * for(… of …)：以任意顺序迭代对象的可枚举属性，遍历（object）键名
  * for(… in …)：遍历可迭代对象定义要迭代的数据，遍历（iterable）键值

可以使用在循环中使用break和continue。

## 字符串
### 定义
```js
let str1 = 'Hello'; // 推荐使用单引号
let str2 = "world";

// ASCII字符可以以\x##形式的十六进制表示
let str3 = '\x74'; // 完全等同于 'J'

// \u####表示一个Unicode字符
let str4 = '\u6211\u662F'; // 完全等同于 '我是'
let str5 = '\u4e2d\u6587'; // 完全等同于 '中文'

// 多行字符串/模版字符串
let str6 = `${str1} ${str2},
${str4} ${str3}avaScript!`
// str6 === 'Hello world,\n我是 tavaScript!'
```

### 特性
字符串是不可变的，如果对字符串的某个索引赋值，不会有任何错误，但也没有任何效果。



## 函数
在JavaScript里，函数是一等公民，和对象平等。
### 定义
```js
命名函数：
function 函数名(形参列表){
    // 函数体
}

匿名函数（定义一个匿名函数，再进行赋值操作）：
let 函数名 = function(形参列表){ /*函数体*/ };
```

### 返回值
如果没有return语句，函数执行完毕后也会返回结果，只是结果为undefined。

### 重载
js函数无重载，后面定义的函数会覆盖前面定义的同名函数。

### arguments
JavaScript允许传入任意个参数而不影响调用，因此传入的参数比定义的参数多也没有问题。

在 function 函数中，可以直接用arguments来获取所有参数的变量，操作类似数组。



## 对象
JavaScript的对象是一种无序的集合数据类型，它由若干键值对组成。

### 定义
```js
// Object 形式
let 变量名 = new Object(); 

// {}花括号形式
var 变量名 = { 
    属性名:属性值, // 定义属性

    函数名:function(参数列表){ /*函数体*/ } // 定义函数
    
    /* 最后一个键值对不需要在末尾加 逗号, */
};
```

### 访问
```js
// 读取
let 属性 = 对象.属性名;
let 属性 = 对象['属性名']; // 下面的同理，可以用“[]”替换“.”
let 函数 = 对象.函数名;

// 调用
let 返回值 = 对象名.函数名();


// 赋值
对象名.xx名 = xx;
对象名.属性名 = 属性;
对象名.函数名 = 函数;
```

要检测对象是否拥有某一属性：
* in操作符（自身拥有的 或者 是继承得到的）
* hasOwnProperty()方法（自身拥有的，不是继承得到的）

为了区分对象的类型，我们用typeof操作符获取对象的类型，它总是返回一个字符串。

### this
this 的多种指向:
1. 在对象方法中， this 指向调用它所在方法的对象。
2. 单独使用 this，它指向全局(Global)对象。
3. 函数使用中，this 指向函数的所属者。
4. 严格模式下函数是没有绑定到 this 上，这时候 this 是 undefined。
5. HTML 事件句柄中，this 指向了接收事件的 HTML 元素。
6. apply 和 call 允许切换函数执行的上下文环境（context），即 this 绑定的对象，可以将 this 引用到任何对象。



## iterable
iterable类型的集合可以通过新的for ... of循环来遍历，Array、Map和Set都属于iterable类型。

### Array
JavaScript的Array可以包含任意数据类型，并通过索引来访问每个元素。

js的数组会自动扩容：
1. 直接给Array的length赋一个新的值
2. 通过给超过范围的索引赋值
```js
> let arr = [1,2]; // 初始化

> arr.length; // 查看数组长度
2

> arr.length = 4;  // 1.

> arr[2]
undefined
> arr[3]
undefined

> arr[5] = 'a';  //  2.

> arr[4]
undefined
>
```
#### 常用方法
* indexOf：搜索一个指定的元素的位置
* slice：截取Array的部分元素，然后返回一个新的Array
* push和pop：push()向Array的末尾添加若干元素，pop()则把Array的最后一个元素删除掉
* unshift和shift：shift()向Array的头部添加若干元素，unshift()则把Array的第一个元素删除掉
* sort：对当前Array进行排序
* reverse：把整个Array的元素给反转
* splice：从指定的索引开始删除若干元素，然后再从该位置添加若干元素
* concat：把当前的Array和另一个Array连接起来，并返回一个新的Array
* join：把当前Array的每个元素都用指定的字符串连接起来，然后返回连接后的字符串

### Map
Map是一组键值对的结构，具有极快的查找速度。
#### 定义
```js
let map = new Map([[key1, value1], [key2, value2]]);
```
#### 常用方法
* get
* set
* has
* delete

### Set
Set是一组key的集合，但不存储value。key不能重复，重复元素在Set中自动被过滤。
#### 定义
```js
let set = new Set([key1, key2, key3]); // 含1, 2, 3
```
#### 常用方法
* add
* delete



## Date
在JavaScript中，Date对象用来表示日期和时间。

时间戳是一个自增的整数，它表示从1970年1月1日零时整的GMT时区开始的那一刻，到现在的毫秒数。假设浏览器所在电脑的时间是准确的，那么世界上无论哪个时区的电脑，它们此刻产生的时间戳数字都是一样的，所以，时间戳可以精确地表示一个时刻，并且与时区无关。

```js
// 获取当前时间（浏览器从本机操作系统获取的时间），假定为某个时刻
let now_date = new Date();  //获取系统当前时间，date
let now_timestamp = Date.now(); //获取当前时间戳，number
let now = now_date; // Wed Jun 24 2015 19:49:22 GMT+0800 (CST)
now.getFullYear(); // 2015, 年份
now.getMonth(); // 5, 月份，注意月份范围是0~11，5表示六月
now.getDate(); // 24, 表示24号
now.getDay(); // 3, 表示星期三
now.getHours(); // 19, 24小时制
now.getMinutes(); // 49, 分钟
now.getSeconds(); // 22, 秒
now.getMilliseconds(); // 875, 毫秒数
now.getTime(); // 1435146562875, 以number形式表示的时间戳
now.toLocaleString(); // '2015/6/24 下午7:49:22'，本地时间（北京时区+8:00），显示的字符串与操作系统设定的格式有关
now.toUTCString(); // 'Wed, 24 Jun 2015 11:49:22 GMT'，UTC时间，与本地时间相差8小时


// 创建一个指定日期和时间的Date对象
// 方式一
let d1 = new Date(2015, 5, 19, 20, 15, 30, 123);
d1; // Fri Jun 19 2015 20:15:30 GMT+0800 (CST)
// 方式二
let d2 = Date.parse('2015-06-24T19:49:22.875+08:00'); // 解析一个符合ISO 8601格式的字符串
d2; // 1435146562875，不是Date对象，而是一个时间戳
let d3 = new Date(1435146562875); // 把时间戳转换为一个Date
d3; // Wed Jun 24 2015 19:49:22 GMT+0800 (CST)
```



## RegExp
### 创建
JavaScript有两种方式创建一个正则表达式：
* 直接通过`/正则表达式/修饰符`写出来 （推荐！）
* 通过new RegExp('正则表达式','修饰符')创建一个RegExp对象

常用的修饰符：
* g：全局匹配/搜索
* i：忽略大小写
* m：执行多行匹配

### 匹配
RegExp对象的test()方法用于测试给定的字符串是否符合条件。

```js
var re = /^\d{3}\-\d{3,8}$/;
// test是模糊匹配，不是精确匹配。
re.test('010-12345'); // true
re.test('010-1234x'); // false
```

### 切分
```js
'a b   c'.split(/\s+/); // ['a', 'b', 'c']
```

### 分组
正则表达式有提取子串的强大功能，用`()`表示的就是要提取的分组（Group）。

```js
var re = /^(\d{3})-(\d{3,8})$/;
re.exec('010-12345'); // ['010-12345', '010', '12345']
re.exec('010 12345'); // null
```
如果正则表达式中定义了组，就可以在RegExp对象上用exec()方法提取出子串来。

exec()方法在匹配成功后，会返回一个Array，第一个元素是正则表达式匹配到的整个字符串，后面的字符串表示匹配成功的子串。

exec()方法在匹配失败时返回null。

### 贪婪匹配
正则匹配默认是贪婪匹配，也就是匹配尽可能多的字符。使用`?`采用非贪婪匹配。

```js
// 由于\d+采用贪婪匹配，直接把后面的0全部匹配了，结果0*只能匹配空字符串了。
var re1 = /^(\d+)(0*)$/;
re1.exec('102300'); // ['102300', '102300', '']

// 加个?就可以让\d+采用非贪婪匹配
var re2 = /^(\d+?)(0*)$/;
re2.exec('102300'); // ['102300', '1023', '00']
```

### 全局搜索
全局匹配可以多次执行exec()方法来搜索一个匹配的字符串。  
当我们指定g标志后，每次运行exec()，正则表达式本身会更新lastIndex属性，表示上次匹配到的最后索引

```js
var s = 'JavaScript, VBScript, JScript and ECMAScript';
var re=/[a-zA-Z]+Script/g;

// 使用全局匹配:
re.exec(s); // ['JavaScript']
re.lastIndex; // 10

re.exec(s); // ['VBScript']
re.lastIndex; // 20

re.exec(s); // ['JScript']
re.lastIndex; // 29

re.exec(s); // ['ECMAScript']
re.lastIndex; // 44

re.exec(s); // null，直到结束仍没有匹配到
```

## JSON
JSON是一种超轻量级的数据交换格式。

### 语法规则
* 数据为 键/值 对。
* 数据由逗号分隔。
* 大括号保存对象
* 方括号保存数组

JSON的字符集必须是UTF-8，JSON的字符串规定必须用双引号""，键也必须用双引号""。

### 数据类型
在JSON中，一共就这么几种数据类型（以及它们的任意组合）：
* number：和JavaScript的number完全一致；
* boolean：就是JavaScript的true或false；
* string：就是JavaScript的string；
* null：就是JavaScript的null；
* array：就是JavaScript的Array表示方式——[]；
* object：就是JavaScript的{ ... }表示方式。

### 序列化
对象 ---序列化---> JSON格式的字符串

JSON.stringify() 方法用于将 JavaScript 值转换为 JSON 字符串。  
语法：`JSON.stringify(value[, replacer[, space]])`  
参数：
* value: 要转换的 JavaScript 值（通常为对象或数组）
* replacer: 用于转换结果的函数或数组
  * 如果 replacer 为函数，则 JSON.stringify 将调用该函数，并传入每个成员的键和值。使用返回值而不是原始值。如果此函数返回 undefined，则排除成员。
  * 如果 replacer 是一个数组，则仅转换该数组中具有键值的成员。成员的转换顺序与键在数组中的顺序一样。
  * 还可以可以给对象本身定义一个toJSON()的方法，直接返回JSON应该序列化的数据。
* space: 文本添加缩进、空格和换行符，如果 space 是一个数字，则返回值文本在每个级别缩进指定数目的空格，如果 space 大于 10，则文本缩进 10 个空格。space 也可以使用非数字，如：\t

### 反序列化
JSON格式的字符串 <---反序列化--- 对象

JSON.parse() 方法用于将一个 JSON 字符串转换为对象。  
语法：`JSON.parse(text[, reviver])`  
参数：
* text: 一个有效的 JSON 字符串
* reviver: 一个转换结果的函数，将为对象的每个成员调用此函数



## void
void 是 JavaScript 中的关键字，该操作符指定要计算一个表达式但是不返回值。

```js
// void()仅仅是代表不返回任何值，但是括号内的表达式还是要运行
javascript:void(alert("Warnning!"));  //“javascript”可省略，但建议加上

javascript:void(0); // 常见用法
```

## generator
Pending

## 面向对象编程
Pending

## 错误处理
Pending