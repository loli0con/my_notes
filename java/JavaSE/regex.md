# 正则表达式

## 基本定义
正则表达式(Regular Expression)是一种**文本模式**，包括普通字符（例如，a 到 z 之间的字母）和特殊字符（称为"元字符"）。正则表达式是用**字符串**描述的一个**匹配规则**，使用正则表达式可以快速判断给定的字符串是否符合匹配规则，并进行查找、提取、分割、替换等操作。



## 正则表达式的字符

### 普通字符
普通字符包括没有显式指定为元字符的所有可打印和不可打印字符，这包括所有大写和小写字母、所有数字、所有标点符号和一些其他符号。正则表达式支持的普通字符：
|字符|说明|
|---|---|
|`x`|字符x（x可代表任何合法的字符）|
|`\0mnn`|八进制数0mnn所表示的字符|
|`\xhh`|十六进制值0xhh所表示的字符|
|`\uhhhh`|十六进制0xhhhh所表示的Unicode字符|
|`\t`|制符表（'\u0009'）|
|`\v`|垂直制表符（'\u000B'）|
|`\n`|换行符（'\u000A'）|
|`\r`|回车符（'\u000D'）|
|`\f`|换页符（'\u000C'）|
|`\a`|报警符（'\u0007'）|
|`\e`|Escape符（'\u001B'）|
|`\cx`|x对应的控制符。例如，\cM匹配Ctrl-M。x值必须为A～Z或a～z之一。|

### 特殊字符
正则表达式中有一些特殊字符，这些特殊字符在正则表达式中有其特殊的用途。如果需要匹配这些特殊字符，就必须首先将这些字符转义，也就是在前面添加一个反斜线（\）。正则表达式支持的部分特殊字符如下：
|字符|说明|
|---|---|
|`\`|用于转译下一个字符，或指定八进制、十六进制字符。要匹配\字符本身，请使用\\\ |
|`()`|标记子表达式的开始和结束位置。子表达式可以获取供以后使用。要匹配这些字符，请使用\\(和\\)|
|`[]`|用于确定中括号表达式的开始和结束位置。要匹配这些字符，请使用\\[和\\]|

由于Java字符串中反斜杠本身需要转义，因此两个反斜杠（\\）实际上相当于一个（前一个用于转义）。

#### 限定符
限定符用来指定正则表达式的一个给定组件必须要出现多少次才能满足匹配。
|字符|说明|
|---|---|
|`\*`|指定前面子表达式可以出现零次或多次。要匹配*字符本身，请使用\\\*|
|`+`|指定前面子表达式可以出现一次或多次。要匹配+字符本身，请使用\\+|
|`?`|指定前面子表达式可以出现零次或一次。要匹配?字符本身，请使用\\?|
|`{}`|用于标记前面子表达式的出现频度。要匹配这些字符，请使用\\{和\\}。|
|`{n}`|n是一个非负整数。匹配确定的n次。|
|`{n,}`|n是一个非负整数。至少匹配n次。|
|`{n,m}`|m和n均为非负整数，其中n<=m。最少匹配n次且最多匹配m次（闭区间）。在逗号和两个数之间不能有空格。|

`*`和`+`限定符都是贪婪的，因为它们会尽可能多的匹配文字，只有在它们的后面加上一个`?`就可以实现非贪婪或最小匹配。

#### 通配符
正则表达式中的“通配符”是可以匹配多个字符的特殊字符，它被称为预定义字符，正则表达式支持的预定义字符如下：
|预定义字符|说明|
|---|---|
|`.`|匹配除换行符\n之外的任何单字符。要匹配.字符本身，请使用\\.|
|`\d`|匹配0～9的所有数字|
|`\D`|匹配非数字|
|`\s`|匹配所有的空白字符，包括空格、制符表、回车符、换页符、换行符等|
|`\S`|匹配所有的非空白字符|
|`\w`|匹配所有的单词字符，包括0～9所有数字、26个英文字符和下划线（_）|
|`\W`|匹配所有的非单词字符|

#### 方括号表达式
若只想匹配**某几个字符**，或者匹配除**某几个字符**之外的所有字符，此时就需要使用方括号表达式，方括号表达式有如下所示的几种形式：
|方括号表达式|说明|
|---|---|
|表示枚举|例如\[abc]，表示a、b、c其中任意一个字符；\[gz]，表示g、z其中任意一个字符|
|表示范围：`-`|例如\[a-f]，表示a～f范围内的任意字符；\[\\u0041-\\u0056]，表示十六进制字符\\u0041到\\u0056范围的字符。范围可以和枚举结合使用，如\[a-cx-z]，表示a～c、x～z范围内的任意字符|
|表示求否：`^`|例如\[^abc]，表示非a、b、c的任意字符；\[^a~f]，表示不是a～f范围内的任意字符|
|表示“与”运算：`&&`|例如\[a-z&&\[def]]，求a～z和\[def]的交集，表示d、e、f；\[a-z&&\[^bc]]，a～z范围内的所有字符，除了b和c之外，即\[ad-z]；\[a-z&&\[^m-p]]，a-z范围内的所有字符，除m～p范围之外的字符，即\[a-lq-z]|
|表示“并”运算|并运算与前面的枚举类似。例如\[a-d\[m-p]]，表示\[a-dm-p]|

#### 圆括号表达式
正则表示支持圆括号表达式，用于将多个表达式组成一个子表达式，圆括号中可以使用或运算符(`|`)。圆括号表达式表示捕获分组，圆括号表达式会把每个分组里的匹配的值保存起来，可以使用`$0…$9`属性从结果"匹配"集合中检索捕获的匹配，多个匹配值也可以通过相应的方法进行查看。

但用圆括号会有一个副作用，使相关的匹配会被缓存，此时可用`?:`放在第一个选项前来消除这种副作用。

##### 正向预查和负向预查
还有两个非捕获元是`?=`和`?!`，这两个还有更多的含义。前者为正向预查，在任何开始匹配圆括号内的正则表达式模式的位置来匹配搜索字符串，后者为负向预查，在任何开始不匹配该正则表达式模式的位置来匹配搜索字符串。
|字符|说明|
|---|---|
|`(?=正则表达式)`|执行正向预测先行搜索的子表达式，该表达式匹配处于匹配 pattern 的字符串的起始点的字符串。它是一个非捕获匹配，即不能捕获供以后使用的匹配。例如，'Windows (?=95\|98\|NT\|2000)' 匹配"Windows 2000"中的"Windows"，但不匹配"Windows 3.1"中的"Windows"。预测先行不占用字符，即发生匹配后，下一匹配的搜索紧随上一匹配之后，而不是在组成预测先行的字符后。|
|`(?!正则表达式)`|执行反向预测先行搜索的子表达式，该表达式匹配不处于匹配 pattern 的字符串的起始点的搜索字符串。它是一个非捕获匹配，即不能捕获供以后使用的匹配。例如，'Windows (?!95\|98\|NT\|2000)' 匹配"Windows 3.1"中的 "Windows"，但不匹配"Windows 2000"中的"Windows"。预测先行不占用字符，即发生匹配后，下一匹配的搜索紧随上一匹配之后，而不是在组成预测先行的字符后。|

[正则表达式的先行断言(lookahead)和后行断言(lookbehind)](https://www.runoob.com/w3cnote/reg-lookahead-lookbehind.html)

#### 边界匹配符
正则表达式支持如下几个边界匹配符：
|边界匹配符|说明|
|---|---|
|`^`|行的开头，匹配输入字符串开始的位置。|
|`$`|行的结尾，匹配输入字符串结尾的位置。|
|`\b`|单词的边界，即字与空格间的位置。|
|`\B`|非单词的边界|
|`\A`|输入的开头|
|`\G`|前一个匹配的结尾|
|`\Z`|输入的结尾，仅用于最后的结束符|
|`\z`|输入的结尾|

#### 数量标识符
正则表达式还提供了数量标识符，正则表达式支持的数量标识符如下：
* Greedy（贪婪模式）：数量表示符默认采用贪婪模式，除非另有表示。贪婪模式的表达式会一直匹配下去，直到无法匹配为止。
* Reluctant（勉强模式）：用问号后缀（`?`）表示，它只会匹配最少的字符。也称为最小匹配模式。

### 运算符优先级
正则表达式从左到右进行计算，并遵循优先级顺序，下表从最高到最低说明了各种正则表达式运算符的优先级顺序：
|运算符|描述|
|---|---|
|`\\`|转义符|
|`()`, `(?:)`, `(?=)`, `[]`|圆括号和方括号|
|`*`, `+`, `?`, `{n}`, `{n,}`, `{n,m}`|限定符|
|`^`, `$`, `\任何元字符`、`任何字符`|定位点和序列（即：位置和顺序|
|`\|`|替换，"或"操作。字符具有高于替换运算符的优先级，使得"m\|food"匹配"m"或"food"。若要匹配"mood"或"food"，请使用括号创建子表达式，从而产生"(m\|f)ood"。|


## 使用正则表达式

### Pattern和Matcher
Pattern对象是正则表达式编译后在内存中的表示形式，正则表达式字符串必须先被编译为Pattern对象，然后再利用该Pattern对象创建对应的Matcher对象。执行匹配所涉及的状态保留在Matcher对象中，多个Matcher对象可共享同一个Pattern对象。

#### 创建示例
```Java
// 将一个字符串编译成Pattern对象
Pattern p = Pattern.compile("正则表达式字符串");
// 使用Pattern对象创建Matcher对象
Mather m = p.matcher("目标字符串");
// 使用Matcher对象进行进一步操作
```

### Matcher方法
Matcher类提供了如下几个常用方法：
* groupCount()：查看表达式有多少个分组，groupCount方法返回一个int值，表示matcher对象当前有多个捕获组。还有一个特殊的组（group(0)），它总是代表整个表达式。该组不包括在groupCount的返回值中。
* group()：返回上一次与Pattern匹配的子串。
* 索引方法：提供了有用的索引值，精确表明输入字符串中在哪能找到匹配
  * start()：返回上一次与Pattern匹配的子串在目标字符串中的开始位置。
  * start(int group)： 返回上一次的匹配操作期间，由给定组所捕获的子序列的初始索引。
  * end()：返回上一次与Pattern匹配的子串在目标字符串中的结束位置加1。
  * end(int group)：返回上一次的匹配操作期间，由给定组所捕获子序列的最后字符之后的偏移量。
* 查找方法：用来检查输入字符串并返回一个布尔值，表示是否找到该模式
  * lookingAt()：返回目标字符串前面部分与Pattern是否匹配。
  * find()：返回目标字符串中是否包含与Pattern匹配的子串。
  * matches()：返回整个目标字符串与Pattern是否匹配。
* 替换方法：替换输入字符串里文本
  * public Matcher appendReplacement(StringBuffer sb, String replacement)：实现非终端添加和替换步骤。
  * public StringBuffer appendTail(StringBuffer sb)：实现终端添加和替换步骤。
  * public String replaceAll(String replacement)：替换模式与给定替换字符串相匹配的输入序列的每个子序列。
  * public String replaceFirst(String replacement)：替换模式与给定替换字符串匹配的输入序列的第一个子序列。
  * public static String quoteReplacement(String s)：返回指定字符串的字面替换字符串。这个方法返回一个字符串，就像传递给Matcher类的appendReplacement 方法一个字面字符串一样工作。
* reset()：将现有的Matcher对象应用于一个新的字符序列。

#### 查找示例
通过Matcher类的find()和group()方法可以从目标字符串中依次取出特定子串（匹配正则表达式的子串），find()方法依次查找字符串中与Pattern匹配的子串，一旦找到对应的子串，下次调用find()方法时将接着向下查找。find()方法还可以传入一个int类型的参数，带int参数的find()方法将从该int索引处向下搜索。start()和end()方法主要用于确定子串在目标字符串中的位置。

#### 替换示例
```Java
import java.util.regex.Matcher;
import java.util.regex.Pattern;
 
public class RegexMatches
{
   private static String REGEX = "a*b";
   private static String INPUT = "aabfooaabfooabfoobkkk";
   private static String REPLACE = "-";
   public static void main(String[] args) {
      Pattern p = Pattern.compile(REGEX);
      // 获取 matcher 对象
      Matcher m = p.matcher(INPUT);
      StringBuffer sb = new StringBuffer();
      while(m.find()){
         m.appendReplacement(sb,REPLACE);
      }
      m.appendTail(sb);
      System.out.println(sb.toString());
      // 输出结果：-foo-foo-foo-kkk
   }
}
```




## 补充

### 修饰符的用法
![regex_guide+20210530154359](https://raw.githubusercontent.com/loli0con/picgo/master/images/regex_guide%2B20210530154359.png%2B2021-05-30-15-44-00)

```Java
String content = "a11c8abcABC";
String regStr = "abc";
String regStr1 = "(?i)abc";
String regStr2 = "a(?i)bc";
String regStr3 = "a((?i)b)c";

Pattern pattern = Pattern.compile(regStr);
Pattern pattern1 = Pattern.compile(regStr,Pattern.CASE_INSENSITIVE);
Matcher matcher = pattern.matcher(content);

while (matcher.find()) {
    System.out.println("找到  " + matcher.group());
}
```

### 转义
大多数特殊符号在中括号`[]`里时，都不需要转义。具体可以参考idea的提示，并在使用前自行测试。

### 命名分组
语法：`(?<分组名>正则表达式)`
![regex_guide+20210530160743](https://raw.githubusercontent.com/loli0con/picgo/master/images/regex_guide%2B20210530160743.png%2B2021-05-30-16-07-45)


## 玩具
之前工作的时候写的，用于匹配markdown语法，并提取出相关的信息。
```Python
import json
import re
from pprint import pprint

# 带换行好看的
LINE_PATTERN = r"^(\w+)(((\[)|(\{)|(\(\()|(\())(\w+)((?(4)\])(?(5)\})(?(6)\)\))(?(7)\)(?!\)))))?-" \
               r"-((\w+)--)?>(\|(\w+)\|)?(\w+)(?#comment-" \
               r"-comment)(((\[)|(\{)|(\(\()|(\())(\w+)((?(17)\])(?(18)\})(?(19)\)\))(?(20)\)(?!\)))))?$"
LINE_REGEX_OBJECT = re.compile(LINE_PATTERN)

# 原始的，容易复制粘贴的
LINE_PATTERN = '^(\\w+)(((\\[)|(\\{)|(\\(\\()|(\\())(\\w+)((?(4)\\])(?(5)\\})(?(6)\\)\\))(?(7)\\)(?!\\)))))?--((\\w+)--)?>(\\|(\\w+)\\|)?(\\w+)(?#comment--comment)(((\\[)|(\\{)|(\\(\\()|(\\())(\\w+)((?(17)\\])(?(18)\\})(?(19)\\)\\))(?(20)\\)(?!\\)))))?$'

ELEMENT_TYPE_MAPPING = {
    "]": "rectangle",
    "}": "diamond",
    ")": "rounded_rectangle",
    "))": "circle"
}


def analysis(line):
    regex_match_object = LINE_REGEX_OBJECT.match(line)
    if not regex_match_object:
        print("illegal line")
        return False

    result_groups = regex_match_object.groups()
    print(result_groups)

    step_element = {
        "step_id": result_groups[0],
        "step_alias": result_groups[7],
        "step_type": ELEMENT_TYPE_MAPPING[result_groups[8]],
        "next": {
            "next_id": result_groups[13],
            "road_name": result_groups[10] or result_groups[12],
            "backup": {
                "backup_next_alias": result_groups[20],
                "backup_next_type": ELEMENT_TYPE_MAPPING[result_groups[21]]
            }
        }
    }
    return step_element


if __name__ == '__main__':
    e = analysis("input_10001((i1))--No-->|Yes|output_200001((o1))")  # 要匹配的内容
    print(json.dumps(e))

```