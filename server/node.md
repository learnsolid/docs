# 安装并运行一个 NodeJS SoLiD 服务器

目前 SoLiD 的服务端实现仅支持 [Node.js](https://nodejs.org/) v8 及以上。整个程序在 Github 上开源，名称为 [solid-server](https://github.com/solid/node-solid-server)。如果你想快速预览，我们提供了一个 [README](https://github.com/solid/node-solid-server#install) 来帮助你快速安装 ```solid-server```。在这个例子中，我们主要演示如何在 Debian 操作系统上安装并运行 solid-server。

## 示例安装：在 Debian GNU/Linux 上安装 SoLiD Server

首先安装 Node.js 和 NPM：

```shell
$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
$ sudo apt-get install -y nodejs build-essential
```

如果你是 ```root``` 用户，可以使用 NPM 全局安装 solid server：

```shell
$ npm install -g solid-server
```

接下来，你需要想一个 hostname，在这个例子中，我们会使用 ```your.host.example.org``` 用来演示。当然，你最好还有一台服务器用来部署 SoLiD Server（你的开发机也行）。SoLiD Server 中的所有配置文件、数据以及元信息数据库都可以分布式存储，但是这个例子中我们把它们都放到一起。

所有的 SoLiD Server 都需要一个 SSL key 和证书用来加密通信，这可以通过 [Let’s Encrypt](https://letsencrypt.org/) 获得。除此之外，最简单的方式是用 ```certbot```，请按照下面这个教程安装 ```certbot```。

首先将以下信息加到 ```/etc/apt/sources.list``` 中：

```shell
$ deb http://ftp.debian.org/debian stretch-backports main
```

然后，通过以下命令安装 ```certbox```：

```shell
$ apt-get update
$ apt-get -t stretch-backports install certbot
```

接着，安装证书：

```shell
$ certbot certonly --authenticator standalone -d your.host.example.org
```

最后，我们初始化 SoLiD：

```shell
$ solid init
```

初始化时会要求你提供很多信息，就像下面这样：

```shell
? Path to the folder you want to serve. Default is /var/www/your.host.example.org/data
? SSL port to run on. Default is 8443 443
? SoLiD server uri (with protocol, hostname and port) https://your.host.example.org
? Enable WebID authentication Yes
? Serve SoLiD on URL path /
? Path to the config directory (for example: /etc/solid-server) /var/www/your.host.example.org/config
? Path to the config file (for example: ./config.json) /var/www/your.host.example.org/config.json
? Path to the server metadata db directory (for users/apps etc) /var/www/your.host.example.org/.db
? Path to the SSL private key in PEM format /etc/letsencrypt/live/your.host.example.org/privkey.pem
? Path to the SSL certificate key in PEM format /etc/letsencrypt/live/your.host.example.org/fullchain.pem
? Enable multi-user mode No
? Do you want to have a CORS proxy endpoint? Yes
? Serve the CORS proxy on this path /proxy
? Do you want to set up an email service? No
config created on /root/config.json
```

Then, you need to create the paths that you entered. You would also need to copy the config.json file to where you indicated it should be.

然后就可以正式启动 SoLiD 了：

```shell
$ solid start
```

我们推荐使用 ```systemd``` 自动重启和停止 SoLiD Server。
为了安全起见，不要以 root 用户运行 SoLiD Server，请创建一个用户来运行，例如：

```shell
$ adduser --system --ingroup www-data --no-create-home solid
```

接着创建一个名为 ```/lib/systemd/system/solid.service``` 的文件，内容为：

```ini
[Unit]
Description=solid - Social Linked Data
Documentation=https://solid.inrupt.com/docs/
After=network.target

[Service]
Type=simple
User=solid
WorkingDirectory=/var/www/your.host.example.org
ExecStart=/usr/bin/solid start -v
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

然后给它一个软链接：

```shell
$ ln -s /lib/systemd/system/solid.service /etc/systemd/system/multi-user.target.wants/
```

现在，给文件夹赋予权限：

```shell
$ cd /var/www/your.host.example.org/
$ chown solid:www-data config/ data/ .db/
```

最后，用 ```systemctl``` 启动 SoLiD Server：

```shell
$ systemctl start solid.service
```

SSL 证书需要每隔几个月刷新一次，请修改 ```/etc/letsencrypt/renewal/your.host.example.org.conf``` 文件为以下内容：

```ini
authenticator = webroot
webroot_path = /var/www/your.host.example.org/data
[[webroot_map]]
your.host.example.org = /var/www/your.host.example.org/data
```

最后，创建一个名为 ```/etc/cron.daily/``` 的文件以创建一个 crontab，内容如下：

```shell
#!/bin/bash
certbot renew -w /var/www/your.host.example.org/data/
```

现在，你的 SoLiD Server 应该已经可以正常工作了！请尽情享受 SoLiD 世界吧！
