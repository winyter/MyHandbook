# GP 高级操作

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

