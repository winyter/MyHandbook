# GreenPlum 检查并修复失败的 Segment 全过程

> Greenplum集群中，很可能出现网络、主机故障，造成的primary mirror切换。这是一种高可用机制，保证Greenplum的正常平稳运行。切换后，一个主机上会有多个segment执行，造成资源非均衡，并且耗费更多的系统资源。所以，及时恢复失败的segment，把所有segment恢复成原有的角色是非常必要的工作。



#### 一、检查失败的 segment

- **方式1：查询失败的segment，status='d'表示该segment不能用，u表示up。**

  ```shell
  $ psql -c "SELECT * FROM gp_segment_configuration WHERE status='d';"
   dbid | content | role | preferred_role | mode | status | port  | hostname | address | replication_port | san_mounts 
  ------+---------+------+----------------+------+--------+-------+----------+---------+------------------+------------
      4 |       2 | m    | p              | s    | d      | 40000 | gp-s3    | gp-s3   |            41000 | 
  (1 row)
  ```

- **方式2：gpstat -e命令，Change Tracking表名对应的mirror已经失败，这里gp-s1的mirror gp-s3已经失败。**

  >$ gpstate -e
  >20160118:13:49:49:023607 gpstate:gp-master:gpadmin-[INFO]:-Starting gpstate with args: -e
  >20160118:13:49:49:023607 gpstate:gp-master:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 4.3.6.2 build 1'
  >20160118:13:49:49:023607 gpstate:gp-master:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.2.15 (Greenplum Database 4.3.6.2 build 1) on x86_64-unknown-linux-gnu, compiled by GCC gcc (GCC) 4.4.2 compiled on Nov 12 2015 23:50:28'
  >20160118:13:49:49:023607 gpstate:gp-master:gpadmin-[INFO]:-Obtaining Segment details from master...
  >20160118:13:49:49:023607 gpstate:gp-master:gpadmin-[INFO]:-Gathering data from segments...
  >. 
  >20160118:13:49:50:023607 gpstate:gp-master:gpadmin-[INFO]:-----------------------------------------------------
  >20160118:13:49:50:023607 gpstate:gp-master:gpadmin-[INFO]:-Segment Mirroring Status Report
  >20160118:13:49:50:023607 gpstate:gp-master:gpadmin-[INFO]:-----------------------------------------------------
  >20160118:13:49:50:023607 gpstate:gp-master:gpadmin-[INFO]:-Segments with Primary and Mirror Roles Switched
  >20160118:13:49:50:023607 gpstate:gp-master:gpadmin-[INFO]:-   Current Primary   Port    Mirror   Port
  >20160118:13:49:50:023607 gpstate:gp-master:gpadmin-[INFO]:-   gp-s1             50000   gp-s3    40000
  >20160118:13:49:50:023607 gpstate:gp-master:gpadmin-[INFO]:-----------------------------------------------------
  >**20160118:13:49:50:023607 gpstate:gp-master:gpadmin-[INFO]:-Primaries in Change Tracking**
  >**20160118:13:49:50:023607 gpstate:gp-master:gpadmin-[INFO]:-   Current Primary   Port    Change tracking size   Mirror   Port**
  >**20160118:13:49:50:023607 gpstate:gp-master:gpadmin-[INFO]:-   gp-s1             50000   101 MB                 gp-s3    40000**

  gpstat -m命令可以看到详细

  > $ gpstate -m
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:-Starting gpstate with args: -m
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 4.3.6.2 build 1'
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.2.15 (Greenplum Database 4.3.6.2 build 1) on x86_64-unknown-linux-gnu, compiled by GCC gcc (GCC) 4.4.2 compiled on Nov 12 2015 23:50:28'
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:-Obtaining Segment details from master...
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:--Current GPDB mirror list and status
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:--Type = Spread
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:-   Mirror   Datadir                       Port    Status              Data Status       
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:-   gp-s2    /home/gpadmin/mirror/gpseg0   50000   Passive             Synchronized
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:-   gp-s3    /home/gpadmin/mirror/gpseg1   50000   Passive             Synchronized
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:-   gp-s1    /home/gpadmin/mirror/gpseg2   50000   Acting as Primary   Change Tracking
  > 20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------
  > **20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[WARNING]:-1 segment(s) configured as mirror(s) are acting as primaries**
  > **20160118:13:56:59:024354 gpstate:gp-master:gpadmin-[WARNING]:-1 mirror segment(s) acting as primaries are in change tracking**



#### 二、修复失败的 segment

1. **检查原失败segment恢复正常**

   - 主机可以访问
   - instance已启动
   - instance的数据目录正常可用

2. **修复失败segment**

   > $ gprecoverseg 
   >
   > 20160118:14:08:37:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Starting gprecoverseg with args: 
   >
   > 20160118:14:08:37:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 4.3.6.2 build 1'
   >
   > 20160118:14:08:37:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.2.15 (Greenplum Database 4.3.6.2 build 1) on x86_64-unknown-linux-gnu, compiled by GCC gcc (GCC) 4.4.2 compiled on Nov 12 2015 23:50:28'
   >
   > 20160118:14:08:37:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Checking if segments are ready
   >
   > 20160118:14:08:37:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Obtaining Segment details from master...
   >
   > 20160118:14:08:37:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Obtaining Segment details from master...
   >
   > 20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Greenplum instance recovery parameters
   >
   > 20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:----------------------------------------------------------
   >
   > 20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Recovery type        = Standard
   >
   > 20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:----------------------------------------------------------
   >
   > 20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Recovery 1 of 1
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:----------------------------------------------------------**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Synchronization mode             = Incremental**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Failed instance host             = gp-s3**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Failed instance address           = gp-s3**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Failed instance directory          = /home/gpadmin/primary/gpseg2**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Failed instance port             = 40000**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Failed instance replication port       = 41000**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Recovery Source instance host        = gp-s1**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Recovery Source instance address       = gp-s1**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Recovery Source instance directory      = /home/gpadmin/mirror/gpseg2**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Recovery Source instance port        = 50000**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Recovery Source instance replication port  = 51000**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-  Recovery Target               = in-place**
   >
   > **20160118:14:08:38:025645 gprecoverseg:gp-master:gpadmin-[INFO]:----------------------------------------------------------**
   >
   > Continue with segment recovery procedure Yy|Nn (default=N):
   >
   > \> y
   >
   > 20160118:14:08:42:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-1 segment(s) to recover
   >
   > 20160118:14:08:42:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Ensuring 1 failed segment(s) are stopped
   >
   > 20160118:14:08:43:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-23723: /home/gpadmin/primary/gpseg2
   >
   >  
   >
   > 20160118:14:08:51:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Ensuring that shared memory is cleaned up for stopped segments
   >
   > updating flat files
   >
   > 20160118:14:08:56:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Updating configuration with new mirrors
   >
   > 20160118:14:08:56:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Updating mirrors
   >
   > . 
   >
   > 20160118:14:08:57:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Starting mirrors
   >
   > 20160118:14:08:57:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Commencing parallel primary and mirror segment instance startup, please wait...
   >
   > .. 
   >
   > 20160118:14:08:59:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Process results...
   >
   > 20160118:14:08:59:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Updating configuration to mark mirrors up
   >
   > 20160118:14:08:59:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Updating primaries
   >
   > 20160118:14:08:59:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Commencing parallel primary conversion of 1 segments, please wait...
   >
   > . 
   >
   > 20160118:14:09:00:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Process results...
   >
   > 20160118:14:09:00:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Done updating primaries
   >
   > 20160118:14:09:00:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-******************************************************************
   >
   > 20160118:14:09:00:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Updating segments for resynchronization is completed.
   >
   > 20160118:14:09:00:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-For segments updated successfully, resynchronization will continue in the background.
   >
   > 20160118:14:09:00:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-
   >
   > 20160118:14:09:00:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-Use  gpstate -s  to check the resynchronization progress.
   >
   > 20160118:14:09:00:025645 gprecoverseg:gp-master:gpadmin-[INFO]:-******************************************************************

3. **确认primary和mirror已同步，**

   状态从不正常变为正常的过程: `Change Tracking>>Resynchronizing>> Synchronized`

   > $ gpstate -m
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:-Starting gpstate with args: -m
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 4.3.6.2 build 1'
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.2.15 (Greenplum Database 4.3.6.2 build 1) on x86_64-unknown-linux-gnu, compiled by GCC gcc (GCC) 4.4.2 compiled on Nov 12 2015 23:50:28'
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:-Obtaining Segment details from master...
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:--Current GPDB mirror list and status
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:--Type = Spread
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:-  Mirror  Datadir            Port   Status        Data Status    
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:-  gp-s2   /home/gpadmin/mirror/gpseg0  50000  Passive       Synchronized
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:-  gp-s3   /home/gpadmin/mirror/gpseg1  50000  Passive       Synchronized
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:-  gp-s1   /home/gpadmin/mirror/gpseg2  50000  Acting as Primary  Resynchronizing
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------
   >
   > 20160118:14:09:16:025774 gpstate:gp-master:gpadmin-[WARNING]:-1 segment(s) configured as mirror(s) are acting as primaries

   终于恢复到Synchronized状态

   > $ gpstate -m20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:-Starting gpstate with args: -m20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 4.3.6.2 build 1'20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.2.15 (Greenplum Database 4.3.6.2 build 1) on x86_64-unknown-linux-gnu, compiled by GCC gcc (GCC) 4.4.2 compiled on Nov 12 2015 23:50:28'20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:-Obtaining Segment details from master...20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:--Current GPDB mirror list and status20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:--Type = Spread20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:-  Mirror  Datadir            Port   Status        Data Status   20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:-  gp-s2   /home/gpadmin/mirror/gpseg0  50000  Passive       Synchronized20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:-  gp-s3   /home/gpadmin/mirror/gpseg1  50000  Passive       Synchronized20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:-  gp-s1   /home/gpadmin/mirror/gpseg2  50000  Acting as Primary  Synchronized20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------20160118:14:23:21:027009 gpstate:gp-master:gpadmin-[WARNING]:-1 segment(s) configured as mirror(s) are acting as primaries

#### 三、恢复所有 segment 到原有状态

当primary segment宕机，mirror成为了primary segment。执行gprecoverseg命令恢复失败segment后，当前primary segment是原来的mirror，原来失败的primary segment变成了现在的mirror。并没有恢复到系统初始化时候的角色状态，造成一个主机上有多个primary segment，会导致系统处于非平衡状态，消耗更多的系统资源。

1. **gpstate -e命令，能够检查出segment是否是原来的角色**

   > $ gpstate -e
   >
   > 20160118:14:51:47:029558 gpstate:gp-master:gpadmin-[INFO]:-Starting gpstate with args: -e
   >
   > 20160118:14:51:47:029558 gpstate:gp-master:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 4.3.6.2 build 1'
   >
   > 20160118:14:51:47:029558 gpstate:gp-master:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.2.15 (Greenplum Database 4.3.6.2 build 1) on x86_64-unknown-linux-gnu, compiled by GCC gcc (GCC) 4.4.2 compiled on Nov 12 2015 23:50:28'
   >
   > 20160118:14:51:47:029558 gpstate:gp-master:gpadmin-[INFO]:-Obtaining Segment details from master...
   >
   > 20160118:14:51:47:029558 gpstate:gp-master:gpadmin-[INFO]:-Gathering data from segments.... 
   >
   > 20160118:14:51:48:029558 gpstate:gp-master:gpadmin-[INFO]:-----------------------------------------------------
   >
   > 20160118:14:51:48:029558 gpstate:gp-master:gpadmin-[INFO]:-Segment Mirroring Status Report
   >
   > 20160118:14:51:48:029558 gpstate:gp-master:gpadmin-[INFO]:-----------------------------------------------------
   >
   > **20160118:14:51:48:029558 gpstate:gp-master:gpadmin-[INFO]:-Segments with Primary and Mirror Roles Switched**
   >
   > 20160118:14:51:48:029558 gpstate:gp-master:gpadmin-[INFO]:-  Current Primary  Port   Mirror  Port  **#gp-s1原来是mirror，现在成为了primary**
   >
   > 20160118:14:51:48:029558 gpstate:gp-master:gpadmin-[INFO]:-  gp-s1       50000  gp-s3   40000 **#gp-s3原来是primary，现在成为了mirror**

2. **运行gpstate -m，确认所有mirrors都是同步的Resynchronizing。**

   **如果mirrors的状态是Resynchronizing，等待完成。**

   > $ gpstate -m
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:-Starting gpstate with args: -m
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 4.3.6.2 build 1'
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.2.15 (Greenplum Database 4.3.6.2 build 1) on x86_64-unknown-linux-gnu, compiled by GCC gcc (GCC) 4.4.2 compiled on Nov 12 2015 23:50:28'
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:-Obtaining Segment details from master...
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:--Current GPDB mirror list and status
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:--Type = Spread
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:-  Mirror  Datadir            Port   Status        Data Status   
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:-  gp-s2   /home/gpadmin/mirror/gpseg0  50000  Passive       Synchronized
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:-  gp-s3   /home/gpadmin/mirror/gpseg1  50000  Passive       Synchronized
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:-  gp-s1   /home/gpadmin/mirror/gpseg2  50000  **Acting as Primary**  Synchronized
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[INFO]:--------------------------------------------------------------
   >
   > 20160118:14:57:38:030224 gpstate:gp-master:gpadmin-[WARNING]:-1 segment(s) configured as mirror(s) are acting as primaries 
   >
   > 
   >
   > *加粗文字，说明gp-s1原来是mirror，现在成为了primary(**Acting as Primary**)

3. **运行gprecoverseg -r命令，将所有segments恢复到原有角色。**

   > $ gprecoverseg -r
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-Starting gprecoverseg with args: -r
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 4.3.6.2 build 1'
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.2.15 (Greenplum Database 4.3.6.2 build 1) on x86_64-unknown-linux-gnu, compiled by GCC gcc (GCC) 4.4.2 compiled on Nov 12 2015 23:50:28'
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-Checking if segments are ready
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-Obtaining Segment details from master...
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-Obtaining Segment details from master...
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-Greenplum instance recovery parameters
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:----------------------------------------------------------
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-**Recovery type        = Rebalance  #恢复均衡操作**
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:----------------------------------------------------------
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-**Unbalanced segment 1 of 2  #非均衡segment**
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:----------------------------------------------------------
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Unbalanced instance host        = gp-s1  
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Unbalanced instance address       = gp-s1
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Unbalanced instance directory      = /home/gpadmin/mirror/gpseg2
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Unbalanced instance port        = 50000
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Unbalanced instance replication port  = 51000
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Balanced role              = Mirror
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Current role              = Primary
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:----------------------------------------------------------
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-Unbalanced segment 2 of 2
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:----------------------------------------------------------
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Unbalanced instance host        = gp-s3
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Unbalanced instance address       = gp-s3
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Unbalanced instance directory      = /home/gpadmin/primary/gpseg2
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Unbalanced instance port        = 40000
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Unbalanced instance replication port  = 41000
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Balanced role              = Primary
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-  Current role              = Mirror
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[INFO]:----------------------------------------------------------
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[WARNING]:-This operation will cancel queries that are currently executing.
   >
   > 20160118:15:31:14:000698 gprecoverseg:gp-master:gpadmin-[WARNING]:-Connections to the database however will not be interrupted.
   >
   > Continue with segment rebalance procedure Yy|Nn (default=N):
   >
   > \> y
   >
   > 20160118:15:31:24:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-Getting unbalanced segments
   >
   > 20160118:15:31:24:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-Stopping unbalanced primary segments...
   >
   > .. 
   >
   > 20160118:15:31:26:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-Triggering segment reconfiguration
   >
   > 20160118:15:31:30:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-Starting segment synchronization
   >
   > ........... 
   >
   > 20160118:15:31:41:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-******************************************************************
   >
   > 20160118:15:31:41:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-**The rebalance operation has completed successfully. #均衡操作执行完毕**
   >
   > 20160118:15:31:41:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-There is a resynchronization running in the background to bring all
   >
   > 20160118:15:31:41:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-segments in sync.  
   >
   > 20160118:15:31:41:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-
   >
   > 20160118:15:31:41:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-Use **gpstate -e** to check the resynchronization progress.
   >
   > 20160118:15:31:41:000698 gprecoverseg:gp-master:gpadmin-[INFO]:-******************************************************************

4. **均衡后，执行gpstate -e确认所有segment恢复到原来的角色**

   > $ gpstate -e
   >
   > 20160118:15:34:59:001246 gpstate:gp-master:gpadmin-[INFO]:-Starting gpstate with args: -e
   >
   > 20160118:15:34:59:001246 gpstate:gp-master:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 4.3.6.2 build 1'
   >
   > 20160118:15:34:59:001246 gpstate:gp-master:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.2.15 (Greenplum Database 4.3.6.2 build 1) on x86_64-unknown-linux-gnu, compiled by GCC gcc (GCC) 4.4.2 compiled on Nov 12 2015 23:50:28'
   >
   > 20160118:15:34:59:001246 gpstate:gp-master:gpadmin-[INFO]:-Obtaining Segment details from master...
   >
   > 20160118:15:35:00:001246 gpstate:gp-master:gpadmin-[INFO]:-Gathering data from segments.... 
   >
   > 20160118:15:35:01:001246 gpstate:gp-master:gpadmin-[INFO]:-----------------------------------------------------
   >
   > 20160118:15:35:01:001246 gpstate:gp-master:gpadmin-[INFO]:-Segment Mirroring Status Report
   >
   > 20160118:15:35:01:001246 gpstate:gp-master:gpadmin-[INFO]:-----------------------------------------------------
   >
   > 20160118:15:35:01:001246 gpstate:gp-master:gpadmin-[INFO]:-**All segments are running normally**
   >
   > 
   >
   > \#查看gp_segment_configuration，没有处于down状态的segment
   >
   > $ psql -c "SELECT * FROM gp_segment_configuration WHERE status='d';" 
   >
   > dbid | content | role | preferred_role | mode | status | port | hostname | address | replication_port | san_mounts 
   >
   > ------+---------+------+----------------+------+--------+------+----------+---------+------------------+------------
   >
   > (0 rows)

#### 四、其他场景恢复 segment

- **场景1：primary segment和它的mirror都宕机，这种场景下Greenplum 数据库处于不可用状态。**

  1. 重启Greenplum database

     \$ gpstop -r

  2. 恢复

     \$ gprecoverseg 

  3. check the status of mirrors

     \$ gpstate -m

  4. If you still have segments in Change Tracking mode, run a full copy recovery:

     \$ gprecoverseg -F

  If a segment host is not recoverable and you have lost one or more segments, recreate your GreenplumDatabase system from backup files. See Backing Up and Restoring Databases.

  

- **场景2:只有Primary，没有mirror的恢复**

  1. Ensure you can connect to the segment host from the master host. For example:

     \$ ping failed_seg_host_address

     

  2. Troubleshoot the problem that is preventing the master host from connecting to the segment host. For example, the host machine may need to be restarted.

     

  3. After the host is online, verify that you can connect to it and restart Greenplum Database. For example:

     \$ gpstop -r

     

  4. Run the gpstate utility to verify that all segment instances are online:

     \$ gpstate

- **场景3：使用mirror恢复Primary**

  如果Primary的主机硬件出现问题，使用它的mirror，可用恢复Primary。

  If a host is nonoperational, for example, due to hardware failure, recover the segments onto a spare set ofhardware resources. If mirroring is enabled, you can recover a segment from its mirror onto an alternatehost using gprecoverseg. For example:

  \$ gprecoverseg -i recover_config_file

  

  Where the format of recover_config_file is:

  filespaceOrder=[filespace1_name[:filespace2_name:...] failed_host_address:

  port:fselocation [recovery_host_address:port:replication_port:fselocation

  [:fselocation:...]]

  

  For example, to recover to a different host than the failed host without additional filespaces configured (besides the default pg_system filespace):

  

  filespaceOrder=sdw5-2:50002:/gpdata/gpseg2 sdw9-2:50002:53002:/gpdata/gpseg2

  

  The gp_segment_configuration and pg_filespace_entry system catalog tables can help determine your current segment configuration so you can plan your mirror recovery configuration. For example, run the following query:

  

  =# SELECT dbid, content, hostname, address, port, replication_port, fselocation as datadir FROM gp_segment_configuration, pg_filespace_entry WHERE dbid=fsedbidORDER BY dbid;

  

  The new recovery segment host must be pre-installed with the Greenplum Database software and configured exactly as the existing segment hosts.

​	
