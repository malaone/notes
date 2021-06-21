### docker安装

```shell
docker run -d --name zookeeper -p 2181:2181 -v /etc/localtime:/etc/localtime wurstmeister/zookeeper

docker run -d --name kafka --publish 9092:9092 --link zookeeper:zookeeper --env KAFKA_BROKER_ID=100 --env HOST_IP=192.168.1.12 --env KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 --env KAFKA_ADVERTISED_HOST_NAME=192.168.1.12 --env KAFKA_ADVERTISED_PORT=9092 --restart=always --volume /etc/localtime:/etc/localtime wurstmeister/kafka

docker run -d --name kafka-manager --link zookeeper:zookeeper --link kafka:kafka -p 9001:9000 --restart=always --env ZK_HOSTS=zookeeper:2181 sheepkiller/kafka-manager
```



### 命令

kafka启动：

```
./zookeeper-server-start.sh -daemon ../config/zookeeper.properties 
./kafka-server-start.sh -daemon ../config/server.properties
```

kafka-manager启动：

```
bin/kafka-manager
kafka-manager 默认的端口是9000，可通过 -Dhttp.port，指定端口; -Dconfig.file=conf/application.conf指定配置文件:

nohup ./kafka-manager -Dconfig.file=conf/application.conf -Dhttp.port=8080 &
```

