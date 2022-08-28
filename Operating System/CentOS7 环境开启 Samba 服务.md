# CentOS7 环境开启 Samba 服务

> 环境说明：
>
> 两台 CentOS 7.5 的服务器节点，需要将一台服务器的目录共享给另外一台。
>
> 挂载端：即需要被共享的目录所在的节点，也就是需要开启 Smb 服务的节点，假设 IP：172.168.1.100，需要共享的目录为：/home/aaa/output，该目录用户属组为：aaa:users
>
> 被挂载端：即需要执行挂载命令的节点。假设IP：172.168.1.101，被挂载的目录为：/home/bbb/input，该目录的用户属组为：bbb:users
>
> 难点：两个节点的共享及被共享目录均非 root 用户，且不允许讲相关共享目录权限修改为 root，这就要求对 Samba 挂载的权限有足够的了解，否则即使挂载成功，也很可能出现无权限读写的问题

## 一、挂载端操作【被挂载端的 root 用户下执行】

修改 Samba 服务配置

```shell
vim /etc/samba/smb.conf
======== 确认如下内容 =========
[global] 配置组的：security 配置项，值为：user，如果不是，请修改

======== 添加如下内容 =========
[share]    
        comment = All users
        path = /home/aaa/output
        read only = no
        inherit acls = Yes
        veto files = /aquota.user/groups/shares/
======== 部分配置说明 =========
[share]：共享配置别名，可根据需要修改，该别名会在执行 mount 命令时用到
path：要共享的目录路径，请填写绝对路径
read only：是否只读，配置为 no，表示可读可写
```

重启 Samba 服务

```shell
systemctl restart smb.service
```

确认服务已运行

```shell
systemctl status smb.service
```

确认共享目录读取正常

```shell
testparm
======== 如果有返回如下内容，即表示读取正常 ========
Processing section "[share]"
Loaded services file OK.
```

将共享目录的所属用户，添加为 Samba 用户（如果有多个不同用户属组的目录需要共享，则请添加多个 Samba 用户）

```shell
smbpasswd -a aaa
# 根据提示，输入两遍密码即可，这个密码可以与操作系统的用户密码相同，也可以不同，建议设置相同的密码
======== 如果最后返回如下内容，即表示添加成功 ========
Added user aaa.
```



## 二、被挂载端操作【在挂载端 root 用户下执行】

> 首先必须确保被挂载端节点能正常通过网络访问挂载端节点

查看 `bbb` 用户的 `uid` 及 `gid`

```shell
id bbb
========= 会返回如下内容，我们主要取 uid 和 gid 两项 ========
uid=1013(bbb) gid=100(users) groups=100(users)
```

执行命令，进行挂载

```shell
mount -t cifs //172.168.1.100/share /home/bbb/input/ -orw,uid=1013,gid=100,username=aaa,password=aaa

========= 命令参数解析 ========
mount -t cifs //[remote_ip]/[samba_share_name] [local_path] -orw,uid=[local_user_id],gid=[local_group_id],username=[remote_username],password=[remote_user_password]
[remote_ip]：远端节点 IP，即挂载端节点的 IP，我们这里是：172.168.1.100
[samba_share_name]：samba 共享目录别名，即我们在第一节，修改/etc/samba/smb.conf时，添加的那个别名，我们这里是：share
[local_path]：本地目录路径，即被挂载的目录路径，我们这里是：/home/bbb/input/
-orw：表示给予读写权限
[local_user_id]：本地用户的 uid，即我们前一步查询的用户 uid，我们这里是：1013
[local_group_id]：本地用户的 gid，即我们前一步查询的用户 gid，我们这里是：100
[remote_username]：远端共享目录用户名，即我们在第一节最后，使用 smbpasswd 命令添加的用户名
[remote_user_password]：远端共享目录用户密码，即我们在第一节最后，使用 smbpasswd 命令设置的用户密码
```

检查挂载是否成功

```shell
df -h
```



## 三、检查用户权限是否有问题

分别登录挂载端的 `aaa` 用户，以及被挂载端的 `bbb` 用户，在挂载的目录下，执行文件/文件夹的增删改等操作，确认是否可以正常执行，如果可以正常执行，即表示权限没有问题

