# ps 命令详解

> Linux ps （英文全拼：process status）命令用于显示当前进程的状态，类似于 windows 的任务管理器。

## 基本内容

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

## ps -aux

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



## ps -ef

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



## 实例

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

