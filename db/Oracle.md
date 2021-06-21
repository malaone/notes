### 创建库

```plsql
--root用户创建文件夹/home/data,并执行chmod -R 777 data
--创建两个数据库的文件（apollo.dbf 和apollo_temp.dbf 两个文件）
CREATE TABLESPACE apollo LOGGING DATAFILE '/home/data/apollo.dbf' SIZE 300M AUTOEXTEND ON NEXT 32M MAXSIZE 800M EXTENT MANAGEMENT LOCAL;

create temporary tablespace apollo_temp tempfile '/home/data/apollo_temp.dbf' size 300M autoextend on next 32M maxsize 800m extent management local;
--创建用户与上面创建的文件形成映射关系（用户名为apollo,密码为apollo）
CREATE USER apollo IDENTIFIED BY apollo DEFAULT TABLESPACE apollo TEMPORARY TABLESPACE apollo_temp;
--添加权限
grant connect,resource,dba to apollo;
grant create session to apollo;
--查询表空间
SELECT * FROM dba_tablespaces;
--删除数据库
DROP TABLESPACE apollo INCLUDING CONTENTS AND DATAFILES;
--删除用户
drop user apollo cascade;

```



RAC 高可用集群，共享磁盘、缓存

DG 数据同步，主备数据同步

所谓的ADG,只不过就是在备库,应用redo log的同事,避免资源的浪费,(10g之前的dg备库必须处于Mount状态,才可以接收应用redo log),11g增加的ADG的功能支持,备库处于open状态(默认为read only模式),同时可以接收并应用redo log

OGG侧重于数据备份,DG侧重于高可用。
OGG可以异构,DG只有ORACLE。

oracle之nomount、mount、open三种状态：

- **Nomount** – The database instance has been started (processes and memory structures have been allocated, but control file is not yet accessed).  数据库实例已经开始（进程和内存已经分配，但是控制文件还没有启动）

- **Mount** – Instance has accessed the control file, but has not yet validated its entry or accessed the datafiles.  实例已经开启控制文件，但是还没有开启数据文件

- **Open** – Instance has validated the entries in the control files and is accessing the datafiles – it is now open for business.  开启数据文件