# Linux 查看分区文件系统信息的几种方法

- **df -T**

  这个是最简单的命令，文件系统类型在Type列输出。只可以查看已经挂载的分区和文件系统类型。如下所示：

  ```shell
  [root@mylnx008 ~]# df -T /dev/sdb
  
  Filesystem     Type 1K-blocks    Used Available Use% Mounted on
  
  /dev/sdb       xfs  315467264 4356404 311110860   2% /mysql
  
  
  [root@mylnx008 ~]# df -T
  
  Filesystem     Type     1K-blocks     Used Available Use% Mounted on
  
  /dev/sda2      xfs       30929148 22455300   8473848  73% /
  
  devtmpfs       devtmpfs   1746644        0   1746644   0% /dev
  
  tmpfs          tmpfs      1757220        0   1757220   0% /dev/shm
  
  tmpfs          tmpfs      1757220    24868   1732352   2% /run
  
  tmpfs          tmpfs      1757220        0   1757220   0% /sys/fs/cgroup
  
  /dev/sda1      xfs         508580    63024    445556  13% /boot
  
  /dev/sdc1      ext4     139203080  8699072 123409840   7% /mnt/resource
  
  tmpfs          tmpfs       351448        0    351448   0% /run/user/1000
  
  /dev/sdb       xfs      315467264  4356404 311110860   2% /mysql
  
  ```

- **parted -l**

  如下所示，parted -l 命令会输出文件系统类型（File system）， 其中参数l表示列出所有设备的分区信息。

  ```shell
  [root@DB-Server ~]# parted -l
  
  Model: ATA ST500DM002-1BD14 (scsi)
  
  Disk /dev/sda: 500GB
  
  Sector size (logical/physical): 512B/512B
  
  Partition Table: msdos
  
   
  
  Number  Start   End    Size   Type     File system  Flags
  
  1      32.3kB  107MB  107MB  primary  ext3         boot
  
  2      107MB   500GB  500GB  primary               lvm
  ```

- **blkid**

  查看已格式化分区的UUID和文件系统。使用blkid可以输出分区或分区的文件系统类型，查看TYPE字段输出。

  ```shell
  [root@DB-Server ~]# blkid
  
  /dev/mapper/VolGroup00-LogVol01: TYPE="swap"
  
  /dev/mapper/VolGroup00-LogVol00: UUID="1c0d5470-1503-4a18-b184-53483466d948" TYPE="ext3"
  
  /dev/sda1: LABEL="/boot" UUID="582b189c-396c-4da8-a7a3-1effaa3e4000" TYPE="ext3"
  
  /dev/VolGroup00/LogVol00: UUID="1c0d5470-1503-4a18-b184-53483466d948" TYPE="ext3"
  
  /dev/VolGroup00/LogVol01: TYPE="swap"
  
  /dev/mapper/VolGroup00-LogVol03: UUID="f037ba1e-77a1-439a-8a10-b78c3cca68ec" SEC_TYPE="ext2" TYPE="ext3"
  
  [root@DB-Server ~]# blkid  /dev/sda1
  
  /dev/sda1: LABEL="/boot" UUID="582b189c-396c-4da8-a7a3-1effaa3e4000" TYPE="ext3"
  ```

- lsblk -f

  有些系统可能没有这个命令，需要安装。注意：lsblk -f也可以查看未挂载的文件系统类型

  ```shell
  [root@mylnx008 ~]# lsblk -f
  
  NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
  
  fd0
  
  sda
  
  ├─sda1 xfs          b98659b2-5f8c-493e-9304-658905ef1391 /boot
  
  └─sda2 xfs          b7559ac5-b3a4-4b00-b98a-a2a2611806d0 /
  
  sdb    xfs          6fcc5417-3c1b-4c71-aac7-344bac7654a4 /mysql
  
  sdc
  
  └─sdc1 ext4         1ad7da45-2366-4c4f-acd4-484600c4153a /mnt/resource
  
  ```

  
