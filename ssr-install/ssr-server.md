# 本脚本适用环境
系统支持：CentOS 6+，Debian 7+，Ubuntu 12+
内存要求：≥128M

# 代理服务器购买
代理服务器推荐：[国外科学上网翻墙 VPS 服务器推荐，新手买哪个好？](https://v2raycn.com/38.html)

搬瓦工官网：http://aff998.com/bwg

搬瓦工购买与优惠码使用可以参考：[搬瓦工购买教程，支持支付宝和微信支付，终身优惠码](https://v2raycn.com/12.html)

VULTR 官网：http://aff998.com/vultr

VULTR 也支持支付宝和微信支付，购买教程：[VULTR 购买教程，支持支付宝与微信支付，中文图解教程](https://v2raycn.com/19.html)

购买完 VPS 后，只需要通过 Xshell 连接 VPS 即可操作 VPS，包括安装脚本、加速等：[利用免费版 Xshell 远程连接你的 Linux VPS，含 Xshell 下载](https://v2raycn.com/30.html)。

# 关于本脚本
1. 一键安装 Shadowsocks-Python， ShadowsocksR， Shadowsocks-Go， Shadowsocks-libev 版（四选一）服务端；
2. 各版本的启动脚本及配置文件名不再重合；
3. 每次运行可安装一种版本；
4. 支持以多次运行来安装多个版本，且各个版本可以共存（注意端口号需设成不同）；
5. 若已安装多个版本，则卸载时也需多次运行（每次卸载一种）；

# 默认配置
`服务器端口`：自己设定（如不设定，默认从 9000-19999 之间随机生成）

`密码`：自己设定（如不设定，默认为 teddysun.com）

`加密方式`：自己设定（如不设定，Python 和 libev 版默认为 aes-256-gcm，R 和 Go 版默认为 aes-256-cfb）

`协议（protocol）`：自己设定（如不设定，默认为 origin）（仅限 ShadowsocksR 版）

`混淆（obfs）`：自己设定（如不设定，默认为 plain）（仅限 ShadowsocksR 版）

`备注`：脚本默认创建单用户配置文件，如需配置多用户，请手动修改相应的配置文件后重启即可。

# 客户端下载
常规版 Windows 客户端:
https://github.com/shadowsocks/shadowsocks-windows/releases

ShadowsocksR 版 Windows 客户端:
https://github.com/shadowsocksrr/shadowsocksr-csharp/releases

# 安装ssr服务端
使用root用户登录，依次运行以下命令：

```
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```


安装完成后，脚本提示如下

```Congratulations, your_shadowsocks_version install completed!
Your Server IP        :your_server_ip
Your Server Port      :your_server_port
Your Password         :your_password
Your Encryption Method:your_encryption_method

Your QR Code: (For Shadowsocks Windows, OSX, Android and iOS clients)
 ss://your_encryption_method:your_password@your_server_ip:your_server_port
Your QR Code has been saved as a PNG file path:
 your_path.png

Welcome to visit:https://teddysun.com/486.html
Enjoy it!
```


## 卸载方法
若已安装多个版本，则卸载时也需多次运行（每次卸载一种）

使用root用户登录，运行以下命令：

```
./shadowsocks-all.sh uninstall
```


## 启动脚本
启动脚本后面的参数含义，从左至右依次为：启动，停止，重启，查看状态。

Shadowsocks-Python 版：
```
/etc/init.d/shadowsocks-python start | stop | restart | status
```

ShadowsocksR 版：
```
/etc/init.d/shadowsocks-r start | stop | restart | status
```

Shadowsocks-Go 版：
```
/etc/init.d/shadowsocks-go start | stop | restart | status
```

Shadowsocks-libev 版：
```
/etc/init.d/shadowsocks-libev start | stop | restart | status
```

各版本默认配置文件
Shadowsocks-Python 版：
```
/etc/shadowsocks-python/config.json
```

ShadowsocksR 版：
```
/etc/shadowsocks-r/config.json
```

Shadowsocks-Go 版：
```
/etc/shadowsocks-go/config.json
```

Shadowsocks-libev 版：
```
/etc/shadowsocks-libev/config.json
```



## BBR plus

```
wget "https://github.com/chiakge/Linux-NetSpeed/raw/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```



# BBR

## 使用BBR加速器
让访问速度加速，飞起来！使用 BBR 加速工具。

### 安装 BBR
```
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
```
获取读写权限
```
chmod +x bbr.sh
```
### 启动BBR安装
```
./bbr.sh
```
接着按任意键，开始安装，坐等一会。安装完成一会之后它会提示我们是否重新启动vps，我们输入 y 确定重启服务器。

重新启动之后，输入 lsmod | grep bbr 如果看到 tcp_bbr 就说明 BBR 已经启动了。

再访问一下 Youtube，1080p 超高清，很顺畅不卡顿！
