### 常用命令

#### 创建用户

useradd testuser 创建用户testuser
passwd testuser 给已创建的用户testuser设置密码
说明：新创建的用户会在/home下创建一个用户目录testuser
usermod --help 修改用户这个命令的相关参数
userdel testuser 删除用户testuser
rm -rf testuser 删除用户testuser所在目录

#### 文件夹权限赋予

xlf:xlf  用户:组

```shell
chown -R xlf:xlf /home/tmp 
```



#### 磁盘

```
1、查看所有磁盘容量
[root@node165 jobs]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cl-root   44G   31G   14G  71% /
devtmpfs             3.9G     0  3.9G   0% /dev
tmpfs                3.9G     0  3.9G   0% /dev/shm
tmpfs                3.9G  105M  3.8G   3% /run
tmpfs                3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1           1014M  139M  876M  14% /boot
tmpfs                783M     0  783M   0% /run/user/0
tmpfs                783M     0  783M   0% /run/user/1001
2、当前目录的每个文件夹的大小
[root@node165 jobs]# du -ah --max-depth=1
7.6G    ./yusp-micro-srv-mgr
150M    ./yusp-job-admin
2.3G    ./yusp-uaa
94M     ./yusp-echain
14G     .

也可以用命令：du -sh ./*
```

​	