### 常见问题

1、连接拒绝

```
docker login --username=evan 192.168.0.203
Error response from daemon: Get https://192.168.0.203/v2/: dial tcp 192.168.0.203:443: connect: connection refused
通过Jenkins构建也会报类似错误

解决办法：添加 insecure-registries
1）vi /etc/docker/daemon.json
"insecure-registries": ["192.168.2.203"] (私有仓库地址)
2）systemctl daemon-reload
3）systemctl restart docker
```

