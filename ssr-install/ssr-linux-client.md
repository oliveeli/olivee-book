参考：
- https://www.zfl9.com/ss-local.html
- https://zzz.buzz/zh/gfw/2018/03/21/install-shadowsocks-client-on-centos-7/

ss-local 是 shadowsocks 的本地 socks5 服务器，如果需要使用 ss-local 提供的 socks5 代理，必须让应用程序使用 socks5 协议与之通信。但是很可惜，除了部分浏览器、软件直接支持 socks5 协议外，其它的都只支持 http 代理。因此，我们需要借助 privoxy 来将 http 代理协议转换为 socks5 代理协议，与后端的 ss-local 进行通信，与此同时我们还可以进行 gfwlist 分流操作。

# ss-local

## 安装

具体安装 shadowsocks-libev 的命令如下：

```bash
# CentOS/RHEL
cd /etc/yum.repos.d/
curl -O https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/repo/epel-7/librehat-shadowsocks-epel-7.repo
yum install -y shadowsocks-libev
```

安装完成后，会有 ss-local, ss-manager, ss-nat, ss-redir, ss-server, ss-tunnel 命令可用。

其中，作为客户端，我们需要的是 ss-local，不过后文中我们将通过服务文件启动 Shadowsocks，而不会直接与 ss-local 命令打交道。

注，如果安装报类似如下错误：

```bash
Error: Package: shadowsocks-libev-3.1.3-1.el7.centos.x86_64 (librehat-shadowsocks)
           Requires: libsodium >= 1.0.4
Error: Package: shadowsocks-libev-3.1.3-1.el7.centos.x86_64 (librehat-shadowsocks)
           Requires: mbedtls
```

说明系统没有启用 EPEL (Extra Packages for Entreprise Linux)。那么我们需要首先启用 EPEL，再安装 shadowsocks-libev：

```bash
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install -y shadowsocks-libev
```


# 添加配置文件
shadowsocks-libev 默认读取位于 /etc/shadowsocks-libev/config.json 的配置文件，我们可以根据需要参考以下配置文件进行修改：

```bash
{
    "server": "1.2.3.4",
    "server_port": 8989,
    "method": "aes-128-cfb",
    "password": "123456",
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "timeout":300
}

## 配置说明：
{
    "server": "1.2.3.4",          # 服务器IP
    "server_port": 8989,          # 服务器Port
    "method": "aes-128-cfb",      # 加密方式
    "password": "123456",         # 端口密码
    "local_address": "127.0.0.1", # 本地监听IP
    "local_port": 1080,           # 本地监听Port
    "timeout": 300,               # 超时时间
    "fast_open": true,            # TCP Fast Open
    "workers": 1                  # worker进程数量
}
```

如果想要变更默认的配置文件，或者提供其他命令行参数，我们可以修改 /etc/sysconfig/shadowsocks-libev：
```bash
# Configuration file
CONFFILE="/etc/shadowsocks-libev/config.json"

# Extra command line arguments
DAEMON_ARGS="-u"
```

其中 CONFFILE 指定了 shadowsocks-libev 所读取的配置文件；DAEMON_ARGS 则指定了额外的命令行参数，此处的 "-u" 表示启用 UDP 协议。

需要注意的是，命令行参数 DAEMON_ARGS 比配置文件 CONFFILE 中指定的选项优先级要更高一些。

## 启动 Shadowsocks 服务

有了 Shadowsocks 客户端的配置文件后，我们通过 systemd 启动 Shadowsocks 的客户端服务：

```bash
systemctl enable --now shadowsocks-libev-local
```

以上命令同时也会配置 Shadowsocks 客户端服务的开机自动启动。

至此，客户端所需要的所有配置就都已经完成了。


# privoxy

## 安装

```bash
## CentOS/RHEL
yum -y install privoxy

## ArchLinux
pacman -S privoxy
```

## 全局

全局模式是最简单最粗暴的，即：所有流量都走 ss-local，不区分什么国内国外。

所以请确定是否需要这种模式，如果不需要，请跳过此段，直接到 - [gfwlist 模式](https://www.zfl9.com/ss-local.html#gfwlist)。

## gfwlist

gfwlist 是由 AutoProxy 官方维护，由众多网民收集整理的中国大陆防火长城的域名屏蔽列表；

因为这是 Firefox 浏览器直接使用的一种格式，要在 privoxy 上使用就需要进行相应的格式转换；

这里我提供一个 shell 转换脚本，除了正则语法无法自动处理外，其它的基本 OK，[gfwlist2privoxy](https://github.com/zfl9/gfwlist2privoxy)。

```bash
# 关于 gfwlist2privoxy 脚本
# 脚本依赖 base64、curl(支持 https)、perl5 v5.10.0+

# 获取 gfwlist2privoxy 脚本
curl -4sSkLO https://raw.github.com/zfl9/gfwlist2privoxy/master/gfwlist2privoxy

# 生成 gfwlist.action 文件
bash gfwlist2privoxy '127.0.0.1:1080'

# 检查 gfwlist.action 文件
more gfwlist.action # 一般有 5000+ 行

# 应用 gfwlist.action 文件
mv -f gfwlist.action /etc/privoxy
echo 'actionsfile gfwlist.action' >>/etc/privoxy/config

# 启动 privoxy.service 服务
systemctl start privoxy.service
systemctl -l status privoxy.service
```


# 环境变量
有两种方式可以实现全局代理，推荐使用 proxychains-ng，因为更彻底。

- http_proxy和https_proxy环境变量：非强制性的环境变量，部分软件可能不遵守；
- proxychains-ng 的LD_PRELOAD环境变量：强制性的环境变量，用来实现动态库替换。

> ~~但实际上，无论哪种方式~~，体验都不是很好，如果你愿意折腾，推荐使用 [ss-redir 透明代理](https://www.zfl9.com/ss-redir.html)。

http_proxy 方式：
```bash
# privoxy 默认监听端口为 8118
proxy="http://127.0.0.1:8118"
export http_proxy=$proxy
export https_proxy=$proxy
export no_proxy="localhost, 127.0.0.1, ::1"

# no_proxy 环境变量是指不经过 privoxy 代理的地址或域名
# 只能填写具体的 IP、域名后缀，多个条目之间使用 ',' 逗号隔开
# 比如: export no_proxy="localhost, 192.168.1.1, ip.cn, chinaz.com"
# 访问 localhost、192.168.1.1、ip.cn、*.ip.cn、chinaz.com、*.chinaz.com 将不使用代理
```

proxychains 方式：
```bash
# 安装 proxychains-ng
## ArchLinux
pacman -S proxychains-ng

## CentOS/RHEL
./configure --prefix=/usr --sysconfdir=/etc
make && make install && make install-config
ln -sf /usr/bin/proxychains4 /usr/bin/proxychains

# 配置 proxychains-ng
vim /etc/proxychains.conf
# 注释 socks4 127.0.0.1 9050
# 添加 http 127.0.0.1 8118

# 替换当前 shell 进程
# 将 bash 替换为你的 shell
exec proxychains -q bash
```

# 代理测试

简单测试
```bash
# 访问网站，有网页源码输出说明 OK
curl -4sSkL https://www.baidu.com
curl -4sSkL https://www.google.com
curl -4sSkL https://www.google.co.jp
curl -4sSkL https://www.google.com.hk
curl -4sSkL https://www.youtube.com
curl -4sSkL https://www.facebook.com
curl -4sSkL https://www.wikipedia.org

# 获取当前 IP 地址，应该显示本机 IP
curl -4sSkL https://myip.ipip.net
```


详细调试

```bash
# 关闭 privoxy.service
systemctl stop privoxy.service

# 运行 privoxy (debug)
privoxy <(cat /etc/privoxy/config; echo -e 'debug 1\ndebug 2\ndebug 1024\ndebug 4096\ndebug 8192')

# 查看 privoxy 的日志
tail -f /var/log/privoxy/logfile

# 访问百度，观察日志
curl -4sSkL https://www.baidu.com
##### 日志输出 [走直连] #####
2018-07-14 12:15:07.955 7f0f23f080c0 Connect: Waiting for the next client connection. Currently active threads: 1
2018-07-14 12:15:07.956 7f0f2308d700 Connect: Accepted connection from 127.0.0.1 on socket 5
2018-07-14 12:15:07.956 7f0f2308d700 Request: www.baidu.com:443/
2018-07-14 12:15:07.956 7f0f2308d700 Connect: to www.baidu.com:443
2018-07-14 12:15:08.012 7f0f2308d700 Connect: Connected to www.baidu.com[14.215.177.39]:443.
2018-07-14 12:15:08.012 7f0f2308d700 Connect: Created new connection to www.baidu.com:443 on socket 6.
2018-07-14 12:15:08.012 7f0f2308d700 Connect: to www.baidu.com:443 successful
2018-07-14 12:15:08.130 7f0f2308d700 Connect: Closing server socket 6 connected to www.baidu.com. Keep-alive 0. Tainted: 1. Socket alive 1. Timeout: 0.
2018-07-14 12:15:08.130 7f0f2308d700 Connect: Closing client socket 5. Keep-alive: 0. Socket alive: 0. Data available: 0. Configuration file change detected: 0. Requests received: 1.
##### 日志输出 [走直连] #####

# 访问谷歌，观察日志
curl -4sSkL https://www.google.com
##### 日志输出 [走代理] #####
2018-07-14 12:15:43.969 7f0f23f080c0 Connect: Waiting for the next client connection. Currently active threads: 1
2018-07-14 12:15:43.969 7f0f2308d700 Connect: Accepted connection from 127.0.0.1 on socket 5
2018-07-14 12:15:43.970 7f0f2308d700 Connect: Overriding forwarding settings based on 'forward-socks5 127.0.0.1:1080 .'
2018-07-14 12:15:43.970 7f0f2308d700 Request: www.google.com:443/
2018-07-14 12:15:43.970 7f0f2308d700 Connect: to www.google.com:443
2018-07-14 12:15:43.970 7f0f2308d700 Connect: Connected to 127.0.0.1[127.0.0.1]:1080.
2018-07-14 12:15:43.972 7f0f2308d700 Connect: Created new connection to www.google.com:443 on socket 6.
2018-07-14 12:15:43.972 7f0f2308d700 Connect: to www.google.com:443 successful
2018-07-14 12:15:44.640 7f0f23f080c0 Connect: Waiting for the next client connection. Currently active threads: 2
2018-07-14 12:15:44.641 7f0f17de8700 Connect: Accepted connection from 127.0.0.1 on socket 7
2018-07-14 12:15:44.641 7f0f17de8700 Connect: Complete client request received.
2018-07-14 12:15:44.642 7f0f17de8700 Connect: Overriding forwarding settings based on 'forward-socks5 127.0.0.1:1080 .'
2018-07-14 12:15:44.642 7f0f17de8700 Request: www.google.com.hk/url?sa=p&hl=zh-CN&pref=hkredirect&pval=yes&q=http://www.google.com.hk/?gws_rd=cr&ust=1531541774610113&usg=AOvVaw1WKKuVWlVYxE2fELvMJW-Q
2018-07-14 12:15:44.642 7f0f17de8700 Connect: to www.google.com.hk
2018-07-14 12:15:44.642 7f0f17de8700 Connect: Connected to 127.0.0.1[127.0.0.1]:1080.
2018-07-14 12:15:44.643 7f0f17de8700 Connect: Created new connection to www.google.com.hk:80 on socket 8.
2018-07-14 12:15:44.643 7f0f17de8700 Connect: to www.google.com.hk successful
2018-07-14 12:15:45.013 7f0f17de8700 Connect: Done reading from server. Content length: 232 as expected. Bytes most recently read: 232.
2018-07-14 12:15:45.013 7f0f17de8700 Connect: Closing server socket 8 connected to www.google.com.hk. Keep-alive 1. Tainted: 0. Socket alive 1. Timeout: 0.
2018-07-14 12:15:45.013 7f0f17de8700 Connect: Waiting for the next client request on socket 7. No server socket to keep open.
2018-07-14 12:15:45.013 7f0f17de8700 Connect: Client request 2 arrived in time on socket 7.
2018-07-14 12:15:45.014 7f0f17de8700 Connect: Complete client request received.
2018-07-14 12:15:45.014 7f0f17de8700 Connect: Overriding forwarding settings based on 'forward-socks5 127.0.0.1:1080 .'
2018-07-14 12:15:45.014 7f0f17de8700 Request: www.google.com.hk/?gws_rd=cr
2018-07-14 12:15:45.014 7f0f17de8700 Connect: to www.google.com.hk
2018-07-14 12:15:45.014 7f0f17de8700 Connect: Connected to 127.0.0.1[127.0.0.1]:1080.
2018-07-14 12:15:45.014 7f0f17de8700 Connect: Created new connection to www.google.com.hk:80 on socket 8.
2018-07-14 12:15:45.014 7f0f17de8700 Connect: to www.google.com.hk successful
2018-07-14 12:15:45.427 7f0f17de8700 Connect: Looks like we reached the end of the last chunk. We better stop reading.
2018-07-14 12:15:45.427 7f0f17de8700 Connect: Done reading from server. Content length: 11519 as expected. Bytes most recently read: 3662.
2018-07-14 12:15:45.427 7f0f17de8700 Connect: Closing server socket 8 connected to www.google.com.hk. Keep-alive 1. Tainted: 0. Socket alive 1. Timeout: 0.
2018-07-14 12:15:45.429 7f0f17de8700 Connect: Waiting for the next client request on socket 7. No server socket to keep open.
2018-07-14 12:15:45.429 7f0f2308d700 Connect: Closing server socket 6 connected to www.google.com. Keep-alive 0. Tainted: 1. Socket alive 1. Timeout: 0.
2018-07-14 12:15:45.429 7f0f17de8700 Connect: Closing client socket 7. Keep-alive: 1. Socket alive: 0. Data available: 0. Configuration file change detected: 0. Requests received: 2.
2018-07-14 12:15:45.429 7f0f2308d700 Connect: Closing client socket 5. Keep-alive: 0. Socket alive: 0. Data available: 0. Configuration file change detected: 0. Requests received: 1.
##### 日志输出 [走代理] #####

# 调试完成，恢复 privoxy
pkill privoxy
systemctl start privoxy.service
```
