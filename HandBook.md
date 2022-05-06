# Handbook

## Linux

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

- Linux 添加PATH 环境变量的几种方法（以添加 PYTHONPATH 为例）

  ```
  Linux下设置PYTHONPATH环境变量有三种方法：一种作用于当前终端，一种作用于当前用户，一种作用于所有用户。
  
  1.作用于当前终端，直接当前终端输入命令
    $ export PYTHONPATH=$PYTHONPATH:<你的要加入的路径>
    $ export PYTHONPATH=$PYTHONPATH:/home/hadoop/MyBI
    注1：'/home/hadoop/MyBI'是项目MyBI的项目名
    注2：作用范围当前终端，一旦当前终端关闭或在另一个终端中，则无效。
    注3：这种方式立即生效。
  
  2.作用于当前用户，修改当前用户目录下的'~/.bashrc'文件
    $ vi ~/.bashrc
    加入内容：
    export PYTHONPATH=$PYTHONPATH:/home/hadoop/MyBI
    也可以加入多个路径，用分号分隔
    export PYTHONPATH=$PYTHONPATH:<你的要加入的路径1>:<你的要加入的路径2>:等等
  
    注1：需要执行如下命令后生效（或者注销后重新登陆）
    $ source ~/.bashrc  
  
  3.作用于所有用户（需要root权限修改），修改'/etc/profile'文件
    $ vi /etc/profile
    加入内容：
    export PYTHONPATH=$PYTHONPATH:/home/hadoop/MyBI
    
    注1：需要执行如下命令后生效（或者注销后重新登陆）
    $ source /etc/profile
  ```



### Linux 标准输出、错误输出、标准输入

| 数字 | 含义         | 标准叫法                 |
| ---- | ------------ | ------------------------ |
| 0    | 标准输入     | stdin = standard input   |
| 1    | 标准输出     | stdout = standard output |
| 2    | 标准错误输出 | stderr = standard error  |

一些重定向的用法

```shell
1.想要把make输出的全部信息，输出到某个文件中，最常见的办法就是：
make xxx > build_output.txt
此时默认情况是没有改变2=stderr的输出方式，还是屏幕，所以，如果有错误信息，还是可以在屏幕上看到的。

2.只需要把make输出中的错误（及警告）信息输出到文件中ing，可以用：
make xxx 2> build_output.txt
相应地，由于1=stdout没有变，还是屏幕，所以，那些命令执行时候输出的正常信息，还是会输出到屏幕上，你还是可以在屏幕上看到的。

3.只需要把make输出中的正常（非错误，非警告）的信息输出到文件中，可以用：
make xxx 1> build_output.txt
相应地，由于2=stderr没有变，还是屏幕，所以，那些命令执行时候输出的错误信息，还是会输出到屏幕上，你还是可以在屏幕上看到的。

4.想要把正常输出信息和错误信息输出到分别的文件中，可以用：
make xxx 1> build_output_normal.txt 2>build_output_error.txt
即联合使用了1和2，正常信息和错误信息，都输出到对应文件中了。

5. 所有的信息都输出到同一个文件中：
make xxx > build_output_all.txt 2>&1
其中的2>&1表示错误信息输出到&1中，而&1，指的是前面的那个文件：build_output_all.txt 。
注意：上面所有的1,2等数字，后面紧跟着大于号'>' ，中间不能有空格
```



### vm.overcommit_memory & vm.overcommit_ratio

overcommit_memory: 有三个值可以配置
- 0: 用户申请内存的时候，系统会判断剩余的内存多少，如果不够的话那么就会失败
- 1: 用户申请内存的时候，系统不进行任何检查任务内存足够用，直到使用内存超过可用内存
- 2: 用户一次申请的内存大小不允许超过可用内存的大小

overcommit_ratio: 仅当 overcommit_memory 为 2 时，该参数才会生效，这个参数决定了系统可用内存的大小

计算公式:

```
(Physical-RAM-Size）* ratio / 100 + (Swap-Size)
  Physical-RAM-Size: 物理内存大小，具体为 free 命令回显中 Mem 行 Total 列的值
  Swap-Size: swap 内存大小，具体为 free 命令回显中 Swap 行 Total 列的值
  ratio: 即 overcommit_ratio 参数的配置值
```



### shell 异常处理的几个方法

1. 判断命令返回值

   ```shell
   #!/bin/sh
   cd /home/xxxx/
   if [ "$?"= "0" ]; then
      rm -rf *
   else
      echo "cannot change directory" 1>&2
      exit 1
   fi 
   ```

2. && 和 ||

   &&: 只有在shell1返回0（即正常）时，shell2才会执行，否则shell2根本就不执行

   ||: 在shell1执行失败时，shell2才会执行，否则shell2是不执行的

   ```shell
   cd /home/xxxx/ && rm -rf *
   cd /home/xxxx || error_exit "Cannot change directory"
   rm -rf *
   
   -------
   function error_exit {
     echo "$1" 1>&2
     exit 1
   }
   ```

3. trap




### Linux 内存管理之 RSS 和 VSZ

**RSS**( Resident Set Size )：常驻内存集合大小。表示相应进程在 RAM 中占用了多少内存，并不包含在 SWAP 中占用的虚拟内存。即使是在内存中的使用了共享库的内存大小也一并计算在内，包含了完整在 stack 和 heap 中的内存

**VSZ**( Virtual Memory Size)：虚拟内存大小。表明了该进程可以访问的所有内存，包括被交换的内存和共享库内存。

eg. 如果进程A的二进制文件大小为500KB，并且链接到了2500KB的共享库，有200KB的stack/heap大小，这200KB中又有100KB位于内存中，100KB位于[SWAP](https://so.csdn.net/so/search?q=SWAP&spm=1001.2101.3001.7020)空间中，并且加载了1000KB的共享库和400KB的自身二进制文件。则

​	RSS：400K + 1000K + 100K = 1500K

​	VSZ：500K + 2500K + 200K = 3200K



### ps 命令详解

> Linux ps （英文全拼：process status）命令用于显示当前进程的状态，类似于 windows 的任务管理器。

#### 基本内容

- 语法

  ```
  ps [options] [--help]
  ```

- 参数

  - ps 的参数非常多, 在此仅列出几个常用的参数并大略介绍含义
  - -A 列出所有的进程
  - -w 显示加宽可以显示较多的资讯
  - -au 显示较详细的资讯
  - -aux 显示所有包含其他使用者的进程

#### ps -aux

- au(x) 输出格式

  ```
  USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
  ```

  - USER: 进程拥有者
  - PID: 进程ID，进程的唯一标识符
  - %CPU: 占用的 CPU 使用率
  - %MEM: 占用的内存使用率
  - VSZ: 占用的虚拟内存大小(表示如果一个程序完全驻留在内存的话需要占用多少内存空间)
  - RSS: 占用的内存大小(常驻集大小)
  - TTY: 终端的次要装置号码 (minor device number of tty)
  - STAT: 该进程的状态:
    - D: 不可中断 Uninterruptible（usually IO）,不可被喚醒的睡眠狀態，通常這个程序可能在等待I/O的情況(ex>列印)
    - R: 运行，正在运行或在运行队列中等待
    - S: 进程处在睡眠状态(idle)，但可以被喚醒(signal)，表明这些进程在等待某些事件发生–可能是用户输入或者系统资源的可用性
    - T: 停止狀態(stop)，可能是在工作控制(背景暫停)或除錯 (traced) 狀態
    - Z: 僵死 ，进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放
    - W: 进入内存交换（从内核2.6开始无效）
    - <: 高优先序的行程
    - N: 低优先序的行程
    - L: 有些页被锁进内存 (实时系统或捱A I/O)
    - X: 死掉的进程
    - <: 高优先级
    - n: 低优先级
    - s: 包含子进程
    - +: 位于后台的进程组；
    - l: 多线程，克隆线程multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
    - WCHAH: 正在等待的进程资源
  - START: 行程开始时间
  - TIME: 执行的时间
  - COMMAND:所执行的指令



#### ps -ef

- -e 显示所有进程
- -f 全格式
- -h 不显示标题
- -l 长格式
- r 只显示正在运行的进程
- x 显示没有控制终端的进程
- a 显示终端上的所有进程，包括其他用户的进程

ps -ef 命令输出格式

```shell
UID       PID       PPID      C     STIME    TTY       TIME         CMD
zzw      14124   13991      0     00:38      pts/0      00:00:00    grep --color=auto dae
```

- UID   ：程序被该 UID 所拥有

- PID   ：就是这个程序的 ID

- PPID  ：则是其上级父程序的ID

- C     ：CPU使用的资源百分比

- STIME ：系统启动时间

- TTY   ：登入者的终端机位置

- TIME  ：使用掉的CPU时间。

- CMD  ：所下达的是什么指令



#### 实例

查找指定进程格式：

```
ps -ef | grep 进程关键字
```

显示所有进程信息，连同命令行

```
ps -ef
```

显示进程信息：

```
ps -A 
```

显示指定用户信息

```
ps -u root //显示root进程用户信息
```



 



### sed 命令

- sed 递归替换当前目录下所有文件，将 x 替换为 y

  ```shell
  find . -type f -print0 | xargs -0 sed -i "s/x/y/g"
  ```





### iconv 命令详解

- 功能

  对于给定文件，把它的内容从一种编码转换成另一种编码

- 描述

  -f encoding :把字符从encoding编码开始转换。 

  -t encoding :把字符转换到encoding编码。 

  -l :列出已知的编码字符集合 

  -o file :指定输出文件 

  -c :忽略输出的非法字符 

  -s :禁止警告信息，但不是错误信息 

  --verbose :显示进度信息 

  -f和-t所能指定的合法字符在-l选项的命令里面都列出来了。 

- 举例

  列出当前支持的字符编码：

  ```shell
  iconv -l
  ```

  将文件file1转码,转后文件输出到fil2中： 

  ```shell
  iconv  -f EUC-JP-MS -t UTF-8 file1 -o file2   //没-o那么会输出到标准输出.
  ```

  实际需求，从hive中取出的数据是utf8的，要load到mysql中，gbk编码。所以在load之前要先对文件进行转码。

  ```shell
  mysql_cmd = "iconv -c -f utf-8 -t gbk ./data/al_ver_" + yesterday_time + ".xls -o ./data/GBK_al_ver_" + yesterday_time + ".xls "
  print(mysql_cmd)
  os.system(mysql_cmd)
   
  mysql_cmd = "mysql -h60.28.200.78 -uroot -pyeelion -A LogStat_RT  -e \"load data local  infile \'./data/GBK_al_ver_" + yesterday_time + ".xls ' into table HiveData_508\""
  print(mysql_cmd)
  os.system(mysql_cmd)
  ```




### Linux 系统负载高德问题排查思路与解决方法

Load 就是对计算机干活多少的度量，Load Average 就是一段时间（1分钟、5分钟、15分钟）内平均Load。

 linux服务器出现高负载的情况下，一般都有一些具体的症状，比如cpu、内存等被耗尽，磁盘IO或者网络等出现问题，下面通过具体命令去分析解决高负载的问题

`htop`命令：实时更新占用cpu、内存等资源的进程，可以通过分析排名最前的进程来定位问题

`iotop`命令：实时监测磁盘IO使用情况，系统高负载一般也有可能是大量的小文件的读写引起的

`vmstat`命令：实时查看虚拟内存swap、cache等使用情况，一般系统高负载的情况下 cache可能会被耗尽

#### 1、工具

> htop collectl iotop vmstat

- htop

  ```
  yum -y install htop
  ```

  命令行输入htop，这个是实时更新占用cpu、内存等资源的进程，可以通过分析排名最前的进程来定位问题

- collectl

  ```
  yum insatll -y collectl
  ```

  直接运行collectl命令，collectl命令可以实时监测网卡的进出口流量，系统的高负载一般与网卡流量有很大关系

- iotop

  通过iotop命令实时监测磁盘IO使用情况，系统高负载一般也有可能是大量的小文件的读写引起的

- vmstat

  通过vmstat命令实时查看虚拟内存swpd、cache等使用情况，一般系统高负载的情况下 cache可能会被耗尽

  同时，也可以通过该命令从操作系统维度查看 CPU 资源的使用情况

  ```shell
  # 表示结果一秒刷新一次
  vmstat -n 1 -n 1
  
  [root@k8s-10 ~]# vmstat -n 1
  procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
   r  b   swpd   free   buff  cache     si   so    bi    bo   in    cs us sy id wa st
   1  1      0 2798000   2076 6375040    0    0    10    76   10    49  6  2 91  1  0
   0  0      0 2798232   2076 6375128    0    0     0   207 7965 12525  7  2 90  2  0
  ```

  返回结果中的主要数据列说明：

  **r**： 表示系统中 CPU 等待处理的线程。由于 CPU 每次只能处理一个线程，所以，该数值越大，通常表示系统运行越慢。

  **b**： 表示阻塞的进程,这个不多说，进程阻塞，大家懂的。

  **us**： 用户CPU时间，我曾经在一个做加密解密很频繁的服务器上，可以看到us接近100,r运行队列达到80(机器在做压力测试，性能表现不佳)。

  **sy**： 系统CPU时间，如果太高，表示系统调用时间长，例如是IO操作频繁。

  **wa**：IO 等待消耗的 CPU 时间百分比。该值较高时，说明 IO 等待比较严重，这可能磁盘大量作随机访问造成的，也可能是磁盘性能出现了瓶颈。

  **id**：处于空闲状态的 CPU 时间百分比。如果该值持续为 0，同时 sy 是 us 的两倍，则通常说明系统则面临着 CPU 资源的短缺。

  **常见问题及解决方法：**

  如果r经常大于4，且id经常少于40，表示cpu的负荷很重。

  如果pi，po长期不等于0，表示内存不足。

  如果disk经常不等于0，且在b中的队列大于3，表示io性能不好

- top

  可以通过 top 从进程维度来查看其 CPU、内存等资源的使用情况。

  ```shell
  [root@k8s-10 ~]# top -c
  top - 19:53:49 up 2 days,  7:57,  3 users,  load average: 0.76, 0.79, 0.58
  Tasks: 282 total,   2 running, 280 sleeping,   0 stopped,   0 zombie
  %Cpu(s):  2.4 us,  1.4 sy,  0.0 ni, 95.0 id,  1.2 wa,  0.0 hi,  0.0 si,  0.0 st
  KiB Mem : 12304204 total,  2800864 free,  3119064 used,  6384276 buff/cache
  KiB Swap:        0 total,        0 free,        0 used.  8164632 avail Mem
  
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
  29884 root      20   0 5346580 929332  14556 S   0.0  7.6   6:19.19 /opt/jdk1.8.0_144/bin/java +
    875 root      20   0  729524 563424  38612 S   3.1  4.6  93:22.70 kube-apiserver --authorizat+
   3870 nfsnobo+  20   0  910376 317248  22812 S   1.6  2.6  42:29.59 /bin/prometheus --config.fi+
  ```

  默认界面上第三行会显示当前 CPU 资源的总体使用情况，下方会显示各个进程的资源占用情况。

  可以直接在界面输入大小字母 P，来使监控结果按 CPU 使用率倒序排列，进而定位系统中占用 CPU 较高的进程。最后，根据系统日志和程序自身相关日志，对相应进程做进一步排查分析，以判断其占用过高 CPU 的原因。

  

#### 2、Load 分析：

- CPU高、Load高

  ```
      通过top命令查找占用CPU最高的进程PID；
      通过top -Hp PID查找占用CPU最高的线程TID;
      对于java程序，使用jstack打印线程堆栈信息（可联系业务进行排查定位）；
      通过printf %x tid打印出最消耗CPU线程的十六进制；
      在堆栈信息中查看该线程的堆栈信息；
  ```

- CPU低、Load高

  Linux 系统没有业务程序运行，通过 top 观察，类似如下图所示，CPU 很空闲，但是 load average 却非常高：

  load average 是对 CPU 负载的评估，其值越高，说明其任务队列越长，处于等待执行的任务越多。

  出现此种情况时，可能是由于僵死进程导致的。可以通过指令`ps -axjf`查看是否存在 D 状态进程。

  D 状态是指不可中断的睡眠状态。该状态的进程无法被 kill，也无法自行退出。只能通过恢复其依赖的资源或者重启系统来解决。

  ```shell
  等待I/O的进程通过处于uninterruptible sleep或D状态；通过给出这些信息我们就可以简单的查找出处在wait状态的进程
  ps -eo state,pid,cmd | grep "^D"; echo "----"
  通过top命令查看CPU等待IO时间，即%wa；
  通过iostat -d -x -m 1 10查看磁盘IO情况；(安装命令 yum install -y sysstat)
  通过sar -n DEV 1 10查看网络IO情况；
  通过如下命令查找占用IO的程序:
  ps -e -L h o state,cmd  | awk '{if($1=="R"||$1=="D"){print $0}}' | sort | uniq -c | sort -k 1nr
  ```

    

### 网卡 bond

#### 1、什么是 bond

  网卡bond是通过多张网卡绑定为一个逻辑网卡，实现本地网卡的冗余，带宽扩容和负载均衡，在生产场景中是一种常用的技术。Kernels 2.4.12及以后的版本均供bonding模块，以前的版本可以通过patch实现。可以通过以下命令确定内核是否支持 bonding：

```shell
cat /boot/config-2.6.32-573.el6.x86_64 |grep -i bonding
```

#### 2、bond 的模式

bond的模式常用的有两种：

- mode=0（balance-rr）

  表示负载分担round-robin，并且是轮询的方式比如第一个包走eth0，第二个包走eth1，直到数据包发送完毕。

  优点：流量提高一倍

  缺点：需要接入交换机做端口聚合，否则可能无法使用

- mode=1（active-backup）

  表示主备模式，即同时只有1块网卡在工作。

  优点：冗余性高

  缺点：链路利用率低，两块网卡只有1块在工作

- bond其他模式：

  - mode=2(balance-xor)(平衡策略)

    表示XOR Hash负载分担，和交换机的聚合强制不协商方式配合。（需要xmit_hash_policy，需要交换机配置port channel）

    特点：基于指定的传输HASH策略传输数据包。缺省的策略是：(源MAC地址 XOR 目标MAC地址) % slave数量。其他的传输策略可以通过xmit_hash_policy选项指定，此模式提供负载平衡和容错能力

  - mode=3(broadcast)(广播策略)

    表示所有包从所有网络接口发出，这个不均衡，只有冗余机制，但过于浪费资源。此模式适用于金融行业，因为他们需要高可靠性的网络，不允许出现任何问题。需要和交换机的聚合强制不协商方式配合。

    特点：在每个slave接口上传输每个数据包，此模式提供了容错能力

  - mode=4(802.3ad)(IEEE 802.3ad 动态链接聚合)

    表示支持802.3ad协议，和交换机的聚合LACP方式配合（需要xmit_hash_policy）.标准要求所有设备在聚合操作时，要在同样的速率和双工模式，而且，和除了balance-rr模式外的其它bonding[负载均衡](https://so.csdn.net/so/search?q=负载均衡&spm=1001.2101.3001.7020)模式一样，任何连接都不能使用多于一个接口的带宽。

    特点：创建一个聚合组，它们共享同样的速率和双工设定。根据802.3ad规范将多个slave工作在同一个激活的聚合体下。外出流量的slave选举是基于传输hash策略，该策略可以通过xmit_hash_policy选项从缺省的XOR策略改变到其他策略。需要注意的是，并不是所有的传输策略都是802.3ad适应的，尤其考虑到在802.3ad标准43.2.4章节提及的包乱序问题。不同的实现可能会有不同的适应性。

    必要条件：

    - 条件1：ethtool支持获取每个slave的速率和双工设定
    - 条件2：switch(交换机)支持IEEE802.3ad Dynamic link aggregation
    - 条件3：大多数switch(交换机)需要经过特定配置才能支持802.3ad模式

  - mode=5(balance-tlb)(适配器传输负载均衡)

    是根据每个slave的负载情况选择slave进行发送，接收时使用当前轮到的slave。该模式要求slave接口的网络设备驱动有某种ethtool支持；而且ARP监控不可用。

    特点：不需要任何特别的switch(交换机)支持的通道bonding。在每个slave上根据当前的负载（根据速度计算）分配外出流量。如果正在接受数据的slave出故障了，另一个slave接管失败的slave的MAC地址。

    必要条件：

    - ethtool支持获取每个slave的速率

  - mode=6(balance-alb)(适配器适应性负载均衡)

    在5的tlb基础上增加了rlb(接收负载均衡receiveload balance).不需要任何switch(交换机)的支持。接收负载均衡是通过ARP协商实现的.

    特点：该模式包含了balance-tlb模式，同时加上针对IPV4流量的接收负载均衡(receiveload balance, rlb)，而且不需要任何switch(交换机)的支持。接收负载均衡是通过ARP协商实现的。bonding驱动截获本机发送的ARP应答，并把源硬件地址改写为bond中某个slave的唯一硬件地址，从而使得不同的对端使用不同的硬件地址进行通信。来自服务器端的接收流量也会被均衡。当本机发送ARP请求时，bonding驱动把对端的IP信息从ARP包中复制并保存下来。当ARP应答从对端到达时，bonding驱动把它的硬件地址提取出来，并发起一个ARP应答给bond中的某个slave。使用ARP协商进行负载均衡的一个问题是：每次广播 ARP请求时都会使用bond的硬件地址，因此对端学习到这个硬件地址后，接收流量将会全部流向当前的slave。这个问题可以通过给所有的对端发送更新（ARP应答）来解决，应答中包含他们独一无二的硬件地址，从而导致流量重新分布。当新的slave加入到bond中时，或者某个未激活的slave重新激活时，接收流量也要重新分布。接收的负载被顺序地分布（round robin）在bond中最高速的slave上当某个链路被重新接上，或者一个新的slave加入到bond中，接收流量在所有当前激活的slave中全部重新分配，通过使用指定的MAC地址给每个 client发起ARP应答。下面介绍的updelay参数必须被设置为某个大于等于switch(交换机)转发延时的值，从而保证发往对端的ARP应答不会被switch(交换机)阻截。

#### 3、配置bond

> 基于 CentOS7

- 通过命令确定内核是否支持 bonding

  ```shell
  cat /boot/config-2.6.32-573.el6.x86_64 |grep -i bonding
  ```

- 先做bond0内网网卡

  - 第一块网卡配置

    ![1](HandBook/20190616194541353.png)

  - 第二块网卡配置

    ![在这里插入图片描述](HandBook/20190616194634991.png)

  - bond0的配置文件信息

    ![在这里插入图片描述](HandBook/20190616195252856.png)

- 再做bond1外网网卡

  - 第三块网卡配置

    ![在这里插入图片描述](HandBook/20190616195349678.png)

  - 第四块网卡配置

    ![在这里插入图片描述](HandBook/2019061619543578.png)

  - bond1的配置文件信息

    ![在这里插入图片描述](HandBook/20190616195523860.png)

- 都配置完后，重启网卡

  ```shell
  service network restart
  ```

- 创建加载模块，让系统支持bonding

  ![在这里插入图片描述](HandBook/20190616195602153.png)

- 加载bond module

  ![在这里插入图片描述](HandBook/20190616195633463.png)

- 查看是否有bond0和bond1这两块网卡信息

  ![在这里插入图片描述](HandBook/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyMjg3OA==,size_16,color_FFFFFF,t_70.png)

- 查看绑定结果

  - bond0网卡信息

    ![在这里插入图片描述](HandBook/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyMjg3OA==,size_16,color_FFFFFF,t_70-165184938288019.png)

  - bond1网卡信息

    ![在这里插入图片描述](HandBook/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzgyMjg3OA==,size_16,color_FFFFFF,t_70-165184939640322.png)







## MySQL











## ElasticSearch

- ES 查询可检索的内容

  ```
  curl 172.168.1.1:9200/_cat/
  ```

- ES 查询未分配的分片

  ```shell
  curl 172.168.1.1:9200/_cat/shards | grep 'UNASSIGNED'
  ```

- ES 查询集群健康状态

  ```shell
  curl 172.168.1.1:9200/_cluster/health?pretty
  ```

- 修改Es分片备份数：

  ```shell
  curl -XPUT 'http://ip:9200/_settings' -H 'Content-Type:application/json' -d '{"index":{"number_of_replicas":0}}'
  ```

- 修改每分钟编译次数

  ```shell
  curl -XPUT 'http://172.168.1.1:9200/_cluster/settings' -H 'Content-Type:application/json' -d '
  {
      "transient" : {
          "script.max_compilations_rate" : "2000/1m"
      }
  }'
  ```

- EsMaster 数量配置

  ```shell
  discovery_zen_minimum_master_nodes
  # 配置为 ES 节点数 / 2 + 1，例如：4 个 ES 节点，则备选举的节点数=4/2+1=3
  ```

  



## YARN

- 下载某个 Yarn 任务的日志

  ```shell
  yarn logs -applicationId <id>
  ```

  





## HBase









## Hive

- Hive 后台登录命令

  ```shell
  beeline -u jdbc:hive2://172.168.1.1:10000 -n hive -p hive
  ```

- Hive 删除数据库

  ```
  drop database if exists [database] cascade;
  # 删除完成后，需要同步删除 HDFS 上的目录
  ```

  







## GP

### GP 常用命令

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

  

### GP 命令详解

- gpstate 命令详解

  ```shell
  # 查看状态
  gpstate
  # 查看mirror的状态
  gpstate -e
  # 查看 standby master 的状态
  gpstate -f
  # 查看整个 GP 群集的状态
  gpstate -s
  # 查看 GP 的版本
  gpstate -i
  # 帮助文档，可以查看gpstate更多用法
  gpstate --help
  # 其他 gpstatus 命令参数
  gpstate -b => 显示简要状态
  gpstate -c => 显示主镜像映射
  gpstart -d => 指定数据目录（默认值：$MASTER_DATA_DIRECTORY）
  gpstate -e => 显示具有镜像状态问题的片段
  gpstate -f => 显示备用主机详细信息
  gpstate -i => 显示GRIPLUM数据库版本
  gpstate -m => 显示镜像实例同步状态
  gpstate -p => 显示使用端口
  gpstate -Q => 快速检查主机状态
  gpstate -s => 显示集群详细信息
  gpstate -v => 显示详细信息
  ```

- gpconfig 命令详解

  ```shell
  命令    参数                              作用
  gpconfig -c => --change param_name  通过在postgresql.conf 文件的底部添加新的设置来改变配置参数的设置。
  gpconfig -v => --value value 用于由-c选项指定的配置参数的值。默认情况下，此值将应用于所有Segment及其镜像、Master和后备Master。
  gpconfig -m => --mastervalue master_value 用于由-c 选项指定的配置参数的Master值。如果指定，则该值仅适用于Master和后备Master。该选项只能与-v一起使用。
  gpconfig -masteronly =>当被指定时，gpconfig 将仅编辑Master的postgresql.conf文件。
  gpconfig -r => --remove param_name 通过注释掉postgresql.conf文件中的项删除配置参数。
  gpconfig -l => --list 列出所有被gpconfig工具支持的配置参数。
  gpconfig -s => --show param_name 显示在Greenplum数据库系统中所有实例（Master和Segment）上使用的配置参数的值。如果实例中参数值存在差异，则工具将显示错误消息。使用-s=>选项运行gpconfig将直接从数据库中读取参数值，而不是从postgresql.conf文件中读取。如果用户使用gpconfig 在所有Segment中设置配置参数，然后运行gpconfig -s来验证更改，用户仍可能会看到以前的（旧）值。用户必须重新加载配置文件（gpstop -u）或重新启动系统（gpstop -r）以使更改生效。
  gpconfig --file => 对于配置参数，显示在Greenplum数据库系统中的所有Segment（Master和Segment）上的postgresql.conf文件中的值。如果实例中的参数值存在差异，则工具会显示一个消息。必须与-s选项一起指定。
  gpconfig --file-compare 对于配置参数，将当前Greenplum数据库值与主机（Master和Segment）上postgresql.conf文件中的值进行比较。
  gpconfig --skipvalidation 覆盖gpconfig的系统验证检查，并允许用户对任何服务器配置参数进行操作，包括隐藏参数和gpconfig无法更改的受限参数。当与-l选项（列表）一起使用时，它显示受限参数的列表。 警告： 使用此选项设置配置参数时要格外小心。
  gpconfig --verbose 在gpconfig命令执行期间显示额外的日志信息。
  gpconfig --debug 设置日志输出级别为调试级别。
  gpconfig -? | -h | --help 显示在线帮助。
  ```

- gpstart 命令详解

  ```shell
  命令     参数   作用
  gpstart -a => 快速启动|
  gpstart -d => 指定数据目录（默认值：$MASTER_DATA_DIRECTORY）
  gpstart -q => 在安静模式下运行。命令输出不显示在屏幕，但仍然写入日志文件。
  gpstart -m => 以维护模式连接到Master进行目录维护。例如：$ PGOPTIONS='-c gp_session_role=utility' psql postgres
  gpstart -R => 管理员连接
  gpstart -v => 显示详细启动信息
  ```

- gpstop 命令详解

  ```shell
  命令     参数   作用
  gpstop -a => 快速停止
  gpstop -d => 指定数据目录（默认值：$MASTER_DATA_DIRECTORY）
  gpstop -m => 维护模式
  gpstop -q => 在安静模式下运行。命令输出不显示在屏幕，但仍然写入日志文件。
  gpstop -r => 停止所有实例，然后重启系统
  gpstop -u => 重新加载配置文件 postgresql.conf 和 pg_hba.conf
  gpstop -v => 显示详细启动信息
  gpstop -M fast          => 快速关闭。正在进行的任何事务都被中断。然后滚回去。
  gpstop -M immediate     => 立即关闭。正在进行的任何事务都被中止。不推荐这种关闭模式，并且在某些情况下可能导致数据库损坏需要手动恢复。
  gpstop -M smart         => 智能关闭。如果存在活动连接，则此命令在警告时失败。这是默认的关机模式。
  gpstop --host hostname  => 停用segments数据节点，不能与-m、-r、-u、-y同时使用
  ```

  

### GP 常用 SQL

- GP 删除数据库

  ```sql
  drop database [database];
  ```

- PSQL 快捷指令

  ```shell
  \l    列举数据库
  \c    选择/切换数据库
  \dt   查看某个库中的所有表
  \d    查看表结构
  \q    退出 psql
  ```

- 建表

  ```sql
  create table test004(id int ,name varchar(128)) distributed randomly;
  ```

- 查询

  ```
  select id,name from testOOl order by id;
  ```

  

### GP 实用 SQL

- GP 查询数据库连接

  ```sql
  select * from pg_stat_activity where DATNAME = '[database]';
  ```

- GP 解除数据库所有连接

  ```sql
  select pg_terminate_backend(procpid) from pg_stat_activity where DATNAME = '[database]';
  ```

- GP 查看 segment 信息

  ```sql
  SELECT * from gp_segment_configuration  order by status,role;
  ```

- 快速导出单表数据

  ```sql
  copy 表名 to ‘/tmp/1.csv’ with ‘csv’;
  ```

- 快速导入单表数据

  ```sql
  copy 表名 from ‘/tmp/1.csv’ with ‘csv’;
  ```

- 将本地文件拷贝到库表中

  ```shell
  psql -h$gphost -U$gpuser -p$gpport -d $gpdatabase -c "\copy $gptable from '$1/$file'"
  ```

- 其他

  ```shell
  # gp 添加分区: 
  alter table 表名 add partition d20190611 values('20190611') ;
  
  # gp 添加字段:
  alter table 表名 add column update_flag varchar(255);
  
  # gp 重命名:
  alter table 表名 rename column old_column_name to new_column_name;  
  
  # gp 删除字段:
  ALTER TABLE 表名 DROP COLUMN rt_gather_price_remark;
  
  # gp 修改字段的类型:
  alter table 表名 alter column rt_gather_price_remark varchar(200) text; 
  
  # gp 删除表:
  drop table if exists 表名;
  
  # gp 删除分区:
  alter table 表名 DROP partition d20190729;
  
  # gp 清空分区数据:
  alter table 表名 truncate partition d20191010;
  
  # gp 删除进程:
  select pg_cancel_backend(407367);
  
  # gp 查看表结构:
  SELECT a.attnum,a.attname AS field,t.typname AS type,a.attlen AS length,a 
  .atttypmod AS lengthvar,a.attnotnull AS notnull from pg_class c,pg_attribute a,pg_type t where 
  c.relname='表名' and a.attnum>0 and a.attrelid=c.oid and a.atttypid=t.oid order by attnum;
  
  #创建用户
  create role dbdream password 'dbdream' createdb login;
  #查看用户
  select * from pg_user;
  #查看权限
  select * from information_schema.table_privileges;
  #修改文件允许远程登陆
  pg_hba.conf
  host    all     dbdream         10.9.15.20/32   md5
  gpstop –u
  #通过pg_roles字典开查看数据库的用户信息
  select rolname,oid from pg_roles;
  #查看表空间
  SELECT oid,* from pg_tablespace;
  #查看表空间及对应文件目录
  WITH spc AS (SELECT * FROM  gp_tablespace_location(24589))
  SELECT seg.role, spc.gp_segment_id as seg_id, seg.hostname, seg.datadir, tblspc_loc 
  FROM spc, gp_segment_configuration AS seg 
  WHERE spc.gp_segment_id = seg.content ORDER BY seg_id;
  
  #查看表、索引、视图对应的物理文件
  select oid,relfilenode,relname from pg_class where relname = 't2';
  #查看表oid
  SELECT a.oid,
  a.relname AS name,
  b.description AS comment
  FROM pg_class a
  LEFT OUTER JOIN pg_description b ON b.objsubid=0 AND a.oid = b.objoid
  WHERE a.relnamespace = (SELECT oid FROM pg_namespace WHERE nspname=‘public’) --用户表一般存储在public模式下
  AND a.relkind=‘r’
  ORDER BY a.relname;
  #查看当前postgresql有几个数据库
  SELECT datname FROM pg_database
  # 查看所有数据库大小
  select datname,pg_size_pretty(pg_database_size(datname)) from pg_database;
  #查看数据库大小
  select pg_size_pretty(pg_database_size(‘gpdw’));
  #查看表空间大小
  select pg_size_pretty(pg_tablespace_size(‘pg_default’));
  #查看索引大小
  select pg_size_pretty(pg_indexes_size(‘t1’));
  select pg_size_pretty(pg_total_relation_size('gp_test'));
  #查看表大小
  select pg_size_pretty(pg_table_size(‘t2’));
  select pg_size_pretty(pg_relation_size('gp_test'));
  # 查看数据分布情况和磁盘空间
  select gp_segment_id,count(*) from gp_test group by gp_segment_id order by 1;
  select dfhostname, dfspace,dfdevice from gp_toolkit.gp_disk_free order by dfhostname;
  #查看表索引
  select
  relname, indexrelname, idx_scan, idx_tup_read, idx_tup_fetch
  from
  pg_stat_user_indexes
  order by
  idx_scan asc, idx_tup_read asc, idx_tup_fetch asc;
  #查看指定schema 里所有的表大小，按从大到小的顺序排列
  select relname, pg_size_pretty(pg_relation_size(relid)) from pg_stat_user_tables where schemaname=‘public’ order by pg_relation_size(relid) desc;
  #查看表定义
  SELECT a.attnum,
  a.attname AS field,
  t.typname AS type,
  a.attlen AS length,
  a.atttypmod AS lengthvar,
  a.attnotnull AS notnull,
  b.description AS comment
  FROM pg_class c,
  pg_attribute a
  LEFT OUTER JOIN pg_description b ON a.attrelid=b.objoid AND a.attnum = b.objsubid,
  pg_type t
  WHERE c.relname = ‘t2’
  and a.attnum > 0
  and a.attrelid = c.oid
  and a.atttypid = t.oid
  ORDER BY a.attnum;
  ```

### GP 高级操作

```sql
   1、表膨胀相关查询
  -- 该视图显示了那些膨胀的（在磁盘上实际的页数超过了根据表统计信息得到预期的页数）正规的堆存储的表。
  select * from gp_toolkit.gp_bloat_diag;

  -- 所有对象的膨胀明细
  select * from gp_toolkit.gp_bloat_expected_pages;


  2、表倾斜的相关信息
  -- 该视图通过计算存储在每个Segment上的数据的变异系数（CV）来显示数据分布倾斜。
  select * from gp_toolkit.gp_skew_coefficients;

  -- 该视图通过计算在表扫描过程中系统空闲的百分比来显示数据分布倾斜，这是一种数据处理倾斜的指示器。
  select * from gp_toolkit.gp_skew_idle_fractions;


  3、锁查询相关的信息
  -- 该视图显示了当前所有表上持有锁，以及查询关联的锁的相关联的会话信息。
  select * from gp_toolkit.gp_locks_on_relation;

  -- 该视图显示当前被一个资源队列持有的所有的锁，以及查询关联的锁的相关联的会话信息。
  select * from gp_toolkit.gp_locks_on_resqueue;


  4、日志查询相关的信息
  -- 该视图使用一个外部表来读取来自整个Greenplum（Master、Segment、镜像）的服务器日志文件并且列出所有的日志项。
  select * from gp_toolkit.gp_log_system;

  -- 该视图用一个外部表来读取在主机上的日志文件同时报告在数据库会话中SQL命令的执行时间
  select * from gp_toolkit.gp_log_command_timings;

  -- 该视图使用一个外部表来读取整个Greenplum系统（主机，段，镜像）的服务器日志文件和列出与当前数据库关联的日志的入口。
  select * from gp_toolkit.gp_log_database;

  -- 该视图使用一个外部表读取来自Master日志文件中日志域的一个子集。
  select * from gp_toolkit.gp_log_master_concise;


  5、资源队列相关查询信息
  -- gp_toolkit.gp_resgroup_config视图允许管理员查看资源组的当前CPU、内存和并发限制
  select * from gp_toolkit.gp_resgroup_config;

  -- gp_toolkit.gp_resgroup_status视图允许管理员查看资源组的状态和活动
  select * from gp_toolkit.gp_resgroup_status;

  -- 该视图允许管理员查看到一个负载管理资源队列的状态和活动。
  select * from gp_toolkit.gp_resqueue_status;

  -- 对于那些有活动负载的资源队列，该视图为每一个通过资源队列提交的活动语句显示一行。
  select * from gp_toolkit.gp_resq_activity;

  -- 对于有活动负载的资源队列，该视图显示了队列活动的总览。
  select * from gp_toolkit.gp_resq_activity_by_queue;

  -- 资源队列的执行优先级
  select * from gp_toolkit.gp_resq_priority_backend;

  -- 该视图为当前运行在Greenplum数据库系统上的所有语句显示资源队列优先级、会话ID以及其他信息
  select * from gp_toolkit.gp_resq_priority_statement;

  -- 该视图显示与角色相关的资源队列。
  select * from gp_toolkit.gp_resq_role;


  6、查看磁盘上(database,schema,table,indexs,view)等的占用大小的相关信息
  -- 外部表在活动Segment主机上运行df（磁盘空闲）并且报告返回的结果
  select * from gp_toolkit.gp_disk_free;

  -- 该视图显示数据库的总大小。
  select * from gp_toolkit.gp_size_of_database;

  -- 该视图显示当前数据库中schema在数据中的大小
  select * from gp_toolkit.gp_size_of_schema_disk;

  -- 该视图显示一个表在磁盘上的大小。
  select * from gp_toolkit.gp_size_of_table_disk;

  -- 该视图查看表的索引
  select * from gp_toolkit.gp_table_indexes;

  -- 该视图显示了一个表上所有索引的总大小。
  select * from gp_toolkit.gp_size_of_all_table_indexes;

  -- 该视图显示分区子表及其索引在磁盘上的大小。
  select * from gp_toolkit.gp_size_of_partition_and_indexes_disk;

  -- 该视图显示表及其索引在磁盘上的大小。
  select * from gp_toolkit.gp_size_of_table_and_indexes_disk;

  -- 该视图显示表及其索引的总大小
  select * from gp_toolkit.gp_size_of_table_and_indexes_licensing;

  -- 该视图显示追加优化（AO）表没有压缩时的大小。
  select * from gp_toolkit.gp_size_of_table_uncompressed;

  7、用户使用的工作空间大小信息
  -- 该视图为当前在Segment上使用磁盘空间作为工作文件的操作符包含一行。
  select * from gp_toolkit.gp_workfile_entries;

  -- GP工作文件管理器使用的磁盘空间
  select * from gp_toolkit.gp_workfile_mgr_used_diskspace;

  -- 每个查询的GP工作文件使用情况
  select * from gp_toolkit.gp_workfile_usage_per_query;

  -- 每个segment在GP工作文件中的使用量
  select * from gp_toolkit.gp_workfile_usage_per_segment;


  8、查看用户创建的信息(数据库,schema,表,索引,函数,视图)等信息
  -- gp 中所有的名字(索引、表、视图、函数)等的名字
  select * from gp_toolkit."__gp_fullname";
  -- gp 中AO表的名字
  select * from gp_toolkit."__gp_is_append_only";
  -- gp 中segment的个数
  select * from gp_toolkit."__gp_number_of_segments";
  -- gp 中用户表的个数
  select * from gp_toolkit."__gp_user_data_tables";
  -- GP用户数据表可读
  select * from gp_toolkit."__gp_user_data_tables_readable";
  -- 用户自己创建的schema信息
  select * from gp_toolkit."__gp_user_namespaces";
  -- 用户自己创建的表信息
  select * from gp_toolkit."__gp_user_tables";


  9、系统中维护的ID信息
  -- gp  本地维护的ID
  select * from gp_toolkit."__gp_localid";

  -- gp master外部的log信息
  select * from gp_toolkit."__gp_log_master_ext";

  -- gp segment外部的log信息
  select * from gp_toolkit."__gp_log_segment_ext";

  -- gp master 的id信息
  select * from gp_toolkit."__gp_masterid";


  10、系统查用的查询信息
  -- 该视图显示那些没有统计信息的表，因此可能需要在表上执行ANALYZE命令。
  select * from gp_toolkit.gp_stats_missing;

  -- 该视图显示系统目录中被标记为down的Segment的信息。
  select * from gp_toolkit.gp_pgdatabase_invalid;

  -- 那些被分类为本地（local）（表示每个Segment从其自己的postgresql.conf文件中获取参数值）的服务器配置参数，应该在所有Segment上做相同的设置。
  select * from gp_toolkit.gp_param_settings_seg_value_diffs;

  -- 该视图显示系统中所有的角色以及指派给它们的成员（如果该角色同时也是一个组角色）。
  select * from gp_toolkit.gp_roles_assigned;

  11、系统中常用查询的函数
   
  select * from gp_toolkit.gp_param_settings();
  select * from gp_toolkit.gp_skew_details(oid);
  select * from gp_toolkit."__gp_aocsseg"(IN  oid);
  select * from gp_toolkit."__gp_aovisimap"(IN  oid);
  select * from gp_toolkit.gp_param_setting(varchar);
  select * from gp_toolkit."__gp_skew_coefficients"();
  select * from gp_toolkit."__gp_workfile_entries_f"();
  select * from gp_toolkit."__gp_skew_idle_fractions"();
  select * from gp_toolkit."__gp_aocsseg_name"(IN  text);
  select * from gp_toolkit."__gp_aovisimap_name"(IN  text);
  select * from gp_toolkit."__gp_aocsseg_history"(IN  oid);
  select * from gp_toolkit."__gp_aovisimap_entry"(IN  oid);
  select * from gp_toolkit."__gp_aovisimap_hidden_typed"(oid);
  select * from gp_toolkit."__gp_param_local_setting"(varchar);
  select * from gp_toolkit."__gp_aovisimap_entry_name"(IN  text);
  select * from gp_toolkit."__gp_aovisimap_hidden_info"(IN  oid);
  select * from gp_toolkit."__gp_workfile_mgr_used_diskspace_f"();
  select * from gp_toolkit."__gp_aovisimap_hidden_info_name"(IN  text);
  select * from gp_toolkit.gp_skew_coefficient(IN targetoid oid, OUT skcoid oid, OUT skccoeff numeric);
  select * from gp_toolkit.gp_skew_idle_fraction(IN targetoid oid, OUT sifoid oid, OUT siffraction numeric);
  select * from gp_toolkit.gp_bloat_diag(IN btdrelpages int4, IN btdexppages numeric, IN aotable bool, OUT bltidx int4, OUT bltdiag text);
  select * from gp_toolkit."__gp_aovisimap_compaction_info"(IN ao_oid oid, OUT content int4, OUT datafile int4, OUT compaction_possible bool, OUT hidden_tupcount int8, OUT total_tupcount int8, OUT percent_hidden numeric);

  --查询某schema下所有函数的属主
  SELECT n.nspname AS "Schema",p.proname AS "Name",pg_catalog.pg_get_userbyid(p.proowner) AS "Owner" 
  FROM pg_catalog.pg_proc p 
  LEFT JOIN pg_catalog.pg_namespace n ON n.oid = p.pronamespace 
  WHERE n.nspname='dwbi02' ORDER BY 2;

  --查询某schema下所有函数的详细信息
  SELECT n.nspname, p.proname, pg_catalog.pg_get_userbyid(p.proowner) AS "Owner",
   CASE WHEN proallargtypes IS NOT NULL THEN
pg_catalog.array_to_string(ARRAY(
  SELECT
    CASE
      WHEN p.proargmodes[s.i] = 'i' THEN ''
      WHEN p.proargmodes[s.i] = 'o' THEN 'OUT '
      WHEN p.proargmodes[s.i] = 'b' THEN 'INOUT '
      WHEN p.proargmodes[s.i] = 'v' THEN 'VARIADIC '
    END ||
    CASE
      WHEN COALESCE(p.proargnames[s.i], '') = '' THEN ''
      ELSE p.proargnames[s.i] || ' ' 
    END ||
    pg_catalog.format_type(p.proallargtypes[s.i], NULL)
  FROM
    pg_catalog.generate_series(1, pg_catalog.array_upper(p.proallargtypes, 1)) AS s(i)
), ', ')
ELSE
pg_catalog.array_to_string(ARRAY(
SELECT
CASE
WHEN COALESCE(p.proargnames[s.i+1], ‘’) = ‘’ THEN ‘’
ELSE p.proargnames[s.i+1] || ’ ’
END ||
pg_catalog.format_type(p.proargtypes[s.i], NULL)
FROM
pg_catalog.generate_series(0, pg_catalog.array_upper(p.proargtypes, 1)) AS s(i)
), ', ')
END AS “data_types”
FROM pg_catalog.pg_proc p
LEFT JOIN pg_catalog.pg_namespace n ON n.oid = p.pronamespace
WHERE 1=1
AND n.nspname <> ‘pg_catalog’
AND n.nspname <> ‘information_schema’
AND n.nspname = ‘dmc’ order by 2,3;

  --某schema下，表数量统计（正则过滤分区表子分区）
  SELECT count(*) FROM pg_tables WHERE tablename !~ '.*_1+_prt_.*$'AND schemaname = 'ctl';

  --查询一个库下各schema占用的空间
  SELECT pg_size_pretty(cast( sum(pg_total_relation_size( schemaname  || '.' || tablename)) AS bigint)), schemaname
  FROM pg_tables t INNER JOIN pg_namespace d ON t.schemaname=d.nspname  GROUP BY schemaname;

  SELECT pg_size_pretty(cast( sum(pg_total_relation_size( schemaname  || '.' || tablename)) AS bigint)), schemaname
  FROM pg_tables t INNER JOIN pg_namespace d ON t.schemaname=d.nspname  WHERE schemaname in ('mad','mad_ods','dwbi_ods','coupon_ods') GROUP BY schemaname;

  SELECT pg_size_pretty(cast( sum(pg_total_relation_size( schemaname  || '.' || tablename)) AS bigint)), schemaname
  FROM pg_tables t INNER JOIN pg_namespace d ON t.schemaname=d.nspname  WHERE schemaname = 'mad' GROUP BY schemaname;

  --统计某张表的分区数量
  SELECT p.schemaname, p.tablename, count(*) prt_count
  FROM pg_partitions p
  WHERE p.tablename='表名（不含schema名称）' 
  GROUP BY 1,2
  ORDER BY 2;

  --统计AO分区表数量（不含子表）
  SELECT count(*) FROM pg_class c 
  LEFT JOIN pg_namespace n 
  ON c.relnamespace = n.oid 
  WHERE relstorage IN ('c','a') 
  AND c.oid NOT IN(SELECT parchildrelid FROM pg_partition_rule);

  --统计AO表分区数量
  SELECT count(*) FROM pg_class c 
  LEFT JOIN pg_namespace n 
  ON c.relnamespace = n.oid 
  WHERE relstorage IN ('c','a') 
  AND c.oid IN(SELECT parchildrelid FROM pg_partition_rule);

  --查询分区数量大于100的AO表
  SELECT * FROM
  (
  SELECT n.nspname, c.relname, count(*) part_count FROM pg_class c 
  LEFT JOIN pg_namespace n 
  ON c.relnamespace = n.oid 
  INNER JOIN pg_partitions p
  ON p.tablename=c.relname
  AND p.schemaname=n.nspname 
  WHERE c.relstorage IN ('c','a') 
  AND c.oid NOT IN(SELECT parchildrelid FROM pg_partition_rule) 
  GROUP BY n.nspname, c.relname 
  ) a
  WHERE part_count >= 100
  ORDER BY 3 DESC;

  --统计分区表大小（不含索引）
  SELECT p.schemaname,p.tablename,sum(sotdsize) 
  FROM pg_partitions p LEFT JOIN  (SELECT autnspname, autrelname, sotdsize  FROM gp_toolkit.__gp_user_tables a, gp_toolkit.gp_size_of_table_disk b  
  WHERE a.autoid = b.sotdoid) b on p.partitionschemaname = b.autnspname AND p.partitiontablename = b.autrelname 
  GROUP BY 1,2 
  ORDER BY 1,2;

  --统计某分区表大小（含索引）
  SELECT p.schemaname, p.tablename, round(sum(pg_total_relation_size(p.schemaname || '.' || p.partitiontablename))/1024/1024) "MB"
  FROM pg_partitions p
  WHERE p.tablename='employee_daily' 
  GROUP BY 1,2
  ORDER BY 2;

  --统计某schema下各分区表大小（含索引）
  SELECT p.schemaname, p.tablename, round(sum(pg_total_relation_size(p.schemaname || '.' || p.partitiontablename))/1024/1024) "MB"
  FROM pg_partitions p
  WHERE p.schemaname ='mad' 
  GROUP BY 1,2 
 ORDER BY 2;

  --查询所有非分区表名
  SELECT t.schemaname, t.tablename 
  FROM pg_tables t, pg_partitions p 
  WHERE t.tablename <> p.tablename AND t.tablename <> p.partitiontablename AND t.schemaname='mad' 
  GROUP BY 1,2
  ORDER BY 2;

  --查询所有表名（不含分区表子表）
  SELECT n.nspname,c.relname 
  FROM pg_class c, pg_namespace n 
  WHERE c.relnamespace = n.oid AND n.nspname NOT LIKE 'pg_%' AND n.nspname NOT LIKE 'gp_%' 
  AND n.nspname <> 'information_schema' AND relkind IN('r') AND relstorage <> 'x' 
  AND c.oid NOT IN(SELECT parchildrelid FROM pg_partition_rule);

  --Greenplum查询争用（表锁）
  SELECT l.locktype, l.database, c.relname, l.relation, l.transactionid, l.pid, l.mode, l.granted,a.current_query 
  FROM pg_locks l, pg_class c, pg_stat_activity a WHERE l.relation=c.oid AND l.pid=a.procpid ORDER BY c.relname;

  --用函数杀SQL
  -------------------------------------
  --一般查询SQL
  SELECT pg_cancel_backend(procpid);

  --其他SQL
  SELECT pg_terminate_backend(procpid);
  -------------------------------------

  --查询某张表的数据分布均衡性
  SELECT gp_segment_id,count(*) FROM schemaname.tablename GROUP BY gp_segment_id ORDER BY count(*) DESC;

  --查询数据分布均匀性
  SELECT * FROM gp_toolkit.gp_skew_idle_fractions; --siffraction字段值大于0.1的需要重新选择分布键

  --保证全库统计信息是最新的情况下，查询表膨胀情况
  SELECT * FROM gp_toolkit.gp_bloat_diag; --bdidiag字段内容显示膨胀程度

  --查询用户情况
  SELECT rolname,rolsuper,rolinherit,rolcreaterole,rolcanlogin,rolconnlimit,rolresqueue,oid FROM pg_roles ORDER BY 8;

  --查询所有外部表
  SELECT n.nspname,c.relname 
  FROM pg_class c, pg_namespace n 
  WHERE c.relnamespace = n.oid AND n.nspname NOT LIKE 'pg_%' AND n.nspname NOT LIKE 'gp_%' 
  AND n.nspname <> 'information_schema' AND relkind IN('r') AND relstorage = 'x' 
  AND c.oid NOT IN(SELECT parchildrelid FROM pg_partition_rule);

  --重组表 消除膨胀
  \d+ dwbi01_ods.boh__bohrscsales
  ALTER TABLE dwbi01_ods.boh__bohrscsales SET WITH (REORGANIZE=TRUE) DISTRIBUTED RANDOMLY;
  ALTER TABLE dwbi01_ods.boh__bohrscsales SET WITH (REORGANIZE=TRUE) DISTRIBUTED BY (transaction_code);
  ANALYZE dwbi01_ods.boh__bohrscsales;

  --检测字段是否适合用作分布键
  select COUNT(bizdate), gp_segment_id from dwbi01_ods.boh__bohrscsales group by 2;

  --查询角色配置情况（使用资源组时）
  SELECT rolname, rolsuper, rolinherit, rolcreaterole, rolcanlogin, rsgname FROM pg_roles, pg_resgroup WHERE pg_roles.rolresgroup=pg_resgroup.oid;


  --查询膨胀率大于20的AO表
  SELECT nspname, relname, AVG(percent_hidden)
  FROM
  (
    SELECT n.nspname, c.relname, (gp_toolkit.__gp_aovisimap_compaction_info(c.oid)).percent_hidden percent_hidden
    FROM pg_class c, pg_namespace n
    WHERE c.relnamespace = n.oid
    AND relstorage IN (‘c’,‘a’)
    AND c.oid NOT IN(SELECT parchildrelid FROM pg_partition_rule)
    ) t
    WHERE t.percent_hidden > 20
    GROUP BY nspname, relname
    ORDER BY 3 DESC;

       --查询溢出文件大小（详细）
        select * from gp_toolkit.gp_workfile_entries;

  --查询溢出文件大小
  select * from gp_toolkit.gp_workfile_usage_per_query;

  --查询每个Segment上的溢出文件大小
  select * from gp_toolkit.gp_workfile_usage_per_segment;

  --查询表的分布键
  SELECT att.nspname,att.relname,string_agg (a.attname, ',') attby 
  FROM 
  (
   SELECT c.oid,n.nspname,c.relname,regexp_split_to_table (array_to_string (d.attrnums, ','),',')::int as attnu
   FROM gp_distribution_policy d 
   LEFT JOIN pg_class c ON c.oid = d.localoid 
   LEFT JOIN pg_namespace n ON n.oid = c.relnamespace  
   WHERE c.oid = 'ods.oracletest'::regclass
  ) att
  LEFT JOIN pg_attribute a ON a.attrelid = att.oid
  WHERE att.attnu = a.attnum
  GROUP BY 1,2;

  --查询前20张大表
  SELECT tabs.nspname AS schema_name,
   COALESCE(parts.tablename, tabs.relname) AS table_name,
   ROUND(SUM(sotaidtablesize) / 1024 / 1024, 3) AS table_MB,
   ROUND(SUM(sotaidtablesize) / 1024 / 1024 / 1024, 3) AS table_GB,
   ROUND(SUM(sotaididxsize) / 1024 / 1024 / 1024, 3) AS index_GB,
   ROUND(SUM(sotaididxsize) / 1024 / 1024, 3) AS index_MB
  FROM gp_toolkit.gp_size_of_table_and_indexes_disk sotd,
(
SELECT c.oid, c.relname, n.nspname
FROM pg_class c, pg_namespace n
WHERE n.oid = c.relnamespace
AND c.relname NOT LIKE '%_err'
) tabs
  LEFT JOIN pg_partitions parts 
  ON tabs.nspname = parts.schemaname
  AND tabs.relname = parts.partitiontablename
  WHERE sotd .sotaidoid = tabs.oid
  GROUP BY tabs.nspname, COALESCE(parts.tablename, tabs.relname)
  ORDER BY 4 DESC
  LIMIT 20;

  --资源队列属性视图
  SELECT * FROM pg_catalog.pg_resqueue_attributes;



  --查询Filespace和Locating
  SELECT a.dbid,a.content,a.role,a.port,a.hostname,b.fsname,c.fselocation FROM gp_segment_configuration a,pg_filespace b,pg_filespace_entry c WHERE a.dbid=c.fsedbid AND b.oid=c.fsefsoid ORDER BY 2,6;

  --日期计算：
  riqi::date + interval '1 month' --"riqi"增加1月
  riqi::date - interval '1 day'   --"riqi"减去1天

  1.最后分析或真空或创建表或等...
     Select * from pg_stat_operations where schemaname='SCHEMA NAME '
   and actionname in ('ANALYZE','VACUUM') order by statime; 

  2.长时间查询空闲：
    Select * from pg_stat_activity order by query_start,backend_start;



  gpdb=# Select * from pg_stat_activity order by query_start,backend_start;
   datid |  datname  | procpid | sess_id | usesysid | usename |
current_query                            | waiting |          query_start
| backend_start | client_addr | client_port | application_name
| xact_start | waiting_reason | rsgid | rsgname | rsgqueueduratio
n
-------±----------±--------±--------±---------±--------±----------------------
---------------------------------------------±--------±---------------------------
—±------------------------------±--------------±------------±-----------------
±------------------------------±---------------±------±--------±---------------
| gpperfmon | 31604 | 2822 | 16558 | gpmon |
| f | 2019-03-20 21:19:40.079557-
| 2019-03-19 05:45:25.082823-04 | 192.168.0.221 | 62596 | gpcc
| | | 0 | unknown |
| gpperfmon | 7652 | 207 | 16558 | gpmon |
| f | 2019-03-20 21:19:45.883945-
| 2019-03-19 01:52:55.080215-04 | 192.168.0.221 | 45824 | gpcc
| | | 0 | unknown |
| gpdb | 6961 | 27890 | 10 | gpadmin | Select * from pg_stat_
activity order by query_start,backend_start; | f | 2019-03-20 21:19:47.667488-
| 2019-03-20 21:19:20.171974-04 | | -1 | psql
| 2019-03-20 21:19:47.667488-04 | | 0 | unknown |
(3 rows)

3.如何在数据库中找到最大的表？
     SELECT relname, relpages FROM pg_class ORDER BY relpages DESC;

  gpdb=#  SELECT relname, relpages FROM pg_class ORDER BY relpages DESC;
                        relname                             | relpages
----------------------------------------------------------------±---------
test_1 | 1672
test_index_1 | 1672
test99 | 1000
gp_disk_free | 1000
__gp_log_segment_ext | 1000
t1 | 1000
__gp_localid | 1000
__gp_masterid | 1000
__gp_log_master_ext | 1000
test_index_1_idx | 56
pg_proc | 20
pg_rewrite | 19
pg_attribute | 14
pg_depend | 14
pg_depend_reference_index | 13
pg_depend_depender_index | 13
pg_proc_proname_args_nsp_index | 9
test66 | 8
pg_statistic | 6
gp_persistent_relation_node | 6
pg_description | 6
pg_attribute_relid_attnam_index | 6
pg_attribute_relid_attnum_index | 5
pg_description_o_c_o_index | 5
test1 | 5
pg_proc_oid_index | 5
test2 | 4
test110 | 4
pg_operator | 4
gpcrondump_history | 3
pg_type | 3

 4.数据库中的前5个最大表
     SELECT relname, relpages FROM pg_class ORDER BY relpages DESC limit 5; 
  gpdb=# SELECT relname, relpages FROM pg_class ORDER BY relpages DESC limit 5;
   relname       | relpages
---------------------±---------
test_index_1 | 1672
test_1 | 1672
__gp_localid | 1000
__gp_masterid | 1000
__gp_log_master_ext | 1000
(5 rows)
   relname– 关系/表的名称. 
   relpages - 关系页面（页数，默认情况下页面为8kb）
   pg_class– 系统表，维护关系的细节 
   limit 5 – 限制输出只显示5行。
   
5.如何计算磁盘中的数据库大小？
    SELECT pg_database_size('Database Name' );
    SELECT pg_size_pretty(pg_database_size( 'Database Name' ));
gpdb=# SELECT pg_database_size(‘gpdb’ );
pg_database_size
(1 row)
gpdb=# SELECT pg_database_size(‘postgres’ );
pg_database_size
(1 row)
gpdb=# SELECT pg_size_pretty(pg_database_size( ‘gpdb’ ));
pg_size_pretty
MB
(1 row)
gpdb=# SELECT pg_size_pretty(pg_database_size( ‘postgres’ ));
pg_size_pretty
MB
(1 row)

  6.如何计算磁盘中的表大小？
    SELECT pg_size_pretty(pg_total_relation_size('public.test1'));
gpdb=# SELECT pg_size_pretty(pg_total_relation_size(‘public.test1’));
pg_size_pretty
kB
(1 row)

  8.如何生成一系列数字并将其插入表格中？
    INSERT INTO test2  (id) VALUES ( generate_series(1,1000));

14.显示已关闭的segments。
  Select * from gp_segment_configuration where status='d';
15.查找当前用户：
  SELECT SESSION_USER, CURRENT_USER;
16.检查活动会话（工作负载）：
 SELECT * FROM pg_stat_activity;
17.正在队列中等待的查询
 SELECT * FROM gp_tookit.gp_resqueue_status; 
18.查看数据库列表
  SELECT datname from pg_database;
  
12.时区设置：
  gpdb=# BEGIN; 
  gpdb=# SELECT NOW(); 
  gpdb=# SET timezone TO '-8';                               
  gpdb=# SELECT NOW();

```




## Zookeeper

Zookeeper 客户端登录命令：

```shell
./zkCli.sh -server ip:port
```

Zookeeper 基本命令：

```
# 查看目录下所有文件夹
ls /xxx/xxx
# 创建 znode
create /xxx/xxx
```



## Kafka

修改 Topic 分区数

```shell
kafka-topics.sh --alter --topic one14 --zookeeper rhino74:2181 --partitions 3
```

查看消费组消费情况

```shell
kafka-consumer-groups.sh --bootstrap-server rhino221:9092 --group zww_topic_group510 --describe
```





## Redis

登录命令

```
./redis-cli -h ip -p port -a password
```





## Python

### 清华pypi镜像部分包的直接地址

```
# pip
https://pypi.tuna.tsinghua.edu.cn/simple/pip/
# setuptools
https://pypi.tuna.tsinghua.edu.cn/simple/setuptools/
```



### namedtuple 用法

> **namedtuple是继承自tuple的子类。namedtuple创建一个和tuple类似的对象，而且对象拥有可访问的属性。**
>
> 因为元组的局限性：不能为元组内部的数据进行命名，所以往往我们并不知道一个元组所要表达的意义，
> 所以在这里引入了 collections.namedtuple 这个工厂函数，来构造一个带字段名的元组。
> 具名元组的实例和普通元组消耗的内存一样多，因为字段名都被存在对应的类里面。这个类跟普通的对象
> 实例比起来也要小一些，因为 Python 不会用 __dict__ 来存放这些实例的属性。

实现方法

```python
from collections import namedtuple
```

源代码

![img](HandBook/70.png)

用法示例

```python
def namedtuple(typename, field_names, *, verbose=False, rename=False, module=None):
#    - typename: 元组名称
#    - field_names : 元组中元素的名称
#    - rename: 如果元素名称中包含python关键字， 必须设置rename=True
```

```python
from collections import  namedtuple

User = namedtuple('User', ['name', 'age', 'gender'])
u = User('villa', 33, 'male')
print(u.name)
print(u.age)
print(u.gender)
print(u)
```





### Pycharm 快捷键

```
批量注释    Ctrl+/
```



### Pychram 设置环境与导出环境

Pycharm 每个项目都需要设置环境，即 Porject Interpreter，这个环境有几种设置方法
1、将 Interpreter 设置为本机安装路径，如下图所示

![899386514cf7b675689a0f36d033e91f](HandBook/899386514cf7b675689a0f36d033e91f.png)

优势：所有项目依赖包只需要装一遍即可
劣势：依赖环境的第三方库会有非常多，在某个项目的第三方库并不需要很多时，如果此时用这个 interpreter 生成 requirement.txt 就会有很多无关紧要的库，不利于打包部署

2、创建 Virtualenv Environment
此方法给项目分配一个单独的 Porject Interpreter
![222](HandBook/222.png)

![333](HandBook/333.png)

![444](HandBook/444.png)

优势：单个项目有一个独立的环境，相关依赖包会与项目紧密相关，不会存在多余无用的依赖包
劣势：不同项目之间需要重新下载依赖包

环境依赖包导出
基本操作
1、进入对应环境，使用 `python.exe -m pip freeze > requirements.txt` 生成该环境的 requirements 文件（Windows）
2、联网环境下：使用这个 requirements.txt 文件，通过命令 `pip install -r requirements.txt`直接安装

离线环境下的操作

1、进入对应环境，使用 `python.exe -m pip freeze > requirements.txt` 生成该环境的 requirements 文件（Windows）

2、在联网环境下，使用 `pip download -r requestments.txt -d ./pip_packages ` 将所有依赖包存储在 pip_packages 目录下

3、将包和环境文件，拷贝到离线环境下，使用 `pip install --no-index --find-links=./pip_packages -r requestments.txt ` 离线安装所有包

​	（--find-links指定的是包文件的存放地址，-r指定的是txt文件的位置）

如果 Pycharm 使用的是系统 Python 环境，则到系统 Python 环境下安装，如果 Pycharm 使用的是虚拟环境，则到项目目录下的 venv/Scripts 下，有一个 python.exe，利用 venv/Scripts 下的 python.exe 生成 requirement.txt





### Python 文件的读写模式

```
r :读取模式（默认值）
w:写入模式
x:独占写入模式
a：附加模式
b:二进制模式
t：文本模式（默认值，与其他模式结合使用）
+：读写模式（与其他模式结合使用）
r+： 打开文件进行读写，会覆盖文件原内容，重新写入；若文件不存在，则会报错
w+：打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件
a+：打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。
```



### pip 命令

```
pip [download/install]
 -i: 指定下载源
 -d: 指定下载目录
# 导出当前 Python 环境到一个文件
pip freeze > requirements.txt
# 下载 Python 环境的所有包到一个目录
pip download -d C:\Users\Administrator\Desktop\我的脚本\ZTCHS_Checker\pack -r requirements.txt# 导出当前 Python 环境到一个文件
pip freeze > requirements.txt
# 下载 Python 环境的所有包到一个目录
pip download -d C:\Users\Administrator\Desktop\我的脚本\ZTCHS_Checker\pack -r requirements.txt
# 指定清华镜像站，下载包到一个指定目录
pip download -d C:\Users\Administrator\Desktop\我的脚本\ZTCHS_Checker\pack -i https://pypi.tuna.tsinghua.edu.cn/simple setuptools
# 使用清华镜像站更新一个包
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
```



### datetime 模块

```python
# 字符串转 datetime 对象
datetime.datetime.strptime(data_str, "%Y-%m-%d %H:%M:%S")
# datetime 对象转字符串
datetime.xxx.xxx.strftime("%Y-%m-%d %H:%M:%S")
# 获取当前时间时间戳
datetime.datetime.now().timestamp()
# 如果不需要带毫秒
int(datetime.datetime.now().timestamp())
```



### @classmethon & @staticmethod

> 结论：
>
> 我们要写一个只在类中运行而不在实例中运行的方法时，可以使用 @classmethon 装饰器来创建方法
>
> 我们要写一个跟类有关系，但在运行时又不需要实例和类参与的方法时，可以使用 @staticmethod 装饰器来创建方法，staticmethod 方法可以在不创建实例的情况下直接调用

下面一个示例来演示这两个装饰器的使用方法和区别：

```python
class Kls(object):
    def __init__(self, data):
    	self.data = data
    def printd(self):
    	print(self.data)
    @staticmethod
    def smethod(*arg):
    	print('Static:', arg)
    @classmethod
    def cmethod(*arg):
    	print('Class:', arg)

 
>>> ik = Kls(23)
>>> ik.printd()
23
>>> ik.smethod()
Static: ()
>>> ik.cmethod()
Class: (<class '__main__.Kls'>,)
>>> Kls.printd()
TypeError: unbound method printd() must be called with Kls instance as first argument (got nothing instead)
>>> Kls.smethod()
Static: ()
>>> Kls.cmethod()
Class: (<class '__main__.Kls'>,)
```

![img](HandBook/format,png.png)





## Spark

- RDD 分区数（在指定分区的情况下：partitionBy() 函数设定的分区值），至少应该和集群中的总核心数一样。
- 不涉及修改键记录的大部分 Spark 都能利用到已有的分区信息（比如我们手动指定的分区），但可能修改键记录的操作则可能导致子RDD 失去父 RDD 的分区信息，从而无法利用已有分区信息。
- 所有引入了将数据根据键跨节点进行混洗的操作，都能从数据分区中获益：cogroup()、groupWith()、join()、leftOuterJoin()、rightOuterJoin()、groupByKey()、reduceByKey()、combineByKey()、lookup()







## Vim

撤销修改：
```
u / :u：撤销上一次更改
U / :U：撤销单行内的所有修改
:undo：用来撤销删除、粘贴、搜索、替换等命令，与u命令有较多重合
```
重做修改：`Ctrl+R` 或 `:redo`





## SQL

### MySQL 添加字段描述

- 创建表时添加，使用关键词 COMMENT 在需要添加的字段定义后面，下方为例

  ```sql
  CREATE TABLE userinfo(
  id INT COMMENT '编号',
  uname VARCHAR(40) COMMENT '用户名',
  address VARCHAR(120) COMMENT '家庭住址',
  hobby VARCHAR(200) COMMENT '爱好'
  )COMMENT = '用户信息表';
  ```

- 修改表以添加

  ```sql
  ALTER TABLE userinfo COMMENT '用户信息资料表';
  ALTER TABLE userinfo MODIFY COLUMN uname VARCHAR(40) COMMENT '姓名';
  ```

- 查看表描述的方法

  ```sql
  # 在生成的 SQL 语句中查看
  SHOW CREATE TABLE userinfo;
  # 在元数据表里查看
  USE information_schema;
  SELECT * FROM TABLES WHERE TABLE_SCHEMA='shoppingcart' AND TABLE_NAME='userinfo';
  ```

- 查看字段描述的方法

  ```sql
  # 在生成的 SQL 语句中查看
  SHOW FULL COLUMNS FROM userinfo;
  # 在元数据的表里面看
  SELECT * FROM COLUMNS WHERE TABLE_SCHEMA='shoppingcart' AND TABLE_NAME='userinfo';
  ```







##  Git

### 常用命令

```shell
# 从版本库中删除某个文件
|-- git rm xxx
|-- git commit -m " remove xxx"
```



 ### .gitignore 文件

- 配置示例

  ```
  # 忽略 .a 文件
  *.a
  
  # 但否定忽略 lib.a, 尽管已经在前面忽略了 .a 文件
  !lib.a
  
  # 仅在当前目录下忽略 TODO 文件， 但不包括子目录下的 subdir/TODO
  /TODO
  
  # 忽略 build/ 文件夹下的所有文件
  build/
  
  # 忽略 doc/notes.txt, 不包括 doc/server/arch.txt
  doc/*.txt
  
  # 忽略所有的 .pdf 文件 在 doc/ directory 下的
  doc/**/*.pdf
  ```

- 配置模板

  ```
  Github 上为开发者提供了各种环境以及各种编程语言的 gitignore 文件配置模板：https://github.com/github/gitignore
  python项目的gitgnore文件 url:https://github.com/github/gitignore/blob/master/Python.gitignore
  ```

- 全局`.gitignore`设置方法( 这个文件将作用于所有 git 项目，并且作用于项目实例中的所有被跟踪的目录)

  ```
  git config --global core.excludesfile ~/.gitignore
  ```

- 局部`.gitignore`直接在项目目录中创建文件即可







## Nginx

### 基于 Nginx 的视频服务器搭建

> Nginx-1.20.2 + nginx-vod-module-1.28
>
> nginx-vod-module 是基于 Nginx 的视频流处理的第三方模块，详细的功能和配置可以上网查询，本文仅讲解最基本的网络点播服务的搭建



- 环境准备

  ```shell
  yum install -y pcre pcre-devel gcc-c++
  ```

- 上传安装包

  ```
  nginx-1.20.2.tar.gz
  nginx-vod-module-1.28.tar.gz
  ```

- 解压这些安装包

- 进入 nginx-1.20.2 目录，配置安装参数

  ```shell
  ./configure --prefix=/home/videoSrv/nginx --with-http_stub_status_module --with-http_ssl_module --with-cc-opt='-O0 -gstabs+3' --with-debug --add-module=/home/videoSrv/nginx-vod-module
  ```

- 编译安装

  ```shell
  make
  make install
  ```

- 配置 Nginx，添加如下内容（基础nginx配置不赘述，正常配置即可）

  ```nginx
  location /vod {
      vod hls;
      vod_mode local;
  
      vod_align_segments_to_key_frames on;
      vod_manifest_segment_durations_mode accurate;
  
      add_header Access-Control-Allow-Headers '*';
      add_header Access-Control-Expose-Headers 'Server,range,Content-Length,Content-Range';
      add_header Access-Control-Allow-Methods 'GET, HEAD, OPTIONS';
      add_header Access-Control-Allow-Origin '*';
  
      alias /home/videoSrv/media;
  }
  ```

- 新建视频存放目录：/home/videoSrv/media

- 存放视频

- 下载 VLC 本地播放器或 H5 播放器，此处以 VLC 播放器为例

- 视频访问链接

  ```
  http://10.0.0.1:8083/vod/example.mp4/index.m3u8
  
  - 10.0.0.1:8083：Nginx 服务的地址和端口
  - example.mp4：为视频名称
  - index.m3u8：为链接默认配置，不需要改动
  ```

- 打开 VLC 播放器

- 配置网络串流

  ![image-20220421114523467](HandBook/image-20220421114523467.png)

  ![image-20220421114548061](HandBook/image-20220421114548061.png)

- 此时，即可正常播放视频



### Nginx 实现文件下载

先看配置文件

```

server {
     listen    80;
     server_name  localhost;
 
     location ^~ /download/ {
        alias /home/webhtml/;
 
        if ($request_filename ~* ^.*?\.(html|doc|pdf|zip|docx|txt)$) {
            add_header Content-Disposition attachment;
            add_header Content-Type application/octet-stream;
        }
            sendfile on;   # 开启高效文件传输模式
            autoindex on;  # 开启目录文件列表
            autoindex_exact_size on;  # 显示出文件的确切大小，单位是bytes
            autoindex_localtime on;  # 显示的文件时间为文件的服务器时间
            charset utf-8,gbk;  # 避免中文乱码
      }
}
```

关键点解析：

```
server_name  localhost;  代表服务名，在这里直接访问 127.0.0.1就可以。如果是别的机器访问这台机器。则直接替换成这台机器的地址即可。

 location ^~ /download/ 带边的是拦截包含 download的请求

 alias /home/webhtml/;  是来配置下载文件的路径，也就是提供下载的文件要在这个路径下。

  if ($request_filename ~* ^.*?\.(html|doc|pdf|zip|docx|txt)$) {
            add_header Content-Disposition attachment;
            add_header Content-Type application/octet-stream;
        }
```

使用方法

首先把一个待下载的文件放在：/home/webhtml/ 目录下

然后就可以用这个url来下载文件了(本机访问)： 127.0.0.1/download/hello.txt

一般都是对外提供下载的（假设我这台部署nginx的机器是：10.2.5.10）：10.2.5.10/download/hello.tx  

我们可以知道的是，带有download的路径被拦截了，加上我们配的文件路径，最后去取文件：

/home/webhtml/hello.txt
