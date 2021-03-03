# start
## 基础
### 安装
```
# 安装
$ pip install django

# 查看版本1
import django
print(django.get_version())

# 查看版本2
$ python -m django --version
```

### 创建项目
```
# 其中“mysite”是项目名
$ django-admin startproject mysite

# 查看目录结构，使用tree命令
$ tree mysite
mysite
├── manage.py
└── mysite
    ├── __init__.py
    ├── asgi.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

#### 目录&文件解析
* 最外层的mysite/根目录只是项目的容器，根目录的名称对项目没有影响
* manage.py是用来管理项目的命令行工具
* 里面一层的mysite/目录包含该项目，是一个纯python包
* mysite/__init__.py，空文件，表示该目录是一个python包
* mysite/settings.py，项目的配置文件
* mysite/urls.py，项目的url调度器，用于将url映射到视图函数
* mysite/asgi.py 和 mysite/wsgi.py，项目入口

> django是web框架，不是web服务器。django通过某些协议和web服务器进行交互。例如wsgi，Web Server Gateway Interface，Web服务器网关接口，这是Python语言定义的Web服务器和Web应用程序或框架之间的一种简单而通用的接口。上述的wsgi.py就是用于和web服务器交互的一个配置文件。

### 简易开发服
Django自带一个用于开发的简易服务器  
通过`python manage.py runserver`命令启动该服务器

需要注意，该服务器不能用于生产环境；该服务器会自动重新加载（例如在修改项目的代码之后）

### 创建应用
应用在Djang里面是一个python包，应用是专门负责做某件事情的网络应用程序。
```
# 所在目录
$ pwd
/Users/xxx/PycharmProjects/mysite

# 创建应用，polls为应用名
python manage.py startapp polls

# 查看应用目录结构
$ tree polls
polls
├── __init__.py
├── admin.py
├── apps.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
└── views.py
```

### 文档

[django-admin文档](https://docs.djangoproject.com/zh-hans/3.1/ref/django-admin/)