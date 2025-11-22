# 集群安装方式

- minikube
  K8S 集群模拟器，只有一个节点的集群，只为测试用，master 和 worker 都在一起
- 直接用云平台 Kubernetes
  可视化搭建，只需简单几步就可以创建好一个集群
  优点：安装简单，生态齐全，负载均衡器、存储等都给你配套好，简单操作就搞定
- 裸机安装（Bare Metal）
  至少需要两台机器（主节点、工作节点各一台），需要自己安装 Kubernetes 组件，配置会稍微麻烦点。
  可以到各云厂商按时租用服务器，费用低，用完就销毁
  缺点：配置麻烦，缺少生态支持，例如负载均衡器、云存储

# 安装 Kubernetes 集群

## 服务器环境

这里选择裸机安装，三台服务器配置如下且均已安装 Docker：

| IP              | 主机名称 | 描述      |
| --------------- | -------- | --------- |
| 192.168.110.20  | master   | 主节点    |
| 192.168.110.232 | node1    | 工作节点1 |
| 192.168.110.233 | node2    | 工作节点2 |

## 通用节点配置

### 分别设置对应主机名

```bash
hostnamectl set-hostname master
hostnamectl set-hostname node1
hostnamectl set-hostname node2
```
### 关闭 selinux

```bash
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

### 关闭 swap

```bash
cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Aug  9 14:09:10 2022
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=6228c8d0-3efa-4ac0-a147-a3ee8dc90897 /boot                   xfs     defaults        0 0
#/dev/mapper/centos-swap swap                    swap    defaults        0 0

# 注释掉最后一行
```

### 修改 hosts

```bash
vim /etc/hosts
192.168.110.20 master
192.168.110.232 node1
192.168.110.233 node2
```

### 关闭防火墙

```bash
systemctl stop firewalld
systemctl disable firewalld
```

### 添加安装源

```bash
# 添加 k8s 安装源
cat <<EOF > kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
mv kubernetes.repo /etc/yum.repos.d/

# 添加 Docker 安装源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 安装所需组件

```bash
yum install -y kubelet-1.22.4 kubectl-1.22.4 kubeadm-1.22.4 docker-ce
```

### 设置开机启动

```bash
systemctl enable kubelet
systemctl start kubelet
systemctl enable docker
systemctl start docker
```

### 修改 Docker 配置

```bash
# kubernetes 官方推荐 docker 等使用 systemd 作为 cgroupdriver，否则 kubelet 启动不了
cat <<EOF > daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://ud6340vz.mirror.aliyuncs.com"]
}
EOF
mv daemon.json /etc/docker/

# 重启生效
systemctl daemon-reload && systemctl restart docker
```

## 主节点配置

 使用 [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/) 初始化集群（仅在主节点跑）

```bash
# 初始化集群控制台 Control plane
kubeadm init --image-repository=registry.aliyuncs.com/google_containers
# 失败了可以用 kubeadm reset 重置
```

集群创建成功后，保存命令

```bash
kubeadm join 192.168.110.20:6443 --token yeaovk.aiaf64mm7phk62jy \
        --discovery-token-ca-cert-hash sha256:7c3a7118b7a0b8f1b997d00531fcf304be457f89ec8ea2e6dcd91710f0752ed5

# 忘记了重新获取：kubeadm token create --print-join-command
```

复制授权文件，以便 kubectl 可以有权限访问集群。如果其他节点需要访问集群，需要从主节点复制这个文件过去其他节点。

```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
# 在其他机器上创建 ~/.kube/config 文件也能通过 kubectl 访问到集群
```

> 有兴趣了解 kubeadm init 具体做了什么的，可以 [查看文档](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/)

## 工作节点配置

把工作节点加入集群（只在工作节点跑）

```bash
kubeadm join 192.168.110.20:6443 --token yeaovk.aiaf64mm7phk62jy \
        --discovery-token-ca-cert-hash sha256:7c3a7118b7a0b8f1b997d00531fcf304be457f89ec8ea2e6dcd91710f0752ed5
```

安装网络插件，否则 node 是 NotReady 状态（主节点跑）

```bash
# 很有可能国内网络访问不到这个资源，你可以网上找找国内的源安装 flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

这里选择直接下载一份 `kube-flannel.yml` 的源码，[源码参考](https://blog.csdn.net/qq_22409661/article/details/113371921) 下载文件后，在文件所在目录执行

```bash
kubectl apply -f kube-flannel.yml 
```

查看节点，要在主节点查看（其他节点有安装 kubectl 也可以查看）

```bash
[root@master k8s]# kubectl get nodes
NAME     STATUS   ROLES                  AGE   VERSION
master   Ready    control-plane,master   27m   v1.22.4
node1    Ready    <none>                 16m   v1.22.4
node2    Ready    <none>                 11m   v1.22.4
```
