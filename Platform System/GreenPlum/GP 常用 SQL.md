# GP 常用 SQL

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

  
