# base64
在http报文（url部分使用[百分号编码](url-encode.md)）里面不支持中文，如果要传输中文则需要使用base64编码。

## 示例
```java
Base64.Encoder encoder = Base64.getEncoder();
String str = encoder.encodeToString("中文".getBytes("utf-8"));
// =?utf-8?B?str?= 是告诉浏览器以Base64进行解码
str = "=?utf-8?B?" + str + "?=";
System.out.println(str);
```
![base64+20210719153619](https://raw.githubusercontent.com/loli0con/picgo/master/images/base64%2B20210719153619.png%2B2021-07-19-15-36-20)