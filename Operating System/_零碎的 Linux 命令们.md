# 零碎的 Linux 命令们

- 查看 Java 程序堆内存使用情况

  ```shell
  jmap -heap <pid>
  ```

- 修改配置文件，将yum安装的rpm自动下载保存

  ```
  vi etc/yum.conf
  将keepcache=0修改为keepcache=1
  保存退出
  这样后面通过yum安装的rpm文件将会自动下载到/var/cache/yum/x86_64/7/*/packages
  项目组可以将packages中rpm文件打包用于现场安装和升级
  ```

- 查看内存槽数、哪些槽位插了内存及内存大小

  ```shell
  dmidecode | grep -P -A5 "Memory\s+Device" | grep Size | grep -v Range
  ```

- 查看最大支持内存数

  ```shell
  dmidecode | grep -P 'Maximum\s+Capacity'
  ```

- 查看槽位上内存的速率，没插的显示 unknown

  ```shell
  dmidecode | grep -A16 "Memory Device" | grep 'Speed'
  ```

