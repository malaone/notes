### ubuntu下安装

1.运行curl

```bash
curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.deb.sh | sudo bash
```

2.import PackageCloud signing key

```bash
wget -O - "https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey" | sudo apt-key add -
```

3.安装包

```bash
sudo apt-get update -y

sudo apt-get install rabbitmq-server -y --fix-missing
```

4.启动

```bash
service rabbitmq-server start

chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/

rabbitmq-plugins enable rabbitmq_management

rabbitmqctl add_user admin admin

rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```



### 安装

按照以下顺序安装：

1、安装openssl-lib

```
#下载
wget ftp://ftp.pbone.net/mirror/ftp.centos.org/7.6.1810/updates/x86_64/Packages/openssl-libs-1.0.2k-16.el7_6.1.x86_64.rpm
#安装
rpm -ivh openssl-libs-1.0.2k-16.el7_6.1.x86_64.rpm --force
```

2、安装erlang

```
rpm -ivh erlang-20.2.2-1.el7.centos.x86_64.rpm

若没有安装openssl-lib，会报错：
error: Failed dependencies:
        libcrypto.so.10(OPENSSL_1.0.2)(64bit) is needed by erlang-20.2.2-1.el7.centos.x86_64
```

3、安装socat

```
rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm
```

4、安装rabbitmq

```
rpm -ivh rabbitmq-server-3.7.6-1.el7.noarch.rpm
```

5、安装管理web插件

```
rabbitmq-plugins enable rabbitmq_management
```

6、启动问题

```
使用rabbitmq-server start启动报错：
1）Error when reading /var/lib/rabbitmq/.erlang.cookie: eacces
解决：
chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
2）ERROR: epmd error for host “192”:badarg (unknown POSIX error)
在文件/etc/rabbitmq/rabbitmq-env.conf里面添加这一行：NODENAME=rabbit@localhost，该文件原来是不存在的
```

home目录：/usr/lib/rabbitmq

配置文件：/usr/lib/rabbitmq/lib/rabbitmq_server-3.7.6/ebin/rabbit.app

### docker安装

```shell
1. 执行docker search rabbitMq搜索镜像
2. 执行docker pull rabbitmq:management 拉取management版本（带有web页面的）的rabbitMq镜像
3. 执行docker images查看所有镜像
4. 执行docker run -dit --name Myrabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq:management
5. 执行docker ps 查看正在运行的容器

集群安装：https://blog.csdn.net/mx472756841/article/details/88800984
```



### 命令

```
后台启动（直接ctrl c不会停止rabbitmq服务）：
rabbitmq-server start &
添加用户：
rabbitmqctl add_user admin admin
给用户赋予管理员权限：
rabbitmqctl set_user_tags admin administrator
设置用户权限
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
关闭服务：
rabbitmq-server stop
```

### 踩过得到坑

#### 1、.erlang.cookie

搭建集群时所有节点的.erlang.cookie内容必须一致，可直接从一个节点复制到其他节点，复制后的个节点md5sum可能会不一致，这个不影响，解压版rabbitmq的.erlang.cookie在当前用户下

#### 2、主机名配置

需要在每个节点的/etc/hosts中添加各节点的ip及主机名：

```
192.168.2.1  node1
192.168.2.2  node2
192.168.2.3  node3
```

#### 3、添加用户命令报错

执行rabbitmqctl add_user admin admin报错内容：Error:undef。。。。

```
可查看rabbitmq的日志rabbit.log：
"load failed, Failed to load NIF library: '/usr/lib64/libcrypto.so.10: version 'OPENSSL_1.0.2' not found (required by /home/hero/app/rabbitmq/erlang/lib/erlang/lib/crypto-4.2.1/priv/lib/crypto.so)'"

解决方式：
1.手动下载可用服务器上的 libcrypto.so.1.0.2k 文件，上传到/usr/lib64目录
2.删除软连接rm -rf /usr/lib64/libcrypto.so.10
3.重新设置ln -s /usr/lib64/libcrypto.so.1.0.2k /usr/lib64/libcrypto.so.10
4.添加运行权限chmod +x /usr/lib64/libcrypto.so.1.0.2k
```

