---
title: SSR + 跳板搭建记录
date: 2018-01-03
slug: proxy/ssr-1
---

鉴于我的 ss 服务商总是带着租金跑路，我决定搭建一个 ss 服务器，考虑到我正在使用和电话拨号速度相当的长城宽带，
还需要架设一个国内服务器用于跳板，预算约是 ss 服务器 2.5$ ，跳板服务器 5$ 。
你可以通过我的 [邀请链接](https://www.vultr.com/?ref=7213278) 购买服务器，我会得到额外的奖励。反感可以去掉链接尾部的 id。


> 如果你的网络服务提供商没有屏蔽 ss 服务器的 IP 或端口，完全可以跳板跳板。

### 环境
我在 [vultr](https://www.vultr.com/?ref=7213278) 购买 2.5$ 一个月的服务器，不被干扰可以正常浏览 1080P youtube ，如果你需要每天看 twitch 这类直播，可能需要 5$ 或更好的服务器。

先创建一个用户用于日常的登录：
```bash
# add new user
useradd {name}
passwd {name}

```   
给新用户加上 sudo 权限：
`sudo vi /etc/sudoers` :
```shell  
root    ALL=(ALL)     ALL
{name}    ALL=(ALL)    ALL    # add
```  

### 多用户 SSR
用脚本安装 SSR ：
```bash
# install  
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/shadowsocks_install/master/shadowsocksR.sh && bash shadowsocksR.sh

# commander  
/etc/init.d/shadowsocks {start/stop/restart/status}

# show log  
cat /var/log/shadowsocks.log

# update  
cd /usr/local/shadowsocks/shadowsocks && gl

# uninstall
./shadowsocksR.sh uninstall
```

#### 多用户
默认的配置在 `/etc/shadowsocks.json` ，如果你希望多用户登录，可以删除原有的帐号密码，在配置文件中增加
端口对于密码的键值对。（记得这是个 JSON 文件，不要有语法错误）

```bash
# close firewalld
sudo systemctl stop firewalld.service

# edit config
vi /etc/shadowsocks.json
```
```json
{
  "server": "0.0.0.0",
  "local_address": "127.0.0.1",
  "local_port": 1080,
  "port_password": {
    "3000": "123456"
  },
  "...other config": ""
}
```  

### 客户端 SSR
虽然 SSR 脚本能够快速生成一个服务端，但不具备流量控制查询，也没有可视化窗口，如果你需要这些，可以根据下面的配置
在服务器上安装一个客户端。就是你常见的那些黑心商人的网站了 🤔

参考 [文档](https://sspanel.xyz/docs)

参考 [项目](https://github.com/shadowsocks/shadowsocks-go)

```bash
## install docker and docker-compose
...

# run compose
wget https://raw.githubusercontent.com/orvice/ss-panel/master/docker-compose.yml
docker-compose up -d

# install ss-go  
go get shadowsocks/shadowsocks-go/cmd/shadowsocks-server

```


### 跳板
可以选择自己用国内的服务器搭设一个跳板，转发 tcp/udp 到 [vultr](https://www.vultr.com/?ref=7213278) 服务器，或者是直接使用专业的网络链路代理。


#### 利用 `haproxy` 实现资源转发：

```bash
apt-get install haproxy
sudo vi /etc/haproxy/haproxy.cfg
```

在 `/etc/haproxy/haproxy.cfg` 中编辑:

```
global
        ulimit-n  51200

defaults
        log	global
        mode	tcp
        option	dontlognull
        contimeout 1000
        clitimeout 150000
        srvtimeout 150000

frontend app
        bind *: {PORT}
        default_backend ss-out

backend app
        server server1 {VULTR_IP:PORT} maxconn 20480
```

运行 `haproxy -f /etc/haproxy/haproxy.cfg` .

其他命令:
```
# 停止
sudo systemctl stop haproxy.service
# 查看状态
sudo systemctl status haproxy.service
# 启动
sudo systemctl start haproxy.service

# 如果你不是 centOS 7, 可以使用
service haproxy start/stop/restart
```

#### 专业的代理提供商

你可以使用 [VLink](https://vnet.link/?rc=52957) 购买专业的链路级网络优化。

略贵，不过优化比较好，比手动在国内搭跳板会更快一些。





