# GP 实用 SQL

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

  ```sql
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
