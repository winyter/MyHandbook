# Linux 环境变量详解

## Linux 环境变量文件

- 系统环境变量

  **/etc/profile**：这个文件预设了几个重要的变量，例如PATH, USER, LOGNAME, MAIL, INPUTRC, HOSTNAME, HISTSIZE, umask等等

  **/etc/bashrc**：这个文件主要预设umask以及PS1。这个PS1就是我们在敲命令时，前面那串字符了，例如 [root@localhost ~]#,当bash shell被打开时,该文件被读取

- 用户环境变量

  **.bash_profile**：定义了用户的个人化路径与环境变量的文件名称。每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次。（在这个文件中有执行.bashrc的脚本）

  **.bashrc**：该文件包含专用于你的shell的bash信息,当登录时以及每次打开新的shell时,该该文件被读取。例如你可以将用户自定义的alias或者自定义变量写到这个文件中。

  **.bash_history**：记录命令历史用的

  **.bash_logout**：当退出shell时，会执行该文件。可以把一些清理的工作放到这个文件中



## 实操：以添加 PYTHONPATH 为例

```shell
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

