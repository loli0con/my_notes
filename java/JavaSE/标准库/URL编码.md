# URL编码
URL编码是浏览器发送数据给服务器时使用的编码，它通常附加在URL的参数部分。之所以需要URL编码，是因为出于兼容性考虑，很多服务器只识别ASCII字符。

URL编码的规则：
* 如果字符是`A`~`Z`，`a`~`z`，`0`~`9`以及`-`、`_`、`.`、`*`，则保持不变；
* 如果是其他字符，先转换为UTF-8编码，然后对每个字节以%XX表示。

Java的`URLEncoder`类可以对任意字符串进行URL编码，Java标准库的`URLDecoder`类可以解码**URL编码的字符串**：
```Java
import java.net.URLDecoder;
import java.nio.charset.StandardCharsets;

public class Main {
    public static void main(String[] args) {
        String encoded = URLEncoder.encode("中文!", StandardCharsets.UTF_8);
        System.out.println(encoded); // %E4%B8%AD%E6%96%87%21

        String decoded = URLDecoder.decode("%E4%B8%AD%E6%96%87%21", StandardCharsets.UTF_8);
        System.out.println(decoded);
    }
}
```
和标准的URL编码稍有不同，URLEncoder把空格字符编码成`+`，而现在的URL编码标准要求空格被编码为%20，不过，服务器都可以处理这两种情况。