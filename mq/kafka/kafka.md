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



### 问题

1. Kafka的用途有哪些？使用场景如何？

   1）日志收集

   2）大数据

   3）用户行为数据收集

2. Kafka中的ISR、AR又代表什么？ISR的伸缩又指什么

   AR是指分区的所有副本，包括leader和follower

   ISR指所有跟leader副本保持一定程度的同步的副本，包含leader本身

   OSR指所有跟leader副本同步滞后的副本

3. Kafka中的HW、LEO、LSO、LW等分别代表什么？

   HW：High Watermark ，消费者可以消费到这个offset之前的消息

   LEO：Log End Offset，每个副本都有自己的LEO，代表下一条消息写入的位置，ISR中所有副本的最小LEO即是HW

   LSO：LastStableOffset，它具体与kafka的事物有关，消费端参数isolation.level，这个参数用来配置消费者事务的隔离级别（read_uncommitted和read_committed），LSO表示消费者所消费到的位置，如果设置为read_committed，那么消费者就会忽略事务未提交的消息，既只能消费到LSO(LastStableOffset)的位置，默认情况下隔离级别为read_uncommitted，既可以消费到HW（High Watermak）的位置

   参考：[Kafka科普系列 | 什么是LSO？](https://hiddenpps.blog.csdn.net/article/details/88985769)

   LW：AR集合中最小的Log Start Offset

   参考：[Kafka科普系列 | 什么是LW和logStartOffset?](https://blog.csdn.net/u013256816/article/details/88939070) 

4. Kafka中是怎么体现消息顺序性的？

   

5. Kafka中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？

   

6. Kafka生产者客户端的整体结构是什么样子的？

   

7. Kafka生产者客户端中使用了几个线程来处理？分别是什么？

   

8. Kafka的旧版Scala的消费者客户端的设计有什么缺陷？

   

9. “消费组中的消费者个数如果超过topic的分区，那么就会有消费者消费不到数据”这句话是否正确？如果正确，那么有没有什么hack的手段？

   

10. 消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1?

    

11. 有哪些情形会造成重复消费？

    

12. 那些情景下会造成消息漏消费？

    

13. KafkaConsumer是非线程安全的，那么怎么样实现多线程消费？

    

14. 简述消费者与消费组之间的关系

    

15. 当你使用kafka-topics.sh创建（删除）了一个topic之后，Kafka背后会执行什么逻辑？

    

16. topic的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？

    

17. topic的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？

    

18. 创建topic时如何选择合适的分区数？

    

19. Kafka目前有那些内部topic，它们都有什么特征？各自的作用又是什么？

    

20. 优先副本是什么？它有什么特殊的作用？

    

21. Kafka有哪几处地方有分区分配的概念？简述大致的过程及原理

    

22. 简述Kafka的日志目录结构

    

23. Kafka中有那些索引文件？

    

24. 如果我指定了一个offset，Kafka怎么查找到对应的消息？

    

25. 如果我指定了一个timestamp，Kafka怎么查找到对应的消息？

    

26. 聊一聊你对Kafka的Log Retention的理解

    

27. 聊一聊你对Kafka的Log Compaction的理解

    

28. 聊一聊你对Kafka底层存储的理解（页缓存、内核层、块层、设备层）

    

29. 聊一聊Kafka的延时操作的原理

    

30. 聊一聊Kafka控制器的作用

    

31. 消费再均衡的原理是什么？（提示：消费者协调器和消费组协调器）

    

32. Kafka中的幂等是怎么实现的

    

33. Kafka中的事务是怎么实现的（这题我去面试6加被问4次，照着答案念也要念十几分钟，面试官简直凑不要脸）

    

34. Kafka中有那些地方需要选举？这些地方的选举策略又有哪些？

    

35. 失效副本是指什么？有那些应对措施？

    

36. 多副本下，各个副本中的HW和LEO的演变过程

    

37. 为什么Kafka不支持读写分离？

    

38. Kafka在可靠性方面做了哪些改进？（HW, LeaderEpoch）

    

39. Kafka中怎么实现死信队列和重试队列？

    

40. Kafka中的延迟队列怎么实现（这题被问的比事务那题还要多！！！听说你会Kafka，那你说说延迟队列怎么实现？）

    

41. Kafka中怎么做消息审计？

    

42. Kafka中怎么做消息轨迹？

    

43. Kafka中有那些配置参数比较有意思？聊一聊你的看法

    

44. Kafka中有那些命名比较有意思？聊一聊你的看法

    

45. Kafka有哪些指标需要着重关注？

    

46. 怎么计算Lag？(注意read_uncommitted和read_committed状态下的不同)

    

47. Kafka的那些设计让它有如此高的性能？

    

48. Kafka有什么优缺点？

    

49. 还用过什么同质类的其它产品，与Kafka相比有什么优缺点？

    

50. 为什么选择Kafka?

    

51. 在使用Kafka的过程中遇到过什么困难？怎么解决的？

    

52. 怎么样才能确保Kafka极大程度上的可靠性？

    

53. 聊一聊你对Kafka生态的理解
