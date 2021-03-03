# URLconf
## 简介
URLconf用于把某个url映射到某个视图函数.

URL模式（简单的正则表达式） 到 python函数（视图函数） 的简单映射。

## 代码
```python3
from django.urls import path,include

urlpatterns = [
    path(...),
    include(...),
    ...
]
```

## 解析
### urlpatterns
规则列表，从第一项开始匹配，直到找到匹配的项。

不匹配GET/POST等参数，不匹配域名

### path函数
path(route,view,kwargs=None,name=None)
* route：匹配URL的准则（类似正则表达式）
* view：调用的视图函数（被捕获的参数也将传递给视图函数）
* kwargs：任意个被传递给目标函数的键值对
* name：给这条规则取名，在项目的其他地方通过该名称引用该规则

### include函数
用于包含其他URLconf模块，将一部分的url分发到其他URLconf模块去处理。

### 文档
[URLconf的简单例子](https://docs.djangoproject.com/zh-hans/3.1/intro/tutorial01/#write-your-first-view)  
[django.urls函数](https://docs.djangoproject.com/zh-hans/3.1/ref/urls/)  
[url调度详解](https://docs.djangoproject.com/zh-hans/3.1/topics/http/urls/)  
