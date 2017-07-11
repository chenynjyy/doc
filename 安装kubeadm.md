环境：  
centos7.3  
主机：master  
从机（可选）：node

kubeadm在1.6版本之后做了不少改动，具体参见：
https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG.md#kubeadm-1  
kubeadm文档索引：  
https://kubernetes.io/docs/admin/kubeadm/

1. 修改hosts，hostname：
>127.0.0.1 master   

如果有从机可添加：
>127.0.0.1 node  
[masterIP] master etcd

2. 添加k8s repo，下载k8s源文件：  


```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo  
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```


3. 禁止selinux（执行完关闭selinux防火墙）  

```
setenforce 0
```


4. 下载并安装k8s  
```
yum install kubeadm kubernetes-cni kubelet docker -y
```

5. 启动docker和kubelet：  
```
systemctl enable docker && systemctl start docker
systemctl enable kubelet && systemctl start kubelet
```
docker为k8s实际操作的容器工具  
kubelet作为本地维护pod的组件，crud与监控，与master上的api-server进行交互

6. 初始化kubeadm  

```
kubeadm init --service-cidr [string]
```

启动k8s master系统，kubeadm帮你配置完默认参数  

tips：服务CIDR需要与后边`kube-dns`的网络插件flannel的CIDR保持一致，默认为10.96.0.0/12

tips：启动报  

```
/proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
```

参考：http://blog.csdn.net/tycoon1988/article/details/40826235  
在/etc/sysctl.conf添加  

```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
```


参考：https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/  
Limitations.3：  
可在`/etc/sysctl.conf`添加
```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

tips：
- 写入上述记录后需执行：  

```
sysctl -p
```

使过滤条件生效，不指定路径默认生效`/etc/sysctl.conf`文件
- k8s v1.6以上的版本是按照RBAC规则进行人员权限的设计，
api-server端口从80转移到了6443，配置文件也有了相应改动，目测是不支持显示配置配置文件，
linux服务器下的yum配置已被修改为deprecate（过时），
增加了addons概念，对非必须组件可进行addon增删，
可使用命令
```
kubectl apply -f [podnetwork].yaml
```
  详情见：http://kubernetes.io/docs/admin/addons/

7. 添加参数，全局生效  
```
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```

8. （可选）添加子节点node  

```
kubeadm join --token 469d6d.9b23db59952363c3 172.20.212.249:6443
```
添加方式参照格式：
```
kubeadm join --token [tokenId] [IP]:[port]
```

9. 添加网络连接组件addon  
可选择多种网络连接组件，此处选择flannel  
参考：https://kubernetes.io/docs/concepts/cluster-administration/addons/  
首先，先加入flannel的角色权限，获取rbac文件：

```
curl https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel-rbac.yml > kube-flannel-rbac.yml
```

添加flannel的虚地址service，添加flannel配置，添加flannel一批守护进程，获取flannel文件：
```
curl https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml > kube-flannel.yml
```

在此需修改命令，使flannel不使用默认回环网络lo：`--iface=eth0`

运行组件：
```
kubectl apply -f kube-flannel-rbac.yml
kubectl apply -f kube-flannel.yml
```

查看k8s的po是否正常运行（组件kube-dns依赖网络连接组件，本身pod有重试周期，可能不会立即生效）：

```
kubectl get po --all-namespaces
```

tips：  
`kube-flannel.yml`文件中
`"Network": "10.244.0.0/16"`需要与上述`kubeadm init`的初始`CIDR`一致，否则`kube-dns`会一直停留在containerCreating阶段

10. 测试  
可参考：https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/
下的 `Installing a sample application`

参考：
http://blog.frognew.com/2017/04/kubeadm-install-kubernetes-1.6.html#使master-node参与工作负载