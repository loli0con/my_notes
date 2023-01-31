# 正则表达式

## reference
https://www.runoob.com/regexp/regexp-tutorial.html  
https://www.bilibili.com/video/BV1Eq4y1E79W  

https://www.cnblogs.com/Bw98blogs/p/9302482.html  
https://www.cnblogs.com/fudashi/p/6856628.html  

## 基本定义
定义1：正则表达式(Regular Expression)是一种**文本模式**，包括普通字符（例如，a 到 z 之间的字母）和特殊字符（称为"元字符"）。

定义2：正则表达式使用**单个字符串**来描述、匹配一系列匹配某个句法规则的字符串。
### 个人见解
从上面的定义抽取两个重要的关键词，“文本模式” 和 “单个字符串”，可以看出正则表达式应该是一个**字符串**，或者说是**文本**。

模式这个概念还是挺抽象的，这里展示一下维基百科的定义，然后举一个生活中的例子来描述一下。
![regex_guide+20210528162111](https://raw.githubusercontent.com/loli0con/picgo/master/images/regex_guide%2B20210528162111.png%2B2021-05-28-16-21-12)
相信每个人对九九乘法表都很熟悉，表里面的每一个元素的*形式*都是“数字a ✖️ 数字b = 数字c”，所以我们可以把上述这种*形式*提炼出来，称为一种模式。

回到正则表达式上面来，我们定义并使用某种文本模式，然后去对另一个字符串进行匹配和其他相关操作。
例如我定一个正则表达式“**主语～谓语～宾语**”，然后用这个表达式，我可以匹配“*我～爱～你*”和“*I～love～you*”这两个字符串，并实现提取**主语**、替换**主语**等相关操作。

## 入门教程
第一次入门正则表达式的，可以在b站搜寻相关培训班的教程，我看的是[韩顺平](https://www.bilibili.com/video/BV1Eq4y1E79W)的，里面有一些小错误，但是总体上我觉得还是挺适合入门的。

[p16](https://www.bilibili.com/video/BV1Eq4y1E79W?p=16)，10分钟左右：
![regex_guide+20210528163646](https://raw.githubusercontent.com/loli0con/picgo/master/images/regex_guide%2B20210528163646.png%2B2021-05-28-16-36-46)
应该写成`[3458]`或`(?:3|4|5|8)`，否则：
![regex_guide+20210528163804](https://raw.githubusercontent.com/loli0con/picgo/master/images/regex_guide%2B20210528163804.png%2B2021-05-28-16-38-05)

不怎么需要做笔记，有很多现成的笔记，收藏一下，忘了去看看就行了，用多次就很熟悉了。

## 进阶教程
入门的其实工作中大体上是够用的，或者说这些轮子遍地都是，随便找。所以掌握一些进阶的，其实也是有必要的。

这边推荐可以看看[菜鸟教程](https://www.runoob.com/regexp)的，里面基本涵盖齐全了。
先行断言和后行断言还是挺有意思的，看懂了这个，正则表达式所有的用法算是基本掌握了，剩下的引擎部分，有兴趣可以自行了解。

菜鸟教程应该是转载别人的，算是笔记的笔记了，所以我就不做笔记了，看菜鸟的就行，没时间去造轮子。

## 补充
在菜鸟教程里有一些没有提到的东西，这里补充一下。

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