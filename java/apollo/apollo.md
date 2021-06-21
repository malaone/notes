### 数据库

#### oracle

```plsql
--apollo配置库创建
CREATE TABLESPACE ApolloConfig LOGGING DATAFILE '/home/data/ApolloConfig.dbf' SIZE 300M AUTOEXTEND ON NEXT 32M MAXSIZE 800M EXTENT MANAGEMENT LOCAL;

create temporary tablespace ApolloConfig_temp tempfile '/home/data/ApolloConfig_temp.dbf' size 300M autoextend on next 32M maxsize 800m extent management local;

CREATE USER ApolloConfig IDENTIFIED BY ApolloConfig DEFAULT TABLESPACE ApolloConfig TEMPORARY TABLESPACE ApolloConfig_temp;

grant connect,resource,dba to ApolloConfig;
grant create session to ApolloConfig;

--apollo管理库创建
CREATE TABLESPACE ApolloPortal LOGGING DATAFILE '/home/data/ApolloPortal.dbf' SIZE 300M AUTOEXTEND ON NEXT 32M MAXSIZE 800M EXTENT MANAGEMENT LOCAL;

create temporary tablespace ApolloPortal_temp tempfile '/home/data/ApolloPortal_temp.dbf' size 300M autoextend on next 32M maxsize 800m extent management local;

CREATE USER ApolloPortal IDENTIFIED BY ApolloPortal DEFAULT TABLESPACE ApolloPortal TEMPORARY TABLESPACE ApolloPortal_temp;

grant connect,resource,dba to ApolloPortal;
grant create session to ApolloPortal;


--使用默认表空间
CREATE USER ApolloConfig IDENTIFIED BY ApolloConfig;
grant connect,resource,dba to ApolloConfig;
grant create session to ApolloConfig;

CREATE USER ApolloPortal IDENTIFIED BY ApolloPortal;
grant connect,resource,dba to ApolloPortal;
grant create session to ApolloPortal;
```

