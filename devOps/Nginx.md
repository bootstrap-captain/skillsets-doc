# 安装

- [下载官网](https://nginx.org/en/download.html)

## 单机

### 编译启动

#### 下载上传

![image-20250305173804196](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250305173804196.png)

```bash
# Centos 7.6上安装编译依赖
sudo yum install -y gcc make openssl-devel pcre-devel zlib-devel

# 下载  pgp
/Users/shuzhan/Desktop/nginx-1.26.3.tar.gz /usr/tmp

# 解压文件
cd usr/tmp
tar -zxvf nginx-1.26.3.tar.gz

# 解压后的目录
CHANGES
CHANGES.ru
LICENSE
Makefile
README
auto
conf
configure
contrib
html
man
objs
src
```

#### 配置编译选项

- 运行configure脚本，编译当前解压的文件

```bash
./configure \
--prefix=/usr/local/nginx \
--sbin-path=/usr/local/nginx/sbin/nginx \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--pid-path=/var/run/nginx.pid \
--with-http_ssl_module \
--with-http_gzip_static_module \
--with-pcre
```

```bash
./configure \
--prefix=/usr/local/nginx \          # 安装主目录
--sbin-path=/usr/local/nginx/sbin/nginx \  # 可执行文件路径
--conf-path=/usr/local/nginx/conf/nginx.conf \  # 配置文件路径
--pid-path=/var/run/nginx.pid \      # PID文件路径
--with-http_ssl_module \             # 启用SSL（HTTPS必需）
--with-http_gzip_static_module \     # 启用Gzip压缩
--with-pcre                          # 正则表达式支持
```

#### 编译并安装

```bash
make install

# 检查版本
/usr/local/nginx/sbin/nginx -v
```

#### 启动

```bash
cd /usr/local/nginx/sbin

# 启动
./nginx 

# 默认端口是80, 访问
http://39.105.210.163/ 

# 启动停止
./nginx                  # 启动
./nginx -s stop          # 快速停止
./nginx -s quit          # 优雅关闭，在退出前完成已经接受的连接请求
./nginx -s reload        # 重新加载配置
```

## 5. 配置Systemd服务

- 先把上面的关闭了

```bash
sudo vi /etc/systemd/system/nginx.service
```

```bash
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=network.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

```bash
# 重新加载系统服务
sudo systemctl daemon-reload

sudo systemctl start nginx          # 启动
sudo systemctl status nginx         # 查看状态
sudo systemctl stop nginx           # 关闭
sudo systemctl restart nginx             # 重启

sudo systemctl enable nginx         # 开机启动
```

## 6. index.html

```bash
# 默认访问的就是该页面
/usr/local/nginx/htm/index.html
```



# 目录结构

```bash
cd /usr/local/nginx

# temp： 是运行nginx后产生的临时文件
client_body_temp
scgi_temp
fastcgi_temp
proxy_temp
uwsgi_temp

# sbin
- 里面就只有nginx一个
- nginx的主程序，启动的命令




logs

# sbin




2 nobody root 4096 Mar  5 18:10 client_body_temp
2 root   root 4096 Mar  5 18:08 conf                         # nginx的主配置文件
2 nobody root 4096 Mar  5 18:10 fastcgi_temp
2 root   root 4096 Mar  5 18:38 html
2 root   root 4096 Mar  5 18:10 logs
2 nobody root 4096 Mar  5 18:10 proxy_temp
2 root   root 4096 Mar  5 18:08 sbin                          # nginx的主程序，启动的命令
2 nobody root 4096 Mar  5 18:10 scgi_temp
2 nobody root 4096 Mar  5 18:10 uwsgi_temp
```

```bash
# conf
- nginx的核心配置文件

## nginx.conf
- 主配置文件，会引用其他的一些配置文件

fastcgi.conf
fastcgi.conf.default
fastcgi_params
fastcgi_params.default
koi-utf
koi-win
mime.types
mime.types.default
nginx.conf
nginx.conf.default
scgi_params
scgi_params.default
uwsgi_params
uwsgi_params.default
win-utf
```

```bash
# html
- index.html                # 使用 http://39.105.210.163/ 使用的默认页
- 50x.html                  # 错误的页面
```

```bash
# logs

access.log                  # 每个用户的访问日志
error.log                   # 错误日志
```

