### 环境

主中心：

主节点 192.168.35.46:6500

主节点 192.168.35.46:6501

主节点 192.168.35.46:6502

同城中心：

从节点 192.168.35.49:6500

从节点 192.168.35.49:6501

从节点 192.168.35.49:6502

### 背景

1.所有主节点都故障了，使用kill -9 pid1;kill -9 pid2模拟故障

2.通过redis故障转移命令cluster failover takeover将存活的从节点升级为主节点

3.当原来主节点恢复时，出现无法加入集群情况

### 恢复

1.启动主中心所有节点后，在同城中心任意节点查询：

```
[xxx@test-demo3 redis-5.0.9]$ ./src/redis-cli -h 192.168.35.49 -p 6500 cluster nodes
64707372b1eb184405696a41b92132ad7b42909b 192.168.35.49:6501@16501 master - 0 1616722238139 6 connected 10923-16383
c90e275fac150c4523b52e56a60b5322a3585b7b 192.168.35.49:6500@16500 myself,master - 0 1616722236000 4 connected 0-5460
5d6728fcd1d0ebc704ccd5e06193cc907fc9e8eb 192.168.35.49:6502@16502 master - 0 1616722237137 5 connected 5461-10922
66c5f14675fc0c623459a0a42f92622ee4bbd520 :0@0 master,fail,noaddr - 1616721992020 1616721989000 1 disconnected
b628d01b93c4c431d6d7f0f3b0ddc56c92ed0ac5 :0@0 master,fail,noaddr - 1616721992020 1616721990000 3 disconnected
40fcf38d9aff153bb480eca7a0c87716ae87a321 :0@0 master,fail,noaddr - 1616721992020 1616721990517 2 disconnected
```

2.剔除故障集群：同城中心每个节点执行命令，命令最后的id为上面0@0 master,fail对应的id：

```
./src/redis-cli -h 192.168.35.49 -p 6500 cluster forget 66c5f14675fc0c623459a0a42f92622ee4bbd520
./src/redis-cli -h 192.168.35.49 -p 6501 cluster forget 66c5f14675fc0c623459a0a42f92622ee4bbd520
./src/redis-cli -h 192.168.35.49 -p 6502 cluster forget 66c5f14675fc0c623459a0a42f92622ee4bbd520

./src/redis-cli -h 192.168.35.49 -p 6500 cluster forget b628d01b93c4c431d6d7f0f3b0ddc56c92ed0ac5
./src/redis-cli -h 192.168.35.49 -p 6501 cluster forget b628d01b93c4c431d6d7f0f3b0ddc56c92ed0ac5
./src/redis-cli -h 192.168.35.49 -p 6502 cluster forget b628d01b93c4c431d6d7f0f3b0ddc56c92ed0ac5

./src/redis-cli -h 192.168.35.49 -p 6500 cluster forget 40fcf38d9aff153bb480eca7a0c87716ae87a321
./src/redis-cli -h 192.168.35.49 -p 6501 cluster forget 40fcf38d9aff153bb480eca7a0c87716ae87a321
./src/redis-cli -h 192.168.35.49 -p 6502 cluster forget 40fcf38d9aff153bb480eca7a0c87716ae87a321
```

3.重新加入集群：同城中心任意节点执行命令

```
./src/redis-cli -h 192.168.35.49 -p 6500 cluster meet 192.168.35.46:6500
./src/redis-cli -h 192.168.35.49 -p 6500 cluster meet 192.168.35.46:6501
./src/redis-cli -h 192.168.35.49 -p 6500 cluster meet 192.168.35.46:6502
```

4.配置主从关系：主中心所有节点执行

```
./src/redis-cli -h 192.168.35.49 -p 6500 cluster replicate 192.168.35.46:6500
./src/redis-cli -h 192.168.35.49 -p 6501 cluster replicate 192.168.35.46:6501
./src/redis-cli -h 192.168.35.49 -p 6502 cluster replicate 192.168.35.46:6502
```

5.查询集群，确认处理结果

```
[xxx@test-demo3 redis-5.0.9]$ ./src/redis-cli -h 192.168.35.49 -p 6500 cluster nodes
64707372b1eb184405696a41b92132ad7b42909b 192.168.35.49:6501@16501 master - 0 1616723347329 6 connected 10923-16383
22194f0ed50443af33ea4d472b2f4e9aa862aa25 192.168.35.46:6501@16501 slave 64707372b1eb184405696a41b92132ad7b42909b 0 1616723345000 7 connected
c90e275fac150c4523b52e56a60b5322a3585b7b 192.168.35.49:6500@16500 myself,master - 0 1616723347000 4 connected 0-5460
bc49ccefe69d66b3a1de1fd331ea83564f19628c 192.168.35.46:6502@16502 slave 5d6728fcd1d0ebc704ccd5e06193cc907fc9e8eb 0 1616723345000 8 connected
5d6728fcd1d0ebc704ccd5e06193cc907fc9e8eb 192.168.35.49:6502@16502 master - 0 1616723348331 5 connected 5461-10922
ea6f0e67ade10cdd45926599b8891432a1c85325 192.168.35.46:6500@16500 slave c90e275fac150c4523b52e56a60b5322a3585b7b 0 1616723346328 4 connected
```

