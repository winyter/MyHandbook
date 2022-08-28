# 记一次 CentOS 7 系统盘分区调整

有一台服务器因为安装人员的失误，导致 `root`根目录只分配了 50G，而`/home` 目录分配了 10T+，这明显是非常不合理的，因此，需要把 `home` 的容量抠出来，给 `root` 使用

```
lvremove /dev/mapper/centos-home 
lvextend -L +1024G /dev/mapper/centos-root 
lvcreate -L 1024G -n home centos
mkfs.xfs /dev/mapper/centos-home 
xfs_growfs /dev/mapper/centos-root 
vim /etc/fstab
mount /dev/mapper/centos-home 
df -h
```

