# Linux 下端口映射

1. 允许数据包转发

   ```shell
   echo 1 >/proc/sys/net/ipv4/ip_forwardiptables -t nat -A POSTROUTING -j MASQUERADEiptables -A FORWARD -i [内网网卡名称] -j ACCEPTiptables -t nat -A POSTROUTING -s [内网网段] -o [外网网卡名称] -j MASQUERADE 
   例：echo 1 >/proc/sys/net/ipv4/ip_forwardiptables -t nat -A POSTROUTING -j MASQUERADEiptables -A FORWARD -i ens33 -j ACCEPTiptables -t nat -A POSTROUTING -s 192.168.50.0/24 -o ens37 -j MASQUERADE
   ```

2. 设置端口映射
```shell
iptables -t nat -A PREROUTING -p tcp -m tcp --dport [外网端口] -j DNAT --to-destination [内网地址]:[内网端口]
例：iptables -t nat -A PREROUTING -p tcp -m tcp --dport 6080 -j DNAT --to-destination 10.0.0.100:6090
```

