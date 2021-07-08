# k8s 集群搭建

标签（空格分隔）： k8s

---

[TOC]

## 使用`kubeadm`工具部署节点

```
# 创建一个 Master 节点
$ kubeadm init

# 将一个 Node 节点加入到当前集群中
$ kubeadm join <Master节点的IP和端口 >
```

## 部署k8s步骤

### 安装要求

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多
- 集群中所有机器之间网络互通
- 可以访问外网，需要拉取镜像
- 禁止swap分区

### 所有节点安装Docker/kubeadm/kubelet

- 安装docker

    ```
    $ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
    $ yum -y install docker-ce-18.06.1.ce-3.el7
    $ systemctl enable docker && systemctl start docker
    $ docker --version
    Docker version 18.06.1-ce, build e68fc7a
    ```

- 添加阿里云YUM软件源

    ```
    $ cat > /etc/yum.repos.d/kubernetes.repo << EOF
    [kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF
    ```

- 安装kubeadm，kubelet和kubectl

    ```
    $ yum install -y kubelet-1.14.0 kubeadm-1.14.0 kubectl-1.14.0
    $ systemctl enable kubelet
    #安装kubelet 后会在/etc下生成文件目录/etc/kubernetes/manifests/
    ```

- 部署Kubernetes Master

    在172.16.3.40（Master）执行:

    ```
    $ kubeadm init \
  --apiserver-advertise-address=172.16.3.40 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.14.0 \
  --service-cidr=10.1.0.0/16 \
  --pod-network-cidr=10.244.0.0/16

  # kubeadm 以后将会在 /etc 路径下生成配置文件和证书文件
  [root@k8s-master etc]# tree kubernetes/
    kubernetes/
    ├── admin.conf
    ├── controller-manager.conf
    ├── kubelet.conf
    ├── manifests
    │   ├── etcd.yaml
    │   ├── kube-apiserver.yaml
    │   ├── kube-controller-manager.yaml
    │   └── kube-scheduler.yaml
    ├── pki
    │   ├── apiserver.crt
    │   ├── apiserver-etcd-client.crt
    │   ├── apiserver-etcd-client.key
    │   ├── apiserver.key
    │   ├── apiserver-kubelet-client.crt
    │   ├── apiserver-kubelet-client.key
    │   ├── ca.crt
    │   ├── ca.key
    │   ├── etcd
    │   │   ├── ca.crt
    │   │   ├── ca.key
    │   │   ├── healthcheck-client.crt
    │   │   ├── healthcheck-client.key
    │   │   ├── peer.crt
    │   │   ├── peer.key
    │   │   ├── server.crt
    │   │   └── server.key
    │   ├── front-proxy-ca.crt
    │   ├── front-proxy-ca.key
    │   ├── front-proxy-client.crt
    │   ├── front-proxy-client.key
    │   ├── sa.key
    │   └── sa.pub
    └── scheduler.conf
    ```

### master节点token

- 查看当前token状态

```
kubeadm token list
```

- 生成新的token

```
# 默认有效期24小时,若想久一些可以结合--ttl参数,设为0则用不过期
kubeadm token create --print-join-command
```

- node节点加入集群

```
 kubeadm join <Master节点的IP和端口 > --token<master token>
```

### 安装Pod网络插件（CNI）

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

由于网络原因，使用以下方式：

```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
ls | grep flannel.yml   #确定下载到了当前目录
kubectl apply -f kube-flannel.yml  #指定下载的.yml文件执行相应命令
```

## 问题

- 安装kubeadm，kubelet和kubectl时报`Error: Package: kubelet-1.14.0-0.x86_64 (kubernetes)`

    解决：
  - 方式一、直接单独安装缺少依赖（可能会造成kubelet版本不匹配）

    ```
    yum -y install kubernetes-cni = 0.7.5
    ```

  - 方式二、不指定安装源单独安装 kubelet

    ```
    yum  -y  install kubelet-1.14.0
    ```

- 部署Kubernetes Master初始化时报`[ERROR Swap]: running with swap on is not supported. Please disable swap`

    解决：关闭swap

- `[ERROR KubeletVersion]: the kubelet version is higher than the control plane version.`

    解决：重新安装对应的 kubelet 版本

    ```
    yum -y remove kubelet
    ```
  
- `[ERROR Port-10250]: Port 10250 is in use`

    解决：执行`kubeadm reset`

- `raw.githubusercontent.com was refused`

    解决：

    ```
    # 执行
    sudo vi /etc/hosts
    # 添加
    151.101.76.133 raw.githubusercontent.com
    ```

- 执行`kubectl apply -f kube-flannel.yml`报错如下：

-

```
unable to recognize "kube-flannel.yml": no matches for kind "PodSecurityPolicy" in version "policy/v1beta1"
unable to recognize "kube-flannel.yml": no matches for kind "ClusterRole" in version "rbac.authorization.k8s.io/v1"
unable to recognize "kube-flannel.yml": no matches for kind "ClusterRoleBinding" in version "rbac.authorization.k8s.io/v1"
unable to recognize "kube-flannel.yml": no matches for kind "ServiceAccount" in version "v1"
unable to recognize "kube-flannel.yml": no matches for kind "ConfigMap" in version "v1"
unable to recognize "kube-flannel.yml": no matches for kind "DaemonSet" in version "apps/v1"
```

解决：

```
# 查看kubeadm版本
kubeadm config images list
# 暂未有解决办法 尝试更新版本
搜到的回答`You must be using Kubernetes version >=1.16`
```
