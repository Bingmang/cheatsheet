# Kubernetes

## 安装

### kvm

minikube依赖virtualbox或kvm，ubuntu安装kvm：https://wiki.ubuntu.org.cn/UbuntuHelp:KVM/Installatio

```sh
sudo aptitude install kvm libvirt-bin ubuntu-vm-builder bridge-utils
virsh -c qemu:///system list
```

### minikube

下载地址：https://github.com/kubernetes/minikube/releases

```bash
wget https://github.com/kubernetes/minikube/releases/download/v1.5.1/minikube-linux-amd64
mv minikube-linux-amd64 minikube
sudo install minikube /usr/local/bin/minikube
```

在安装minikube的时候，国内会被墙，导致镜像下载失败，参考阿里云的安装教程即可：https://yq.aliyun.com/articles/221687

```bash
minikube start --image-mirror-country cn \
    --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.5.0.iso \
    --registry-mirror=https://xxxxxx.mirror.aliyuncs.com
```

### kubectl

```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version
```

## 常用指令

注：`alias k=kubectl`

```sh
# 集群相关
k cluster-info

# 状态描述
k describe node/pod/svc [Name] 

# 获取信息
k get nodes/pods/svc/rc [-o wide]
# 获取已部署pod的完整YAML
k get po [PodName] -o yaml

# 暴露ReplicationController，分配负载均衡的Service
k expose rc [ReplicationControllerName] --type=LoadBalancer --name [ServiceName]

# 伸缩pods
k scale rc [ReplicationControllerName] --replicas=3
```
