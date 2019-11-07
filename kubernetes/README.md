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

```sh
wget https://github.com/kubernetes/minikube/releases/download/v1.5.1/minikube-linux-amd64
mv minikube-linux-amd64 minikube
sudo install minikube /usr/local/bin/minikube
```

在安装minikube的时候，国内会被墙，导致镜像下载失败，参考阿里云的安装教程即可：https://yq.aliyun.com/articles/221687

```sh
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

假设现有资源：`kubia-pod`、`kubia-service`、`kubia-node`、`kubia-container`、`kubia-rc`、`kubia-rs`、`kubia-ns`等

假设使用的标签为：`env: (dev, debug, prod)`

### 查询

```sh
# 集群相关
k cluster-info

# 状态描述
k describe node/pod/svc kubia 

# 获取信息
k get nodes/pods/svc/rc/ns [-o wide] [--show-labels] [-L [LabelKey,]]

# 获取日志（如果有多个容器则需要指定容器）
k logs kubia-pod [-c kubia-container] [--previous]

# 获取已部署pod的完整YAML
k get po kubia-pod -o yaml

# 获取API描述
k explain pods/pod.spec/pod.metadata
```

### 网络

```sh
# 暴露ReplicationController，分配负载均衡的Service
k expose rc kubia-rc --type=LoadBalancer --name kubia-service-http
minikube service kubia-service-http

# 本地端口转发（调试用，前台进程）
k port-forward kubia-pod 8888:8080

# 关闭网络连接接口（模拟故障）
minikube ssh kubia-node
sudo ifconfig eth0 down
```

### 资源操作

```sh
# 伸缩pods
k scale rc kubia-rc --replicas=3

# 使用配置文件创建资源
k create -f kubia-manual.yaml
KUBE_EDITOR="/bin/zsh" k edit po kubia-pod

# 移除Pod（通过标签删除需要30秒）
k delete po kubia-pod kubia-pod2
k delete po -l env=prod
k delete po --all

# 删除当前命名空间下的所有资源（30秒）
k delete all --all

# 删除Controller，不删除Pods
k delete kubia-rc --cascade=false
```

### 标签、注解

```sh
# 加标签（改标签）
k label po kubia-pod env=prod [--overwrite]
k label node kubia-node gpu=true

# 使用标签筛选器
k get po -l env=prod
k get po -l env in (prod, debug)
k get po -l env notin (prod, debug)
k get po -l 'env!=prod'
k get po -l '!env'

# 添加注解
k annotate pod kubia-pod jd.com/group="wxt yr"
k describe pod kubia-pod | grep Annotations
```

### 命名空间

```sh
# 查询命名空间下的pods
k get po -n kube-system

# 创建命名空间
k create ns kubia-ns

# 创建Pod
k create -f kubia-pod.yaml -n kubia-ns

# 快速切换命名空间
alias kcd='kubectl config set-context $(kubectl config current-context) --namespace'
kcd kubia-ns

# 删除命名空间（连带下面的pods，需要30秒）
k delete ns kubia-ns
```

## 配置文件

### Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: kubia-gpu

spec:
  # 节点选择器
  nodeSelector:
    gpu: "true"
  containers:
  - image: bingmang/kubia
    name: kubia
    # 存活探针
    livenessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 15 # 容器启动后15秒再开始探测
```

### ReplicaSet

```yaml
apiVersion: apps/v1beta2
kind: ReplicaSet

metadata:
  name: kubia

spec:
  replicas: 3
  # ReplicaSet比ReplicationController提供了更多的选择器
  #（如果标签简单，可以省略selector，k8s会自动匹配template中的label）
  selector:
    matchLabels:
      app: kubia
    matchExpressions:
      - key: app
        operator: In  # In/NotIn/Exists/DoesNotExist
        values:
          - kubia
  # Pod 模板
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: bingmang/kubia
```

### DaemonSet

```yaml
apiVersion: apps/v1beta2
kind: DaemonSet

metadata:
  name: prod-monitor

spec:
  selector:
    matchLabels:
      app: prod-monitor
  template:
    metadata:
      lebels:
        app: prod-monitor
    spec:
      nodeSelector:
        env: prod
      containers:
      - name: main
        image: bingmang/prod-monitor
```