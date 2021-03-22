# 实验2：用户及权限管理

## 实验目的：

掌握用户管理、角色管理、权根维护与分配的能力，掌握用户之间共享对象的操作技能。

## 实验内容：

Oracle有一个开发者角色resource，可以创建表、过程、触发器等对象，但是不能创建视图。本训练要求：

- 在pdborcl插接式数据中创建一个新的本地角色con_res_view，该角色包含connect和resource角色，同时也包含CREATE VIEW权限，这样任何拥有con_res_view的用户就同时拥有这三种权限。
- 创建角色之后，再创建用户new_user，给用户分配表空间，设置限额为50M，授予con_res_view角色。
- 最后测试：用新用户new_user连接数据库、创建表，插入数据，创建视图，查询表和视图的数据。

## 实验步骤：

- 第1步：以system登录到pdborcl，创建角色con_res_view和用户yuanhaiyang，并授权和分配空间：

  ```sql
  $ sqlplus system/123@pdborcl
  SQL> CREATE ROLE con_res_view;
  Role created.
  SQL> GRANT connect,resource,CREATE VIEW TO con_res_view;
  Grant succeeded.
  SQL> CREATE USER yuanhaiyang IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;
  User created.
  SQL> ALTER USER yuanhaiyang QUOTA 50M ON users;
  User altered.
  SQL> GRANT con_res_view TO yuanhaiyang;
  Grant succeeded.
  SQL> exit
  ```

  ![用户创建和授权](image\用户创建和授权.png)

- 第2步：yuanhaiyang连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。

  ```sql
  $ sqlplus yuanhaiyang/123@pdborcl
  SQL> show user;
  USER is "YUANHAIYANG"
  SQL> CREATE TABLE mytable (id number,name varchar(50));
  Table created.
  SQL> INSERT INTO mytable(id,name)VALUES(1,'zhang');
  1 row created.
  SQL> INSERT INTO mytable(id,name)VALUES (2,'wang');
  1 row created.
  SQL> CREATE VIEW myview AS SELECT name FROM mytable;
  View created.
  SQL> SELECT * FROM myview;
  NAME
  --------------------------------------------------
  zhang
  wang
  SQL> GRANT SELECT ON myview TO hr;
  Grant succeeded.
  SQL>exit
  ```

  ![创建表mytable](image\创建表mytable.png)

- 第3步：用户hr连接到pdborcl，查询YUANHAIYANG授予它的视图myview

  ```sql
  $ sqlplus hr/123@pdborcl
  SQL> SELECT * FROM yuanhaiyang.myview;
  NAME
  --------------------------------------------------
  zhang
  wang
  SQL> exit
  ```

  ![hr查看myview](image\hr查看myview.png)

## 查看数据库的使用情况

```sql
SQL>SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';

SQL>SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
 free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
 where  a.tablespace_name = b.tablespace_name;
```

![查看数据库的使用情况](image\查看数据库的使用情况.png)

## 用户界面操作授权

![表授权](image\表授权.png)

## 实验总结：

在本次实验中，主要是创建用户和表以及视图，在创建这些东西的时候因为对于SQL语句不是很熟悉，就造成了许多不该出现的简单错误，但最终还是完成了本次实验，还是要多加练习SQL语句的编写加深对数据库的理解。