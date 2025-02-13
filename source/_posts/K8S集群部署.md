---
title: K8S集群部署
categories:
  - 技术
  - K8S
tags:
  - K8S
  - Kubernetes
abbrlink: 92b6432e
date: 2023-11-06 18:05:48
---

Kubeadm 是一个K8s 部署工具，提供kubeadm init 和kubeadm join，用于快速部署Kubernetes 集群。

<!--more-->

##  完整的卸载k8s（有需要的话）

```bash
# 首先清理运行到k8s群集中的pod，使用
kubectl delete node --all

# 使用脚本停止所有k8s服务
for service in kube-apiserver kube-controller-manager kubectl kubelet etcd kube-proxy kube-scheduler; 
do
    systemctl stop $service
done

# 使用命令卸载k8s
kubeadm reset -f

# 卸载k8s相关程序
yum -y remove kube*

# 删除相关的配置文件
modprobe -r ipip
lsmod

# 然后手动删除配置文件和flannel网络配置和flannel网口：
rm -rf /etc/cni
rm -rf /root/.kube
# 删除cni网络
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1

# 删除残留的配置文件
rm -rf ~/.kube/
rm -rf /etc/kubernetes/
rm -rf /etc/systemd/system/kubelet.service.d
rm -rf /etc/systemd/system/kubelet.service
rm -rf /etc/systemd/system/multi-user.target.wants/kubelet.service
rm -rf /var/lib/kubelet
rm -rf /usr/libexec/kubernetes/kubelet-plugins
rm -rf /usr/bin/kube*
rm -rf /opt/cni
rm -rf /var/lib/etcd
rm -rf /var/etcd

# 更新镜像
yum clean all
yum makecache
```

------

## 安装kube集群（4节点）

k8s重置命令（如果初始化的过程出现了错误就使用重置命令）：`kubeadm reset`

### 准备工作（所有的节点都执行）

编辑4台服务器的 `/etc/hosts` 文件 ,添加下面内容（每个节点都执行一遍）：

```text
192.168.2.1 node1
192.168.2.2 node2
192.168.2.3 node3
192.168.2.4 node4
```

设置hostname（以node1为例）：

```bash
hostnamectl set-hostname  node1  # node1 是自定义名字
```

或者修改 `/etc/hostname` 文件，写入`node1`（其他的子节点都一样）：

```bash
vim /etc/hostname
```

修改之后`/etc/hostname`的内容为：

```bash
node1
```

所有节点执行时间同步：

```bash
# 启动chronyd服务
systemctl start chronyd
systemctl enable chronyd
date
```

所有节点禁用SELinux和Firewalld服务：

```bash
systemctl stop firewalld
systemctl disable firewalld

sed -i 's/enforcing/disabled/' /etc/selinux/config # 重启后生效
```

所有节点禁用swap分区：

```bash
# 临时禁用swap分区
swapoff -a

# 永久禁用swap分区
vi /etc/fstab 
# 注释掉下面的设置
# /dev/mapper/centos-swap swap
# 之后需要重启服务器生效
```

所有节点添加网桥过滤和地址转发功能：

```bash
cat > /etc/sysctl.d/kubernetes.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# 然后执行,生效
sysctl --system
```

然后所有节点安装docker-ce（略）

需要注意的是要配置docker的cgroupdriver：

打开docker配置文件进行修改，该文件默认情况下不存在，可以新建一个，不同操作系统的文件位置不一样。

```bash
vim /etc/docker/daemon.json
```

```json
{
  // 添加这行
  "exec-opts": ["native.cgroupdriver=systemd"],
}
```

重新加载配置并重启服务：

```bash
systemctl daemon-reload
systemctl restart docker
```

最终配置文件内容如下：

```json
{
        "exec-opts": [
                        "native.cgroupdriver=systemd"
        ]
}
```



所有节点的kubernetes镜像切换成国内源：

```bash
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

所有节点安装指定版本 kubeadm，kubelet 和 kubectl（我这里选择`1.23.0`版本的）：

```bash
yum install -y kubelet-1.23.0 kubeadm-1.23.0 kubectl-1.23.0

# 设置kubelet开机启动（看你自己）
systemctl enable kubelet
```

### 更改kubelet的容器路径（如果需要的话，不需要可以跳过）

```bash
vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
```

修改完之后配置文件如下：

```bash
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --root-dir=/mnt/sdb_new/kubelet/ --kubeconfig=/etc/kubernetes/kubelet.conf"
```

使配置生效：

```bash
systemctl daemon-reload
systemctl restart docker
systemctl restart kubelet
```

## 部署Kubernetes集群

### 覆盖kubernetes的镜像地址（只需要在master节点上操作初始化命令）

1. 首先要覆盖kubeadm的镜像地址，因为这个是外网的无法访问，需要替换成国内的镜像地址，使用此命令列出集群在配置过程中需要哪些镜像：

```bash
[root@node1 home]# kubeadm config images list
I0418 18:26:04.047449   19242 version.go:255] remote version is much newer: v1.27.1; falling back to: stable-1.23
k8s.gcr.io/kube-apiserver:v1.23.17
k8s.gcr.io/kube-controller-manager:v1.23.17
k8s.gcr.io/kube-scheduler:v1.23.17
k8s.gcr.io/kube-proxy:v1.23.17
k8s.gcr.io/pause:3.6
k8s.gcr.io/etcd:3.5.1-0
k8s.gcr.io/coredns/coredns:v1.8.6
```

2. 更改为阿里云的镜像地址：

```bash
[root@node1 home]# kubeadm config images list  --image-repository registry.aliyuncs.com/google_containers
I0418 18:28:18.740057   20021 version.go:255] remote version is much newer: v1.27.1; falling back to: stable-1.23
registry.aliyuncs.com/google_containers/kube-apiserver:v1.23.17
registry.aliyuncs.com/google_containers/kube-controller-manager:v1.23.17
registry.aliyuncs.com/google_containers/kube-scheduler:v1.23.17
registry.aliyuncs.com/google_containers/kube-proxy:v1.23.17
registry.aliyuncs.com/google_containers/pause:3.6
registry.aliyuncs.com/google_containers/etcd:3.5.1-0
registry.aliyuncs.com/google_containers/coredns:v1.8.6
```

3. 然后将镜像手动拉取下来，这样在初始化的时候回更快一些（还有一个办法就是直接在docker上把镜像pull下来，docker只要配置一下国内源即可快速的将镜像pull下来）：

```bash
[root@node1 home]# kubeadm config images pull  --image-repository registry.aliyuncs.com/google_containers
I0418 18:28:31.795554   20088 version.go:255] remote version is much newer: v1.27.1; falling back to: stable-1.23
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-apiserver:v1.23.17
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-controller-manager:v1.23.17
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-scheduler:v1.23.17
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-proxy:v1.23.17
[config/images] Pulled registry.aliyuncs.com/google_containers/pause:3.6
[config/images] Pulled registry.aliyuncs.com/google_containers/etcd:3.5.1-0
[config/images] Pulled registry.aliyuncs.com/google_containers/coredns:v1.8.6
```

### 初始化kubernetes（只需要在master节点上操作初始化命令）

初始化 Kubernetes，指定网络地址段 和 镜像地址（后续的子节点可以使用join命令进行动态的追加）：

```bash
[root@node1 home]# kubeadm init \
  --apiserver-advertise-address=192.168.2.1 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.23.0 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --ignore-preflight-errors=all

# –apiserver-advertise-address # 集群通告地址(master 机器IP，这里用的万兆网)
# –image-repository # 由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址
# –kubernetes-version #K8s版本，与上面安装的一致
# –service-cidr #集群内部虚拟网络，Pod统一访问入口，可以不用更改，直接用上面的参数
# –pod-network-cidr #Pod网络，与下面部署的CNI网络组件yaml中保持一致，可以不用更改，直接用上面的参数
```

执行完之后要手动执行一些参数（尤其是 **加入集群的join命令** 需要复制记录下载）：

```bash
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.2.1:6443 --token ochspx.15in9qkiu5z8tx2y \
        --discovery-token-ca-cert-hash sha256:1f31202107af96a07df9fd78c3aa9bb44fd40076ac123e8ff28d6ab691a02a31
```

执行参数：

```bash
[root@node1 home]# mkdir -p $HOME/.kube
[root@node1 home]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@node1 home]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
[root@node1 home]# 
[root@node1 home]# vim /root/.bash_profile
```

加入以下这段：

```bash
# 超级用户变量
export KUBECONFIG=/etc/kubernetes/admin.conf
# 设置别名
alias k=kubectl
# 设置kubectl命令补齐功能
source <(kubectl completion bash)
```

激活 `.bash_profile`：

```bash
[root@node1 home]# source /root/.bash_profile
```

这段要**复制记录**下来（来自k8s初始化成功之后出现的`join`命令，需要先配置完Flannel才能加入子节点），后续子节点加入master节点需要执行这段命令：

```bash
kubeadm join 192.168.2.1:6443 --token ochspx.15in9qkiu5z8tx2y \
        --discovery-token-ca-cert-hash sha256:1f31202107af96a07df9fd78c3aa9bb44fd40076ac123e8ff28d6ab691a02a31
```

### 设定kubeletl网络（主节点部署）

部署容器网络，CNI网络插件(在Master上执行，著名的有flannel、calico、canal和kube-router等，简单易用的实现是为CoreOS提供的flannel项目)，这里使用Flannel实现。

下载`kube-flannel.yml`：

```bash
[root@node1 home]# wget https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

然后修改配置文件，找到如下位置，修改 `Newwork` 与执行 `kubeadm init` 输入的网段一致：

```bash
net-conf.json: |
  {
    "Network": "10.244.0.0/16",
    "Backend"": {
      "Type": "vxlan"
  }
}
```

修改配置之后安装组件（如果安装的时候卡在pull镜像的时候，试一试手动用docker将镜像拉取下来）：

```bash
[root@node1 home]# kubectl apply -f kube-flannel.yml
```

查看`flannel pod`状态（必须要为`Running`状态，如果`kube-flannel`起不来，那么就用`kubectl describe pod kube-flannel-ds-f5jn6 -n kube-flannel`命令查看`pod`起不来的原因，然后去搜度娘获取解决方案）：

```bash
[root@node1 home]# # 必须所有的容器都是Running
[root@node1 home]# kubectl get pod --all-namespaces
NAMESPACE      NAME                                 READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-f5jn6                1/1     Running   0          8m21s
kube-system    coredns-6d8c4cb4d-ctqw5              1/1     Running   0          42m
kube-system    coredns-6d8c4cb4d-n52fq              1/1     Running   0          42m
kube-system    etcd-k8s-master                      1/1     Running   0          42m
kube-system    kube-apiserver-k8s-master            1/1     Running   0          42m
kube-system    kube-controller-manager-k8s-master   1/1     Running   0          42m
kube-system    kube-proxy-swpkz                     1/1     Running   0          42m
kube-system    kube-scheduler-k8s-master            1/1     Running   0          42m
```

查看通信状态：

```bash
[root@node1 home]# kubectl get pod -n kube-system
NAME                                 READY   STATUS    RESTARTS   AGE
coredns-6d8c4cb4d-ctqw5              1/1     Running   0          52m
coredns-6d8c4cb4d-n52fq              1/1     Running   0          52m
etcd-k8s-master                      1/1     Running   0          53m
kube-apiserver-k8s-master            1/1     Running   0          53m
kube-controller-manager-k8s-master   1/1     Running   0          53m
kube-proxy-swpkz                     1/1     Running   0          52m
kube-scheduler-k8s-master            1/1     Running   0          53m
[root@node1 home]# 
[root@node1 home]# 获取主节点的状态
[root@node1 home]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true","reason":""}
[root@node1 home]# kubectl get node
NAME         STATUS   ROLES                  AGE   VERSION
node1        Ready    control-plane,master   52m   v1.23.0
```

查看节点状态（此时还只有主节点，还没添加子节点）：

```bash
[root@node1 home]# kubectl get node
NAME         STATUS   ROLES                  AGE   VERSION
node1        Ready    control-plane,master   53m   v1.23.0
```

**至此 K8s master主服务器 已经部署完成！**

### 子节点加入集群（在子节点上操作）

初始化会生成`join`命令，需要在**子节点**执行即可，以下`token`作为举例，以实际为主，例如：

```bash
[root@node2 home]# kubeadm join 192.168.2.1:6443 --token ochspx.15in9qkiu5z8tx2y         --discovery-token-ca-cert-hash sha256:1f31202107af96a07df9fd78c3aa9bb44fd40076ac123e8ff28d6ab691a02a31
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

默认的 `join token` 有效期限为24小时，当过期后该 `token` 就不能用了，这时需要重新创建 `token`，创建新的`join token`需要在主节点上创建，创建命令如下：

```bash
[root@node1 home]# kubeadm token create --print-join-command
```

加入之后再在主节点查看集群中节点的状态（**必须要都为`Ready`状态**）：

```bash
[root@node1 home]# kubectl get nodes
NAME         STATUS     ROLES                  AGE     VERSION
node1        Ready      control-plane,master   63m     v1.23.0
node2        Ready      <none>                 3m57s   v1.23.0
node3        Ready      <none>                 29s     v1.23.0
```

**如果所有的节点`STATUS`都为`Ready`的话，那么到此，所有的子节点加入完成！**

### 删除子节点（在master主节点上操作）

```bash
# kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
# 其中 <node name> 是在k8s集群中使用 <kubectl get nodes> 查询到的节点名称
# 假设这里删除 node3 子节点
[root@node1 home]# kubectl drain node3 --delete-local-data --force --ignore-daemonsets
[root@node1 home]# kubectl delete node node3
```

然后在删除的子节点上操作重置k8s（重置k8s会删除一些配置文件），这里在`node3`子节点上操作：

```bash
[root@node3 home]# # 子节点重置k8s
[root@node3 home]# kubeadm reset
[reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] Are you sure you want to proceed? [y/N]: y
[preflight] Running pre-flight checks
W0425 01:59:40.412616   15604 removeetcdmember.go:80] [reset] No kubeadm config, using etcd pod spec to get data directory
[reset] No etcd config found. Assuming external etcd
[reset] Please, manually reset etcd to prevent further issues
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
[reset] Deleting contents of stateful directories: [/var/lib/kubelet /var/lib/dockershim /var/run/kubernetes /var/lib/cni]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
```

然后在被删除的子节点上手动删除k8s配置文件、flannel网络配置文件 和 flannel网口：

```bash
[root@node3 home]# rm -rf /etc/cni/net.d/
[root@node3 home]# rm -rf /root/.kube/config
[root@node3 home]# # 删除cni网络
[root@node3 home]# ifconfig cni0 down
[root@node3 home]# ip link delete cni0
[root@node3 home]# ifconfig flannel.1 down
[root@node3 home]# ip link delete flannel.1
```

------

## 部署k8s dashboard（这里使用Kubepi）

**Kubepi**是一个简单高效的k8s集群图形化管理工具，方便日常管理K8S集群，高效快速的查询日志定位问题的工具

部署KubePI（随便在哪个节点部署，我这里在主节点部署）：

```bash
[root@node1 home]# docker pull kubeoperator/kubepi-server
[root@node1 home]# # 运行容器
[root@node1 home]# docker run --privileged -itd --restart=unless-stopped --name kube_dashboard -v /home/docker-mount/kubepi/:/var/lib/kubepi/ -p 8000:80 kubeoperator/kubepi-server
```

登录：

```bash
# 地址: http://192.168.2.1:8000
# 默认用户名：admin
# 默认密码：kubepi
```

填写集群名称，默认认证模式，填写`apisever`地址及`token`：

![](K8S集群部署/image-20231106182459560.png)

kubepi导入集群:

获取登录需要用到的ip地址和登录token：

```bash
[root@node1 home]# # 在 k8s 主节点上创建用户，并获取token
[root@node1 home]# kubectl create sa kubepi-user --namespace kube-system
serviceaccount/kubepi-user created
[root@node1 home]# kubectl create clusterrolebinding kubepi-user --clusterrole=cluster-admin --serviceaccount=kube-system:kubepi-user
clusterrolebinding.rbac.authorization.k8s.io/kubepi-user created
[root@node1 home]# 
[root@node1 home]# # 在主节点上获取新建的用户 kubeapi-user 的 token
[root@node1 home]# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kubepi-user | awk '{print $1}') | grep token: | awk '{print $2}'
eyJhbGciOiJSUzI1NiIsImtpZCI6IkhVeUtyc1BpU1JvRnVacXVqVk1PTFRkaUlIZm1KQTV6Wk9WSExSRllmd0kifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcGktdXNlci10b2tlbi10cjVsMiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcGktdXNlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjJiYzlhZDRjLWVjZTItNDE2Mi04MDc1LTA2NTI0NDg0MzExZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcGktdXNlciJ9.QxkR1jBboqTYiVUUVO4yGhfWmlLDA5wHLo_ZnjAuSLZQDyVevCgBluL6l7y7UryRdId6FmBZ-L0QitvOuTsurcjGL2QHxPE_yZsNW7s9K7eikxJ8q-Q_yOvnADtAueH_tcMGRGW9Zyec2TlmcGTZCNaNUme84TfMlWqX7oP3GGJGMbMGN7H4fPXh-Qqrdp-0MJ3tP-dk3koZUEu3amrq8ExSmjIAjso_otrgFWbdSOMkCXKsqb9yuZzaw7u5Cy18bH_HW6RbNCRT5jGs5aOwzuMAd0HQ5iNm-5OISI4Da6jGdjipLXejcC1H-xWgLlJBx0RQWu41yoPNF57cG1NubQ
[root@node1 home]# 
[root@node1 home]# # 在主节点上获取 apiserver 地址
[root@node1 home]# cat ~/.kube/config | grep server: | awk '{print $2}'
https://192.168.2.1:6443
```

将上面获取的`api地址`和`token`填入页面即可，name可以自己随意取。

![](K8S集群部署/image-20231106182537167.png)

**到此，KubePI安装完成！**

## 安装metrics k8s集群监控插件

k8s metrics插件提供了 `top` 命令可用于统计 k8s集群资源 的使用情况，它包含有 `node` 和 `pod` 两个⼦命令，分别显⽰ `node` 节点和 `Pod` 对象的资源使⽤信息。

kubectl top 命令依赖于 metrics 接口。k8s 系统默认未安装该接口，需要单独部署：

```text
[root@k8s-master k8s-install]# kubectl top pod
error: Metrics API not available
```

### 下载部署文件

下载 metrics 接口的部署文件 metrics-server-components.yaml

```text
[root@k8s-master k8s-install]# wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml -O metrics-server-components.yaml
--2022-10-11 00:13:01--  https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
正在解析主机 github.com (github.com)... 20.205.243.166
正在连接 github.com (github.com)|20.205.243.166|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 302 Found
位置：https://github.com/kubernetes-sigs/metrics-server/releases/download/metrics-server-helm-chart-3.8.2/components.yaml [跟随至新的 URL]
--2022-10-11 00:13:01--  https://github.com/kubernetes-sigs/metrics-server/releases/download/metrics-server-helm-chart-3.8.2/components.yaml
再次使用存在的到 github.com:443 的连接。
已发出 HTTP 请求，正在等待回应... 302 Found
位置：https://objects.githubusercontent.com/github-production-release-asset-2e65be/92132038/d85e100a-2404-4c5e-b6a9-f3814ad4e6e5?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20221010%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20221010T161303Z&X-Amz-Expires=300&X-Amz-Signature=efa1ff5dd16b6cd86b6186adb3b4c72afed8197bdf08e2bffcd71b9118137831&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=92132038&response-content-disposition=attachment%3B%20filename%3Dcomponents.yaml&response-content-type=application%2Foctet-stream [跟随至新的 URL]
--2022-10-11 00:13:02--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/92132038/d85e100a-2404-4c5e-b6a9-f3814ad4e6e5?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20221010%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20221010T161303Z&X-Amz-Expires=300&X-Amz-Signature=efa1ff5dd16b6cd86b6186adb3b4c72afed8197bdf08e2bffcd71b9118137831&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=92132038&response-content-disposition=attachment%3B%20filename%3Dcomponents.yaml&response-content-type=application%2Foctet-stream
正在解析主机 objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.109.133, 185.199.111.133, 185.199.108.133, ...
正在连接 objects.githubusercontent.com (objects.githubusercontent.com)|185.199.109.133|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：4181 (4.1K) [application/octet-stream]
正在保存至: “metrics-server-components.yaml”

100%[============================================================================================================================>] 4,181       --.-K/s 用时 0.01s   

2022-10-11 00:13:10 (385 KB/s) - 已保存 “metrics-server-components.yaml” [4181/4181])
```

### 修改镜像地址

将部署文件中镜像地址修改为国内的地址。大概在部署文件的第 140 行。

原配置是：

```text
image: k8s.gcr.io/metrics-server/metrics-server:v0.6.1
```

修改后的配置是：

```text
image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server:v0.6.1
```

可使用如下命令实现修改：

```text
sed -i 's/k8s.gcr.io\/metrics-server/registry.cn-hangzhou.aliyuncs.com\/google_containers/g' metrics-server-components.yaml
```

### 部署 metrics 接口

```text
[root@k8s-master k8s-install]# kubectl create -f metrics-server-components.yaml 
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```

查看该 metric pod 的运行情况：

```text
[root@k8s-master k8s-install]# kubectl get pods --all-namespaces | grep metrics
kube-system   metrics-server-6ffc8966f5-84hbb      0/1     Running   0              2m23s
```

查看该 pod 的情况，发现是探针问题：Readiness probe failed: HTTP probe failed with statuscode: 500

```text
[root@k8s-master k8s-install]# kubectl describe pod metrics-server-6ffc8966f5-84hbb -n kube-system
Name:                 metrics-server-6ffc8966f5-84hbb
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 k8s-slave2/192.168.100.22
Start Time:           Tue, 11 Oct 2022 00:27:33 +0800
Labels:               k8s-app=metrics-server
                      pod-template-hash=6ffc8966f5
Annotations:          <none>
Status:               Running
IP:                   10.244.2.9
IPs:
  IP:           10.244.2.9
Controlled By:  ReplicaSet/metrics-server-6ffc8966f5
Containers:
  metrics-server:
    Container ID:  docker://e913a075e0381b98eabfb6e298f308ef69dfbd7c672bdcfb75bb2ff3e4b5a0a4
    Image:         registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server:v0.6.1
    Image ID:      docker-pullable://registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server@sha256:5ddc6458eb95f5c70bd13fdab90cbd7d6ad1066e5b528ad1dcb28b76c5fb2f00
    Port:          4443/TCP
    Host Port:     0/TCP
    Args:
      --cert-dir=/tmp
      --secure-port=4443
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      --kubelet-use-node-status-port
      --metric-resolution=15s
    State:          Running
      Started:      Tue, 11 Oct 2022 00:27:45 +0800
    Ready:          False
    Restart Count:  0
    Requests:
      cpu:        100m
      memory:     200Mi
    Liveness:     http-get https://:https/livez delay=0s timeout=1s period=10s #success=1 #failure=3
    Readiness:    http-get https://:https/readyz delay=20s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /tmp from tmp-dir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-x2spb (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  tmp-dir:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-x2spb:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              kubernetes.io/os=linux
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                     From               Message
  ----     ------     ----                    ----               -------
  Normal   Scheduled  7m27s                   default-scheduler  Successfully assigned kube-system/metrics-server-6ffc8966f5-84hbb to k8s-slave2
  Normal   Pulling    7m26s                   kubelet            Pulling image "registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server:v0.6.1"
  Normal   Pulled     7m15s                   kubelet            Successfully pulled image "registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server:v0.6.1" in 10.976606194s
  Normal   Created    7m15s                   kubelet            Created container metrics-server
  Normal   Started    7m15s                   kubelet            Started container metrics-server
  Warning  Unhealthy  2m17s (x31 over 6m47s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 500

进而查看 pod 的日志：
[root@k8s-master k8s-install]# kubectl logs metrics-server-6ffc8966f5-84hbb -n kube-system 
I1010 16:27:46.228594       1 serving.go:342] Generated self-signed cert (/tmp/apiserver.crt, /tmp/apiserver.key)
I1010 16:27:46.633494       1 secure_serving.go:266] Serving securely on [::]:4443
I1010 16:27:46.633585       1 requestheader_controller.go:169] Starting RequestHeaderAuthRequestController
I1010 16:27:46.633616       1 shared_informer.go:240] Waiting for caches to sync for RequestHeaderAuthRequestController
I1010 16:27:46.633653       1 dynamic_serving_content.go:131] "Starting controller" name="serving-cert::/tmp/apiserver.crt::/tmp/apiserver.key"
I1010 16:27:46.634221       1 tlsconfig.go:240] "Starting DynamicServingCertificateController"
W1010 16:27:46.634296       1 shared_informer.go:372] The sharedIndexInformer has started, run more than once is not allowed
I1010 16:27:46.634365       1 configmap_cafile_content.go:201] "Starting controller" name="client-ca::kube-system::extension-apiserver-authentication::requestheader-client-ca-file"
I1010 16:27:46.634370       1 shared_informer.go:240] Waiting for caches to sync for client-ca::kube-system::extension-apiserver-authentication::requestheader-client-ca-file
I1010 16:27:46.634409       1 configmap_cafile_content.go:201] "Starting controller" name="client-ca::kube-system::extension-apiserver-authentication::client-ca-file"
I1010 16:27:46.634415       1 shared_informer.go:240] Waiting for caches to sync for client-ca::kube-system::extension-apiserver-authentication::client-ca-file
E1010 16:27:46.641663       1 scraper.go:140] "Failed to scrape node" err="Get \"https://192.168.100.22:10250/metrics/resource\": x509: cannot validate certificate for 192.168.100.22 because it doesn't contain any IP SANs" node="k8s-slave2"
E1010 16:27:46.645389       1 scraper.go:140] "Failed to scrape node" err="Get \"https://192.168.100.20:10250/metrics/resource\": x509: cannot validate certificate for 192.168.100.20 because it doesn't contain any IP SANs" node="k8s-master"
E1010 16:27:46.652261       1 scraper.go:140] "Failed to scrape node" err="Get \"https://192.168.100.21:10250/metrics/resource\": x509: cannot validate certificate for 192.168.100.21 because it doesn't contain any IP SANs" node="k8s-slave1"
I1010 16:27:46.733747       1 shared_informer.go:247] Caches are synced for RequestHeaderAuthRequestController 
I1010 16:27:46.735167       1 shared_informer.go:247] Caches are synced for client-ca::kube-system::extension-apiserver-authentication::client-ca-file 
I1010 16:27:46.735194       1 shared_informer.go:247] Caches are synced for client-ca::kube-system::extension-apiserver-authentication::requestheader-client-ca-file 
E1010 16:28:01.643646       1 scraper.go:140] "Failed to scrape node" err="Get \"https://192.168.100.22:10250/metrics/resource\": x509: cannot validate certificate for 192.168.100.22 because it doesn't contain any IP SANs" node="k8s-slave2"
E1010 16:28:01.643805       1 scraper.go:140] "Failed to scrape node" err="Get \"https://192.168.100.21:10250/metrics/resource\": x509: cannot validate certificate for 192.168.100.21 because it doesn't contain any IP SANs" node="k8s-slave1"
E1010 16:28:01.646721       1 scraper.go:140] "Failed to scrape node" err="Get \"https://192.168.100.20:10250/metrics/resource\": x509: cannot validate certificate for 192.168.100.20 because it doesn't contain any IP SANs" node="k8s-master"
I1010 16:28:13.397373       1 server.go:187] "Failed probe" probe="metric-storage-ready" err="no metrics to serve"
```

基本可以确定 pod 异常是因为：Readiness Probe 探针检测到 Metris 容器启动后对 http Get 探针存活没反应，具体原因是：cannot validate certificate for 192.168.100.22 because it doesn't contain any IP SANs" node="k8s-slave2"

查看 metrics-server 的文档（[https://github.com/kubernetes...](https://link.zhihu.com/?target=https%3A//link.segmentfault.com/%3Fenc%3DLHp8vLvsXQ8%2Byt1%2FzV9Stg%3D%3D.brVN9B7OHIL7n9%2Fn8A4cIZe2cEyp%2BZMhJKHHxr%2FREeY5Tqsd8Ap7rZUwRhb47K%2Fw1RJybc8%2BCUL06q%2FyPx4BJg%3D%3D)），有如下一段说明：

```text
Kubelet certificate needs to be signed by cluster Certificate Authority (or disable certificate validation by passing 
--kubelet-insecure-tls to Metrics Server)
```

意思是：kubelet 证书需要由集群证书颁发机构签名(或者通过向 Metrics Server 传递参数 --kubelet-insecure-tls 来禁用证书验证)。
由于是测试环境，我们选择使用参数`禁用证书验证，生产环境不推荐这样做！！！`

在大概 139 行的位置追加参数：

```text
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
```

apply 部署文件：

```text
[root@k8s-master k8s-install]# kubectl apply -f metrics-server-components.yaml
Warning: resource serviceaccounts/metrics-server is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
serviceaccount/metrics-server configured
Warning: resource clusterroles/system:aggregated-metrics-reader is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader configured
Warning: resource clusterroles/system:metrics-server is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
clusterrole.rbac.authorization.k8s.io/system:metrics-server configured
Warning: resource rolebindings/metrics-server-auth-reader is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader configured
Warning: resource clusterrolebindings/metrics-server:system:auth-delegator is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator configured
Warning: resource clusterrolebindings/system:metrics-server is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server configured
Warning: resource services/metrics-server is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
service/metrics-server configured
Warning: resource deployments/metrics-server is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/metrics-server configured
Warning: resource apiservices/v1beta1.metrics.k8s.io is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io configured
```

metrics pod 已经正常运行：

```text
[root@k8s-master k8s-install]# kubectl get pod -A | grep metrics
kube-system   metrics-server-fd9598766-8zphn       1/1     Running   0              89s
```

再次执行 kubectl top 命令成功：

```text
[root@k8s-master k8s-install]# kubectl top pod
NAME                            CPU(cores)   MEMORY(bytes)   
front-end-59bc6df748-699vb      0m           3Mi             
front-end-59bc6df748-r7pkr      0m           3Mi             
kucc4                           1m           2Mi             
legacy-app                      1m           1Mi             
my-demo-nginx-998bbf8f5-9t9pw   0m           0Mi             
my-demo-nginx-998bbf8f5-lfgvw   0m           0Mi             
my-demo-nginx-998bbf8f5-nfn7r   1m           0Mi             
nginx-kusc00401                 0m           3Mi
[root@k8s-master k8s-install]# 
[root@k8s-master k8s-install]# kubectl top node
NAME         CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
k8s-master   232m         5%     1708Mi          46%       
k8s-slave1   29m          1%     594Mi           34%       
k8s-slave2   25m          1%     556Mi           32%
```

## k8s常用命令集合

k8s常用命令集合：

```bash
# 查看当前集群的所有的节点
kubectl get node
# 显示 Node 的详细信息（一般用不着）
kubectl describe node node1

# 查看所有的pod
kubectl get pod --all-namespaces
# 查看pod的详细信息
kubectl get pods -o wide --all-namespaces

# 查看所有创建的服务
kubectl get service

# 查看所有的deploy
kubectl get deploy

# 重启 pod（这个方式会删除原来的pod，然后再重新生成一个pod达到重启的目的）
# 有yaml文件的重启
kubectl replace --force -f xxx.yaml
# 无yaml文件的重启
kubectl get pod <POD_NAME> -n <NAMESPACE> -o yaml | kubectl replace --force -f -

# 查看pod的详细信息
kubectl describe pod nfs-client-provisioner-65c77c7bf9-54rdp -n default

# 根据 yaml 文件创建Pod资源
kubectl apply -f pod.yaml
# 删除基于 pod.yaml 文件定义的Pod 
kubectl delete -f pod.yaml

#获取 Kubernetes 集群中的 Pod 列表，并将详细结果显示为宽表格
kubectl get pod -o wide
#基于现有的pod导出yaml文件
kubectl get pod <pod_name> -o yaml > pod.yaml

#根据CPU使用率自动扩展 Pod 的数量
kubectl autoscale deployment <deployment_name> --min=<min_replicas> --max=<max_replicas> --cpu-percent=<cpu_percent>

# 查看容器的日志
kubectl logs <pod-name>
# 实时查看日志
kubectl logs -f <pod-name>
# 若 pod 只有一个容器，可以不加 -c
kubectl log  <pod-name> -c <container_name>
# 返回所有标记为 app=frontend 的 pod 的合并日志
kubectl logs -l app=frontend

# 通过bash获得 pod 中某个容器的TTY，相当于登录容器
# kubectl exec -it <pod-name> -c <container-name> -- bash
eg:
kubectl exec -it redis-master-cln81 -- bash

# 查看 endpoint 列表
kubectl get endpoints

# 查看已有的token
kubeadm token list

#滚动升级（在yaml文件里修改新的images）
kubectl apply -f svc-zipkin.yaml --record
或者直接用命令，更新名为my-deployment的部署中my-container容器的镜像
kubectl set image deployment/my-deployment my-container=my-image:my-tag
#暂停升级
kubectl rollout pause deployment zipkin-server
#继续升级
kubectl rollout resume deployment zipkin-server
#查看升级历史
kubectl rollout history deployment zipkin-server
#回滚到上一级
kubectl rollout undo deployment zipkin-server
#回滚制定版本（根据rollout history的查看结果）
kubectl rollout undo deployment zipkin-server --to-revision=13
```
