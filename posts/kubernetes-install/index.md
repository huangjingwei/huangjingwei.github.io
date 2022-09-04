# Kubernetes 安装


## 安装kubeadm三件套

三件套：

- kubeadm：用来初始化集群的指令。
- kubelet：运行在集群节点上的组件。
- kubectl：和集群交互的CLI客户端。

### 官方源安装

```sh
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl #固定其版本
```

参考[官方安装教材法文版]。

### 国内源安装

```sh
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main
EOF

sudo apt-get update
```

添加源之后，执行 apt-get update报错。原因是没有公钥，无法验证签名。添加操作可以执行下面的命令，命令中的`BA07F4FB`为报错信息中提示的缺失公钥的后8位。

```sh
gpg --keyserver keyserver.ubuntu.com --recv-keys  BA07F4FB
gpg --export --armor E084DAB9 | sudo apt-key add -
```

安装

```sh
sudo apt-get install -y kubectl kubeadm kubectl
```

### 命令自动补全

```sh
kubectl completion bash > kubectl
sudo mv kubectl /etc/bash_completion.d/

kubeadm completion bash > kubeadm
sudo mv kubeadm /etc/bash_completion.d/
```

## 禁用swap

如果在安装系统的时候分配了`swap`。需要将其关闭。

```sh
sudo swapoff -a

#查看
free -h
```

## 创建集群

```sh
kubeadm config images pull #pull Google提供的相关镜像。
sudo kubeadm init #初始化并创建Kubernetes集群。
```

如果是国内的网络，镜像拉取会失败。

- 配置代理拉取google镜像：[dockerd代理配置](../docker_proxy/#dockerd代理配置)
- 使用阿里的镜像：

    ```sh
    kubeadm config image pull --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
    ```

## Kuberneres v1.24 适配docker

在Kubernetes v1.24及更早版本中，依然可以通过dockershim来使用Docker Engine。
但是dockershim组件在Kubernetes v1.24发行版本中已被移除。
不过，可以通过配置[容器运行时接口CRI]使用[cri-dockerd]。

### 安装cri-dockerd

```sh
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.2.5/cri-dockerd_0.2.5.3-0.ubuntu-jammy_amd64.deb
sudo dpkg -i cri-dockerd_0.2.5.3-0.ubuntu-jammy_amd64.deb
```

推荐在Release中找到最新版本下载安装。

### 修改kubeadm配置

```sh
$ kubeadm config print init-defaults 

apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: 1.24.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

将以上配置文件写入文件，并修改其内容。

```sh
kubeadm config print init-defaults > init.yaml

# 修改init.yaml:

apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.3.6 #本机ip
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///run/cri-dockerd.sock #改成cri-dockerd
  imagePullPolicy: IfNotPresent
  name: walle #本机hostname
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io # 这里也可以改成阿里的惊喜
kind: ClusterConfiguration
kubernetesVersion: 1.24.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.172.0.0/12
  podSubnet: 10.244.0.0/16 # --pod-network-cidr Specify range of IP addresses for the pod network. If set, the control plane will automatically allocate CIDRs for every node.

scheduler: {}

sudo kubeadm init --config init.yaml 
# 如果在命令后面指定--pod-network-cidr则会报“can not mix '--config' with arguments [pod-network-cidr]“，参考：https://github.com/kubernetes/kubeadm/issues/1899
```

## 配置kubectl客户端

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

配置完，`kubectl`在当前用户就可以用了。

```sh
$ kubectl --namespace=kube-system get pods
NAME                            READY   STATUS              RESTARTS   AGE
coredns-6d4b75cb6d-lvct9        1/1     ContainerCreating   0          31s
coredns-6d4b75cb6d-zpwcc        1/1     ContainerCreating   0          31s
etcd-walle                      1/1     Running             0          31s
kube-apiserver-walle            1/1     Running             0          31s
kube-controller-manager-walle   1/1     Running             0          31s
kube-proxy-rbgzm                1/1     Running             0          31s
kube-scheduler-walle            1/1     Running             0          31s
```

在未配置网络插件（CNI Plugin）时，可以发现coredns是无法启动的。

## 网络插件配置

如果之前安装过其他的插件，需要先清理配置：

```sh
sudo rm -f /etc/cni/net.d/*
```

### 安装flannel

执行命令安装`flannel`：

```sh
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

安装后所有的Pod都正常启动

```sh
$ kubectl --namespace=kube-system get pods
NAME                            READY   STATUS    RESTARTS   AGE
coredns-6d4b75cb6d-lvct9        1/1     Running   0          9h
coredns-6d4b75cb6d-zpwcc        1/1     Running   0          9h
etcd-walle                      1/1     Running   0          9h
kube-apiserver-walle            1/1     Running   0          9h
kube-controller-manager-walle   1/1     Running   0          9h
kube-proxy-rbgzm                1/1     Running   0          9h
kube-scheduler-walle            1/1     Running   0          9h
```

### 检查podCIDR配置

```sh
$ kubectl get node -o yaml | grep CIDR
    podCIDR: 10.244.0.0/24
    podCIDRs:
```

## master 节点默认不能运行 pod

如果用 kubeadm 部署一个单节点集群，默认情况下无法使用，请执行以下命令解除限制

```sh
$ kubectl taint nodes --all node-role.kubernetes.io/master-

# 恢复默认值
# $ kubectl taint nodes NODE_NAME node-role.kubernetes.io/master=true:NoSchedule
```

[官方安装教材法文版]:https://kubernetes.io/fr/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
[容器运行时接口CRI]:https://kubernetes.io/zh-cn/docs/concepts/overview/components/#container-runtime
[cri-dockerd]:https://github.com/Mirantis/cri-dockerd

