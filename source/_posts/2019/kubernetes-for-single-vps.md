
---
title: 穷人也能用得起 K8s - VPS 单节点部署 Kubernetes 的方法与对比
s: kubernetes-for-single-vps
date: 2019-02-22 14:06:44
published: true
tags:    
 - Docker  
 - Kubernetes 
 - Minikube
---

Docker 在生产环境的部署方案，目前的最优解显然是 Kubernetes 了，Kubernetes （下称 K8s）提供了非常完备的功能，几乎能覆盖所有能想到的运维场景，这点无需多言。

唯独对于流量不大，对于机器资源要求比较严苛的小项目，K8s 就显得不那么友好了: 一套[高可用的 K8s 集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/#cluster-diagram)，至少需要 3 个  Master 节点，Worker 节点虽然没有明确要求，但至少 2 个 Worker 节点显然是比较恰当的。也就是说哪怕只是一个静态的 html 页面，K8s 也至少需要 4-5 台主机。

而在实际项目中，往往有一些新想法要试水，或者临时上线一些非常简单的页面，为此单独部署一套 K8s 高可用 5 节点集群显然是浪费的。将其随便塞进一个已经在运行的不相干的集群，也是对运维秩序的扰乱。所以比较折中的做法是牺牲高可用性，基于 1 台低配置的 VPS 或虚拟主机，部署一套 K8s 单节点环境。

当然也可以不用 K8s，直接通过命令行或 docker-compose 操作 Docker 也能实现同样目的。这里非要死磕 K8s 主要基于 3 个原因：

- 其一是统一运维的基础设施，如果同一个运维团队里，同时存在 K8s、Docker Swarm、Shell 脚本等多种运维工具，维护起来难免顾此失彼，容易产生问题，也不利于技术积累。
- 其二是基于 K8s 可以快速水平扩展，特别是一个新想法上线如果短期内获得了不错的反应，流量会有快速的增长，基于 K8s 的配置可以在数分钟内从单机扩展到上百台机器的集群。
- 其三是可以使用成熟的 K8s 部署策略，即便是单节点，仍然可以通过简单配置就实现[滚动更新(RollingUpdate)](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)、回滚等。同时还可以有 K8s dashboard 这样好用的 UI 界面，而其他方案则要自己实现这些，并不划算。

当然单节点部署 K8s 也需要付出代价，由于单节点既充当了 Master 节点，又是 Worker 节点，会有额外的 CPU 和内存消耗。实测 2 核 4G 的机器，裸 K8s 会额外占用掉 0.2~0.5 核左右的 CPU 资源和 600 MB 左右内存资源，建议单节点机器至少也应该有 2 核 4G 及以上规格。

以下以 [Linode 4GB Plan](https://www.linode.com/?r=a33af5735a21b63c784f7cd2cf87dba00fd319a2) 为例，记录单节点 K8s 的部署过程。

## 基于 minikube 在 VPS 快速部署单节点 Kubernetes

[minikube](https://github.com/kubernetes/minikube) 原本是用于在开发环境快速安装 K8s 的工具，由于 Docker 需要系统为 Linux 且内核支持 [LXC](https://linuxcontainers.org/)，因此在 Windows、macOS 下目前都是通过虚拟机来实现 Docker 的安装及运行的。而 Minikube 支持 Windows、macOS、Linux 三种 OS，会根据平台不同，下载对应的虚拟机镜像，并在镜像内安装 k8s。

目前的虚拟机技术都是基于 [Hypervisor](https://en.wikipedia.org/wiki/Hypervisor) 来实现的，Hypervisor 规定了统一的虚拟层接口，由此 Minikube 就可以无缝切换不同的虚拟机实现，如 macOS 可以切换 [hyperkit](https://github.com/moby/hyperkit) 或 VirtualBox， Windows 下可以切换 [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) 或 VirtualBox 等。虚拟机的切换可以通过 `--vm-driver` 实现，如
 
```bash
minikube start --vm-driver hyperkit
minikube start --vm-driver hyperv
```
 
如果 Minikube 安装在内核原生就支持 LXC 的 OS 内，如 Ubuntu 等，再安装一次虚拟机显然就是对资源的浪费了，Minikube 提供了直接对接 OS 底层的方式

```bash
minikube start --vm-driver=none
```

因此在 VPS 上，可以借助 Minikube，快速安装 K8s 环境，同时又最大限度的使用 OS 系统资源。

OS 以 Ubuntu 18.04 为例

#### 1. 安装 Docker Engine

由于 [minikube 限制](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/util/system/docker_validator.go#L41)，必须安装版本不高于 v18.09 的 Docker (截止 2019 年 2 月 19 日）

可以用 `apt-cache policy docker-ce` 查看可以安装的 Docker 版本， 这里我们选择 18.06.2

```bash
apt-get install -y apt-transport-https ca-certificates curl  software-properties-common && \
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - && \
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs)  stable" && \
  apt-get update && apt-get install docker-ce=18.06.2~ce~3-0~ubuntu
```

#### 2. 安装 minikube

```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo cp minikube /usr/local/bin/ && rm minikube

export MINIKUBE_WANTUPDATENOTIFICATION=false
export MINIKUBE_WANTREPORTERRORPROMPT=false
export MINIKUBE_HOME=$HOME
export CHANGE_MINIKUBE_NONE_USER=true
mkdir -p $HOME/.minikube
```

#### 3. 启动 minikube

```bash
minikube start --vm-driver=none
```

如果一切顺利，可以看到 minikube 将下载并安装 kubeadm kubelet 等必要组件，最后按照 minikube 的提示，[安装 kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)


```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

之后就可以通过 `kubectl get pods -n kube-system` 查看容器的运行情况了 


为了方便操作，建议配置 kubectl 的指令补全， 并将命名空间切换至 kube-system， 后文指令都将忽略命名空间 `kube-system`。

```bash
kubectl completion bash >> ~/.bashrc
source ~/.bashrc
kubectl config set-context minikube --namespace=kube-system
```

#### 解决 CoreDNS Forwarding loop 问题

在我的 VPS 上完成上述安装后， 可以看到 CoreDNS 服务并没有完成启动

```plain
# kubectl get pods
NAMESPACE     NAME                               READY   STATUS             RESTARTS   AGE
kube-system   coredns-86c58d9df4-jjhwl           0/1     CrashLoopBackOff   5          3m59s
kube-system   coredns-86c58d9df4-zzj4m           0/1     CrashLoopBackOff   5          3m59s
```

通过查看日志可以看到 CoreDNS 服务检测到了循环 DNS 查询

```plain
# kubectl logs coredns-86c58d9df4-jjhwl
 [FATAL] plugin/loop: Forwarding loop detected in "." zone. Exiting. See https://coredns.io/plugins/loop#troubleshooting. Probe query: "HINFO 6451342721392587444.9147583119172762210.".
```

这一问题在 Ubuntu 上非常容易遭遇，是因为 Ubuntu 使用 systemd-resolved 作为 DNS 解析器，当网络状态变化时，systemd-resolved 有可能将 `127.0.0.53`作为域名服务器写入 `/etc/resolv.conf`，而 CoreDNS 默认又会读取宿主机上 systemd-resolved 所使用的文件，造成死循环。

解决这个问题的一种方法是在启动 minikube 时直接指定 `resolv.conf` 文件位置， 如

```bash
minikube --vm-driver=none start --extra-config=kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
```

如果启动 minikube 后产生了这个问题，则可以删除`resolv.conf`软链接， 自己生成一个`resolv.conf`，当 systemd-resolved 检测到 `/etc/resolv.conf` 不是软链接后，就不会再覆盖这个文件。

```bash
rm /etc/resolv.conf && echo "nameserver 114.114.114.114 > /etc/resolv.conf"
kubectl delete pods -n kube-system --selector k8s-app=kube-dns
```

也可以将 `/etc/resolv.conf` 指向主机商提供的 DNS 解析文件。

```bash
rm /etc/resolv.conf && cp /run/systemd/resolve/resolv.conf /etc/resolv.conf
kubectl delete pods -n kube-system --selector k8s-app=kube-dns
```

#### minikube 不使用虚拟机的注意点

由于 minikube 主要设计是基于虚拟机安装的，[`vm-driver=none` 也会带来一些影响](https://github.com/kubernetes/minikube/blob/master/docs/vmdriver-none.md), 主要包括

- minikube 在安装时将覆盖主机原有的 `/usr/local/bin/kubeadm`, `/usr/local/bin/kubectl`, `/etc/kubernetes` 等目录
- 无法在一台主机上使用 `vm-driver=none` 安装 2 个 minikube
- `minikube dashboard`, `minikube mount`, `minikube ssh` 等基于虚拟机的指令均无法使用
- `minikube delete` 将删除 `/etc/kubernetes` 目录

为了避免问题，建议使用全新的主机安装，使用完后直接回收主机即可。

##  基于 microk8s 在 VPS 快速部署单节点 Kubernetes

minikube 虽然是官方出品，但主要还是基于虚拟机做的设计。在 Linux 生产环境下，[microk8s](https://microk8s.io/) 可能是一个更合适的选择。

microk8s 基于 [snap](https://snapcraft.io/) 进行安装，ubuntu 16.04 及之后的版本都已经预装了 snap，如果是其他发行版的 Linux 需要先[安装 snap](https://docs.snapcraft.io/installing-snapd/6735)。 安装前 microk8s 可以通过

```plain
snap info microk8s
```

了解可安装版本和操作指令等信息。

microk8s 的独特之处在于直接包含了 Kubernetes 单节点必须的如 `kubectl`, `kubelet` 等所有工具，甚至直接包含了 docker。因此在一台裸机上运行

```plain
snap install microk8s --classic
```

之后，就可以在进程里看到 `/snap/microk8s` 为前缀的一系列熟悉的进程， 并且这些进程直接运行在主机，并未使用容器，k8s 也没有使用默认的端口 `6443`，而改用`8080`。这样做带来的一个好处是即便系统之前已经安装了`kubectl`, `docker`等，也可以进行安装，不会冲突；不过就不能用 `kubectl`、 `docker`直接进行操作， 而需要使用 `microk8s.docker`, `microk8s.kubectl` 等， 与原来的指令是完全一样的。

安装完成后可以通过 `microk8s.status` 查看运行状态及插件的开启情况。也可以用 `microk8s.inspect` 查看已经安装的服务。

通常来说安装了 microk8s 后，我们也不会再次安装 `kubectl`， 为了操作方面，可以为 `microk8s.kubectl` 开启设置`kubectl`为别名， 并开启输入补全。

```plain
snap alias microk8s.kubectl kubectl
source <(microk8s.kubectl completion bash)
```

也可以随时取消别名。

```plain
snap unalias kubectl
```

和 minikube 相同，microk8s 默认只安装最核心的功能，可以通过 `microk8s.enable dns dashboard` 等开启附加的插件。

## 使用 kubeadm 部署单节点 Kubernetes

通过 minikube 或 microk8s 安装 K8s 虽然方便，但是由于很多安装细节被屏蔽，直接用在生产环境也难免让人心存疑虑，那么也可以[使用 kubeadm 部署单节点 kubernetes](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)。

同样以 Ubuntu18.04 为例， 首先[安装 Docker v18.06.2 后](https://docs.docker.com/install/linux/docker-ce/ubuntu/) （K8s 同样对 Docker 版本有要求），参考官方文档， 可以使用 `apt-get` 直接安装 `kubelet`，`kubeadm`, `kubectl`， 并且限制这些服务不能自动升级。

```bash
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

接下来关闭 swap，这应该是为了[避免 swap 对 CPU 和内存的 limit 造成意外影响](https://github.com/kubernetes/kubernetes/issues/53533)。

```bash
swapoff -a
```

然后运行 

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16
```

就会开始 Kubernetes master 节点的安装。安装完成后 CoreDNS 是处于 `Pending` 状态的，这是由于网络插件尚未安装，Kubernetes 使用 [CNI](https://github.com/containernetworking/cni) 作为容器网络的接口，因此可以根据主机实际情况选择不同的 CNI 实现，这里以比较流行的 [flannel](https://github.com/coreos/flannel) 为例。

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
```

之后可以看到 CoreDNS 的状态从 `Pending` 变为 `Running`。至此 K8s 的 master 节点所需服务就已经安装完毕。可以参考安装后的提示，配置 `kubectl` 

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

现在尝试部署一个 pod

```bash
kubectl run --rm --restart=Never -it --image=hello-world test-pod
```

发现 pod 状态卡在 `Pending`，查看 pod 信息看到如下报错

```bash
$ kubectl describe pod test-pod
0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
```

这是由于 K8s 默认禁止在 master 节点部署 pod，可以使用以下指令取消这一限制。

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

之后可以看到 pod 状态已经变为 `Completed`

##  Kubernetes 单节点部署方式对比

至此我们分别用 minikube、microk8s、kubeadm 3 种方式完成了单节点 k8s 的部署。从安装的服务来看，k8s 单节点必要的服务包括: 

- 容器运行时: 默认是 Docker
- etcd: key-value 存储服务，用于保存集群的状态
- kube-apiserver: 集群资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制
- kube-controller-manager: 维护集群的状态，比如故障检测、自动扩展、滚动更新等
- kube-scheduler: 负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上
- kubelet: 负责维持容器的生命周期，同时也负责 Volume（CVI）和网络（CNI）的管理
- kube-proxy: 负责为 Service 提供 cluster 内部的服务发现和负载均衡

三者差异之处在于:

- microk8s 部署 k8s 是直接安装到主机而不是以容器方式安装
- microk8s 虽然默认没有安装 DNS 服务，但通常情况下为了使用服务发现，还是需要安装的，microk8s 默认的 DNS 服务是 kube-dns，而不是目前官方推荐的 CoreDNS 。
- 网络插件 kubeadm 需要自己选择， microk8s 和 minikube 都会自己安装
- minikube 默认额外安装了 `storage-provisioner` 用于虚拟机挂载磁盘

从易用角度来看，microk8s 是安装最简单，门槛最低的；minikube 适合对 minikube 比较熟悉的用户。

无论以何种方式安装 k8s， 都需要注意安全问题， 因为在 k8s 的设计中， Master 节点是不会暴露到外网的，用户服务都会安装到 Worker 节点，但是在单节点的情况下，k8s 所监听的端口都没有设防，容器的权限也有可能过大，这些[安全问题在 minikube 的文档中也有提到](https://github.com/kubernetes/minikube/blob/master/docs/vmdriver-none.md#can-the-none-driver-be-used-outside-of-a-vm)， 需要对网络端口设置 iptables 限制可访问的 IP 等方式来提升安全性，如果是安全性敏感的项目，建议放弃单节点 k8s 的方案。

References:

- [Kubernetes 架构原理](https://kubernetes.feisky.xyz/he-xin-yuan-li/architecture)
- [Kubernetes 网络相关总结](http://codemacro.com/2018/04/01/kube-network/)
