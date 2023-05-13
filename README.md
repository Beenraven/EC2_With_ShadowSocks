# EC2_With_ShadowSocks
基于AWS_EC2搭建SS服务

![image](https://github.com/Beenraven/EC2_With_ShadowSocks/assets/129687108/b04c6245-005f-41f3-bb7f-92fe9295179f)

yum -y install python-pip

pip install --upgrade pip

pip install shadowsocks

vi /etc/shadowsocks.json

{
 "server":"0.0.0.0",
 "server_port":端口,
 "local_address":"127.0.0.1",
 "local_port":1080,
 "password":"密码",
 "timeout":300,
 "method":"aes-256-cfb",
 "fast_open":false,
 "workers": 1
}

修改上面的中文：“端口” 和 “密码”，这两个将在之后使用手机连接ss时要使用到，请务必记下来

配置文件说明

server 服务端监听地址(IPv4或IPv6)
server_port 服务端端口，一般为443
local_address 本地监听地址，缺省为127.0.0.1
local_port 本地监听端口，一般为1080
password 用以加密的密匙
timeout 超时时间（秒）
method 加密方法，默认为aes-256-cfb，更多请查阅Encryption
fast_open 是否启用TCP-Fast-Open，true或者false
workers worker数量，如果不理解含义请不要改（这个只在Unix和Linux下有用）


编辑启动脚本
vim /etc/systemd/system/shadowsocks.service
[Unit]
Description=Shadowsocks

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target



运行ss服务端
ssserver -c /etc/shadowsocks.json -d start

错误处理

运行时产生告警：

AttributeError: /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup

参考文章：Kali2.0 update到最新版本后安装shadowsocks服务报错问题

这是由于在 openssl1.1.0版本中，EVP_CIPHER_CTX_cleanup函数被替换为EVP_CIPHER_CTX_reset。

修改方法：
1.用vim打开文件：vim /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py (该路径请根据自己的系统情况自行修改，如果不知道该文件在哪里的话，可以使用find命令查找文件位置)
2.跳转到52行（shadowsocks2.8.2版本，其他版本搜索一下cleanup）
3.进入编辑模式
4.将第52行libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,) 改为libcrypto.EVP_CIPHER_CTX_reset.argtypes = (c_void_p,)
5.再次搜索cleanup（全文件共2处，此处位于111行），将libcrypto.EVP_CIPHER_CTX_cleanup(self._ctx) 改为libcrypto.EVP_CIPHER_CTX_reset(self._ctx)
6.保存并退出
7.启动shadowsocks服务：service shadowsocks start 或 sslocal -c ss配置文件目录

8.问题解决
