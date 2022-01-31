# 基本引用类型

引用值(或者对象)是某个特定引用类型的实例。在 ECMAScript 中，引用类型是把数据和功能组织到一起的结构。引用类型有时候也被称为对象定义，因为它们描述了自己的对象应有的属性和方法。

ECMAScript 缺少传统的面向对象编程语言所具备的某些基本结构，包括类和接口。

新对象通过使用 new 操作符后跟一个构造函数(constructor)来创建，构造函数就是用来创建新对象的函数。


## Date
ECMAScript 的 Date 类型将日期保存为自协调世界时(UTC，Universal Time Coordinated)时间 1970 年 1 月 1 日午夜(零时)至今所经过的毫秒数。

要创建日期对象，就使用 new 操作符来调用 Date 构造函数:
```js
let now = new Date();
```
在不给 Date 构造函数传参数的情况下，创建的对象将保存当前日期和时间。要基于其他日期和时 间创建日期对象，必须传入其毫秒表示(UNIX 纪元 1970 年 1 月 1 日午夜之后的毫秒数)。

ECMAScript 为此提供了两个辅助方法:Date.parse()和 Date.UTC()。

### Date.parse()
Date.parse()方法接收一个表示日期的字符串参数，尝试将这个字符串转换为表示该日期的毫秒数。所有实现都必须支持下列日期格式:
* “月/日/年”，如"5/23/2019";
* “月名 日, 年”，如"May 23, 2019";
* “周几 月名 日 年 时:分:秒 时区”，如"Tue May 23 2019 00:00:00 GMT-0700";
* ISO 8601扩展格式“YYYY-MM-DDTHH:mm:ss.sssZ”，如2019-05-23T00:00:00。

如果直接把表示日期的字 符串传给 Date 构造函数，那么 Date 会在后台调用 Date.parse()。

### Date.UTC()
Date.UTC()方法返回日期的毫秒表示，传给 Date.UTC()的参数是年、零起点月数(1 月是 0，2 月是 1，以此类推)、日(1~31)、时(0~23)、分、秒和毫秒。这些参数中，只有前两个(年和月)是必需的。如果不提供日，那么默认为 1 日。其他参数的默认值都是 0。

Date.UTC()也会被 Date 构造函数隐式调用，但创建的是本地日期(由于系统设置决定的)，不是 GMT 日期。

### Date.now()
ECMAScript 还提供了 Date.now()方法，返回表示方法执行时日期和时间的毫秒数。


### 继承的方法
Date 类型重写了 toLocaleString()、toString()和 valueOf()方法：
* toLocaleString()方法返回与浏览器运行的本地环境一致的日期和时间。这通常意味着格式中包含针对时间的 AM(上午)或 PM(下午)，但不包含时区信息(具体格式可能因浏览器而不同)。
* toString()方法通常返回带时区信息的日期和时间，而时间也是以 24 小时制(0~23)表示的。
* valueOf()方法返回的是日期的毫秒表示。

### 日期格式化方法
Date 类型有几个专门用于格式化日期的方法，它们都会返回字符串:
* toDateString()显示日期中的周几、月、日、年(格式特定于实现);
* toTimeString()显示日期中的时、分、秒和时区(格式特定于实现);
* toLocaleDateString()显示日期中的周几、月、日、年(格式特定于实现和地区);
* toLocaleTimeString()显示日期中的时、分、秒(格式特定于实现和地区);
* toUTCString()显示完整的 UTC 日期(格式特定于实现)。

### 日期/时间组件方法
Date 类型剩下的方法(见下表)直接涉及取得或设置日期值的特定部分。
|方法|说明|
|---|---|
|getTime()|返回日期的毫秒表示;与 valueOf()相同|
|setTime(milliseconds)|设置日期的毫秒表示，从而修改整个日期|
|getFullYear()|返回 4 位数年(即 2019 而不是 19)|
|getUTCFullYear()|返回 UTC 日期的 4 位数年|
|setFullYear(year)|设置日期的年(year 必须是 4 位数)|
|setUTCFullYear(year)|设置 UTC 日期的年(year 必须是 4 位数)|
|getMonth()|返回日期的月(0 表示 1 月，11 表示 12 月)|
|getUTCMonth()|返回 UTC 日期的月(0 表示 1 月，11 表示 12 月)|
|setMonth(month)|设置日期的月(month 为大于 0 的数值，大于 11 加年)|
|setUTCMonth(month)|设置 UTC 日期的月(month 为大于 0 的数值，大于 11 加年)|
|getDate()|返回日期中的日(1~31)|
|getUTCDate()|返回 UTC 日期中的日(1~31)|
|setDate(date)|设置日期中的日(如果 date 大于该月天数，则加月)|
|setUTCDate(date)|设置 UTC 日期中的日(如果 date 大于该月天数，则加月)|
|getDay()|返回日期中表示周几的数值(0 表示周日，6 表示周六)|
|getUTCDay()|返回 UTC 日期中表示周几的数值(0 表示周日，6 表示周六)|
|getHours()|返回日期中的时(0~23)|
|getUTCHours()|返回 UTC 日期中的时(0~23)|
|setHours(hours)|设置日期中的时(如果 hours 大于 23，则加日)|
|setUTCHours(hours)|设置 UTC 日期中的时(如果 hours 大于 23，则加日)|
|getMinutes()|返回日期中的分(0~59)|
|getUTCMinutes()|返回 UTC 日期中的分(0~59)|
|setMinutes(minutes)|设置日期中的分(如果 minutes 大于 59，则加时)|
|setUTCMinutes(minutes)|设置 UTC 日期中的分(如果 minutes 大于 59，则加时)|
|getSeconds()|返回日期中的秒(0~59)|
|getUTCSeconds()|返回 UTC 日期中的秒(0~59)|
|setSeconds(seconds)|设置日期中的秒(如果 seconds 大于 59，则加分)|
|setUTCSeconds(seconds)|设置 UTC 日期中的秒(如果 seconds 大于 59，则加分)|
|getMilliseconds()|返回日期中的毫秒|
|getUTCMilliseconds()|返回 UTC 日期中的毫秒|
|setMilliseconds(milliseconds)|设置日期中的毫秒|
|setUTCMilliseconds(milliseconds)|设置 UTC 日期中的毫秒|
|getTimezoneOffset()|返回以分钟计的 UTC 与本地时区的偏移量(如美国 EST 即“东部标准时间” 返回 300，进入夏令时的地区可能有所差异)|


## RegExp
ECMAScript 通过 RegExp 类型支持正则表达式。正则表达式使用类似 Perl 的简洁语法来创建:
```js
let expression = /pattern/flags;
```
这个正则表达式的 pattern(模式)可以是任何简单或复杂的正则表达式，包括字符类、限定符、分组、向前查找和反向引用。每个正则表达式可以带零个或多个 flags(标记)，用于控制正则表达式的行为。下面给出了表示匹配模式的标记:
* g:全局模式，表示查找字符串的全部内容，而不是找到第一个匹配的内容就结束。
* i:不区分大小写，表示在查找匹配时忽略 pattern 和字符串的大小写。
* m:多行模式，表示查找到一行文本末尾时会继续查找。
* y:粘附模式，表示只查找从 lastIndex 开始及之后的字符串。
* u:Unicode 模式，启用 Unicode 匹配。
* s:dotAll 模式，表示元字符.匹配任何字符(包括\n 或\r)。

所有元字符在模式中也必须转义，包括:`([{\^$|)]}?*+.`。元字符在正则表达式中都有一种或多种特殊功能，所以要匹配上面这些字符本身，就必须使用反斜杠来转义。

正则表达式也可以使用 RegExp 构造函数来创建，它接收两个参数:模式字符串和(可选的)标记字符串。任何使用字面量定义的正则表达式也可以通过构造函数来创建。RegExp 构造函数的两个参数都是字符串。因为 RegExp 的模式参数是字符串，所以在某些情况下需要二次转义。所有元字符都必须二次转义，包括转义字符序列，如\n(\转义后的字符串是\\，在正则表达式字符串中则要写成\\\\)。此外，使用 RegExp 也可以基于已有的正则表达式实例，并可选择性地修改它们的标记。

### RegExp实例属性
每个 RegExp 实例都有下列属性，提供有关模式的各方面信息：
* global:布尔值，表示是否设置了 g 标记。
* ignoreCase:布尔值，表示是否设置了 i 标记。
* unicode:布尔值，表示是否设置了 u 标记。
* sticky:布尔值，表示是否设置了 y 标记。
* lastIndex:整数，表示在源字符串中下一次搜索的开始位置，始终从 0 开始。
* multiline:布尔值，表示是否设置了 m 标记。
* dotAll:布尔值，表示是否设置了 s 标记。
* source:正则表达式的字面量字符串(不是传给构造函数的模式字符串)，没有开头和结尾的斜杠。
* flags:正则表达式的标记字符串。始终以字面量而非传入构造函数的字符串模式形式返回(没有前后斜杠)。

source 和 flags 属性返回的是规范化之后可以在字面量中使用的形式。

### RegExp实例方法
RegExp 实例的主要方法是 exec()，主要用于配合捕获组使用。这个方法只接收一个参数，即要应用模式的字符串。如果找到了匹配项，则返回包含第一个匹配信息的数组;如果没找到匹配项，则返回 null。返回的数组虽然是 Array 的实例，但包含两个额外的属性:index 和 input。index 是字符串中匹配模式的起始位置，input 是要查找的字符串。这个数组的第一个元素是匹配整个模式的字符串，其他元素是与表达式中的捕获组匹配的字符串。如果模式中没有捕获组，则数组只包含一个元素。

如果模式设置了全局标记，则每次调用 exec()方法会返回一个匹配的信息。如果没有设置全局标记，则无论对同一个字符串调用多少次 exec()，也只会返回第一个匹配的信息。

lastIndex 在非全局模式下始终不变。如果在这个模式上设置了 g 标记，则每次调用 exec()都会在字符串中向前搜索下一个匹配项，直到搜索到字符串末尾，模式的 lastIndex 属性每次都会变化。在全局匹配模式下，每次调用 exec()都会更新 lastIndex 值，以反映上次匹配的最后一个字符的索引。

如果模式设置了粘附标记 y，则每次调用 exec()就只会在 lastIndex 的位置上寻找匹配项。粘附标记覆盖全局标记。

正则表达式的另一个方法是 test()，接收一个字符串参数。如果输入的文本与模式匹配，则参数返回 true，否则返回 false。这个方法适用于只想测试模式是否匹配，而不需要实际匹配内容的情况。

无论正则表达式是怎么创建的，继承的方法 toLocaleString()和 toString()都返回正则表达式的字面量表示。正则表达式的valueOf()方法返回正则表达式本身。

### RegExp构造函数属性
RegExp 构造函数本身也有几个属性。(在其他语言中，这种属性被称为静态属性。)这些属性适用于作用域中的所有正则表达式，而且会根据最后执行的正则表达式操作而变化。通过这些属性可以提取出与 exec()和 test()执行的操作相关的信息。

这些属性还有一个特点，就是可以通过两种不同的方式访问它们。换句话说，每个属性都有一个全名和一个简写。下表列出了RegExp 构造函数的属性。

|全名|简写|说明|
|---|---|---|
|input|$_|最后搜索的字符串(非标准特性)|
|lastMatch|$&|最后匹配的文本|
|lastParen|$+|最后匹配的捕获组(非标准特性)|
|leftContext|$`|input 字符串中出现在 lastMatch 前面的文本|
|rightContext|$'|input 字符串中出现在 lastMatch 后面的文本|

大多数简写形式都不是合法的 ECMAScript 标识符，要使用中括号语法来访问。

RegExp 还有其他几个构造函数属性，可以存储最多 9 个捕获组的匹配项。这些属性通过 RegExp.$1~RegExp.$9 来访问，分别包含第 1~9 个捕获组的匹配项。在调用 exec()或 test()时，这些属性就会被填充。

RegExp 构造函数的所有属性都没有任何 Web 标准出处，因此不要在生产环境中使用它们。

### 模式局限
下列特性目前还没有得到 ECMAScript 的支持：
* \A 和\Z 锚(分别匹配字符串的开始和末尾)
* 联合及交叉类
* 原子组
* x(忽略空格)匹配模式
* 条件式匹配
* 正则表达式注释


## 原始值包装类型
ECMAScript 提供了 3 种特殊的引用类型:Boolean、Number 和 String。

每当用到某个原始值的方法或属性时，后台都会创建一个相应原始包装类型的对象，从而暴露出操作原始值的各种方法。

后台执行的步骤可以描述为：
1. 创建一个 原始值 类型的实例;
2. 调用实例上的特定方法
3. 销毁实例

这种行为可以让原始值拥有对象的行为。

引用类型与原始值包装类型的主要区别在于对象的生命周期。在通过 new 实例化引用类型后，得到的实例会在离开作用域时被销毁，而自动创建的原始值包装对象则只存在于访问它的那行代码执行期间。这意味着不能在运行时给原始值添加属性和方法（因为在访问过后，实例会被销毁）。

在原始值包装类型的实例上调用 typeof 会返回"object"，所有原始值包装对象都会转换为布尔值 true。

Object 构造函数作为一个工厂方法，能够根据传入值的类型返回相应原始值包装类型的实 例。如果传给 Object 的是字符串，则会创建一个 String 的实例。如果是数值，则会创建 Number 的实例。布尔值则会得到 Boolean 的实例。

使用 new 调用原始值包装类型的构造函数，与调用同名的转型函数并不一样。

### Boolean
Boolean 是对应布尔值的引用类型。要创建一个 Boolean 对象，就使用 Boolean 构造函数并传入 true 或 false。

Boolean 的实例会重写 valueOf()方法，返回一个原始值 true 或 false。toString()方法被调用时也会被覆盖，返回字符串"true"或"false"。

强烈建议永远不要使用 Boolean 对象。

### Number
Number 是对应数值的引用类型。要创建一个 Number 对象，就使用 Number 构造函数并传入一个数值。

Number 类型重写了 valueOf()、toLocaleString() 和 toString()方法。valueOf()方法返回 Number 对象表示的原始数值，另外两个方法返回数值字符串。toString() 方法可选地接收一个表示基数的参数，并返回相应基数形式的数值字符串。

Number 类型还提供了几个用于将数值格式化为字符串的方法：
* toFixed()方法返回包含指定小数点位数的数值字符串。如果数值本身的小数位超过了参数指定的位数，则四舍五入到最接近的小数位。toFixed()方法可以表示有 0~20 个小数位的数值。
* toExponential()方法返回以科学记数法(也称为指数记数法)表示的数值字符串。接收一个参数，表示结果中小数的位数。
* toPrecision()方法会根据情况返回最合理的输出结果，可能是固定长度，也可能是科学记数法形式。这个方法接收一个参数，表示结果中数字的总位数(不包含指数)。本质上，toPrecision()方法会根据数值和精度来决定调用 toFixed()还是 toExponential()。

为了以正确的小数位精确表示数值，这 3 个方法都会向上或向下舍入。

#### 整数方法
Number.isInteger()方法，用于辨别一个数值是否保存为整数。

IEEE 754 数值格式有一个特殊的数值范围，在这个范围内二进制值可以表示一个整数值。这个数值范围从 Number.MIN_SAFE_INTEGER(-2^53 + 1)到 Number.MAX_SAFE_INTEGER(2^53 - 1)。对超出这个范围的数值，即使尝试保存为整数，IEEE 754 编码格式也意味着二进制值可能会表示一个完全不同的数值。为了鉴别整数是否在这个范围内，可以使用 Number.isSafeInteger()方法。


### String
String 是对应字符串的引用类型。要创建一个 String 对象，使用 String 构造函数并传入一个数值。

String 对象的方法可以在所有字符串原始值上调用。3 个继承的方法 valueOf()、toLocaleString() 和 toString()都返回对象的原始字符串值。

每个 String 对象都有一个 length 属性，表示字符串中字符的数量。注意，即使字符串中包含双字节字符(而不是单字节的 ASCII 字符)，也仍然会按单字符来计数。

#### JavaScript 字符
JavaScript 字符串由 16 位码元(code unit)组成。对多数字符来说，每 16 位码元对应一个字符。换句话说，字符串的 length 属性表示字符串包含多少 16 位码元。

charAt()方法返回给定索引位置的字符，由传给方法的整数参数指定。具体来说，这个方法查找指定索引位置的 16 位码元，并返回该码元对应的字符。

JavaScript 字符串使用了两种 Unicode 编码混合的策略:UCS-2 和 UTF-16。对于可以采用 16 位编码的字符(U+0000~U+FFFF)，这两种编码实际上是一样的。

使用 charCodeAt()方法可以查看指定码元的字符编码。这个方法返回指定索引位置的码元值，索引以整数指定。

fromCharCode()方法用于根据给定的 UTF-16 码元创建字符串中的字符。这个方法可以接受任意多个数值，并返回将所有数值对应的字符拼接起来的字符串。

对于 U+0000~U+FFFF 范围内的字符，length、charAt()、charCodeAt()和 fromCharCode() 返回的结果都跟预期是一样的。这是因为在这个范围内，每个字符都是用 16 位表示的，而这几个方法也都基于 16 位码元完成操作。只要字符编码大小与码元大小一一对应，这些方法就能如期工作。


这个对应关系在扩展到 Unicode 增补字符平面时就不成立了。问题很简单，即 16 位只能唯一表示 65536 个字符。这对于大多数语言字符集是足够了，在 Unicode 中称为**基本多语言平面(BMP)**。为了表示更多的字符，Unicode 采用了一个策略，即每个字符使用另外 16 位去选择一个**增补平面**。这种每个字符使用两个 16 位码元的策略称为**代理对**。

在涉及增补平面的字符时，前面讨论的字符串方法就会出问题。只有fromCharCode()方法能返回正确的结果，因为它实际上是基于提供的二进制表示直接组合成字符串。

为正确解析既包含单码元字符又包含代理对字符的字符串，可以使用 codePointAt()来代替 charCodeAt()。codePointAt()接收 16 位码元的索引并返回该索引位置上的码点(code point)。**码点**是 Unicode 中一个字符的完整标识。码点可能是 16 位，也可能是 32 位，而 codePointAt()方法可以从指定码元位置识别完整的码点。如果传入的码元索引并非代理对的开头，就会返回错误的码点。这种错误只有检测单个字符的时候才会出现，可以通过从左到右按正确的码元数遍历字符串来规避。迭代字符串可以智能地识别代理对的码点。

与 charCodeAt()有对应的 codePointAt()一样，fromCharCode()也有一个对应的 fromCodePoint()。这个方法接收任意数量的码点，返回对应字符拼接起来的字符串。

#### normalize()方法
某些 Unicode 字符可以有多种编码方式。有的字符既可以通过一个 BMP 字符表示，也可以通过一个代理对表示。

比如:
```js
// U+00C5:上面带圆圈的大写拉丁字母A
console.log(String.fromCharCode(0x00C5)); // Å


// U+212B:长度单位“埃”
console.log(String.fromCharCode(0x212B)); // Å


// U+004:大写拉丁字母A
// U+030A:上面加个圆圈
console.log(String.fromCharCode(0x0041, 0x030A)); // Å
```

比较操作符不在乎字符看起来是什么样的，因此这 3 个字符互不相等。

为解决这个问题，Unicode 提供了 4 种规范化形式，可以将类似上面的字符规范化为一致的格式，无论底层字符的代码是什么。这 4 种规范化形式是:NFD(Normalization Form D)、NFC(Normalization Form C)、NFKD(Normalization Form KD)和 NFKC(Normalization Form KC)。可以使用 normalize()方法对字符串应用上述规范化形式，使用时需要传入表示哪种形式的字符串:"NFD"、"NFC"、"NFKD"或"NFKC"。

通过比较字符串与其调用 normalize()的返回值，就可以知道该字符串是否已经规范化了。选择同一种规范化形式可以让比较操作符返回正确的结果。


#### 字符串操作方法
以下的的方法不会修改调用它们的字符串，而会返回新字符串值。

concat()，用于将一个或多个字符串拼接成一个新字符串。concat()方法可以接收任意多个参数，因此可以一次性拼接多个字符串。虽然 concat()方法可以拼接字符串，但更常用的方式是使用加号操作符(+)。而且多数情况下，对于拼接多个字符串来说，使用加号更方便。


ECMAScript 提供了 3 个从字符串中提取子字符串的方法:slice()、substr()和 substring()。第一个参数表示子字符串开始的位置，第二个参数表示子字符串结束的位置。对 slice()和 substring()而言，第二个参数是提取结束的位置(即该位置之前的字符会被提取出来)。对 substr()而言，第二个参数表示返回的子字符串数量。任何情况下，省略第二个参数都意味着提取到字符串末尾。

当某个参数是负值时，这 3 个方法的行为又有不同。slice()方法将所有负值参数都当成字符串长度加上负参数值。substr()方法将第一个负参数值当成字符串长度加上该值，将第二个负参数值转换为 0。substring()方法会将所有负参数值都转换为 0。

#### 字符串位置方法
有两个方法用于在字符串中定位子字符串:indexOf()和 lastIndexOf()。这两个方法从字符串中搜索传入的字符串，并返回位置(如果没找到，则返回-1)。两者的区别在于，indexOf()方法从字符串开头开始查找子字符串，而 lastIndexOf()方法从字符串末尾开始查找子字符串。

这两个方法都可以接收可选的第二个参数，表示开始搜索的位置。这意味着，indexOf()会从这个参数指定的位置开始向字符串末尾搜索，忽略该位置之前的字符;lastIndexOf()则会从这个参数指定的位置开始向字符串开头搜索，忽略该位置之后直到字符串末尾的字符。

#### 字符串包含方法
ECMAScript 有 3 个用于判断字符串中是否包含另一个字符串的方法:startsWith()、endsWith()和 includes()。这些方法都会从字符串中搜索传入的字符串，并返回一个表示是否包含的布尔值。它们的区别在于，startsWith()检查开始于索引 0 的匹配项，endsWith()检查开始于索引(string.length - substring.length)的匹配项，而 includes()检查整个字符串。

startsWith()和 includes()方法接收可选的第二个参数，表示开始搜索的位置。如果传入第二个参数，则意味着这两个方法会从指定位置向着字符串末尾搜索，忽略该位置之前的所有字符。

endsWith()方法接收可选的第二个参数，表示应该当作字符串末尾的位置。如果不提供这个参数，那么默认就是字符串长度。如果提供这个参数，那么就好像字符串只有那么多字符一样。

#### trim()方法
ECMAScript 在所有字符串上都提供了 trim()方法。这个方法会创建字符串的一个副本，删除前、后所有空格符，再返回结果，原始字符串不受影响。

另外，trimLeft()和 trimRight()方法分别用于从字符串开始和末尾清理空格符。

#### repeat()方法
ECMAScript 在所有字符串上都提供了 repeat()方法。这个方法接收一个整数参数，表示要将字符串复制多少次，然后返回拼接所有副本后的结果。

#### padStart()和 padEnd()方法
padStart()和 padEnd()方法会复制字符串，如果小于指定长度，则在相应一边填充字符，直至满足长度条件。这两个方法的第一个参数是长度，第二个参数是可选的填充字符串，默认为空格 (U+0020)。

可选的第二个参数并不限于一个字符。如果提供了多个字符的字符串，则会将其拼接并截断以匹配指定长度。此外，如果长度小于或等于字符串长度，则会返回原始字符串。

#### 字符串迭代与解构
字符串的原型上暴露了一个@@iterator 方法，表示可以迭代字符串的每个字符。可以像下面这样手动使用迭代器:
```js
let message = "abc";
let stringIterator = message[Symbol.iterator]();

console.log(stringIterator.next()); // {value: "a", done: false}
console.log(stringIterator.next()); // {value: "b", done: false}
console.log(stringIterator.next()); // {value: "c", done: false}
console.log(stringIterator.next()); // {value: undefined, done: true}
```

在 for-of 循环中也可以通过这个迭代器按序访问每个字符:
```js
for (const c of "abcde") {
    console.log(c);
}
// a
// b
// c
// d
// e
```

有了这个迭代器之后，字符串就可以通过解构操作符来解构了。比如，可以更方便地把字符串分割为字符数组:

```js
let message = "abcde";
console.log([...message]); // ["a", "b", "c", "d", "e"]
```

#### 字符串大小写转换
大小写转换包括 4 个方法:toLowerCase()、toLocaleLowerCase()、toUpperCase()和 toLocaleUpperCase()。

toLocaleLowerCase()和 toLocaleUpperCase()方法旨在基于特定地区实现。

#### 字符串模式匹配方法
match()方法接收一个参数，可以是一个正则表达式字符串，也可以是一个 RegExp 对象。match()方法返回的数组与 RegExp 对象的 exec()方法返回的数组是一样的:第一个元素是与整个模式匹配的字符串，其余元素则是与表达式中的捕获组匹配的字符串(如果有的话)。

search()方法接收一个参数，可以是一个正则表达式字符串，也可以是一个 RegExp 对象。search()方法返回模式第一个匹配的位置索引，如果没找到则返回-1。search() 始终从字符串开头向后匹配模式。

ECMAScript 提供了 replace()方法简化子字符串替换操作。这个方法接收两个参数，第一个参数可以是一个 RegExp 对象或一个字符串(这个字符串不会转换为正则表达式)，第二个参数可以是一个字符串或一个函数。如果第一个参数是字符串，那么只会替换第一个子字符串。要想替换所有子字符串，第一个参数必须为正则表达式并且带全局标记。第二个参数是字符串的情况下，有几个特殊的字符序列，可以用来插入正则表达式操作的值。ECMA-262 中规定了下表中的值。

|字符序列|替换文本|
|---|---|
|$$|$|
|$&|匹配整个模式的子字符串。与 RegExp.lastMatch 相同|
|$'|匹配的子字符串之前的字符串。与 RegExp.rightContext 相同|
|$`|匹配的子字符串之后的字符串。与 RegExp.leftContext 相同|
|$n|匹配第 n 个捕获组的字符串，其中 n 是 0~9。比如，$1 是匹配第一个捕获组的字符串，$2 是匹配第二个捕获组的字符串，以此类推。如果没有捕获组，则值为空字符串|
|$nn|匹配第 nn 个捕获组字符串，其中 nn 是 01~99。比如，$01 是匹配第一个捕获组的字符串，$02 是匹配第二个捕获组的字符串，以此类推。如果没有捕获组，则值为空字符串|

使用这些特殊的序列，可以在替换文本中使用之前匹配的内容。

replace()的第二个参数可以是一个函数。在只有一个匹配项时，这个函数会收到 3 个参数:与整个模式匹配的字符串、匹配项在字符串中的开始位置，以及整个字符串。在有多个捕获组的情况下，每个匹配捕获组的字符串也会作为参数传给这个函数，但最后两个参数还是与整个模式匹配的开始位置和原始字符串。这个函数应该返回一个字符串，表示应该把匹配项替换成什么。


#### localeCompare()方法
localeCompare()，这个方法比较两个字符串，返回如下 3 个值中的一个：
* 如果按照字母表顺序，字符串应该排在字符串参数前头，则返回负值。(通常是-1，具体还要看与实际值相关的实现。)
* 如果字符串与字符串参数相等，则返回 0。
* 如果按照字母表顺序，字符串应该排在字符串参数后头，则返回正值。(通常是 1，具体还要看与实际值相关的实现。)

localeCompare()的独特之处在于，实现所在的地区(国家和语言)决定了这个方法如何比较字符串。

#### HTML 方法
早期的浏览器开发商认为使用 JavaScript 动态生成 HTML 标签是一个需求。因此，早期浏览器扩展了规范，增加了辅助生成 HTML 标签的方法。下表总结了这些 HTML 方法。不过，这些方法基本上已经没有人使用了，因为结果通常不是语义化的标记。


|方法|输出|
|---|---|
|anchor(name)|`<a name="name">string</a>`|
|big()|`<big>string</big>`|
|bold()|`<b>string</b>`|
|fixed()|`<tt>string</tt>`|
|fontcolor(color)|`<font color="color">string</font>`|
|fontsize(size)|`<font size="size">string</font>`|
|italics()|`<i>string</i>`|
|link(url)|`<a href="url">string</a>`|
|small()|`<small>string</small>`|
|strike()|`<strike>string</strike>`|
|sub()|`<sub>string</sub>`|
|sup()|`<sup>string</sup>`|

## 单例内置对象
ECMA-262 对内置对象的定义是“任何由 ECMAScript 实现提供、与宿主环境无关，并在 ECMAScript 程序开始执行时就存在的对象”。

包括 Object、Array 和 String，以及 Global 和 Math。

### Global
ECMA-262 规定 Global 对象为一种兜底对象，它所针对的是不属于任何对象的属性和方法。事实上，不存在全局变量或全局函数这种东西。在全局作用域中定义的变量和函数都会变成 Global 对象的属性。包括 isNaN()、isFinite()、parseInt()和 parseFloat()等常见函数，实际上都是 Global 对象的方法。

除了这些，Global 对象上还有另外一些方法。

#### URL 编码方法
encodeURI()和 encodeURIComponent()方法用于编码统一资源标识符(URI)，以便传给浏览器。有效的 URI 不能包含某些字符，比如空格。使用 URI 编码方法来编码 URI 可以让浏览器能够理解它们，同时又以特殊的 UTF-8 编码替换掉所有无效字符。

ecnodeURI()方法用于对整个 URI 进行编码，而 encodeURIComponent()方法用于编码 URI 中单独的组件。这两个方法的主要区别是，encodeURI()不会编码属于 URL 组件的特殊字符，比如冒号、斜杠、问号、井号，而 encodeURIComponent()会编码它发现的所有非标准字符(非字母字符)。

与 encodeURI()和 encodeURIComponent()相对的是 decodeURI()和 decodeURIComponent()。decodeURI()只对使用 encodeURI()编码过的字符解码。


#### eval()方法
这个方法就是一个完整的 ECMAScript 解释器，它接收一个参数，即一个要执行的 ECMAScript(JavaScript)字符串。

当解释器发现 eval()调用时，会将参数解释为实际的 ECMAScript 语句，然后将其插入到该位置。通过 eval()执行的代码属于该调用所在上下文，被执行的代码与该上下文拥有相同的作用域链。

通过 eval()定义的任何变量和函数都不会被提升，这是因为在解析代码的时候，它们是被包含在一个字符串中的。它们只是在 eval()执行的时候才会被创建。

#### Global 对象属性 
Global 对象有很多属性，像 undefined、NaN 和 Infinity 等特殊
值都是 Global 对象的属性。此外，所有原生引用类型构造函数，比如 Object 和 Function，也都是 Global 对象的属性。下表列出了所有这些属性。

|属性|说明|
|---|---|
|undefined|特殊值 undefined|
|NaN|特殊值 NaN|
|Infinity|特殊值 Infinity|
|Object|Object 的构造函数|
|Array|Array 的构造函数|
|Function|Function 的构造函数|
|Boolean|Boolean 的构造函数|
|String|String 的构造函数|
|Number|Number 的构造函数|
|Date|Date 的构造函数|
|RegExp|RegExp 的构造函数|
|Symbol|Symbol 的伪构造函数|
|Error|Error 的构造函数|
|EvalError|EvalError 的构造函数|
|RangeError|RangeError 的构造函数|
|ReferenceError|ReferenceError 的构造函数|
|SyntaxError|SyntaxError 的构造函数|
|TypeError|TypeError 的构造函数|
|URIError|URIError 的构造函数|


#### window 对象
虽然 ECMA-262 没有规定直接访问 Global 对象的方式，但浏览器将 window 对象实现为 Global 对象的代理。因此，所有全局作用域中声明的变量和函数都变成了 window 的属性。

另一种获取 Global 对象的方式是使用如下的代码:
```js
let global = function() {
  return this;
}();
```
这段代码创建一个立即调用的函数表达式，返回了 this 的值。如前所述，当一个函数在没有明确(通过成为某个对象的方法，或者通过 call()/apply())指定 this 值的情况下执行时，this 值等于 Global 对象。因此，调用一个简单返回 this 的函数是在任何执行上下文中获取 Global 对象的通用方式。

### Math
ECMAScript 提供了 Math 对象作为保存数学公式、信息和计算的地方。Math 对象提供了一些辅助 计算的属性和方法。

#### Math 对象属性

|属性|说明|
|---|---|
|Math.E|自然对数的基数 e 的值 |
|Math.LN10|10 为底的自然对数|
|Math.LN2|2 为底的自然对数|
|Math.LOG2E|以 2 为底 e 的对数|
|Math.LOG10E|以 10 为底 e 的对数|
|Math.PI|π 的值|
|Math.SQRT1_2|1/2 的平方根|
|Math.SQRT2|2 的平方根|

#### min()和 max()方法
min()和 max()方法用于确定一组数值中的最小值和最大值。这两个方法都接收任意多个参数，使用这两个方法可以避免使用额外的循环和 if 语句来确定一组数值的最大最小值。


#### 舍入方法
用于把小数值舍入为整数的 4 个方法:Math.ceil()、Math.floor()、Math.round() 和 Math.fround()。这几个方法处理舍入的方式如下所述：
* Math.ceil()方法始终向上舍入为最接近的整数。
* Math.floor()方法始终向下舍入为最接近的整数。
* Math.round()方法执行四舍五入。
* Math.fround()方法返回数值最接近的单精度(32 位)浮点值表示。

#### random()方法
Math.random()方法返回一个 0~1 范围内的随机数，其中包含 0 但不包含 1。

可以基于如下公式使用 Math.random()从一组整数中随机选择一个数:
```js
number = Math.floor(Math.random() * total_number_of_choices + first_possible_value)
```

如果是为了加密而需要 生成随机数(传给生成器的输入需要较高的不确定性)，那么建议使用 window.crypto.getRandomValues()。

#### 其他方法
Math 对象还有很多涉及各种简单或高阶数运算的方法，下表还是总结了 Math 对象的其他方法。

|方法|说明|
|---|---|
|Math.abs(x)|返回 x 的绝对值|
|Math.exp(x)|返回Math.E的x次幂|
|Math.expm1(x)|等于 Math.exp(x) - 1|
|Math.log(x)|返回 x 的自然对数|
|Math.log1p(x)|等于 1 + Math.log(x)|
|Math.pow(x, power)|返回x的power次幂|
|Math.hypot(...nums)|返回 nums 中每个数平方和的平方根|
|Math.clz32(x)|返回 32 位整数 x 的前置零的数量|
|Math.sign(x)|返回表示 x 符号的 1、0、-0 或-1|
|Math.trunc(x)|返回 x 的整数部分，删除所有小数|
|Math.sqrt(x)|返回 x 的平方根|
|Math.cbrt(x)|返回 x 的立方根|
|Math.acos(x)|返回 x 的反余弦|
|Math.acosh(x)|返回 x 的反双曲余弦|
|Math.asin(x)|返回 x 的反正弦|
|Math.asinh(x)|返回 x 的反双曲正弦|
|Math.atan(x)|返回 x 的反正切|
|Math.atanh(x)|返回 x 的反双曲正切|
|Math.atan2(y, x)|返回 y/x 的反正切|
|Math.cos(x)|返回x的余弦|
|Math.sin(x)|返回x的正弦|
|Math.tan(x)|返回x的正切|

即便这些方法都是由 ECMA-262 定义的，对正弦、余弦、正切等计算的实现仍然取决于浏览器，因 为计算这些值的方式有很多种。结果，这些方法的精度可能因实现而异。
