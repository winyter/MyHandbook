# GP 常用命令

- GP 后台登录命令

  ```shell
  psql -h [ip] -p [port] [database] [username]
  ```

- GP服务启停

  ```shell
  # 正常启动
  gpstart
  # 正常关闭
  gpstop
  # 快速关闭
  gpstop -M fast
  # 重启
  gpstop –r
  # 重新加载配置文件
  gpstop –u
  ```

- 集群恢复

  ```shell
  命令     参数   作用
  gprecoverseg -a => 快速恢复
  gprecoverseg -i => 指定恢复文件
  gprecoverseg -d => 指定数据目录
  gprecoverseg -l => 指定日志文件
  gprecoverseg -r => 平衡数据
  gprecoverseg -s => 指定配置空间文件
  gprecoverseg -o => 指定恢复配置文件
  gprecoverseg -p => 指定额外的备用机
  gprecoverseg -S => 指定输出配置空间文件
  ```

- 激活备库流程

  ```shell
    命令     参数   作用
    gpactivatestandby -d 路径 | 使用数据目录绝对路径，默认：$MASTER_DATA_DIRECTORY
    gpactivatestandby -f | 强制激活备份主机
    gpactivatestandby -v | 显示此版本信息
  ```

- 初始化备 Master

  ```shell
    命令     参数   作用
    gpinitstandby -s 备库名称 => 指定新备库
    gpinitstandby -D => debug 模式
    gpinitstandby -r => 移除备用机
  ```

  