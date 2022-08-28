# Linux 查询网卡 UUID

某个网卡的UUID改错了或者删除了，重新配置的时候需要UUID怎么办

在Linux或CentOS中，可以通过如下命令获取网卡的uuid信息：

```shell
[root@ligle2 ~]# uuidgen eth1
07d07031-eb0f-4691-8606-befb46645433
```

获取到eth1网卡的uuid，即可完成对ifcfg-eth1配置文件的修改，最后通过service network restart命令重启网卡，OK。

