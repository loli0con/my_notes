# 字符编码

## 定义
字符编码（英语：Character encoding）是把 **字符集中的字符** *编码为* **指定集合中某一对象**（例如：比特模式、自然数序列、8位组或者电脉冲），以便文本在计算机中存储和通过通信网络的传递。

## 编码模型
现代编码模型分为5个层次，所用的术语列在下面：
1. 抽象字符表：解决包含的字符范围的问题，声明都有哪些字符。第一层抽象字符表是对人类理解的抽象字符的总结，明确了抽象字符范围。每个抽象字符可能字形不同（写法不同），在不同语境下字符表示的含义不同，但从字符本身的角度逻辑相同，并采用字符名称等方式唯一的标识该字符。
2. 编码字符集：解决如何用数字信号唯一的表示字符集中的每个字符。第二层编码字符集则为抽象字符编号，将抽象字符表示成数学形式，类似模电和数电的关系，因为只有数字才能进一步保存到计算机。但注意，这一层并不涉及计算机，数学编号也是人类意义上的编号，但将形式上的符号抽象为数学编号表示，是用计算机建模现实事务的关键一步。
3. 字符编码表：解决如何用某种数据类型描述字符编码后的数字信号。第三层字符编码形式是真正用计算机表示字符的第一步，这里采用计算机的抽象数据类型（码元）来表示人类对字符的抽象描述（数学编码）。
4. 字符编码方案：解决数据类型在计算机中（用字节序列）的表示方法。第四层字符编码方案则进一步将用计算机的抽象数据类型表示的字符映射到计算机真正的底层表示——字节流上。到这一层，字符已经完全转化为计算机的表示方式，计算机可以基于上述模型栈（其顺序处理的形式可以理解为栈）对字符进行读写或其他操作，并在计算机底层表示和人类的抽象字符间相互转化。
5. 传输编码语法：解决描述字符串的字节序列在传输过程中的编码与压缩问题。第五层传输编码语法是对计算机底层数据流额外的附加处理，来提升传输效率或满足传输要求。

字符编码模型可以进一步总结为一种计算机建模的通用思想：
1. 明确现实事物
2. 建模事物
3. 用计算机数据类型表示
4. 用计算机底层字节序列表示
5. 对字节序列的优化处理

### 抽象字符表 - 定义“抽象字符”
**抽象字符表**（Abstract character repertoire）是一个系统支持的所有抽象字符的集合。字符表可以是封闭的，即除非创建一个新的标准（ASCII），否则不允许添加新的符号；字符表也可以是开放的，即允许添加新的符号（Unicode）。

### 编码字符集 - 映射“抽象字符”到“码位”
**编码字符集**（CCS:Coded Character Set）是将字符集C中每个字符映射到1个坐标（整数值对：x, y）或者表示为1个非负整数N。

字符集及码位映射称为**编码字符集**。例如，在一个给定的字符表中，表示大写拉丁字母“A”的字符被赋予整数65、字符“B”是66，如此继续下去。

多个编码字符集可以表示同样的字符表，例如ISO-8859-1和IBM的代码页037和代码页500含盖同样的字符表但是将字符映射为不同的整数。

由此产生了编码空间（encoding space）的概念：简单说就是包含所有字符的表的维度。
* 可以用一对整数来描述，例如：GB 2312的汉字编码空间是94 x 94。
* 可以用一个整数来描述，例如：ISO-8859-1的编码空间是256。
* 也可以用字符的存储单元尺寸来描述，例如：ISO-8859-1是一个8比特的编码空间。

编码空间还可以用其子集来表述，如行、列、面（plane）等。编码空间中的一个位置（position）称为**码位**（code point）。一个字符所占用的码位称为码位值（code point value）。**编码字符集**就是把抽象字符映射为码位值。

#### Unicode（统一码）
Unicode，全称为Unicode标准（The Unicode Standard），其官方机构Unicode联盟所用的中文名称为统一码，又译作万国码、统一字元码、统一字符编码，是信息技术领域的业界标准，其整理、编码了世界上大部分的文字系统，使得电脑能以通用划一的字符集来处理和显示文字，不但减轻在不同编码系统间切换和转换的困扰，更提供了一种跨平台的乱码问题解决方案。

每个 Unicode 字符编码可以表示为：`U + 6个十六进制数字`，比如：'0' 表示为 '\U000030'。

Unicode 采用`平面 + 16-bit 编码`方式，每个平面的编码空间为 2^16（用'\U000030'的后四位表示，使用两个字节），共 17 个平面（用'\U000030'的前两位表示，使用一个字节），理论上能表示的字符数 = 平面数(17) × 平面编码空间大小(2^16) = 1114112。17个平面编号为0-16（0x00-0x10），如下图。

![Character-encoding+20230121231311](https://raw.githubusercontent.com/loli0con/picgo/master/images/Character-encoding%2B20230121231311.png%2B2023-01-21-23-13-12)

日常中常用的字符都定义在 0 号平面，该平面的码点表示时可以省略前两个十六进制位的平面号。平面中不是每个位置都定义了对应的字符，还有不少空间保留或作特殊用途。

每个抽象字符在 Unicode 中采用唯一且不可变的字符名称来表示，如：拉丁字母 K 在 Unicode 中的字符名称是“Latin Capital Letter K”，码位是 004B。

### 字符编码表 - 映射“码位”到“码元”
**字符编码表**（CEF:Character Encoding Form），也称为"storage format"，是将编码字符集的非负整数值（即抽象的码位）转换成有限比特长度的整型值（称为**码元**code units）的序列。码元也称“代码单元”，是指一个已编码的文本中具有最短的比特组合的单元。对于UTF-8来说，码元是8比特长；对于UTF-16来说，码元是16比特长；对于UTF-32来说，码元是32比特长。

这对于定长编码来说是个到自身的映射（null mapping），但对于变长编码来说，该映射比较复杂，把一些码位映射到一个码元，把另外一些码位映射到由多个码元组成的序列。例如，使用16比特长的存储单元保存数字信息，系统每个单元只能够直接表示从0到65,535的数值，但是如果使用多个16位单元就能够表示更大的整数。这就是CEF的作用，它可以把Unicode从0到140万的码空间范围的每个码位映射到单个或多个在0到65,5356范围内的码值。最简单的**字符编码表**就是单纯地选择足够大的单位，以保证编码字符集中的所有数值能够直接编码（一个码位对应一个码值）。这对于能够用使用八比特组来表示的编码字符集（如多数传统的非CJK的字符集编码）是合理的，对于能够使用十六比特来表示的编码字符集（如早期版本的Unicode）来说也足够合理。但是，随着编码字符集的大小增加（例如，现在的Unicode的字符集至少需要21位才能全部表示），这种直接表示法变得越来越没有效率，并且很难让现有计算机系统适应更大的码值。因此，许多使用新近版本Unicode的系统，或者将Unicode码位对应为可变长度的8位字节序列的UTF-8，或者将码位对应为可变长度的16位序列的UTF-16。

第二层的**编码字符集**和第三层的**字符编码表**是多对多关系：
1. 一种编码方式也可应用多种字符集，如：EUC 编码方式可以用于 GB 2312，也可以用于 JIS X 0208（一种日语字符集编码标准）
2. 一种字符集何以对应多种编码方式，如：Unicode 对应UTF-8、UTF-16、UTF-32等编码方法

#### UTF
Unicode的实现方式不同于编码方式。一个字符的Unicode编码确定，但是在实际传输过程中，由于不同系统平台的设计不一定一致，以及出于节省空间的目的，对Unicode编码的实现方式有所不同。Unicode的实现方式称为Unicode转换格式（Unicode Transformation Format，简称为UTF）。

##### UTF8
UTF-8 是 Unicode 的一种边长编码，码元为 8 bit，采用 1 - 4 个码元（字节）表示一个字符，根据字符码位值的不同变换表示长度。编码规则如下：
|Unicode 十六进制码点范围|UTF-8 二进制|
|---|---|
|0000 0000 - 0000 007F|0xxxxxxx|
|0000 0080 - 0000 07FF|110xxxxx 10xxxxxx|
|0000 0800 - 0000 FFFF|1110xxxx 10xxxxxx 10xxxxxx|
|0001 0000 - 0010 FFFF|11110xxx 10xxxxxx 10xxxxxx 10xxxxxx|

1. 对于单个字节的字符，第一位设为 0，后面的 7 位对应这个字符的 Unicode 码点。因此，对于英文中的 0 - 127 号字符，与 ASCII 码完全相同。这意味着 UTF-8 完全兼容过去用 ASCII 编码的文档。
2. 对于需要使用 N 个字节来表示的字符（N > 1），第一个字节的前 N 位都设为 1，第 N + 1 位设为 0，剩余的 N - 1 个字节的前两位都设位 10，剩下的二进制位则使用这个字符的 Unicode 码点来填充。

##### UTF16
UTF-16 则采用 16bit（两个字节） 码元，编码规则为：基本平面的字符占用 2 个字节，辅助平面的字符占用 4 个字节。而确定是用一个码元还是两个码元是通过基本平面中 U+D800 - U+DFFF 的编码留白实现的。

辅助平面的字符位共有 2^20 个，因此表示这些字符至少需要 20 个二进制位。UTF-16 将这 20 个二进制位分成两半，前 10 位映射在 U+D800 到 U+DBFF（空间大小 2^10），称为高位（H），后 10 位映射在 U+DC00 到 U+DFFF（空间大小 2^10），称为低位（L）。这意味着，一个辅助平面的字符，被拆成两个基本平面的字符表示。

### 字符编码方案 - 映射“码元”到“8位字节序列”
**字符编码方案**（CES:Character Encoding Scheme），也称作"serialization format"。将定长的整型值（即码元）映射到8位字节序列，以便编码后的数据的文件存储或网络传输。抽象字符的码位值可以通过具体数据类型的码元表示，但由于这些数据类型可能需要多个字节才能表示，**字符编码方案**主要解决码元如何用字节序列表示的问题。在使用Unicode的场合，使用一个简单的字符来指定字节顺序是大端序或者小端序（但对于UTF-8来说并不需要专门指明字节序）。然而，有些复杂的字符编码机制（如ISO/IEC 2022）使用控制字符转义序列在几种编码字符集或者用于减小每个单元所用字节数的压缩机制（如SCSU、BOCU和Punycode）之间切换。

#### 大小端
计算机硬件有两种储存数据的方式：大端字节序（big endian）和小端字节序（little endian）。以汉字严为例，Unicode 码是`4E25`，需要用两个字节存储，一个字节是`4E`，另一个字节是`25`。存储的时候，`4E`在前，`25`在后，这就是 Big endian 方式；`25`在前，`4E`在后，这是 Little endian 方式。同理，`0x1234567`的大端字节序和小端字节序的写法如下图。

![Character-encoding+20230122145948](https://raw.githubusercontent.com/loli0con/picgo/master/images/Character-encoding%2B20230122145948.png%2B2023-01-22-14-59-48)

为什么会有小端字节序？答案是，计算机电路先处理低位字节，效率比较高，因为计算都是从低位开始的。所以，计算机的内部处理都是小端字节序。（人类进行加减乘除运算的时候也是从低位开始计算的，执行两个大数目运算的时候，计算机运算的过程和人类类似，先执行低位运算，临时保存进位情况后再进行高一位的运算，直到最高一位的运算结束）但是，人类还是习惯读写大端字节序。（在进行读写的时候，是按照“从左往右，从上往下”的方式进行的）所以，除了计算机的内部处理，其他的场合几乎都是大端字节序，比如网络传输和文件储存。

解决字节序问题，一般有两种方案：
1. 强制规定使用某种字节序。如网络传输强制要求网络字节序使用大端序。
2. 使用字节序标记说明当前使用的字节序。字符集编码一般采用这种方案。Unicode 编码方案中有个叫 BOM（Byte Order Mark）的东西，就是用来做这事的。Unicode 规范定义，每一个文件的最前面分别加入一个表示编码顺序的字符，这个字符的名字叫做"零宽度非换行空格"（zero width no-break space），用`FEFF`表示。这正好是两个字节，而且FF比FE大1。如果一个文本文件的头两个字节是`FE FF`，就表示该文件采用大头方式；如果头两个字节是`FF FE`，就表示该文件采用小头方式。

当然，对于码元为单字节的情况下，不存在字节序问题，如 UTF-8，这也是 UTF-8 广泛使用的原因之一。但一些 UTF-8 文件也存在 BOM 头，但这不是必须的，只是用来标识该文件采用 UTF-8 编码。


### 传输编码语法 - 处理“8位字节序列”
**传输编码语法**（transfer encoding syntax），用于处理上一层次的字符编码方案提供的字节序列。一般其功能包括两种：一是把字节序列的值映射到一套更受限制的值域内，以满足传输环境的限制，例如Email传输时Base64或者quoted-printable，都是把8位的字节编码为7位长的数据；另一是压缩字节序列的值，如LZW或者行程长度编码等无损压缩技术。



## 编码历史

### 背景
全世界很多个国家都在为自己的文字编码，并且互不想通，不同的语言字符编码值相同却代表不同的符号（例如：韩文编码EUC-KR中“한국어”的编码值正好是汉字编码GBK中的“茄惫绢”）。因此，同一份文档，拷贝至不同语言的机器，就可能成了乱码，于是人们就想：我们能不能定义一个超大的字符集，它可以容纳全世界所有的文字字符，再对它们统一进行编码，让每一个字符都对应一个不同的编码值，从而就不会再有乱码了。

### Unicode与ISO 10646
如果说“各个国家都在为自己文字独立编码”是百家争鸣，那么“建立世界统一的字符编码”则是一统江湖，谁都想来做这个武林盟主。早前就有两个机构试图来做这个事：
1. 国际标准化组织（ISO），他们于1984年创建ISO/IEC JTC1/SC2/WG2工作组，试图制定一份“通用字符集”（Universal Character Set，简称UCS），并最终制定了**ISO 10646标准**。
(2) 统一码联盟，他们由Xerox、Apple等软件制造商于1988年组成，并且开发了**Unicode标准**（The Unicode Standard，这个前缀Uni很牛逼哦---Unique, Universal, and Uniform）。

1991年前后，两个项目的参与者都认识到，世界不需要两个不兼容的字符集。于是，它们开始合并双方的工作成果，并为创立一个单一编码表而协同工作。**从Unicode 2.0开始**，Unicode采用了与ISO 10646-1相同的字库和字码；ISO也承诺，ISO 10646将不会替超出U+10FFFF的UCS-4编码赋值，以使得**两者保持一致**。两个项目仍都独立存在，并独立地公布各自的标准。不过由于Unicode这一名字比较好记，因而它使用更为广泛。

### UTF-32与UCS-4
在Unicode与ISO 10646合并之前，ISO 10646标准为“通用字符集”（UCS）定义了一种31位的编码形式（即**UCS-4**），其编码固定占用4个字节，编码空间为0x00000000~0x7FFFFFFF（可以编码20多亿个字符）。

UCS-4有20多亿个编码空间，但实际使用范围并不超过`0x10FFFF`，并且为了兼容Unicode标准，ISO也承诺将不会为超出`0x10FFFF`的UCS-4编码赋值。由此**UTF-32**编码被提出来了，它的编码值与UCS-4相同，只不过其编码空间被限定在了`0~0x10FFFF`之间。因此也可以说：**UTF-32是UCS-4的一个子集**。

### UTF-16与UCS-2
除了UCS-4，ISO 10646标准为“通用字符集”（UCS）定义了一种16位的编码形式（即**UCS-2**），其编码固定占用2个字节，它包含65536个编码空间（可以为全世界最常用的63K字符编码，为了兼容Unicode，0xD800-0xDFFF之间的码位未使用）。例：“汉”的UCS-2编码为6C49。

但俩个字节并不足以正真地“一统江湖”（a fixed-width 2-byte encoding could not encode enough characters to be truly universal），于是**UTF-16**诞生了，与UCS-2一样，它使用两个字节为全世界最常用的63K字符编码，不同的是，它使用4个字节对不常用的字符进行编码。UTF-16属于变长编码。

UTF-16中如何对“辅助平面”进行编码呢？Unicode的码位区间为0\~0x10FFFF，除“基本多语言平面”外，还剩0xFFFFF个码位（并且其值都大于或等于0x10000）。对于“辅助平面”内的字符来说，如果用它们在Unicode中码位值减去0x10000，则可以得到一个0\~0xFFFFF的区间（该区间中的任意值都可以用一个20-bits的数字表示）。该数字的前10位(bits)加上0xD800，就得到UTF-16四字节编码中的前两个字节；该数字的后10位(bits)加上0xDC00，就得到UTF-16四字节编码中的后两个字节。

### UTF-8、UTF-16、UTF-32、UCS-2、UCS-4
|对比|UTF-8|UTF-16|UTF-32|UCS-2|UCS-4|
|---|---|---|---|---|---|
|编码空间|0-10FFFF|0-10FFFF|0-10FFFF|0-FFFF|0-7FFFFFFF|
|最少编码字节数|1|2|4|2|4|
|最多编码字节数|4|4|4|2|4|
|是否依赖字节序|否|是|是|是|是|

## Java编码
Java的char类型使用固定两个字节表示，实现的是UCS-2标准，所以Java中单个char类型描述的字符是有限的，单个char只能描述unicode中的BMP范围的码位，也就意味着BMP范围外的字符char是无法表示的。char脱离了原有字符的语意，理解为码元更加合适。String本意为char类型的序列，如今理解为码元的序列也许更合适。

## 参考
* [深入理解“字符编码模型”](https://www.cnblogs.com/zhe-si/p/16631919.html)
* [字符编码笔记：ASCII，Unicode 和 UTF-8](https://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
* [理解字节序](https://www.ruanyifeng.com/blog/2016/11/byte-order.html)
* [细说：Unicode, UTF-8, UTF-16, UTF-32, UCS-2, UCS-4](https://www.cnblogs.com/malecrab/p/5300503.html)
* [Java中关于Char存储中文到底是2个字节还是3个还是4个？ - 子非鱼的回答](https://www.zhihu.com/question/279539793/answer/1830657398)