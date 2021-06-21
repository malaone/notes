### Mysql

#### 权限

```mysql
--root登录调整权限
--查看用户权限
show grants for 'microservice'@'%';
--赋予表（microservice_cluster.*表示所有表）权限
grant all privileges on microservice_cluster.* to 'microservice'@'%' identified by 'microservice';
--取消权限
revoke all privileges on microservice_cluster from 'microservice'@'%';
```

