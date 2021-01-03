# SSH

## SSH 与 OpenSSH

### SSH（Secure Shell 的缩写）是一种网络协议，用于加密两台计算机之间的通信，并且支持各种身份验证机制

### OpenSSH 是 SSH  协议的免费开源实现

## SSH 客户端

### 安装

- OpenSSH 官网

- OpenSSH 的客户端是二进制程序 ssh

  它在 Linux/Unix 系统的位置是/usr/local/bin/ssh，Windows 系统的位置是\Program Files\OpenSSH\bin\ssh.exe。

### 基本命令使用

- ssh hostname

  此方式为不指定用户名。hostname是主机名，它可以是域名，也可能是IP 地址或局域网内部的主机名。

- ssh user@hostname

  指定用户名时，用户名和主机名用 @ 连接

- ssh -l username host

  指定用户名时，ssh -l 用参数指定，不用再使用@链接

- ssh -p 8821 host

  ssh 默认连接服务器的22端口，-p参数可以指定其他端口。

### 连接流程

- ssh 首次连接服务器，会有一个验证过程，提醒是否确认连接
- 每台 ssh 服务器都有唯一一对密钥，用于跟客户端通信，其中公钥的哈希值就可以用来识别服务器，简称服务器指纹
- ssh 会将本机连接过的所有服务器公钥的指纹，都储存在本机的 ~/.ssh/known_hosts 文件中

### 服务器密钥变更

- 问题：主机服务器密钥变更(比如重装ssh服务器)，导致公钥指纹不匹配，连接会失败
- 解决方式：1. 将 known_hosts 中保存的旧公钥指纹删除，然后再次 ssh 连接。2. 使用命令替换：ssh-keygen -R hostname

### 命令行配置项

- -c 指定加密算法：ssh -c blowfish,3des server.example.com
- -C 表示压缩数据传输:  ssh -C server.example.com
- -d 设置打印的 debug 信息级别，数值越高，输出的内容越详细: ssh –d 1 foo.com
- -D 指定本机的 Socks 监听端口，该端口收到的请求将转发到远程的 SSH 主机，又称动态端口转发: ssh -D 1080 server
- -f 表示 SSH 连接在后台运行
- -F 指定配置文件： ssh -F /usr/local/ssh/other_config
- -i 指定私钥，默认私钥为 ~/.ssh/id_dsa
- -l 指定远程登录的账户名
- -L 设置本地端口转发：ssh  -L 9999:targetServer:80 user@remoteserver

  所有发向本地9999端口的请求，都会经过remoteserver发往 targetServer 的 80 端口，这就相当于直接连上了 targetServer 的 80 端口。

- -m 指定校验数据完整性的算法:  ssh -m hmac-sha1,hmac-md5 server.example.com
- -p 指定 SSH 客户端连接的服务器端口: ssh -p 2035 server.example.com
- -v 显示详细信息，-v可以重复多次，表示信息的详细程度：ssh -vvv server.example.com
- -V 输出 ssh 客户端的版本
- -1 指定使用 SSH 1 协议，-2 指定使用 SSH 2 协议
- -4 指定使用 IPv4 协议（默认值），-6 指定使用 IPv6 协议

### 配置文件

- 全局配置文件是 /etc/ssh/ssh_config，用户个人的配置文件在 ~/.ssh/config，优先级高于全局配置文件
- ~/.ssh 目录下常见文件

	- ~/.ssh/id_ecdsa：用户的 ECDSA 私钥
	- ~/.ssh/id_ecdsa.pub：用户的 ECDSA 公钥
	- ~/.ssh/id_rsa：用于 SSH 协议版本2 的 RSA 私钥
	- ~/.ssh/id_rsa.pub：用于SSH 协议版本2 的 RSA 公钥
	- ~/.ssh/identity：用于 SSH 协议版本1 的 RSA 私钥
	- ~/.ssh/identity.pub：用于 SSH 协议版本1 的 RSA 公钥
	- ~/.ssh/known_hosts：包含 SSH 服务器的公钥指纹

### 配置命令

- 语法

	- 配置文件的每一行，就是一个配置命令
	- 配置命令与对应的值之间，可以使用空格，也可以使用等号
	- #开头的行表示注释，会被忽略。空行等同于注释

- 主要命令及范例值

  AddressFamily inet：表示只使用 IPv4 协议。如果设为inet6，表示只使用 IPv6 协议。
  BindAddress 192.168.10.235：指定本机的 IP 地址（如果本机有多个 IP 地址）。
  CheckHostIP yes：检查 SSH 服务器的 IP 地址是否跟公钥数据库吻合。
  Ciphers blowfish,3des：指定加密算法。
  Compression yes：是否压缩传输信号。
  ConnectionAttempts 10：客户端进行连接时，最大的尝试次数。
  ConnectTimeout 60：客户端进行连接时，服务器在指定秒数内没有回复，则中断连接尝试。
  DynamicForward 1080：指定动态转发端口。
  GlobalKnownHostsFile /users/smith/.ssh/my_global_hosts_file：指定全局的公钥数据库文件的位置。
  Host server.example.com：指定连接的域名或 IP 地址，也可以是别名，支持通配符。Host命令后面的所有配置，都是针对该主机的，直到下一个Host命令为止。
  HostKeyAlgorithms ssh-dss,ssh-rsa：指定密钥算法，优先级从高到低排列。
  HostName myserver.example.com：在Host命令使用别名的情况下，HostName指定域名或 IP 地址。
  IdentityFile keyfile：指定私钥文件。
  LocalForward 2001 localhost:143：指定本地端口转发。
  LogLevel QUIET：指定日志详细程度。如果设为QUIET，将不输出大部分的警告和提示。
  MACs hmac-sha1,hmac-md5：指定数据校验算法。
  NumberOfPasswordPrompts 2：密码登录时，用户输错密码的最大尝试次数。
  PasswordAuthentication no：指定是否支持密码登录。不过，这里只是客户端禁止，真正的禁止需要在 SSH 服务器设置。
  Port 2035：指定客户端连接的 SSH 服务器端口。
  PreferredAuthentications publickey,hostbased,password：指定各种登录方法的优先级。
  Protocol 2：支持的 SSH 协议版本，多个版本之间使用逗号分隔。
  PubKeyAuthentication yes：是否支持密钥登录。这里只是客户端设置，还需要在 SSH 服务器进行相应设置。
  RemoteForward 2001 server:143：指定远程端口转发。
  SendEnv COLOR：SSH 客户端向服务器发送的环境变量名，多个环境变量之间使用空格分隔。环境变量的值从客户端当前环境中拷贝。
  ServerAliveCountMax 3：如果没有收到服务器的回应，客户端连续发送多少次keepalive信号，才断开连接。该项默认值为3。
  ServerAliveInterval 300：客户端建立连接后，如果在给定秒数内，没有收到服务器发来的消息，客户端向服务器发送keepalive消息。如果不希望客户端发送，这一项设为0。
  StrictHostKeyChecking yes：yes表示严格检查，服务器公钥为未知或发生变化，则拒绝连接。no表示如果服务器公钥未知，则加入客户端公钥数据库，如果公钥发生变化，不改变客户端公钥数据库，输出一条警告，依然允许连接继续进行。ask（默认值）表示询问用户是否继续进行。
  TCPKeepAlive yes：客户端是否定期向服务器发送keepalive信息。
  User userName：指定远程登录的账户名。
  UserKnownHostsFile /users/smith/.ssh/my_local_hosts_file：指定当前用户的known_hosts文件（服务器公钥指纹列表）的位置。
  VerifyHostKeyDNS yes：是否通过检查 SSH 服务器的 DNS 记录，确认公钥指纹是否与known_hosts文件保存的一致。

## SSH 密钥登录

*XMind - Trial Version*