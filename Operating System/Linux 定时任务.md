# Linux 定时任务

## 原理与 crond 进程

在LINUX中，周期执行的任务一般由 cron 这个守护进程来处理 `[ps -ef|grep cron]`。cron 读取一个或多个配置文件，这些配置文件中包含了命令行及其调用时间。

cron的配置文件称为“crontab”，是“cron table”的简写。

1. cron在3个地方查找配置文件：

   1. /var/spool/cron/ 这个目录下存放的是每个用户包括root的crontab任务，每个任务以创建者的名字命名，比如tom建的crontab任务对应的文件就是/var/spool/cron/tom。
      一般一个用户最多只有一个crontab文件。
   2. /etc/crontab 这个文件负责安排由系统管理员制定的维护系统以及其他任务的crontab。
   3. /etc/cron.d/ 这个目录用来存放任何要执行的crontab文件或脚本。

2. 权限
   crontab权限问题到/var/adm/cron/下一看，文件cron.allow和cron.deny是否存在
   用法如下： 
   1、如果两个文件都不存在，则只有root用户才能使用crontab命令。 
   2、如果cron.allow存在但cron.deny不存在，则只有列在cron.allow文件里的用户才能使用crontab命令，如果root用户也不在里面，则root用户也不能使用crontab。 
   3、如果cron.allow不存在, cron.deny存在，则只有列在cron.deny文件里面的用户不能使用crontab命令，其它用户都能使用。 
   4、如果两个文件都存在，则列在cron.allow文件中而且没有列在cron.deny中的用户可以使用crontab，如果两个文件中都有同一个用户，
   以cron.allow文件里面是否有该用户为准，如果cron.allow中有该用户，则可以使用crontab命令。

3. cron服务
   cron是一个**linux**下 的定时执行工具，可以在无需人工干预的情况下运行作业。

   ```shell
   /sbin/service crond start  //启动服务
   /sbin/service crond stop   //关闭服务
   /sbin/service crond restart //重启服务
   /sbin/service crond reload  //重新载入配置
   /sbin/service crond status  //查看服务状态 
   ```

4. 每次编辑完某个用户的cron设置后，cron自动在/var/spool/cron下生成一个与此用户同名的文件，此用户的cron信息都记录在这　个文件中，这个文件是不可以直接编辑的，只可以用crontab -e　来编辑。cron启动后每过一份钟读一次这个文件，检查是否要执行里面的命令。因此此文件修改后不需要重新启动cron服务。

5. 查看crontab 执行的日志，可以在/var/log/cron* 查看，或者 0 3 * * * /root/mysqlbak.sh >/var/log/mysqlbak.log 2>&1 把日志定向出来查看。



## crontab 命令

```shell
# 查看当前用户所有定时任务
crontab -l
# 编辑当前用户定时任务列表
crontab -e
# 删除当前用户下的定时任务
crontab -r
# 指定用户
crontab -u   // crontab -u user -r/-e/-l
```



## crontab 定时配置方法解析

```shell
0    12   *   *   *   COMMAND
分   时   日   月  周  COMMAND
```

| 代表意义 | 分钟 | 小时 | 日期 | 月份 | 周   |
| -------- | ---- | ---- | ---- | ---- | ---- |
| 数字范围 | 0~59 | 0~23 | 1~31 | 1~12 | 0~7  |

常见辅助字符解析：

| 特殊字符 | 代表意义                                                     |
| -------- | ------------------------------------------------------------ |
| *(星号)  | 代表任何时刻都接受的意思。举例来说，范例一内那个日、月、周都是*，就代表着不论何月、何日的礼拜几的12：00都执行后续命令的意思。 |
| ,(逗号)  | 代表分隔时段的意思。举例来说，如果要执行的工作是3：00与6：00时，就会是：0 3,6 * * * command，时间还是有五列，不过第二列是 3,6 ，代表3与6都适用 |
| -(减号)  | 代表一段时间范围内，举例来说，8点到12点之间的每小时的20分都进行一项工作：20 8-12 * * * command，仔细看到第二列变成8-12.代表 8,9,10,11,12 都适用的意思 |
| /n(斜线) | 那个n代表数字，即是每隔n单位间隔的意思，例如每五分钟进行一次，则：*/5 * * * * command，用*与/5来搭配，也可以写成0-59/5，意思相同 |



## crontab 高级特性

- 用户所建立的Crontab文件存于 /var/spool/cron 中，其文件名与用户名一致

- Linux 定时任务日志解析

  -  /var/log/cron 记录了是否执行了某些计划的脚本

  -  具体执行是否正确以及脚本执行过程中的一些信息则linux会每次都发邮件到该用户下，如 root 用户的定时任务执行日志会存放到 /var/spool/mail/root 文件中，有crontab执行日志的记录，用tail -f /var/spool/mail/root 即可查看最近的 crontab 执行情况

- /etc/crontab 文件

  用 crontab 配置是针对某个用户的，而编辑 /etc/crontab 是针对系统的任务。此文件的格式是：

  ```shell
  SHELL=/bin/bash  
  PATH=/sbin:/bin:/usr/sbin:/usr/bin 
  MAILTO=root //如果出现错误，或者有数据输出，数据作为邮件发给这个帐号  
  HOME=/ //使用者运行的路径,这里是根目录
  # run-parts
  01   *   *   *   *     root run-parts /etc/cron.hourly         //每小时执行/etc/cron.hourly内的脚本  
  02   4   *   *   *     root run-parts /etc/cron.daily           //每天执行/etc/cron.daily内的脚本  
  22   4   *   *   0     root run-parts /etc/cron.weekly       　　//每星期执行 /etc/cron.weekly内的脚本  
  42   4   1   *   *     root run-parts /etc/cron.monthly     　　//每月去执行/etc/cron.monthly内的脚本
  ```

  需要注意”run-parts”这个参数，如果去掉这个参数的话，后面就可以写要运行的某个脚本名，而不是文件夹名了
  也可以通过直接编辑 /etc/crontab 文件，添加相应任务(不建议)

  ```shell
  11 2 21 10 * rm -rf /mnt/fb  
  ```

