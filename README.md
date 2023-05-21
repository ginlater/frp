# frp
记一次痛苦的内网vpn经历，被easyconnect折磨到死，还是frp好用

## 1 frp 内网穿透教程 与 可能的问题解决方案

---- 这是一次关于使用easyconnect折磨了我8个小时无法ssh到内网主机的教训经验

---- 到现在为止，我使用easyconnect ssh还是连不上内网服务器

----最终用阿里云配置frp成功内网穿透 打到内网服务器上



### easyconnect 失败教训

垃圾，真难用

1. docker配置方案失败
2. easyconnect网页登录 + easyconnect客户端 失败
3. 关闭了服务器hosts.deny ufw都不行
4. 可以访问http服务tcp可以，但是不能ssh，ping也不行icmp连接不起来。



### frp 内网穿透成功

具体教程参考 https://developer.aliyun.com/article/937673

但是有一些大坑：

1. 如果第一次客户端或者服务器frpc.ini / frps.ini配置错了，重新修改后不能直接`sudo systemctl start frps` 来覆盖之前的服务配置。需要`sudo systemctl restart start frps`来覆盖。
   1. 可以用`sudo netstat -tunlp`查看本机正在运行的进程,ip和监听的端口号
   2. 可以用`journalctl -u frps.service`查看服务端frps日志，对应的`journalctl -u frpc.service`查看客户端frps日志。
   3. 搭配`| tail -10可以查看最新的10条日志`
2. 使用`ssh username@ip -p port`连接的时候，千万注意：
   1. username ： 是你内网主机的用户名，不是阿里云服务器的
   2. ip ： 是阿里云服务器的公网ip
   3. port是frp 客户端与 阿里云服务端建立tcp连接的端口号，不是22
   4. 后面输入的密码，是你内网服务器的密码！！！！！
   5. 可能会出现以下报错：
      1. ssh: connect to host x.x.x.x port x: Connection timed out 解决方案 ：阿里云防火墙安全组 没有添加 port 入口规则允许。
      2. ssh permission denied, please try again. 解决方案 ： 检查一下自己内网服务器/etc/hosts.allow 是否允许了localhosts或者127.0.0.1； 然后也可以重启一下sshd服务：`sudo service sshd restart`

## 2 github终端访问过慢，配代理方法
> windows终端不走代理的可能的原因和解决方法有以下几种：

- windows终端没有设置代理环境变量，导致无法使用代理软件提供的代理服务¹²。你可以在终端中输入以下命令来设置代理环境变量，其中端口值要根据你的代理软件的设置来修改：

```bash
set HTTP_PROXY=http://127.0.0.1:端口值
set HTTPS_PROXY=http://127.0.0.1:端口值
```

- windows终端设置的代理环境变量不正确，导致无法使用代理软件提供的代理服务³。你可以检查你的代理软件的协议类型和端口号，然后根据相应的格式来设置代理环境变量。例如，如果你的代理软件使用socks5协议，你可以在终端中输入以下命令来设置代理环境变量：

```bash
set HTTP_PROXY=socks5://127.0.0.1:端口值
set HTTPS_PROXY=socks5://127.0.0.1:端口值
```

- windows终端设置的代理环境变量只对当前会话有效，导致关闭终端后失效¹²。你可以在系统的环境变量中添加或修改相应的代理环境变量，这样可以使其永久生效。具体的操作步骤如下：

  - 右键点击我的电脑，选择属性，然后选择高级系统设置。
  - 在高级选项卡中，点击环境变量按钮。
  - 在系统变量中，添加或修改HTTP_PROXY和HTTPS_PROXY这两个变量，值为相应的代理地址和端口号。
  - 点击确定，保存修改。

