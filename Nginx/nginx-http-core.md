# Nginx Http Core

## 一|安装

基于官方文档说明安装 [Installing nginx](http://nginx.org/en/docs/install.html)

基于各平台包管理器安装：

```bash
# CentOS
yum install nginx;

# Ubuntu
sudo apt-get install nginx;

# Mac
brew install nginx;
```

## 二|基本命令

```bash
# 启动
nginx -s start;

# 重新启动，热启动，修改配置重启不影响线上
nginx -s reload;

# 关闭
nginx -s stop;

# 修改配置后，可以通过下面的命令测试是否有语法错误
nginx -t;

# 说明
-s，signal，意思就是向 nginx 发送 start|reload|stop 等命令。
```

## 三|默认配置文件

默认全局配置文件 **/etc/nginx/nginx.conf**

```conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;

    client_max_body_size 1024m;
    client_body_buffer_size 1024k;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

最后一行 `include /etc/nginx/conf.d/*.conf;` 可见该全局配置文件会 include `/etc/nginx/conf.d/` 目录下的所有以 `.conf` 结尾的配置文件。

默认子配置文件 **/etc/nginx/nginx.d/default.conf**

```conf
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

## 四|指令：listen

详见 [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen)

- Sets the `address` and `port` for IP, or the `path` for a UNIX-domain socket on which the server will accept requests.
- Both `address` and `port`, or only `address` or only port can be specified.
- An `address` may also be a hostname.
- If only `address` is given, the port 80 is used.

```conf
# address and port
listen 127.0.0.1:8000;
listen 127.0.0.1;
listen 8000;
listen *:8000;
listen localhost:8000;

# IPv6 addresses (0.7.36) are specified in square brackets:
listen [::]:8000;
listen [::1];

# UNIX-domain sockets (0.8.21) are specified with the “unix:” prefix:
listen unix:/var/run/nginx.sock;
```

## 五|指令：server_name

详见 [server_name](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name)

server_name 指令设置虚拟主机的名称。

```conf
# 常规
server {
    server_name example.com www.example.com;
}

# 通配符
server {
    server_name example.com *.example.com www.example.*;
}

# .符
server {
    server_name .example.com;
}

# 正则
server {
    server_name www.example.com ~^www\d+\.example\.com$;
}

# 正则-捕获(0.7.40)
server {
    server_name ~^(www\.)?(.+)$;
    location / {
        root /sites/$2;
    }
}
server {
    server_name _;
    location / {
        root /sites/default;
    }
}

# 正则-具名捕获(0.7.40)
server {
    server_name ~^(www\.)?(?<domain>.+)$;
    location / {
        root /sites/$domain;
    }
}
server {
    server_name _;
    location / {
        root /sites/default;
    }
}

# 空主机名(0.7.11)：用于处理未带 “Host” header 的请求
server {
    server_name www.example.com "";
}
```

## 六|指令：root & alias & index

详见 [root](http://nginx.org/en/docs/http/ngx_http_core_module.html#root) [alias](http://nginx.org/en/docs/http/ngx_http_core_module.html#alias)

原型：

```conf
Syntax: root path;
Default: root html;
Context: http, server, location, if in location
```

```conf
Syntax: alias path;
Default: —
Context: location
```

```conf
location /i/ {
    root /data/w3;
}
```

- **root**: Sets the root directory for requests.
- The /data/w3/i/top.gif file will be sent in response to the “/i/top.gif” request.
- The path value can contain variables, except `$document_root` and `$realpath_root`.

```conf
location /i/ {
    alias /data/w3/images/;
}
```

- **alias**: Defines a replacement for the specified location.
- on request of “/i/top.gif”, the file /data/w3/images/top.gif will be sent.
- The path value can contain variables, except $document_root and $realpath_root.

```conf
location ~ ^/users/(.+\.(?:gif|jpe?g|png))$ {
    alias /data/w3/images/$1;
}
```

- If alias is used inside a location defined with a regular expression then such regular expression should contain captures and alias should refer to these captures (0.7.40).

## 七|指令：location

详见 [location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)

原型：

```conf
Syntax: location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
Default: —
Context: server, location
```

    根据请求 URI 设置配置。

    匹配是基于规范化的 RUI 进行的（解码 URL-encode，解析 . 和 .. 的相对引用，两个及以上的斜杠压缩成单斜杠）

    location 可以是前缀字符串（前缀location）或正则表达式（正则location）。正则表达式以 `~*`（不区分大小写的匹配） 或 `~`（区分大小写的匹配） 开头。正则表达式可包含捕获 (0.7.40) 。

    匹配过程：
    - 先匹配一个最长匹配的 “前缀location”。
    - 然后根据 “正则location” 在配置文件中的定义顺序，匹配 “正则location” 。遇到第一个匹配的 “正则location”，则使用该 “正则location”，并结束匹配。
    - 如果没有匹配到 “正则location”，则使用前面匹配到的最长匹配的 “前缀location”。

    匹配特例：
    - 如果匹配到的最长匹配 “前缀location” 有 `^~`，则不进行后续的 “正则location” 匹配。
    - 使用`=` “全等location” 精确匹配后，则不继续后续的匹配。

示例：

```conf
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

- The “/” request will match configuration A.
- the “/index.html” request will match configuration B.
- the “/documents/document.html” request will match configuration C.
- the “/images/1.gif” request will match configuration D.
- the “/documents/1.jpg” request will match configuration E.

“@” 用于定义命名location，这种location不用于匹配处理，而是用于请求重定向，不可嵌套，也不可被嵌套。

如果一个 “前缀location” 以 `/` 结尾，且请求由 “proxy_pass, fastcgi_pass, uwsgi_pass, scgi_pass, memcached_pass, grpc_pass” 之一处理，则会做特殊处理：

假设一个请求等于该 “前缀location” ，但不以 `/` 结尾，则会响应一个301的永久重定向，并在请求 URI 加上  `/` 。如果这不是预期想要的，则使用精确匹配：

```conf
location /user/ {
    proxy_pass http://user.example.com;
}

location = /user {
    proxy_pass http://login.example.com;
}
```

## 八|指令：try_files

详见 [try_files](http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files)

原型：

```conf
Syntax: try_files file ... uri;
        try_files file ... =code;
Default: —
Context: server, location
```

以指定的顺序检查文件是否存在，并使用第一个存在的文件（或文件夹，`/` 结尾表示文件夹）。（The path to a file is constructed from the file parameter according to the root and alias directives.）。如果无一文件被找到，将会做一个内部重定向到指定的最后一个参数。例如：

```conf
location /images/ {
    try_files $uri /images/default.gif;
}

location = /images/default.gif {
    expires 30s;
}
```

最后一个参数，可以是具名location：

```conf
location / {
    try_files /system/maintenance.html
              $uri $uri/index.html $uri.html
              @mongrel;
}

location @mongrel {
    proxy_pass http://mongrel;
}
```

从版本 0.7.51 起，最后一个参数，也可以是状态码 code：

```conf
location / {
    try_files $uri $uri/index.html $uri.html =404;
}
```

在代理中的使用示例：

```conf
location / {
    try_files /system/maintenance.html
              $uri $uri/index.html $uri.html
              @mongrel;
}

location @mongrel {
    proxy_pass http://mongrel;
}
```

其他注意点：

- 最后一个参数是回退URI且必须存在，否则会出现内部500错误。
- 与rewrite指令不同，如果回退URI不是命名的location那么$args不会自动保留，如果你想保留$args，则必须明确声明。例如：try_files $uri $uri/ /index.php?q=$uri&$args;
- 参看这个博客：[Nginx的try_files指令使用实例](https://www.hi-linux.com/posts/53878.html)

## 八|指令：

## x|内置全局变量

## X|How nginx processes a request

详见 [How nginx processes a request](http://nginx.org/en/docs/http/request_processing.html)