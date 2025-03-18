---
title: windows 11 安装 minikube
date: 2022-05-13 16:23:57
tags:
---
## windows 11 安装 minikube

1.安装 [docker]([Home - Docker](https://www.docker.com/))

2.访问官网 [minikube]([minikube start | minikube (k8s.io)](https://minikube.sigs.k8s.io/docs/start/))

#### 下载安装包
选择windows安装包
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ce853e6fe59ec21712e20f5fcb01665e.png)

#### 设置环境变量
以管理员运行 PowerShell ，运行以下命令
```
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){ `
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine) `
}
```
或者手动设置
系统设置 - 环境变量 - 添加系统变量
```
Path 添加 C:\minikube
```

#### 启动 minikube
因为网络问题，可能下载image时会产生问题，需要添加一些参数，官方有说明
```
--image-mirror-country
Country code of the image mirror to be used. Leave empty to use the global one. For Chinese mainland users, set it to cn.

--image-repository
Alternative image repository to pull docker images from. This can be used when you have limited access to gcr.io. Set it to "auto" to let minikube decide one for you. For Chinese mainland users, you may use local gcr.io mirrors such as registry.cn-hangzhou.aliyuncs.com/google_containers.

--driver
Used to specify the driver to run Kubernetes in. The list of available drivers depends on operating system.
```
所以需要运行
```
minikube start --driver=docker --image-mirror-country=cn --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```
查看状态
```
minikube status
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3db280eef438e6340e4c667bbd927d3d.png)
#### 部署应用
官方命令
```
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
```
这个 k8s.gcr.io/echoserver:1.4 image 因为网络问题无法下载，在docker中查找该image的镜像
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a6b831a0ff05cb7458927348e94cdf5.png)
这里就采用第一个，需要注意的是官方的镜像监听的是8080端口，该镜像是80端口
```
// 创建一个名叫 hello-minikube 的部署
kubectl create deployment hello-minikube --image=cilium/echoserver
```
#### NodePort 方式
```
// 使用 NodePort  方式将集群的80端口暴露，作为一个 service
kubectl expose deployment hello-minikube --type=NodePort --port=80
```
下面是官方指令 minikube service
```
// minikube 提供的快捷访问 node 的 ip 和 service 的端口
minikube service hello-minikube
```
在官方文档中有提及，在 windows 下使用 docker 网络将受到限制，无法访问 node
```
The network is limited if using the Docker driver on Darwin, Windows, or WSL, and the Node IP is not reachable directly.
Running minikube on Linux with the Docker driver will result in no tunnel being created.
```
只能通过 port-forward 来访问
```
// 将宿主机器的7080端口转发到集群的80端口
kubectl port-forward service/hello-minikube 7080:80
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff7c1a4025855125afd425dab537a422.png)
访问 [http://localhost:7080](http://localhost:7080)，看到如下信息
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/73ce2f42e9de5ec91d44df7a4e0f6feb.png)

#### LoadBalancer 方式
```
// 创建一个名叫 hello-minikube-loadbalancer 的部署
kubectl create deployment hello-minikube-loadbalancer --image=cilium/echoserver
```
```
// 使用 LoadBalancer 方式将集群的80端口暴露，作为一个 service
kubectl expose deployment hello-minikube-loadbalancer --type=LoadBalancer --port=80
```
```
// 查看名为 hello-minikube-loadbalancer 的 service 信息
kubectl get services hello-minikube-loadbalancer
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d3575db59949c620e9809c3e4668fa0d.png)
可以看到该service 的 external-ip（外部ip） 一直处于 pending（待获取） 状态，此时需要**另外打开一个命令行**输入
```
minikube tunnel
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0e4be7dac17d27b4914faa52502f204a.png)
再次查看 service 的信息
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9dffba764210e21ebcfe4aef05069499.png)
访问 [http://localhost/](http://localhost/) 或者 [http://127.0.0.1/](http://127.0.0.1/) 看到如下信息
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/41e4f55f5e27702680b39426399e77ed.png)


#### 相关命令
```
kubectl get pod -o wide		// 获取当前所有 pod 信息
kubectl get deployment		// 获取当前所有 deployment 信息
kubectl get service			// 获取当前所有 service 信息
kubectl get all				// 获取当前所有信息
kubectl describe pod		// 查看pod详细信息
kubectl logs pod			// 查看pod的日志
kubectl scale deployment hello-minikube --replicas=5  	// 将标签为 hello-minikube 的 pod 设置副本为 5

minikube dashboard			// minikube 提供可视化仪表盘
minikube stop 				// 停止你的集群
minikube delete --all 		// 删除所有集群和配置
minikube ip					// 获取集群node ip
```
