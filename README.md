# kubernetes
kubernetes 安装配置

## 安装 kubelet kubeadm kubectl docker

通过阿里云源进行安装

```sh
sudo apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add - 
sudo cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl docker.io=18.06.1-0ubuntu1~16.04.2
```

## 通过 kubeadm 部署 kubernetes

拉取镜像

```sh
kubeadm config --kubernetes-version 1.13.2 images pull
```

国内无法拉取镜像可以替换为阿里云

```sh
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.13.2
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.13.2
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.13.2
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.13.2
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.2.24
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.2.6
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.13.2 k8s.gcr.io/kube-controller-manager:v1.13.2
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.13.2 k8s.gcr.io/kube-scheduler:v1.13.2
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.13.2 k8s.gcr.io/kube-proxy:v1.13.2
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.13.2 k8s.gcr.io/kube-apiserver:v1.13.2
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.2.24 k8s.gcr.io/etcd:3.2.24
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.2.2 k8s.gcr.io/coredns:1.2.2
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
```

初始化

```sh
sudo kubeadm init --kubernetes-version=v1.13.2 --pod-network-cidr=10.244.0.0/16
```

安装网络

```sh
kubectl apply -f kube-flannel.yaml
```

## 查看运行状态

查看状态, 如果状态都为running则kubernete部署正常

```sh
kubectl get pods --all-namespaces
```

## 安装 dashboard

安装部署 dashboard

```sh
kubectl apply -f kubernetes-dashboard.yaml
```

添加授权用户

```sh
kubectl apply -f kubernetes-dashboard-rbac.yaml
```

获取用户token

```sh
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

浏览器访问 http://{ip}:30000 , 选择token验证, 输入token。

## 部署监控


检出目录 https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus

```sh
kubectl delete -f manifests/ || true
```

## 其他

### 修改动态端口范围

修改文件 sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml， 添加命令参数: --service-node-port-range=1000-60000

### 设置node label

```sh
kubectl label nodes {nodename} nodetype=storage
```

### Q: 0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```