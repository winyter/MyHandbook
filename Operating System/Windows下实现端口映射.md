# Windows下实现端口映射

> 通常服务器会有许多块网卡，因此也可能会连接到不同的网络，在隔离的网络中，某些服务可能会需要进行通信，此时服务器经过配置就可以承担起了转发数据包的功能。



1. 查询端口映射情况

   ```shell
   netsh interface portproxy show v4tov4
   ```

2. 查询某一个IP的所有端口映射情况

   ```shell
   netsh interface portproxy show v4tov4 | find "[IP]"
   例：netsh interface portproxy show v4tov4 | find "192.168.1.1"
   ```

3. 增加一个端口映射

   ```shell
   netsh interface portproxy add v4tov4 listenaddress=[外网IP] listenport=[外网端口] connectaddress=[内网IP] connectport=[内网端口]
   例：netsh interface portproxy add v4tov4 listenaddress=2.2.2.2 listenport=8080 connectaddress=192.168.1.50 connectport=80
   ```

4. 删除一个端口映射

   ```shell
   netsh interface portproxy delete v4tov4 listenaddress=[外网IP] listenport=[外网端口]
   例：netsh interface portproxy delete v4tov4 listenaddress=2.2.2.2 listenport=8080
   ```

   
