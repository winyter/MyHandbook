# Linux 命令执行卡死与 strace 命令

1、首先使用strace ls命令跟踪查看执行到哪一步卡死

```shell
strace ls /…
lgetxattr(“/MegaSAS.log”, “system.posix_acl_default”, 0x0, 0) = -1 ENODATA (No data available) lstat(“/net”, {st_mode=S_IFDIR|0755, st_size=0, …}) = 0
lgetxattr(“/net”, “security.selinux”, 0x258f850, 255) = -1 EOPNOTSUPP (Operation not supported) lstat(“/net”, {st_mode=S_IFDIR|0755, st_size=0, …}) = 0
lgetxattr(“/net”, “system.posix_acl_access”, 0x0, 0) = -1 EOPNOTSUPP (Operation not supported) lstat(“/chaichuan_test”, {st_mode=S_IFDIR|0755, st_size=4096, …}) = 0
lgetxattr(“/chaichuan_test”, “security.selinux”, 0x258f850, 255) = -1 ENODATA (No data available) lstat(“/chaichuan_test”, {st_mode=S_IFDIR|0755, st_size=4096, …}) = 0
lgetxattr(“/chaichuan_test”, “system.posix_acl_access”, 0x0, 0) = -1 ENODATA (No data available)
lgetxattr(“/chaichuan_test”, “system.posix_acl_default”, 0x0, 0) = -1 ENODATA (No data available) lstat(“/mnt”, {st_mode=S_IFDIR|0775, st_size=4096, …}) = 0
lgetxattr(“/mnt”,
```

可以看到,命令执行到 /mnt这个目录时,停止不动了

也可以使用cat /proc/mounts 查看当前mount状态,发现确实有对mnt目录的记录

2、fuser查看问题目录进程

问题定位/mnt目录,使用fuser 查看此目录占用进程

fuser -m /mnt 无法获取 /proc/4110/fd/255 的文件状态: 失效文件句柄

无法获取 /proc/9492/fd/255 的文件状态: 失效文件句柄

无法获取 /proc/29965/fd/255 的文件状态: 失效文件句柄

3、kill命令解决

找到进程号直接kill

> strace命令是一个集诊断、调试、统计与一体的工具，我们可以使用strace对应用的系统调用和信号传递的跟踪结果来对应用进行分析，以达到解决问题或者是了解应用工作过程的目的, 具体使用可以查看帮助。

