---
title: Docker 常用命令
category: Technology
toc: true
date: 2016-10-23 22:32:05
tags: Docker
---
&emsp;&emsp;初学Docker，记录一些常用命令与配置方法。

## 信息查看
```bash
docker info                  # 查看docker基本信息
docker images                # 查看镜像列表
docker ps                    # 查看运行中的容器列表
docker logs 819f822966a6     # 查看容器运行log
docker search (image-name)   # 仓库中镜像查找
docker stats                 # 查看docker使用cpu、内存、网络、io情况
```

## 启动和终止容器
```bash
docker run busybox /bin/echo Hello Docker          # 容器名称“Helo Docker”
docker start 819f822966a6                          # 运行
docker stop 819f822966a6                           # 停止
docker restart 819f822966a6                        # 重启
docker rm 819f822966a6                             # 删除
docker commit 819f822966a6 job1                    # 将容器保存为镜像

# 进入容器内部
docker exec -it 775c7c9ee1e1 /bin/bash  
```

## 导入和导出容器
```bash
docker export 7691a814370e > ubuntu.tar   # export
```

## CentOS 7 迁移Docker目录
```bash
cp -r /var/lib/docker/ /data/docker  
vim /lib/systemd/system/docker.service
   ExecStart=/usr/bin/dockerd --graph="/data/docker"
systemctl daemon-reload   #重新加载配置
service docker restart    #重启服务
```
[参考](https://docs.docker.com/engine/admin/systemd/)