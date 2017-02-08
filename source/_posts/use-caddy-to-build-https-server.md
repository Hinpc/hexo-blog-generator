title: 使用 Caddy 快速搭建高效自动的 HTTPS 服务器  
date: 2016-11-24 22:01  
tag:
- caddy
- https
- 服务器

photos:
- /img/2016/201611-ipad-605420_1920.jpg

---

Caddy 是一个用 Golang 写的高效 Web Server，相比 Nginx，它的配置和使用要简单很多，并且支持自动化 HTTPS。

<!--more-->

官网：[https://caddyserver.com/](https://caddyserver.com/)

官方文档：[https://caddyserver.com/docs/getting-started](https://caddyserver.com/docs/getting-started)



这里介绍本站的 Caddy 服务器配置过程。从 Nginx 换到 Caddy 整个过程不到10分钟，官方甚至有一个28秒的[配置视频](https://www.youtube.com/watch?time_continue=28&v=nk4EWHvvZtI)。



## 1. 快速上手

### 1.1 下载安装

进入 [下载页面](https://caddyserver.com/download) 勾选需要的模块，选择平台进行下载 。（我的是 Ubuntu x64，因此选择了Linux 64-bit，下载后再上传到 VPS，因为链接是生成的，没有固定链接，囧）



下载后的压缩文件 `caddy_linux_amd64_custom.tar.gz` 放到网站根目录（跟 index.html 同级），进行解压：

```bash
$ tar zxvf caddy_linux_amd64_custom.tar.gz
```

会得到 caddy 文件以及 init 文件夹和其他杂项文件。



### 1.2 运行

在 caddy 解压出来的目录（网站根目录）执行 `./caddy` ，这时访问 `localhost:2015` 你的网站应该已经愉快地跑起来了。如果你在 vps 上进行的此操作，远程访问 http://你的ip:2015 应该也能看到你的站点了。



## 2. 配置 Caddy

如何自定义 Caddy 服务呢？在你的站点目录里创建一个名为 `Caddyfile` 的文件，文件的第一行写上监听地址，如：

```
localhost:8080
```

重新启动 Caddy：执行  `./caddy` ，你会发现站点已经运行在8080端口了。

Caddyfile 可以在其他目录，也可以命名为其他你喜欢的名字。只需要在启动 Caddy 时告诉它配置文件在哪：

```shell
$ caddy -conf="../path/to/Caddyfile"
```

Caddyfile 语法：[查看](https://caddyserver.com/docs/caddyfile)



## 3. 实现自动化 HTTPS

Caddy 自动地为你的站点启用 HTTPS。注意：需要配置 host 不能为空，也不能是 localhost 或 ip 地址。

我的静态博客的配置是：

```
blog.hinpc.com {
    root /home/blog.hinpc.com/www #换成你的网站路径
    gzip
    log ../access.log
}
```

执行  `./caddy` ，网站已启动，并且支持 HTTPS 了。它的证书是由 [Let’s Encrypt](https://letsencrypt.org/) 实现的，并且自动为你的网站申请和续期。



## 4. 使用 supservisor 来运行 Caddy

在你的 `supervisord.conf` 文件中，`[supervisord]` 区域下配置：

```
minfds=4096
```

 `supervisord.conf` 中添加：

```
[program:caddy]
directory=/etc/caddy/ ## 换成你的目录
command=/etc/caddy/caddy -conf="/etc/caddy/Caddyfile" ## 换成你的配置文件
user=www-data
autostart=true
autorestart=true
stdout_logfile=/var/log/supervisor/caddy_stdout.log
stderr_logfile=/var/log/supervisor/caddy_stderr.log
```

启用：

```shell
# supervisorctl reread
# supervisorctl add caddy
# supervisorctl start caddy
```

---

### 参考

https://caddyserver.com/docs/getting-started

http://depado.markdownblog.com/2015-12-07-setting-up-caddy-server-on-debian
