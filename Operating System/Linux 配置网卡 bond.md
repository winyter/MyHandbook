# Linux 配置网卡 bond

> 基于 CentOS7

- 通过命令确定内核是否支持 bonding

  ```shell
  cat /boot/config-2.6.32-573.el6.x86_64 |grep -i bonding
  ```

- 先做bond0内网网卡

  - 第一块网卡配置

    ![1123](pic/20190616194541353.png)

  - 第二块网卡配置

    ![在这里插入图片描述](pic/20190616194634991.png)

  - bond0的配置文件信息

    ![在这里插入图片描述](pic/20190616195252856.png)

- 再做bond1外网网卡

  - 第三块网卡配置

    ![在这里插入图片描述](pic/20190616195349678.png)

  - 第四块网卡配置

    ![在这里插入图片描述](pic/2019061619543578.png)

  - bond1的配置文件信息

    ![在这里插入图片描述](pic/20190616195523860.png)

- 都配置完后，重启网卡

  ```shell
  service network restart
  ```

- 创建加载模块，让系统支持bonding

  ![在这里插入图片描述](pic/20190616195602153.png)

- 加载bond module

  ![在这里插入图片描述](pic/20190616195633463.png)

- 查看是否有bond0和bond1这两块网卡信息

  ![在这里插入图片描述](pic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyMjg3OA==,size_16,color_FFFFFF,t_70.png)

- 查看绑定结果

  - bond0网卡信息

    ![在这里插入图片描述](pic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyMjg3OA==,size_16,color_FFFFFF,t_70-165184938288019.png)

  - bond1网卡信息

    ![在这里插入图片描述](pic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyMjg3OA==,size_16,color_FFFFFF,t_70-165184939640322.png)

