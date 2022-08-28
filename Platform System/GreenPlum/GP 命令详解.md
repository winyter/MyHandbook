# GP 命令详解

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

  
