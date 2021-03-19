> 阿里云esc  
> 操作系统:centos 7.6

### 安装ftp

* 安装ftp：yum install vsftpd -y

* 启动ftp：systemctl start vsftpd

* 修改配置文件，配置文件根目录：/etc/vsftp。该目录下有三个文件：ftpusers（该文件中配置的用户不能访问FTP服务器）、user_list（该文件里的用户账户仅当在vsftpd .conf配置文件里设置了userlist_enable=NO选项时才可以访问FTP）、vsftpd.conf（主配置文件）。配置/etc/vsftp/vsftpd.conf

```properties
#禁止匿名用户登录
anonymous_enable=NO
#允许本地用户登录
local_enable=YES
write_enable=YES
#本地用户登录文件访问权限(022代表访问权限为：777-022=755)
local_umask=022
#本地用户登录默认目录并锁定
local_root=/ftp-home/
chroot_local_user=YES
allow_writeable_chroot=YES
```

* 添加ftp用户(第一次安装完vsftp会默认创建ftp用户，如需使用可以直接在root权限下：*passwd ftp* 修改密码，也可也先root执行命令：*userdel ftp* 删除ftp用户，再重新创建ftp用户)

```shell
useradd ftp
passwd ftp
```

* 配置本地用户登录默认目录权限(在root权限下进行)

```
#创建ftp-home目录
mkdir /ftp-home
#修改用户组
chgrp ftp /ftp-home
#修改目录控制权限
chmod 775 /ftp-home
```

* 重启vsftpd使配置生效并加入开机自启动

```shell
#重启vsftpd
systemctl restart vsftpd
#查看vsftpd运行状态
systemctl status vsftpd
```

### 安装nginx

* 安装nginx：yum install nginx -y
* 修改nginx配置文件：/etc/nginx/nginx.conf

```
server {
	   #默认监听端口
        listen       8001 default_server;
        listen       [::]:8001 default_server;
        server_name  _;
        #资源根目录，设置为ftp根目录即可
        root         /ftp-home;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

* 启动nginx并加入开机自启动

```shell
#启动nginx
systemctl start nginx
#加入开机自启动
systemctl enable nginx
```