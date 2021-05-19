## 1. 安装

### 1.1 加载netfilter驱动

```shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

```

### 1.2 配置sysctl

```shell
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

cat <<EOF | sudo tee /etc/sysctl.d/ip_forward.conf
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

### 1.3 关闭交换分区

swapoff -a

注释掉/etc/fstab

### 1.4 配置k8s的repo，yum install kubeadm kubectl kubelet

### 1.5 kubeadm config images list列出要用的镜像，然后从国内源下载下来，再tag成原来的名字

### 1.6 配置

- ```shell
  #显示默认配置，可以用来生成config.yml文件
  kubeadm config print init-defaults
  
  #初始化前先添加hosts记录
  cat <<EOF >> /etc/hosts
  2.2.11.79 centos79
  2.2.11.80 c80
  2.2.11.81 c81
  EOF
  #master导入镜像后，执行初始化
  kubeadm init [--config=kubeadm-config.yaml | tee kubeadm-init.log] 
  
  kubeadm init --apiserver-advertise-address=2.2.11.79 --service-cidr=10.2.0.0/16 --pod-network-cidr=10.244.0.0/16
  
  --image-repository registry.aliyuncs.com/google_containers 
  
  #初始化后会启动多个docker容器，自动生成的配置文件保存在
  /var/lib/kubelet/config.yaml
  
  	
  #都添加完后执行kubectl get nodes，会发现所有节点的状态都是NotReady,因为没有安装网络插件
  
  mkdir -p /etc/cni/net.d
  cat >/etc/cni/net.d/10-mynet.conf <<-EOF
  {
      "cniVersion": "0.3.0",
      "name": "mynet",
      "type": "bridge",
      "bridge": "cni0",
      "isGateway": true,
      "ipMasq": true,
      "ipam": {
          "type": "host-local",
          "subnet": "10.244.0.0/16",
          "routes": [
              {"dst": "0.0.0.0/0"}
          ]
      }
  }
  EOF
  cat >/etc/cni/net.d/99-loopback.conf <<-EOF
  {
      "cniVersion": "0.3.0",
      "type": "loopback"
  }
  EOF
  
  kubectl create -f kube-flannel.yml
  kubectl delete -f kube-flannel.yml
  
  
  #等待k8s 把 pod：flannel生效 可以用命令查看状态进度：
  
  kubectl get pods -n kube-system
  等 flannel 状态显示 Running后
  
  
  #coredns的pod一直处于containerCreating状态，使用命令查看
   kubectl describe pod coredns-b7d8c5745-4qxnh -n kube-system
  
  ```


```
1.在master节点上输出增加节点的命令
kubeadm token create --print-join-command
2.在node节点上执行
eval $(ssh 2.2.11.79 "kubeadm token create --print-join-command")

3.#执行查看k8s集群运行状态命令：
sudo kubectl get nodes

4.给其他节点打标签
kubectl label nodes c80 node-role.kubernetes.io/node=
kubectl label nodes c81 node-role.kubernetes.io/node=
```

```
4.支持命令补全
yum install bash-completion -y
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
kubectl completion bash >/etc/bash_completion.d/kubectl
```



**kubectl默认使用~/.kube/config作为配置文件，其中包含了元数据、连接信息、认证token和证书**

当连接多个集群时，可以通过KUBECONFIG环境变量来指定使用哪个配置文件

# 重启kubeadm部署的k8s集群

master节点执行systemctl restart kubelet

## 2. 组件

**master**组件：

​	-1.The API server

​	-2.Etcd

​	-3.controller manager，It contains the replication controller, the pod controller, the services controller, the endpoints controller, and others

​	-4.kube-scheduler，is responsible for scheduling pods into nodes

​	-5.dns，It is scheduled as a regular pod

**node组件：**

​	-1.Kube-proxy

​	-2.kubelet

## 3. 创建deployment

> rc滚动升级会引起服务中断，于是引入了deployment资源

### 3.1 create

```shell
#创建一个deployment source
kubectl create deployment ds_name --image=image_name

#查看状态
kubectl get deployment ds_name
#或者
kubectl get deployment.apps

#列出详细信息，用于调试
kubectl describe deployment ds_name

#编辑配置文件
kubectl edit deployment ds_name
#imagePullPolicy设置为IfNotPresent(如果本地没有，才从远程仓库拉取) 或者 Never(只从本地拉取)
```

> **显示镜像拉取错误：**

```shell
#单节点要让在master可以调度
kubectl taint nodes --all node-role.kubernetes.io/master-

#一主多从要保证被调度到的节点上有该镜像
#在从节点上运行容器(container)前，会先创建相应的pod，可以通过docker ps查看到
#pod资源至少由2个容器组成，pod基础容器和业务容器
```

### 3.2 expose

```shell
kubectl expose deployment hello-go --type=LoadBalancer --port=8180

#查看被expose的外部端口，然后就可以通过外网访问容器了
kubectl get service -o wide
#查看endpoints
kubectl describe svc svc_name

#如果访问不通，可能是node节点上kube-proxy服务没开

#查看日志
kubectl logs -l app=hello-go -f
kubectl logs pod_name container_name
```

### 3.3 scale

```shell
#将已有容器扩展至4个；这是让生成了4个pod资源，而不是在一个pod内启动4个hello-go
kubectl scale deployments/hello-go --replicas=4

$ kubectl get deployment   hello-go
NAME          READY      UP-TO-DATE      AVAILABLE     AGE
hello-go      4/4         4                      4      35m


#批量查看日志
for pod in $(kubectl get po -l app=hello-go -oname);do echo $pod;kubectl logs $pod;done
```

### 3.4 备份 k8s yaml 配置

```shell
kubectl get -o yaml pod pod_name 
# --export去掉集群相关信息
kubectl get -o yaml 资源类型 资源名称 --export > file.yml
```



## 4. 使用ansible

### 4.1 利用docker build的中间镜像来构建最终镜像

```dockerfile
FROM golang:1-alpine as build

WORKDIR /app
COPY cmd cmd
RUN go build cmd/hello/hello.go

FROM alpine:latest

WORKDIR /app
COPY --from=build /app/hello /app/hello

EXPOSE 8180
ENTRYPOINT ["./hello"]

```

