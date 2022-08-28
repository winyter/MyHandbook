# SFTP

> sFTP（安全文件传输程序）是一种安全的交互式文件传输程序，其工作方式与 FTP（文件传输协议）类似。 然而，sFTP 比 FTP 更安全；它通过加密 SSH 传输处理所有操作。
>
> 它可以配置使用几个有用的 SSH 功能，如公钥认证和压缩。 它连接并登录到指定的远程机器，然后切换到交互式命令模式，在该模式下用户可以执行各种命令。

## 常用操作

- SFTP 登录使用类似于 SSH 的密码方式，同时，为了安全并简化连接操作，也可以创建和使用 SSH 无密码登录

- 为了防止用户访问远程主机上的整个文件系统，出于安全原因，你可以使用 chroot Jail将 sFTP 用户限制到其主目录中。

- 常用命令

  ```shell
  # 登录命令
  sftp user@ip
  # 基本文件操作
  sftp> ls            #list directory 
  sftp> pwd           #print working directory on remote host 
  sftp> lpwd          #print working directory on local host 
  sftp> mkdir uploads     #create a new directory
  # 上传文件夹，需要注意：上传时，需提前在远程主机上新建同名文件夹，否则上传会报错
  sftp> put -r fold_name 
  # 要保留修改时间、访问时间以及被传输的文件的模式，需添加 -p 参数
  sftp> put -pr fold_name
  # 下载文件夹
  sftp> get -r fold_name
  # 退出 sftp
  sftp> bye  或  sftp> exit
  ```

## FTP 默认 21 和 20 端口号的区别

> ftp端口号20和21的区别是：一个是数据端口，一个是控制端口，控制端口一般为21，而数据端口不一定是20，这和FTP的应用模式有关，如果是主动模式，应该为20，如果为被动模式，由服务器端和客户端协商而定。



## FTP Port模式和FTP Passive模式(主动与被动模式)

> FTP是仅基于TCP的服务，不支持UDP。 与众不同的是FTP使用2个端口，一个数 据端口和一个命令端口（也可叫做控制端口）。通常来说这两个端口是21（命令端 口）和20（数据端口）。但FTP工作方式的不同，数据端口并不总是20。这就是主 动与被动FTP的最大不同之处。

- **FTP主动模式工作流程**

  主动方式的FTP是这样的：

  客户端从一个任意的非特权端口N（N>1024）连接到FTP服务器的命令端口，也就是21端口。然后客户端开始 监听端口N+1，并发送FTP命令“port N+1”到FTP服务器。接着服务器会从它自己的数据端口（20）连接到客户端指定的数据端口（N+1）。

  针对FTP服务器前面的防火墙来说，必须允许以下通讯才能支持主动方式FTP：

  1. 任何大于1024的端口到FTP服务器的21端口。（客户端初始化的连接）
  2. FTP服务器的21端口到大于1024的端口。 （服务器响应客户端的控制端口）
  3. FTP服务器的20端口到大于1024的端口。（服务器端初始化数据连接到客户端的数据端口）
  4. 大于1024端口到FTP服务器的20端口（客户端发送ACK响应到服务器的数据端口）

  - **简明概括：**
    PORT（主动）方式的连接过程是：客户端向服务器的FTP端口（默认是21）发送连接请求，服务器接受连接，建立一条命令链路。当需要传送数据时，客户端在命令链路上用PORT命令告诉服务器：“我打开了XXXX端口，你过来连接我”。于是服务器从20端口向客户端的XXXX端口发送连接请求，建立一条数据链路来传送数据。

  - **开启主动模式：**
    pasv_enable=no
    若设置为YES，则使用PASV工作模式；若设置为NO，则使用PORT模式。默认值为YES，即使用PASV工作模式。

  - **主动模式下：**
    SecureFX工具去连接ftp，客户没有允许开放端口，服务器没法与客户端相连接，关闭客户端防火墙

- **FTP被动模式**

  为了解决服务器发起到客户的连接的问题，人们开发了一种不同的FTP连接方式。这就是所谓的被动方式，或者叫做PASV，当客户端通知服务器它处于被动模式时才启用。

  在被动方式FTP中，命令连接和数据连接都由客户端发起，这样就可以解决从服务器到客户端的数据端口的入方向连接被防火墙过滤掉的问题。

  当开启一个 FTP连接时，客户端打开两个任意的非特权本地端口（N > 1024和N+1）。第一个端口连接服务器的21端口，但与主动方式的FTP不同，客户端不会提交PORT命令并允许服务器来回连它的数据端口，而是提交 PASV命令。这样做的结果是服务器会开启一个任意的非特权端口（P > 1024），并发送PORT P命令给客户端。然后客户端发起从本地端口N+1到服务器的端口P的连接用来传送数据。

  对于服务器端的防火墙来说，必须允许下面的通讯才能支持被动方式的FTP:

  1. 从任何大于1024的端口到服务器的21端口 （客户端初始化的连接）
  2. 服务器的21端口到任何大于1024的端口 （服务器响应到客户端的控制端口的连接）
  3. 从任何大于1024端口到服务器的大于1024端口 （客户端初始化数据连接到服务器指定的任意端口）
  4. 服务器的大于1024端口到远程的大于1024的端口（服务器发送ACK响应和数据到客户端的数据端口）

  - **简明概括：**
    PASV（被动）方式的连接过程是：客户端向服务器的FTP端口（默认是21）发送连接请求，服务器接受连接，建立一条命令链路。当需要传送数据时，服务器在命令链路上用PASV命令告诉客户端：“我打开了XXXX端口，你过来连接我”。于是客户端向服务器的XXXX端口发送连接请求，建立一条数据链路来传送数据。

  - **开启被动模式**

    ```shell
    默认是开启的，但是要指定一个端口范围，打开vsftpd.conf文件，在后面加上
    pasv_enable=yes
    若设置为YES，则使用PASV工作模式；若设置为NO，则使用PORT模式。默认值为YES，即使用PASV工作模式。
    pasv_min_port=30000
    在PASV工作模式下，数据连接可以使用的端口范围的最大端口，0 表示任意端口。默认值为0。
    pasv_max_port=30999
    在PASV工作模式下，数据连接可以使用的端口范围的最小端口，0 表示任意端口。默认值为0。
    ```

  表示端口范围为30000~30999，这个可以随意改。改完重启一下vsftpd由于指定这段端口范围，iptables也要相应的开启这个范围，所以像上面那样打开iptables文件。

  也是在21上下面另起一行，更那行差不多，只是把21 改为30000:30999,然后:wq保存，重启下iptables。这样就搞定了。

- **主动与被动FTP优缺点：**

  主动FTP对FTP服务器的管理有利，但对客户端的管理不利。因为FTP服务器企图与客户端的高位随机端口建立连接，而这个端口很有可能被客户端的防火墙阻塞掉。

  被动FTP对FTP客户端的管理有利，但对服务器端的管理不利。因为客户端要与服务器端建立两个连接，其中一个连到一个高位随机端
  口，而这 个端口很有可能被服务器端的防火墙阻塞掉。

  幸运的是，有折衷的办法。既然FTP服务器的管理员需要他们的服务器有最多的客户连接，那么必须得支持被动FTP。我们可以通过为FTP服务器指定一个有限的端口范围来减小服务器高位端口的暴露。这样，不在这个范围的任何端口会被服务器的防火墙阻塞。虽然这没有消除所有针对服务器的危险，但它大大减少了危险。

  

  - **简而言之：**
    主动模式（PORT）和被动模式（PASV）。主动模式是从服务器端向客户端发起连接；被动模式是客户端向服务器端发起连接。两者的共同点是都使
    用21端口进行用户验证及管理，差别在于传送数据的方式不同，PORT模式的FTP服务器数据端口固定在20，而PASV模式则在1025-65535之间随机。

  - **常见的FTP客户端软件的PASV方式的关闭方法**
    大部分FTP客户端默认使用PASV方式。IE默认使用PORT方式。 在大部分FTP客户端的设置里，常见到的字眼都是“PASV”或“被动模式”，
    极少见到“PORT”或“主动模式”等字眼。因为FTP的登录方式只有两种：PORT和PASV，取消PASV方式，就意味着使用PORT方式。
    （1）IE：工具 -> Internet选项 -> 高级 -> “使用被动FTP”（需要IE6.0以上才支持）。
    （2）CuteFTP：Edit -> Setting -> Connection -> Firewall -> “PASV Mode” 或File -> Site Manager，在左边选中站
    点 -> Edit -> “Use PASV mode” 。
    （3）FlashGet：工具 -> 选项 -> 代理服务器 -> 直接连接 -> 编辑 -> “PASV模式”。
    （4）FlashFXP：选项 -> 参数选择 -> 代理/防火墙/标识 -> “使用被动模式” 或 站点管理 -> 对应站点 -> 选项 -> 
    “使用被动模式”或快速连接 -> 切换 -> “使用被动模式”。

- **主动模式配置实操**

  ```shell
  主动模式
  Port_enable=YES               开启主动模式
  Connect_from_port_20=YES      当主动模式开启的时候 是否启用默认的20端口监听
  Ftp_date_port=%portnumber%    上一选项使用NO参数是 指定数据传输端口
  ```

- **被动模式配置实操**

  ```shell
  被动模式
  PASV_enable=YES   开启被动模式
  PASV_min_port=%number% 被动模式最低端口
  PASV_max_port=%number% 被动模式最高端口
  iptables中开放这段端口
  service iptables start 打开防火墙
  iptables -I INPUT  -p tcp  --dport 10020:10040  -j ACCEPT
  iptables -A INPUT -p tcp -m tcp --dport 21 -j ACCEPT
  
  在被动模式，服务器做了NAT，例如云主机，这时候我们用特定的IP访问机器，其实还转了一层。FTP客户端访问机器可能会没响应。具体情况为登录成功，但是list目录和文件的时候卡住。
  这时候我们用lsof -i:21
  vsftpd   22411   nobody    0u  IPv4  68905      0t0  TCP 10.140.41.65:ftp->10.10.10.98:43380 (ESTABLISHED)
  vsftpd   22411   nobody    1u  IPv4  68905      0t0  TCP 10.140.41.65:ftp->10.10.10.98:43380 (ESTABLISHED)
  这时候可以看到机器的真正IP。
  我们需要设置
  pasv_address=本机ip【就是我们能访问的外网IP】
  pasv_addr_resolve=yes
  这样ftp客户端就可以解析IP，访问成功
  ```

- **总结**

  1. 被动模式

     第二次请求过程中，客户端跟服务端建立数据通道； 

     服务端被动将数据响应给客户端。

     第二次请求数据传输，会随机生成一个服务端口。被防火墙禁用。 

  2. 主动模式

     服务端主动向客户端发送数据，会被客户端的防火墙禁掉。 

     多数客户端不支持主动模式，不安全。





## CentOS7 vsftpd 服务配置解析

- 安装 vsftpd

  ```shell
  yum install -y vsftpd
  ```

- /etc/vsftpd/vsftpd.conf 配置文件解析

  ```shell
  不允许匿名访问
  anonymous_enable=NO
  
  本地用户登录ftp后的根目录
  local_root=/var/ftp
  
  本地ftp用户权限
  file_open_mode=0755
  
  解决新版本vsftpd本地用户主目录如果有写权限时，无法登录的问题
  allow_writeable_chroot=YES
  
  登录用户是否有写入的权限
  write_enable=YES
  
  锁定登录用户不能访问主目录的上级目录
  chroot_local_user=YES
  
  ftp数据传输使用的接口
  ftp_data_port=20
  
  启用被动传输模式
  pasv_enable=YES
  
  被动模式传输时开始的端口号
  pasv_min_port=40000
  
  被动模式传输时结束的端口号
  pasv_max_port=40100
  ```

- 创建一个ftp本地账户

  useradd ftpuser -s /sbin/nologin 添加账户

  passwd ftpuser 设置密码

- 把根目录的权限设置为此用户可以写入删除

  chmod o+w+r /var/ftp/

- 在防火墙中添加被动传输的端口

  ```shell
  firewall-cmd --zone=public --add-port=20-21/tcp --permanent
  firewall-cmd --zone=public --add-port=40000-40100/tcp --permanent
  firewall-cmd --reload
  ```

- 查看selinux配置是否正常（允许用户登录）或关闭 selinux

  ```
  sestatus -b | grep ftp
  如果全部都是off，则根据需求进行打开，比如：
  （setsebool -P ftpd_full_access on systemctl restart vsftpd #重启vsftpd）
  ```

- 把服务设置成开机启动 

  ```shell
  systemctl enable sshd.service
  ```



## vsftpd 开启 SSL/TLS 证书加密

> 基于 Ubuntu 系统

生成证书

```shell
openssl req -x509 -nodes -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/private/vsftpd.pem -days 365 -newkey rsa:2048
```

会提示你配置相关信息，提供参考

```shell
Country Name (2 letter code) [XX]:IN
State or Province Name (full name) []:Lower Parel
Locality Name (eg, city) [Default City]:Mumbai
Organization Name (eg, company) [Default Company Ltd]: mi-di.cn
Organizational Unit Name (eg, section) []:midi
Common Name (eg, your name or your server's hostname) []:midi
Email Address []:1076735024@qq.com
```

接下来配置防火墙，放通TLS连接证书的通信端口（下面命令以 Ubuntu 为例，其他发行版本按情况修改命令）

```shell
sudo ufw allow 990/tcp
```

打开vsftpd配置文件

```shell
vi /etc/vsftpd.conf
```

进行下面的配置

```shell
rsa_cert_file=/etc/ssl/private/vsftpd.pem 
rsa_private_key_file=/etc/ssl/private/vsftpd.key 

ssl_enable=YES 
ssl_tlsv1=YES 
ssl_sslv2=NO 
ssl_sslv3=NO

allow_anon_ssl=NO 
force_local_data_ssl=YES 
force_local_logins_ssl=YES 

require_ssl_reuse=NO 
ssl_ciphers=HIGH  
debug_ssl=YES 
```

开启了DEBUG，其中出了什么问题可以去/var/log/vsftpd.log下进行查看

重启服务

```shell
systemctl restart vsftpd 
```

记得使用支持SSL/TLS加密的FTP软件哦，有一些是不支持的，所以连不上该服务器，推荐使用FileZilla

