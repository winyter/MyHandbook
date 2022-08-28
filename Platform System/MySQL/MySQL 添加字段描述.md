# MySQL 添加字段描述

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

