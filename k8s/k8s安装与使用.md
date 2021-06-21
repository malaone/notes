### 安装k8s

1. 安装并启动 docker

   ```shell
   yum install docker-ce -y && systemctl enable docker --now
   ```

2. 配置镜像加速器

   ```shell
   tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://zgmke8qe.mirror.aliyuncs.com"]
   }
   EOF
   ```

3. 重新加载docker并重启

   ```shell
   systemctl daemon-reload  &&  systemctl restart docker
   ```

4. 安装minikube

   ```shell
   minikube start --registry-mirror=https://registry.docker-cn.com
   
   #由于配置了镜像加速器，所以不需要指定  --registry-mirror
   minikube start --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.9.0.iso --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers  --driver=none
   # 如果安装中提示（X Sorry, Kubernetes 1.18.3 requires conntrack to be installed in root's path） 可执行以下命令安装
    yum install conntrack -y
    
   #启动k8s	
   minikube start --vm-driver=none --registry-mirror=https://registry.docker-cn.com
   ```

### k8s使用

#### 启动web管理端

```shell
kubectl proxy --port=46473 --address='192.168.1.23' --accept-hosts='^.*'

访问地址：http://192.168.1.23:46473/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

##### 安装应用

1. 创建文件夹/home/xulifei/redis6

2. 创建一个namespace（可以忽略，使用默认的namespace）

   创建文件namespace.yaml ，内容如下：

   ```yml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: redis6
   ```

   创建namespace并查看：

   ```shell
   kubectl apply -f namespace.yaml
   
   # 查看 namespace
   kubectl get namespace
   ```

3. 创建service

   创建文件 redis_service.yaml，内容如下：

   ```yml
   apiVersion: v1
   kind: Service
   metadata:
     name: redis6-service
     labels:
       name: redis6-redis
     namespace: redis6
   spec:
     type: NodePort
     selector:
       app: redis6-redis
     ports:
     - port: 6379
       protocol: TCP
       targetPort: 6379
       name: http
   ```

   创建service并查看：

   ```shell
   create -f redis_service.yaml --record
   #查看service详情
   kubectl describe svc/redis6-service --namespace=redis6
   ```

4. 创建configmap

   ```shell
   #创建redis配置文件，并填写配置
   mkdir config && cd config
   vim redis.conf
   
   # 在 redis6 namespace 中创建 configmap，根据上面的文件创建
   kubectl create configmap redis6-redis-conf --from-file=redis.conf -n redis6
   
   # 查看configmap
   kubectl describe cm redis6-redis-conf -n redis6
   
   # 修改configmap（可参考：https://www.qttc.net/503-how-change-exist-configmap-on-kubernetes.html），重启pod生效
   kubectl edit cm redis6-redis-conf -n redis6
   
   ```

5. 创建 redis 容器

   创建文件 redis_deploy.yaml，内容如下：

   ```yml
   kind: Deployment
   metadata:
     name: redis6-redis
     namespace: redis6
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: redis6-redis
     template:
       metadata:
        labels:
          app: redis6-redis
       spec:
        containers:
        - name: redis6-redis
          image: redis
          volumeMounts:
          - name: foo
            mountPath: "/usr/local/etc"
          command:
            - "redis-server"
          args:
            - "/usr/local/etc/redis/redis.conf"
        volumes:
        - name: foo
          configMap:
            name: redis6-redis-conf
            items:
              - key: redis.conf
                path: redis/redis.conf
   ```

   创建和查看 pod

   ```shell
   kubectl apply -f redis_deploy.yaml 
   # 查看 pod
   kubectl get pods -n redis6
   NAME                            READY   STATUS    RESTARTS   AGE
   redis6-redis-5d749fbcd6-m6p65   1/1     Running   0          14m
   
   # 进入容器
   exec -it --namespace=redis6 redis6-redis-5d749fbcd6-m6p65 -- /bin/bash
   
   # pod重启
   kubectl replace --force -f redis_deploy.yaml
   
   # 删除pod
   kubectl delete pod redis6-redis-5d749fbcd6-m6p65
   ```

   



