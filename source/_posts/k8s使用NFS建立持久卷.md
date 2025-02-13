---
title: k8s使用NFS建立持久卷
categories:
  - 技术
  - NFS
tags:
  - K8S
  - NFS
abbrlink: 2698ba09
date: 2024-06-17 16:32:50
---

由于服务在k8s集群是以容器方式启动，容器重启的话数据会丢失，所以我们一般会把重要的数据挂载到机器上，这里使用NFS做k8s的持久卷。

<!--more-->

### 部署NFS 服务端

关闭防火墙

```bash
$ systemctl stop firewalld.service
$ systemctl disable firewalld.service
```


安装配置 nfs

```
$ yum -y install nfs-utils rpcbind
```


共享目录设置权限：以 /data/k8s目录存放数据

```
chmod 755 /data/k8s/
```


配置 nfs，nfs 的默认配置文件在 /etc/exports 文件下，在该文件中添加下面的配置信息：

```
$ vi /etc/exports
/data/k8s  *(rw,sync,no_root_squash)
```


配置说明：

/data/k8s：是共享的数据目录
*：表示任何人都有权限连接，当然也可以是一个网段，一个 IP，也可以是域名
rw：读写的权限
sync：表示文件同时写入硬盘和内存
no_root_squash：当登录 NFS 主机使用共享目录的使用者是 root 时，其权限将被转换成为匿名使用者，通常它的 UID 与 GID，都会变成 nobody 身份
启动服务 nfs 需要向 rpc 注册，rpc 一旦重启了，注册的文件都会丢失，向他注册的服务都需要重启

```
$ systemctl start rpcbind.service
$ systemctl enable rpcbind
$ systemctl status rpcbind
● rpcbind.service - RPC bind service
   Loaded: loaded (/usr/lib/systemd/system/rpcbind.service; disabled; vendor preset: enabled)
   Active: active (running) since Tue 2018-07-10 20:57:29 CST; 1min 54s ago
  Process: 17696 ExecStart=/sbin/rpcbind -w $RPCBIND_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 17697 (rpcbind)
    Tasks: 1
   Memory: 1.1M
   CGroup: /system.slice/rpcbind.service
           └─17697 /sbin/rpcbind -w
```


 running 证明启动成功了。

然后启动 nfs 服务

```
$ systemctl start nfs.service
$ systemctl enable nfs
$ systemctl status nfs

● nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
  Drop-In: /run/systemd/generator/nfs-server.service.d
           └─order-with-mounts.conf
   Active: active (exited) since Fri 2020-12-11 16:59:51 CST; 2 weeks 4 days ago
 Main PID: 83896 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/nfs-server.service
```

acticve代表启动成功

另外我们还可以通过下面的命令确认下：

```
$ rpcinfo -p|grep nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100227    3   udp   2049  nfs_acl
```


查看具体目录挂载权限：

```
$ cat /var/lib/nfs/etab
/data/k8s    *(rw,sync,wdelay,hide,nocrossmnt,secure,no_root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,secure,no_root_squash,no_all_squash)
```

到这里我们就把 nfs server 给安装成功了。

在其他机器查看共享目录

```
sudo showmount -e 192.168.108.100
```

返回值如下，表示创建成功

```
Export list for 192.168.108.100:

/data/k8s	192.168.108.0/24
```

### 设置存储分配器的权限

创建nfs-client-provisioner-authority.yaml文件

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner

replace with namespace where provisioner is deployed

namespace: default

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:

  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]

---

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:

  - kind: ServiceAccount
    name: nfs-client-provisioner

    # replace with namespace where provisioner is deployed

    namespace: default
    roleRef:
      kind: ClusterRole
      name: nfs-client-provisioner-runner
      apiGroup: rbac.authorization.k8s.io

---

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner

  # replace with namespace where provisioner is deployed

  namespace: default
rules:

  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]

---

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner

  # replace with namespace where provisioner is deployed

  namespace: default
subjects:

  - kind: ServiceAccount
    name: nfs-client-provisioner

    # replace with namespace where provisioner is deployed

    namespace: default
    roleRef:
      kind: Role
      name: leader-locking-nfs-client-provisioner
      apiGroup: rbac.authorization.k8s.io
```

### 创建NFS存储分配器

创建nfs-client-provisioner.yaml文件

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:

   - name: nfs-client-provisioner
     image: quay.io/external_storage/nfs-client-provisioner:latest
     volumeMounts:

       - name: nfs-client-root
         mountPath: /persistentvolumes
         env:

       # 存储分配器名称

       - name: PROVISIONER_NAME
         value: nfs-provisioner

       # NFS服务器地址，设置为自己的IP

       - name: NFS_SERVER
         value: 192.168.108.100

       # NFS共享目录地址

       - name: NFS_PATH
         value: /data/k8s
           volumes:
            - name: nfs-client-root
              nfs:

       # 设置为自己的IP

       server: 192.168.108.100

       # 对应NFS上的共享目录

       path: /data/k8s
```

### 创建StorageClass

创建nfs-storage-class.yaml文件

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-data

# 存储分配器的名称

# 对应“nfs-client-provisioner.yaml”文件中env.PROVISIONER_NAME.value

provisioner: nfs-provisioner

# 允许pvc创建后扩容

allowVolumeExpansion: True

parameters:

  # 资源删除策略，“true”表示删除PVC时，同时删除绑定的PV

  archiveOnDelete: "true"
```

查看StorageClass

```
kubectl get storageclass
```

### 创建PVC

创建nfs-pvc.yaml文件

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:

  # 注意，后面Deployment申请资源需要用到此处的名称

  name: nfs-pvc
spec:

  # 设置资源的访问策略，ReadWriteMany表示该卷可以被多个节点以读写模式挂载；

  accessModes:
    - ReadWriteMany

  # 设置资源的class名称

  # 注意，此处的名称必须与“nfs-storage-class.yaml”中的storageClassName相同

  storageClassName: nfs-data

  # 设置申请的资源大小

  resources:
    requests:
      storage: 100Mi
```

查看PVC

```
kubectl get pvc
```

### 创建Deployment控制器

创建nfs-deployment-python.yaml文件

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-deployment-python
spec:
  replicas: 2
  selector:
    matchLabels:
      app: python-nfs
  template:
    metadata:
      labels:
        app: python-nfs
    spec:
      containers:
        - name: python-nfs
          image: python:3.8.2
          imagePullPolicy: IfNotPresent
          command: ['/bin/bash', '-c', '--']
          # 启动"python -m http.server 80"服务，“>>”表示向文件中追加数据
          args: ['echo "<p> The host is $(hostname) </p>" >> /containerdata/podinfor; python -m http.server 80']
          # 设置80端口
          ports:
            - name: http
              containerPort: 80
          # 设置挂载点
          volumeMounts:
            # 此处的名称与volumes有对应关系
            - name: python-nfs-data
              mountPath: /containerdata

  # 配置nfs存储卷

  volumes:

    # 此处的名称需与spec.containers.volumeMounts.name相同

   - name: python-nfs-data

     # 向PVC申请资源，此处的名称对应# nfs-pvc.yaml文件中的metadata.name

     persistentVolumeClaim:
       claimName: nfs-pvc
```

