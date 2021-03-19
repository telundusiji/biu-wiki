> 操作系统：CentOS-7.8  
> nginx版本：1.18.0

### 一、下载源码

访问nginx官网，下载nginx包，[http://nginx.org/en/download.html](http://nginx.org/en/download.html)

### 二、安装依赖

* 安装gcc的c++编译环境：yum install gcc-c++

* 安装解析正则表达式的库：yum install -y pcre pcre-devel

* 安装数据压缩函式库：yum install -y zlib zlib-devel

* 安装用于安全通信的库：yum install -y openssl openssl-devel

### 三、编译安装

* 解压nginx包：tar -xzvf nginx-1.18.0.tar.gz

* 进入编译解压后的目录，进行构建：./configure 选择需要配置的选项

可用配置如下，<font color=#00bfa5>使用到路径的，需要路径先创建好</font>

我在安装的配置使用到了/var/temp/nginx/这个目录，所以先创建目录

`makedir -p /var/temp/nginx/`

然后执行构建脚本命令如下：

```shell

./configure \
    --prefix=/usr/local/nginx \
    --pid-path=/var/run/nginx/nginx.pid \
    --lock-path=/var/lock/nginx.lock \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --with-http_gzip_static_module \
    --http-client-body-temp-path=/var/temp/nginx/client \
    --http-proxy-temp-path=/var/temp/nginx/proxy \
    --http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
    --http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
    --http-scgi-temp-path=/var/temp/nginx/scgi

```
可用配置如下见文末附表。

* 进行编译，命令：make

* 进行安装，命令：make install

* 启动nginx，进入自己配置的prefix指定的目录，执行命令

* 停止：./nginx -s stop
* 重新加载：./nginx -s reload

#### pid文件不存在、或者pid数字无效错误处理办法

1.首先检查pid目录是否存在，上面命令构建命名配置项--pid-path，所配置的目录，不存在则创建一个

2.如pid目录存在，则执行命令：./nginx -c {nginx.conf的路径}

3.重新加载nginx：./nginx -s reload

|配置|说明|
|:--- | :--- |
| 测--prefix=<path>测 | Nginx安装路径。如果没有指定，默认为 /usr/local/nginx。 |
| --sbin-path=<path> | Nginx可执行文件安装路径。只能安装时指定，如果没有指定，默认为<prefix>/sbin/nginx。 |
| --conf-path=<path> | 在没有给定-c选项下默认的nginx.conf的路径。如果没有指定，默认为<prefix>/conf/nginx.conf。 |
| --pid-path=<path> | 在nginx.conf中没有指定pid指令的情况下，默认的nginx.pid的路径。如果没有指定，默认为 <prefix>/logs/nginx.pid。 |
| --lock-path=<path> | nginx.lock文件的路径。 |
| --error-log-path=<path> | 在nginx.conf中没有指定error_log指令的情况下，默认的错误日志的路径。如果没有指定，默认为 <prefix>/logs/error.log。 |
| --http-log-path=<path> | 在nginx.conf中没有指定access_log指令的情况下，默认的访问日志的路径。如果没有指定，默认为 <prefix>/logs/access.log。 |
| --user=<user> | 在nginx.conf中没有指定user指令的情况下，默认的nginx使用的用户。如果没有指定，默认为 nobody。 |
| --group=<group> | 在nginx.conf中没有指定group指令的情况下，默认的nginx使用的组。如果没有指定，默认为 nobody。 |
| --builddir=DIR | 指定编译的目录 |
| --with-rtsig_module | 启用 rtsig模块 |
| --with-select_module --without-select_module | 允许或不允许开启SELECT模式，如果 configure 没有找到更合适的模式，比如：kqueue(sun os),epoll (linux kenel 2.6+),rtsig（实时信号）或者/dev/poll（一种类似select的模式，底层实现与SELECT基本相 同，都是采用轮训方法） SELECT模式将是默认安装模式 |
| --with-poll_module --without-poll_module | 是否开启poll模块，如果configure未发现更合适的方法（如kqueue、epoll、rtsig或/dev/poll），则默认情况下启用此模块。 |
| --with-http_ssl_module | 开启HTTP SSL模块 |
| --with-http_realip_module | 启用 ngx_http_realip_module |
| --with-http_addition_module | 启用 ngx_http_addition_module |
| --with-http_sub_module | 启用 ngx_http_sub_module |
| --with-http_dav_module | 启用 ngx_http_dav_module |
| --with-http_flv_module | 启用 ngx_http_flv_module |
| --with-http_stub_status_module | 启用 "server status" 页 |
| --without-http_charset_module | 禁用 ngx_http_charset_module |
| --without-http_gzip_module | 禁用 ngx_http_gzip_module. 如果启用，需要 zlib。 |
| --without-http_ssi_module | 禁用 ngx_http_ssi_module |
| --without-http_userid_module | 禁用 ngx_http_userid_module |
| --without-http_access_module | 禁用 ngx_http_access_module |
| --without-http_auth_basic_module | 禁用 ngx_http_auth_basic_module |
| --without-http_autoindex_module | 禁用 ngx_http_autoindex_module |
| --without-http_geo_module | 禁用 ngx_http_geo_module |
| --without-http_map_module | 禁用 ngx_http_map_module |
| --without-http_referer_module | 禁用 ngx_http_referer_module |
| --without-http_rewrite_module | 禁用 ngx_http_rewrite_module. 如果启用需要 PCRE。 |
| --without-http_proxy_module | 禁用 ngx_http_proxy_module |
| --without-http_fastcgi_module | 禁用 ngx_http_fastcgi_module |
| --without-http_memcached_module | 禁用 ngx_http_memcached_module |
| --without-http_limit_zone_module | 禁用 ngx_http_limit_zone_module |
| --without-http_empty_gif_module | 禁用 ngx_http_empty_gif_module |
| --without-http_browser_module | 禁用 ngx_http_browser_module |
| --without-http_upstream_ip_hash_module | 禁用 ngx_http_upstream_ip_hash_module |
| --with-http_perl_module | 启用 ngx_http_perl_module |
| --with-perl_modules_path=PATH | 指定 perl模块的路径 |
| --with-perl=PATH | 指定 perl 执行文件的路径 |
| --http-log-path=PATH | 设置http access日志路径 |
| --http-client-body-temp-path=PATH | 设置http 客户端请求体的临时文件目录 |
| --http-proxy-temp-path=PATH | 设置http代理的临时文件目录 |
| --http-fastcgi-temp-path=PATH |设置http fastcgi临时文件目录 |
| --without-http | 禁用 HTTP server |
| --with-mail | 启用 IMAP4/POP3/SMTP 代理模块 |
| --with-mail_ssl_module | 启用 ngx_mail_ssl_module |
| --with-cc=PATH | 指定 C编译器的路径 |
| --with-cpp=PATH | 指定 C预处理器的路径 |
| --with-cc-opt=OPTIONS | 将添加到变量CFLAGS的其他参数。在FreeBSD中使用系统库PCRE时，有必要用cc opt=“-I/usr/local/include”指示。如果我们使用select（），并且需要增加文件描述符的数量，那么也可以在这里分配：-使用cc opt=“-D FD_SETSIZE=2048”。 |
| --with-ld-opt=OPTIONS | 传递给链接器的其他参数。在FreeBSD中使用系统库PCRE时，有必要用ld opt=“-L/usr/local/lib”指示。 |
| --with-cpu-opt=CPU | 为特定的 CPU 编译，有效的值包括：pentium,pentiumpro,pentium3,pentium4,athlon,opteron,amd64,sparc32,sparc64,ppc64 |
| --without-pcre | 禁止 PCRE 库的使用。同时也会禁止 HTTP rewrite 模块。在 "location" 配置指令中的正则表达式也需要 PCRE。 |
| --with-pcre=DIR | 指定 PCRE 库的源代码的路径。 |
| --with-pcre-opt=OPTIONS | 为构建PCRE设置其他选项 |
| --with-md5=DIR | 设置md5库源的路径 |
| --with-md5-opt=OPTIONS | 为md5构建设置其他选项 |
| --with-md5-asm | 使用md5汇编程序源 |
| --with-sha1=DIR | 设置sha1库源的路径 |
| --with-sha1-opt=OPTIONS | 为sha1构建设置其他选项。 |
| --with-sha1-asm | 使用sha1汇编程序源 |
| --with-zlib=DIR | 设置zlib库源的路径 |
| --with-zlib-opt=OPTIONS | 为zlib构建设置其他选项 |
| --with-zlib-asm=CPU | 使用为指定CPU优化的zlib汇编程序源，有效值为：pentium、pentiumpro |
| --with-openssl=DIR | 设置OpenSSL库源的路径 |
| --with-openssl-opt=OPTIONS | 为OpenSSL构建设置其他选项 |
| --with-debug | 启用调试日志 |
| --add-module=PATH | 在目录路径中添加第三方模块 |



> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片

  
> 觉得不错就点个赞叭QAQ
