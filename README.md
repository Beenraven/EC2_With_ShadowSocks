# EC2_With_ShadowSocks（基于AWS_EC2搭建SS服务）

[购买云服务](#购买云服务)

[服务器端配置](#服务器端配置)

[常见错误处理](#常见错误处理)

# 购买云服务

![image](https://github.com/Beenraven/EC2_With_ShadowSocks/assets/129687108/b04c6245-005f-41f3-bb7f-92fe9295179f)

# 服务器端配置
__1、远程登录__

常用的主要是两种

	1、通过实例会话(EC2 Instance Connect)
	
	2、通过SSH客户端(Wins对应命令提示端，Mac对应Terminate)
	
登录成功如图所示

![image](https://github.com/Beenraven/EC2_With_ShadowSocks/assets/129687108/1459f031-6816-47df-bf20-99ba5f1ddab3)



__2、软件安装与更新__

>- python-pip
>- 更新pip版本
>- 安装shadowsocks



__2.1、安装python-pip__

	yum -y install python-pip
>> pip 是 Python 的包管理器。这意味着它是一个工具，允许你安装和管理不属于标准库的其他库和依赖。

__2.2、更新pip版本__

	pip install --upgrade pip

__2.3、安装shadowsocks__

	pip install shadowsocks



__3、配置shadowsocks__
>-ss配置文件编辑
>-ss启动脚本编辑

__3.1、SS配置文件编辑__	

	vim /etc/shadowsocks.json  

**单用户场景**

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
	
**多用户场景**	

	{
	    “server”:“0.0.0.0”,
	    “local_address”:“127.0.0.1”,
	    “local_port”:1080,
	    “port_password”:{
	       “端口1”:“密码1”, 
	       “端口2”:“密码2”,
	       “端口3”:“密码3”,
	       “端口4”:“密码4”,
	       “端口5”:“密码5”
	    },
	    “timeout”:300,
	    “method”:“aes-256-cfb”,
	    “fast_open”: false
	}
*修改上面的中文：“端口” 和 “密码”，这两个将在之后使用手机连接ss时要使用到，请务必记下来*

>- server 服务端监听地址(IPv4或IPv6)
>- server_port 服务端端口，一般为443
>- local_address 本地监听地址，缺省为127.0.0.1
>- local_port 本地监听端口，一般为1080
>- password 用以加密的密匙
>- timeout 超时时间（秒）
>- method 加密方法，默认为aes-256-cfb，更多请查阅Encryption
>- fast_open 是否启用TCP-Fast-Open，true或者false
>- workers worker数量，如果不理解含义请不要改（这个只在Unix和Linux下有用）


__3.2、SS启动脚本编辑__

	vim /etc/systemd/system/shadowsocks.service

具体参数如下

	[Unit]
	Description=Shadowsocks

	[Service]
	TimeoutStartSec=0
	ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json

	[Install]
	WantedBy=multi-user.target



__4、启动shadowsocks服务__

停止服务

	systemctl enable shadowsocks
	
启用服务
	
	systemctl start shadowsocks
	
查看状态

	systemctl status shadowsocks -l
![image](https://github.com/Beenraven/EC2_With_ShadowSocks/assets/129687108/f0264701-f9c5-49f1-b446-674fd0175687)



__5、配置安全组__

__通过配置安全组，用户可以通过指定的ip/端口号访问服务器__

![image](https://github.com/Beenraven/EC2_With_ShadowSocks/assets/129687108/ff0e32f9-c529-450f-abb1-b0a39c5d1fa4)
![image](https://github.com/Beenraven/EC2_With_ShadowSocks/assets/129687108/00341baa-0e74-4d6d-abe4-21cbfe3359d7)




# 常见错误处理
>- undefined symbol: EVP_CIPHER_CTX_cleanup
>- OSError: [Errno 98] Address already in use

__1、undefined symbol: EVP_CIPHER_CTX_cleanup__

错误截图
![1683959257087](https://github.com/Beenraven/EC2_With_ShadowSocks/assets/129687108/4d8ff71a-e853-4db6-93ec-c72de5b39f42)
**openssl版本升级,函数EVP_CIPHER_CTX_cleanup被替换为EVP_CIPHER_CTX_reset导致的报错**

__解决方案__

>打开问题的py文件```vi /usr/local/lib/python3.9/site-packages/shadowsocks/crypto/openssl.py```
>
>替换对应函数2处为EVP_CIPHER_CTX_reset，保存并退出


![1683961045159](https://github.com/Beenraven/EC2_With_ShadowSocks/assets/129687108/76b00608-cb6c-4a05-a982-8be830f8f019)

![1683961095483](https://github.com/Beenraven/EC2_With_ShadowSocks/assets/129687108/4a0ceafb-4f6a-4182-8f8f-b13b60f5c6ef)

>重新启动shadowsocks服务


__2、OSError: [Errno 98] Address already in use__
错误截图
![image](https://github.com/Beenraven/EC2_With_ShadowSocks/assets/129687108/6a5b39cb-1a83-45c8-bcb3-a02ac2a9d319)
***shoadowsocks配置的端口号被占用**

__解决方案__

>查询当前与shadowsocks配置文件中冲突的端口号	```netstat -tunlp |grep 端口号```

	
![image](https://github.com/Beenraven/EC2_With_ShadowSocks/assets/129687108/0728ded7-4979-4a0e-97c5-cf37da378e94)

>重新配置shadowsocks中的端口号，避免冲突（参考SS配置文件编辑）
>
>或者杀掉当前占用端口的进程(不推荐)```sudo kill -9 进程号(pid)```
