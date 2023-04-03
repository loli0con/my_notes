# nginx

## 配置文件

### 基本介绍
Nginx配置文件nginx.conf位于Nginx主目录下的conf文件夹，默认的配置也存放在此。在nginx.conf的注释符号为：`#`。

### 组成部分
nginx.conf配置文件一共由三部分组成，分别为**全局块**、**events块**和**http块**。在http块中，又包含http全局块、多个server块。每个server块中，可以包含server全局块和多个location块。在同一配置块中嵌套的配置块，各个之间不存在次序关系。
1. 全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
2. events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
3. http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
4. server块：配置虚拟主机的相关参数。
5. location块：配置请求的路由，以及各种页面的处理情况。

整个配置文件的结构大致如下：
```conf
#全局块
...

#events块
events {
   ...
}

#http块
http {
    #http全局块
    ...

    #server块
    server {
        #server全局块
        ...

        #location块
        location [PATTERN] {
            ...
        }
        location [PATTERN] {
            ...
        }
    }
    server {
      ...
    }

    #http全局块
    ...
}
```

### 作用域和优先级
配置文件支持大量可配置的指令，绝大多数指令不是特定属于某一个块的。同一个指令放在不同层级的块中，其作用域也不同，一般情况下，高一级块中的指令可以作用于自身所在的块和此块包含的所有低层级块。如果某个指令在两个不同层级的块中同时出现，则采用“就近原则”，即以较低层级块中的配置为准。比如，某指令同时出现在http全局块中和server块中，并且配置不同，则应该以server块中的配置为准。

### 全局变量
Nginx提供了以$符号开头的全局变量：
* `$args`: 请求中的参数;
* `$content_length`: HTTP请求信息里的"Content-Length";
* `$content_type`: 请求信息里的"Content-Type";
* `$document_root`: 针对当前请求的根路径设置值;
* `$document_uri:` 与$uri相同;
* `$host`: 请求信息中的"Host"，如果请求中没有Host行，则等于设置的服务器名;
* `$limit_rate`: 对连接速率的限制;
* `$request_method`: 请求的方法，比如"GET"、"POST"等;
* `$remote_addr`: 客户端地址;
* `$remote_port`: 客户端端口号;
* `$remote_user`: 客户端用户名，认证用;
* `$request_filename`: 当前请求的文件路径名
* `$request_body_file`: 当前请求的文件
* `$request_uri`: 请求的URI，带查询字符串;
* `$query_string`: 与$args相同;
* `$scheme`: 所用的协议，比如http或者是https;
* `$server_protocol`: 请求的协议版本，"HTTP/1.0"或"HTTP/1.1";
* `$server_addr`: 服务器地址;
* `$server_name`: 请求到达的服务器名;
* `$server_port`: 请求到达的服务器端口号;
* `$uri`: 请求的URI，可能和最初的值有不同，比如经过重定向之类的。



## 配置详解

### 全局块
全局块是默认配置文件从开始到events块之间的一部分内容，主要设置一些影响Nginx服务器整体运行的配置指令。通常包括配置运行Nginx服务器的用户（组）、允许生成的worker process数、Nginx进程PID存放路径、日志的存放路径和类型以及配置文件引入等。

#### 用户（组）
```conf
user [user] [group];

# user nobody nobody;
```
* 指定可以运行nginx服务的用户和用户组，只能在全局块配置
* 将user指令注释掉，或者配置成nobody的话所有用户都可以运行
* user指令在Windows上不生效，如果指定了具体用户和用户组会报警告

#### worker process
```conf
worker_processes number | auto;

# worker_processes 1;
```
* 指定工作进程数，可以是具体的进程数，也可使用自动模式，只能在全局块配置
* Nginx除了工作进程外，还有一个master主进程

#### pid文件
```conf
pid logs/nginx.pid;
```
* 指定pid文件存放的路径，只能在全局块配置

#### 错误日志
```conf
error_log [path] [debug | info | notice | warn | error | crit | alert | emerg];

# error_log  logs/error.log  notice;
# error_log  logs/error.log  info;
```
* 指定错误日志的路径和日志级别，可以在全局块、http块、server块以及location块中配置
* 其中debug级别的日志需要编译时使用--with-debug开启debug开关





### events块
events块涉及的指令主要影响Nginx服务器与用户的网络连接。常用到的设置包括是否开启对多worker process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型处理连接请求，每个worker process可以同时支持的最大连接数等。

这一部分的指令对Nginx服务器的性能影响较大，在实际配置中应该根据实际情况灵活调整。

#### 连接序列化
```conf
accept_mutex on | off;
```
* 当某一时刻只有一个网络连接到来时，多个睡眠进程会被同时叫醒，但只有一个进程可获得连接。如果每次唤醒的进程数目太多，会影响一部分系统性能。在Nginx服务器的多进程下，就有可能出现这样的问题。开启的时候，将会对多个Nginx进程接收连接进行序列化，防止多个进程对连接的争抢
* 默认是开启状态，只能在events块中进行配置

#### 接受多连接
```conf
multi_accept on | off;
```
* 如果multi_accept为off了，nginx一个工作进程只能同时接受一个新的连接。否则，一个工作进程可以同时接受所有的新连接
* 如果nginx使用kqueue连接方法，那么这条指令会被忽略，因为这个方法会报告在等待被接受的新连接的数量
* 默认是off状态，只能在event块配置

#### 网络IO模型
```conf
use [ select | poll | kqueue | epoll | rtsig | /dev/poll | eventport];
```
* 指定使用哪种网络IO模型，一般操作系统不是支持上面所有模型的
* 只能在events块中进行配置

#### 每个worker的最大连接数
```conf
worker_connections 1024;
```
* 设置允许每一个worker process同时开启的最大连接数，当每个工作进程接受的连接数超过这个值时将不再接收连接
* 当所有的工作进程都接收满时，连接进入logback，logback满后连接被拒绝
* 这个值不能超过超过系统支持打开的最大文件数，也不能超过单个进程支持打开的[最大文件数](https://cloud.tencent.com/developer/article/1114773)
* 只能在events块中进行配置




### http块
http块是Nginx服务器配置中的重要部分，代理、缓存和日志定义等绝大多数的功能和第三方模块的配置都可以放在这个模块中。可以在http全局块中配置的指令包括文件引入、MIME-Type定义、日志自定义、是否使用sendfile传输文件、连接超时时间、单连接请求数上限等。

#### MIME Type
```conf
include mime.types;
```
* 常用的浏览器中，可以显示的内容有HTML、XML、GIF及Flash等种类繁多的文本、媒体等资源，浏览器为区分这些资源，需要使用MIME Type（文件扩展名与文件类型映射表）。Nginx服务器作为Web服务器，必须能够识别前端请求的资源类型。
* include指令，用于包含其他的配置文件，可以放在配置文件的任何地方，但是要注意包含进来的配置文件一定符合配置规范.比如说include进来的配置是worker_processes指令的配置，而将这个指令包含到了http块中，肯定是不行的（上面已经介绍过worker_processes指令只能在全局块中）。
* mime.types和nginx.conf同级目录，不同级的话需要指定具体路径

#### 默认文件类型
```conf
default_type application/octet-stream;
```
* 如果不加此指令，默认值为text/plain
* 还可以在http块、server块或者location块中进行配置

#### 访问日志
```conf
access_log path [format [buffer=size]];

# 如果要关闭access_log可以使用下面的命令
# access_log off;
```
* 记录Nginx服务器提供服务过程应答客户端请求的日志
* 可以在http块、server块或者location块中进行设置

#### 日志格式
```conf
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';

# 定义了上面的日志格式后，可以以下面的形式使用日志
# access_log logs/access.log  main;
```
* 用于定义日志格式
* 只能在http块中进行配置

#### sendfile
```conf
sendfile on | off;
```
* 开启或关闭sendfile方式传输文件(0拷贝)
* 可以在http块、server块或者location块中进行配置

#### sendfile最大数据量
```conf
sendfile_max_chunk size;

# sendfile_max_chunk 128k;
```
* 设置sendfile最大数据量
* 此指令可以在http块、server块或location块中配置
* size值如果大于0，Nginx进程的每个worker process每次调用sendfile()传输的数据量最大不能超过这个值；如果设置为0，则无限制
* 默认值为0

#### 连接超时时间
```conf
keepalive_timeout timeout [header_timeout];
```
* 与用户建立会话连接后，Nginx服务器可以保持这些连接打开一段时间
  * timeout是服务器端对连接的保持时间。默认值为75s
  * header_timeout，可选项，在应答报文头部的Keep-Alive域设置超时时间：`Keep-Alive: timeout= header_timeout`
* 可以在http块、server块或location块中配置

#### 单连接请求数上限
```conf
keepalive_requests number;
```
* Nginx服务器端和用户端建立会话连接后，用户端通过此连接发送请求
* 指令keepalive_requests用于限制用户通过某一连接向Nginx服务器发送请求的次数，默认是100
* 可以在http块、server块或location块中配置

#### 请求体大小限制
```
client_max_body_size 1m;
```
* 用于控制上传文件的大小限制，默认1m

#### 文件压缩
```conf
# 开启gzip压缩功能
gzip on;

# 设置允许压缩的页面最小字节数; 这里表示如果文件小于10k，压缩没有意义.
gzip_min_length 10k;

# 设置压缩比率，最小为1，处理速度快，传输速度慢；
# 9为最大压缩比，处理速度慢，传输速度快; 推荐6
gzip_comp_level 6;

# 设置压缩缓冲区大小，此处设置为16个8K内存作为压缩结果缓冲
gzip_buffers 16 8k;

# 设置哪些文件需要压缩,一般文本，css和js建议压缩。图片视需要要锁。
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript; 
```



### server块
server块和“虚拟主机”的概念有密切联系：  
虚拟主机，又称虚拟服务器、主机空间或是网页空间，它是一种技术。该技术是为了节省互联网服务器硬件成本而出现的。这里的“主机”或“空间”是由实体的服务器延伸而来，硬件系统可以基于服务器群，或者单个服务器等。虚拟主机技术主要应用于HTTP、FTP及EMAIL等多项服务，将一台服务器的某项或者全部服务内容逻辑划分为多个服务单位，对外表现为多个服务器，从而充分利用服务器硬件资源。从用户角度来看，一台虚拟主机和一台独立的硬件主机是完全一样的。

在使用Nginx服务器提供Web服务时，利用虚拟主机的技术就可以避免为每一个要运行的网站提供单独的Nginx服务器，也无需为每个网站对应运行一组Nginx进程。虚拟主机技术使得Nginx服务器可以在同一台服务器上只运行一组Nginx进程，就可以运行多个网站。

每一个http块都可以包含多个server块，而每个server块就相当于一台虚拟主机，它内部可有多台主机联合提供服务，一起对外提供在逻辑上关系密切的一组服务（或网站）。

在server全局块中，最常见的两个配置项是本虚拟主机的监听配置和本虚拟主机的名称或IP配置。

#### 监听配置
server块中最重要的指令就是listen指令，这个指令有三种配置语法。这个指令默认的配置值是：`listen *:80 | *:8000；`。只能在server块种配置这个指令。

```conf
# 第一种
listen address[:port] [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];

# 第二种
listen port [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];

# 第三种（可以不用重点关注）
listen unix:path [default_server] [ssl] [http2 | spdy] [proxy_protocol] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];

# 示例
listen 127.0.0.1:8000;  #只监听来自127.0.0.1这个IP，请求8000端口的请求
listen localhost:8000;  #只监听来自localhost这个域名，请求8000端口的请求
listen 127.0.0.1;       #只监听来自127.0.0.1这个IP，请求80端口的请求（不指定端口，默认80）
listen 8000;            #监听来自所有IP，请求8000端口的请求
listen *:8000;          #监听来自所有IP，请求8000端口的请求
```

重要参数做说明：
* address：监听的IP地址（请求来源的IP地址），如果是IPv6的地址，需要使用中括号“\[]”括起来，比如\[fe80::1]等。
* port：端口号，如果只定义了IP地址没有定义端口号，就使用80端口。多个虚拟主机可以同时监听同一个端口，但server_name需要设置成不一样。如果没有配置listen指令，默认情况如下：
  * 如果nginx以超级用户权限运行，则使用*:80
  * 如果nginx不以超级用户权限运行，则使用*:8000
* default_server：假如通过Host没匹配到对应的虚拟主机，则通过这台虚拟主机处理。如果所有**server**都没有显式定义，则会选取第一个定义的**server**作为default_server。
* backlog=number：设置监听函数listen()最多允许多少网络连接同时处于挂起状态，在FreeBSD中默认为-1，其他平台默认为511。
* accept_filter=filter，设置监听端口对请求的过滤，被过滤的内容不能被接收和处理。本指令只在FreeBSD和NetBSD 5.0+平台下有效。filter可以设置为dataready或httpready，感兴趣的读者可以参阅Nginx的官方文档。
* bind：标识符，使用独立的bind()处理此address:port；一般情况下，对于端口相同而IP地址不同的多个连接，Nginx服务器将只使用一个监听命令，并使用bind()处理端口相同的所有连接。
* ssl：标识符，设置会话连接使用SSL模式进行，此标识符和Nginx服务器提供的HTTPS服务有关。

#### 虚拟主机名
```conf
server_name name ...;
```
server_name指令用于配置虚拟主机的名称，可以只有一个名称，也可以由多个名称并列，之间用空格隔开。每个名字就是一个域名，由两段或者三段组成，之间由点号“.”隔开。

##### name的格式

###### 通配符
在name中可以使用通配符“*”，但通配符只能用在由三段字符串组成的名称的首段或尾段，或者由两段字符串组成的名称的尾段。
```conf
server_name *.myserver.com www.myserver.* myserver.*
```

###### 正则表达式
name还支持正则表达式，详见[Location中的`~`和`~^`标识符](#location块)。

##### 虚拟主机选择
在包含有多个虚拟主机的配置文件中，可能会出现一个名称被多个虚拟主机的server_name匹配成功，来自这个名称的请求按照以下的优先级选择虚拟主机：
1. 对于匹配方式不同的，排在前面的虚拟主机优先处理请求：
   1. 准确匹配server_name的虚拟主机
   2. 通配符在开始时匹配server_name成功的虚拟主机
   3. 通配符在结尾时匹配server_name成功的虚拟主机
   4. 正则表达式匹配server_name成功的虚拟主机
2. 在以上四种匹配方式中，如果server_name被处于同一优先级的匹配方式多次匹配成功，则首次匹配成功的虚拟主机处理请求。

### 错误页面
```conf
error_page 5xx /5xx.html;

error_page  500 501 502 503 504  /50x.html;
location = /50x.html {
    root html;
}
```

### rewrite
```conf
rewrite regex replacement [flag];
```
* rewrite功能就是使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。
* rewrite只能放在`server{}`、`location{}`和`if{}`中，并且只能对域名后边的除去传递的参数外的字符串起作用。
* 执行顺序：
  1. 执行server块的rewrite指令(这里的块指的是server关键字后{}包围的区域，其它xx块类似)
  2. 执行location匹配
  3. 执行选定的location中的rewrite指令
  4. 如果其中某步URI被重写，则重新循环执行1-3，直到找到真实存在的文件
  5. 如果循环超过10次，则返回500 Internal Server Error错误
* flag标志位：
  * last: 终止继续在本location块中处理接收到的URI，并将此处重写的URI作为一个新的URI，使用各location块进行处理。该标志将重写后的URI重写在server块中执行，为重写后的URI提供了转入到其他location块的机会。
  * break: 将此处重写的URI作为一个新的URI，在本块中继续进行处理。该标志将重写后的地址在当前的location块中执行，不会将新的URI转向其他的location块。
  * redirect: 将重写后的URI返回给客户端，状态码为302，指明是临时重定向URI，主要用在replacement变量不是以"http://"或者"https://"开头的情况。
  * permanent: 将重写后的URI返回给客户端，状态码为301，指明是永久重定向URI，主要用在replacement变量不是以"http://"或者"https://"开头的情况。


### location块
每个server块中可以包含多个location块，location块的主要作用是基于Nginx服务器接收到的请求字符串（例如， server_name/uri-string），对除虚拟主机名称（也可以是IP别名）之外的字符串（前例中“/uri-string”部分）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能都是在这部分实现，许多第三方模块的配置也是在location块中提供功能。

```conf
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
```
* uri变量是待匹配的请求字符串，可以是不含正则表达的字符串（标准uri），也可以是包含有正则表达的字符串（正则uri）。
* 方括号里的部分是可选项，用来改变请求字符串与uri的匹配方式
  * 在不添加此选项时，Nginx服务器首先在server块的多个location块中搜索是否有标准uri和请求字符串匹配，如果有多个可以匹配，就记录匹配度最高（前缀匹配字符数量最多，即最大前缀匹配）的一个。然后，服务器再用location块中的正则uri和请求字符串匹配，当第一个正则uri匹配成功，结束搜索，并使用这个location块处理此请求；如果正则匹配全部失败，就使用刚才记录的匹配度最高的location块处理此请求。
  * 各个标识的含义：
    * `=`：用于标准uri前，要求请求字符串与uri严格匹配。如果已经匹配成功，就停止继续向下搜索并立即处理此请求。
    * `^～`：用于标准uri前，要求Nginx服务器找到标识uri和请求字符串匹配度最高的location后，立即使用此location处理请求，而不再使用location块中的正则uri和请求字符串做匹配。
    * `～`：用于表示uri包含正则表达式（即正则uri），并且区分大小写。
    * `～*`：用于表示uri包含正则表达式（即正则uri），并且不区分大小写。
  * 所有类型location存在时，`=标准uri匹配` > `^~标准uri匹配` > `正则uri匹配` > `最大前缀匹配`
* `/`：用于通用匹配，任何的请求都会匹配到。
  * location中的字符有没有`/`都没有影响。也就是说`/user/`和`/user`是一样的。
  * 如果URL结构是 https://domain.com/ 的形式，尾部有没有`/`都不会造成重定向。因为浏览器在发起请求的时候，默认加上了`/`，虽然很多浏览器在地址栏里也不会显示`/`。
  * 如果URL的结构是 https://domain.com/some-dir/ 。尾部如果缺少`/`将导致重定向。因为根据约定，URL尾部的`/`表示目录，没有`/`表示文件。所以访问`/some-dir/`时，服务器会自动去该目录下找对应的默认文件。如果访问`/some-dir`的话，服务器会先去找`some-dir`文件，找不到的话会将`some-dir`当成目录，重定向到`/some-dir/`，去该目录下找默认文件。

#### 文件路径
```conf
root path
alias path
```
nginx指定文件路径有两种方式root和alias。root与alias主要区别在于nginx如何解释location后面的uri，这会使两者分别以不同的方式将请求映射到服务器文件上。
* root
  * root是最上层目录的定义
  * root的处理结果是：root路径＋location路径
  * root可以不放在location中
* alias
  * alias是一个目录别名的定义
  * alias的处理结果是：使用alias路径替换location路径
  * 使用alias时，目录名后面一定要加"/"（root可有可无）
  * alias在使用正则匹配时，必须捕捉要匹配的内容并在指定的内容处使用
  * alias只能位于location块中

#### 转发
```conf
# 可用于反向代理
proxy_pass uri;

# proxy_pass http://localhost:8080;
```

#### 返回
```conf
return code [text];
return code URL;

# return 400;
# return 404 not_found; 
# return 301 https://$server_name$request_uri;;
```

#### 缓存
```conf
expires 30d;
```