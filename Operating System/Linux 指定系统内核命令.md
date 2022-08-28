# Linux 指定系统内核命令
- 查看系统可用内核
``` shell
#查看系统可用内核，并设置启动项
$ sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (5.17.1-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-1160.53.1.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
3 : CentOS Linux (0-rescue-20220208145000711038896885545492) 7 (Core)
```
- 指定开机启动内核版本
``` shell
#指定开机启动内核版本
$ grub2-set-default 0 
或者 
$ grub2-set-default 'CentOS Linux (5.17.1-1.el7.elrepo.x86_64) 7 (Core)'
```
- 生成 grub 配置文件
``` shell
$ grub2-mkconfig -o /boot/grub2/grub.cfg
```

- 重启系统验证

```shell
$ uname -a
```

