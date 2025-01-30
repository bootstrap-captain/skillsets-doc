# 容器化部署

```bash
# 优点
- 容器化技术，在一个server上，即使一个app发生内存泄漏，不会影响其他的app

# 缺点
- 一个server上部署多份的app，多个server部署不同的应用，管理比较困难
```

![image-20250126102012367](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250126102012367.png)

# 架构

- 多个服务器上，运行着多个容器，容器的统一管理不方便
- 大规模的容器编排技术
- [官方文档](https://kubernetes.io/)
- N master node + N worker node， N>=1

```bash
# master node 
- 一般不部署应用，是用来管理work-node的
- master-node 为多个，高可用集群
- master-node发生异常时， master-node采用投票方式选取最新的master-node
```

![image-20250126112310410](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250126112310410.png)

## Controller Manager

- 决策者，决定当前集群需要做什么事情

## ETCD

- 集群的资料库
- cluster集群的核心数据，都可以保存在这里
- k-v存储库，类似redis

## API-SERVER

- 秘书部

## Scheduler

- 根据不同的worker-node，决定这个活到底让哪个worker-node去干

## Kubelet

- 每个node的厂长
- 监控当前node的所有情况，并控制当前node
- 可以对当前node的所有的app进行启动停止

## Kube Proxy

- 看门大爷，清楚各个其他的node情况
- 不同的kube-porxy保持同步

## Cloud Controller Manger

- 云决策者
- 要不要和外面的厂商合作

# 安装

## 1. 安装组件

- 每个机器必须先安装docker
- 一主二从

```bash
# kubectl
- kube controller
- 客户端，向k8s发号施令的组件
- 一般装在master-node即可，这里选择全部装

# kubeadm
- kube admin
- 通过kubeadmin来安装集群

# kubelet
- node的厂长
- 每个节点都必须安装

# 在master节点
- 执行kubeadm init 后，该节点的kubelet，后自动拉对应的组件的镜像
- scheduler，kube-proxy，api-server， etcd， controller-manager

# worker-node
- 执行kubeadmin join 加入集群，该节点的kublet，会自动拉对应的kube-proxy的镜像
```

![image-20250126210003784](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250126210003784.png)

## 2. 云服务器创建

- 2核4G
- 必须在同一个VPC网络下
- 按照k8s官方的要求做下面的修改

### 2.1 VPC

- 选取173.31.0.0/16

![image-20250127133819190](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250127133819190.png)

![image-20250127133745760](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250127133745760.png)

![image-20250127133926612](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250127133926612.png)

### 2.2 三台服务器

- 加入到上面的VPC中
- 开放所有端口(仅仅为了测试方便)

![image-20250127135056465](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250127135056465.png)

### 2.3 预配置

```bash
# 1. 修改主机名，主机名不要重复
hostnamectl set-hostname erick-k8s-master
hostnamectl set-hostname erick-k8s-first-node
hostnamectl set-hostname erick-k8s-second-node

# 2.将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0                                                                      # 临时禁用
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config            # 配置文件永久禁用

# 3. 关闭swap
swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab

# 4. 允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# 5. 刷新配置
sudo sysctl --system
```

## 3. 安装Docker

- kubernetes运行时需要容器化环境，docker是其中一个方案
- 三台都安装且设置为服务启动后自启动

```bash
sudo yum -y install docker-ce-20.10.7 docker-ce-cli-20.10.7 containerd.io-1.4.6
# 设置成开机自启动

Docker version 20.10.7, build f0df350
```

## 4. 安装kubelet/kubeadm/kubectl

- 三台都安装

```bash
# 配置kubernetes的yum源: 使用的是阿里云的镜像
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# 安装kubelet, kubeadm, kubectl
sudo yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9 --disableexcludes=kubernetes

# 开机自启动kublet
sudo systemctl enable --now kubelet

# 检查kubelete的状态
# 现在每隔几秒就会重启，因为它陷入了一个等待 kubeadm 指令的死循环
systemctl status kubelet
```

## 5. kubeadm引导集群

- kube-apiserver，kube-proxy，kube-controller，coredns，etcd，pause都是以容器化的方式运行
- 理论上只需要在master-node上拉对应的镜像，但是kube-proxy在worker-node也需要，暂时就在每个节点都拉取到image

### 5.1 拉镜像

```bash
# 进入对应的目录
cd opt

# 创建shell脚本
sudo tee ./images.sh <<-'EOF'
#!/bin/bash
images=(
kube-apiserver:v1.20.9
kube-proxy:v1.20.9
kube-controller-manager:v1.20.9
kube-scheduler:v1.20.9
coredns:1.7.0
etcd:3.4.13-0
pause:3.2
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/$imageName
done
EOF

# 执行脚本拉image
chmod +x ./images.sh && ./images.sh
```

![image-20250127111735621](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250127111735621.png)

### 5.2 主节点初始化

```bash
# 在每个节点执行： 私有ip， 域名， 域名映射
# 让每个节点都知道master节点在哪里
echo "173.31.0.216  erick-k8s-endpoint" >> /etc/hosts

# 在每个节点ping，保证能ping通
ping erick-k8s-endpoint
```

```bash
# 主节点初始化
# 只在主节点运行

     # 私网ip：master节点的私网ip 173.31.0.216
     # 域名： 和上面对应
kubeadm init \
--apiserver-advertise-address=173.31.0.216 \
--control-plane-endpoint=erick-k8s-endpoint \
--image-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16

# 所有网络范围不重叠
pod-network-cidr: 后面分配的集群中的pod的网络范围

# 启动成功后后，可以用docker ps -a 查看
```

```bash
# 主节点按照下面日志
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
  
#查看集群所有节点
kubectl get nodes

NAME               STATUS     ROLES                  AGE   VERSION
erick-k8s-master   NotReady   control-plane,master   10m   v1.20.9
```

```bash
# 成功日志
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

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join erick-k8s-endpoint:6443 --token wmgxps.fx79sapcvvteyrtw \
    --discovery-token-ca-cert-hash sha256:832c6409b917e721a11ba197571e6b83b87f15bec7887f79ff7df5b2b8c0d90b \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join erick-k8s-endpoint:6443 --token wmgxps.fx79sapcvvteyrtw \
    --discovery-token-ca-cert-hash sha256:832c6409b917e721a11ba197571e6b83b87f15bec7887f79ff7df5b2b8c0d90b 
```

### 5.3 网络插件

- [网络插件官网](https://kubernetes.io/docs/concepts/cluster-administration/addons/)

```bash
# 下载 calico的网络插件的配置文件
# 执行成功后，会在命令执行的目录产生一个 calico.yaml
curl https://docs.projectcalico.org/v3.20/manifests/calico.yaml -O

# 安装：从该配置文件去安装一个应用
kubectl apply -f calico.yaml
```

```bash
# 执行下面的的指令，只能master节点执行
# 查看集群节点
kubectl get nodes

# 根据配置文件，给集群创建资源
kubectl apply -f xxx.yaml

# 查看集群部署了哪些应用
kubectl get pods -A
kubectl get pods -A -w                       # 实时查看
   # 运行中的应用在docker中叫容器，在k8s中叫Pod
```

![image-20250127154607906](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250127154607906.png)

### 5.4 master-节点就绪

```bash
kubectl get nodes

NAME               STATUS   ROLES                  AGE   VERSION
erick-k8s-master   Ready    control-plane,master   98m   v1.20.9
```

## 6. worker-node

- 在两个worker-node分别执行下面的命令，加入集群

```bash
# 24h有效
kubeadm join erick-k8s-endpoint:6443 --token wmgxps.fx79sapcvvteyrtw \
    --discovery-token-ca-cert-hash sha256:832c6409b917e721a11ba197571e6b83b87f15bec7887f79ff7df5b2b8c0d90b
```

![image-20250127160354654](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250127160354654.png)

```bash
# 等待worker-node  就绪后，在主机点执行
 kubectl get nodes
 
 NAME                    STATUS   ROLES                  AGE    VERSION
erick-k8s-first-node    Ready    <none>                 25m    v1.20.9
erick-k8s-master        Ready    control-plane,master   129m   v1.20.9
erick-k8s-second-node   Ready    <none>                 25m    v1.20.9
```

## 7. dashboard

- kubernetes官方提供的可视化界面
- [github地址](https://github.com/kubernetes/dashboard)
- 在master-node操作

### 7.1 安装

```bash
# 安装完成后，就会产生一个pod应用
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml

kubectl get pod -A

# 设置访问端口
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard

- 将type: ClusterIP 改为 type: NodePort

# 找到运行端口30449，在安全组放行
# 每次启动的端口可能不同
kubectl get svc -A |grep kubernetes-dashboard

kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP   10.96.176.74    <none>        8000/TCP                 5m49s
kubernetes-dashboard   kubernetes-dashboard        NodePort    10.96.149.182   <none>        443:30449/TCP            5m49s

https://139.198.165.238:32759
# 使用任意一个node的公网ip+端口即可访问
- https://123.56.85.41:30449
- https://101.200.181.171:30449
- https://182.92.195.98:30449
```

- 使用google浏览器时候，页面会出现无法加载

```bash
# 不用刷新页面
直接在页面中，输入 thisisunsafe， 页面就会自动刷新并且进入
```

### 7.2 创建访问令牌

```yaml
#创建访问账号，准备一个yaml文件； vim dash.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

```bash
# 安装令牌应用
kubectl apply -f dash.yaml

# 有一个admin-user
# serviceaccount/admin-user created
# clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

```bash
#获取访问令牌
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

```bash
# 得到的令牌
eyJhbGciOiJSUzI1NiIsImtpZCI6InpJWlcyMWJDMnlaOUNYbkFETVc5M1l3WkNHS0FScFBjOXNMb1ktVUZWRGMifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXFqbWxsIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmZjA4ZjhlOS03NTE0LTQ5ZmUtYThmMy01NmNmOGNmODAxYWYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.1esQofJjeGC5hJPbbGYn32KZ8fi-x20K5gGl9N3pDRqp0Okzcub_3omOslXSo0z0T702tFC-_0WYLisJjCQg6KPiHOATT8lP1Ruq8tfB9p1s5S_XBrA92uqDHOe0564teVAK1SC3hjfSgG7Xl1i2I9--fMLRx-oW84B6VoJ6FtUO3DLrHtlTY3J0Xqa1-YITyRcvisnxPasJFMoECQhVVa2-XLNaCl9ERmSq75N22LqaJCdBpTg3pU8E0oZrRmcuCZLLKkZPB4fkldB10IcmxUtEp9mrWIOMhaBP8nUheXFHJ_BanMQDglun0WSa19TmDEa4mHIcMNQKKWdeyaJL3Q
```

![image-20250127213001931](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250127213001931.png)

# 自动重启

## 1. 自动重启

### 1.1 reboot

- 每个节点都进行reboot后，可以自动恢复
-  kubectl get nodes查看对应的集群状态

### 1.2 云服务释放

- 释放后依然可行，如果公网IP发生变化，在后续注意更换即可

## 2. token 过期

```bash
# 在master节点 重新生成令牌信息
kubeadm token create --print-join-command
```

# Namespace

- 用来隔离资源

## 1. 简介

- 默认只隔离资源，不隔离网络(可以通过配置的方式隔离网络)

![image-20250127215004236](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250127215004236.png)

## 2. 查看

```bash
# 查看当前的所有namespace
kubectl get namespace
kubectl get ns


NAME                   STATUS   AGE
default                Active   7h33m
kube-node-lease        Active   7h33m
kube-public            Active   7h33m
kube-system            Active   7h33m
kubernetes-dashboard   Active   51m
```

```bash
# 查看当前集群所有的pod，能够看到对应的namespace
kubectl get pods -A

NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
kube-system            calico-kube-controllers-577f77cb5c-xdj69     1/1     Running   1          6h12m
kube-system            calico-node-8kbmr                            1/1     Running   2          5h49m
kube-system            calico-node-lt7rr                            1/1     Running   2          5h49m
kube-system            calico-node-m96d8                            1/1     Running   2          6h12m
kube-system            coredns-5897cd56c4-bq7pm                     1/1     Running   1          7h34m
kube-system            coredns-5897cd56c4-tw8rt                     1/1     Running   1          7h34m
kube-system            etcd-erick-k8s-master                        1/1     Running   2          7h34m
kube-system            kube-apiserver-erick-k8s-master              1/1     Running   2          7h34m
kube-system            kube-controller-manager-erick-k8s-master     1/1     Running   2          7h34m
kube-system            kube-proxy-6w82t                             1/1     Running   2          5h49m
kube-system            kube-proxy-77hf2                             1/1     Running   2          7h34m
kube-system            kube-proxy-s86gw                             1/1     Running   2          5h49m
kube-system            kube-scheduler-erick-k8s-master              1/1     Running   2          7h34m
kubernetes-dashboard   dashboard-metrics-scraper-79c5968bdc-t2qm4   1/1     Running   0          52m
kubernetes-dashboard   kubernetes-dashboard-658485d5c7-xsjpm        1/1     Running   0          52m

# 获取default下面的所有pods
kubectl get pods

# 查看某个指定namespace下面的pod
kubectl get pod -n kubernetes-dashboard 
```

```bash
部署的资源，如果不加namespace，默认都会创建在default的namespace下面
```

## 3. 创建/删除

- 一般情况下，通过哪种方式创建的资源，就通过哪种方式来删

### 3.1 命令行方式

```bash 
# 创建
kubectl create ns erick-prod

# 删除
# 删除namespace时，会把该namespace下面部署的所有资源都删除
kubectl delete ns erick-prod
```

 ### 3.2 资源配置方式

```bash
# 本地创建一个erick-test.yaml的文档，把下面内容复制进去
vim erick-test.yaml

# 1. 创建 
kubectl apply -f erick-test.yaml

# 2. 删除 : 根据配置文件来删除
kubectl delete -f erick-test.yaml 
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: erick-test
```

# Pods

## 1. 简介

- docker中是一个容器
- 为了管理方便，在容器上封装一个pod，pod中可以包含一组容器
- 运行中的一组容器，pod是kubernetes中应用的最小单位
- k8s操作的是pod，pod的启停，会将里面的容器进行启停
- k8s统计的信息，也是通过pod来进行统计

![image-20250127230748469](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250127230748469.png)

```bash
# ready中： 容器总数量/容器就绪数量
# 只有全部的容器都在运行，这个pod才是Running状态

NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
kube-system            calico-kube-controllers-577f77cb5c-xdj69     1/1     Running   1          7h27m
```

## 2. 单容器

- 在一个pod中部署一个容器
- 是在master-node执行

### 2.1 命令行方式

#### 创建

```bash
# 创建
# pod名称：        erick-single-nginx
# docker-image：  nginx:1.20,  不加版本号，就是默认latest
# namespace：     erick-dev， 不加的话，默认在default的namespace中创建
kubectl run erick-single-nginx --image=nginx:1.20 -n erick-dev

# 查看
kubectl get pod -n erick-dev

# 描述, 发现这个pod在second-node上运行的
kubectl describe pod erick-single-nginx -n erick-dev
    # second-node
   - kubelete拉nginx:1.20镜像
   - kubelete创建pod
```

![image-20250127235755317](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250127235755317.png)

```bash
 # 进入second-node查看
 # 只有second-node会下载对应的image
 docker ps 
```

![image-20250128000414840](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250128000414840.png)

#### 删除

```bash
# pod名字： erick-single-nginx
# 命名空间: erick-dev
#  会删除对应的docker的容器，但是image不会删除
kubectl delete pod erick-single-nginx -n erick-dev
```

### 2.2 资源配置方式

#### 创建

```bash
# pod的名字
metadata.name：erick-single-nginx

# 镜像名字
spec.containers.image

# namespace
metadata.namespace

# spec.containers.name
- 容器名字
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mynginx
  name: erick-single-nginx
  namespace: erick-dev
spec:
  containers:
  - image: nginx:1.20
    name: abc-nginx
```

```bash
# 创建配置文件, 把上面内容粘贴
vim nginx-by-config.yaml

# 创建
kubectl apply -f nginx-by-config.yaml
```

#### 删除

```bash
kubectl delete -f nginx-by-config.yaml
```

### 2.3 pod操作

```bash
# 1. 查看该pod的运行日志信息
kubectl logs erick-single-nginx -n erick-dev

# 查看pod更加完善的信息
kubectl get pod -owide -n erick-dev

        # ip： 192.168.63.131
        # node： erick-k8s-second-node
        
# 2. 每个pod启动后，k8s都会给分配一个ip 
# 可以使用ip+端口访问
# 在每个node都可以访问
# 只能在集群内访问，但是不能通过公网访问
curl 192.168.63.131:80

# 3. 进入pod里面的容器
# 只能在master节点执行
kubectl exec -it erick-single-nginx -n erick-dev -- /bin/bash
cd /usr/share/nginx/html
echo "111" > index.html 
exit
```

## 3. 多容器

- 如果在同一个pod中部署了两个相同的应用，或者两个端口冲突的应用，则会报错

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nike
  name: replay-service
  namespace: erick-dev
spec:
  containers:
  - image: nginx:1.20
    name: first-nginx
  - image: tomcat
    name: erick-tomcat
```

```bash
# 查看日志: 指定一个容器
kubectl logs replay-service first-nginx  -n erick-dev

# 进入一个容器
# -c 容器名字
kubectl exec -it replay-service -c first-nginx -n erick-dev -- /bin/bash

# 互相访问
# 同一个pod内，多个容器，共享网络空间，存储空间
# 进入nginx，访问tomcat
curl http://localhost:8080
curl http://127.0.0.1:80
```

# Deployment

- 控制pod，使pod拥有多副本，自愈，扩缩容等能力
- 一般用这种方式启停pod

## 1. 自愈能力

- 使用pod来创建或删除时候，删除就是真的删除了

```bash
# 创建
# deployment/deploy
kubectl create deployment my-nginx-by-deploy --image=nginx:1.20 -n erick-dev

# 查看deploy
kubectl get deploy -n erick-dev

NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
my-nginx-by-deploy   1/1     1            1           41s

# 查看本次的deployment启动的pod
kubectl get pods -n erick-dev

NAME                                  READY   STATUS    RESTARTS   AGE
my-nginx-by-deploy-74bbf76dc4-cnb9t   1/1     Running   0          82s

# 手动删除该pod: 删除老的pod，重新启动一个新的pod
kubectl delete pod my-nginx-by-deploy-74bbf76dc4-cnb9t -n erick-dev

NAME                                  READY   STATUS    RESTARTS   AGE
my-nginx-by-deploy-74bbf76dc4-krhlk   1/1     Running   0          13s

# 删除该deployment，才能彻底删除对应的deploy和pod
kubectl delete deploy my-nginx-by-deploy -n erick-dev
```

## 2. 多副本

```bash
# 创建
kubectl create deploy nginx-dep-three --image=nginx:1.20 --replicas=3 -n erick-dev

# deploy的信息
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
nginx-dep-three   3/3     3            3           14s
# pod的信息
NAME                               READY   STATUS    RESTARTS   AGE
nginx-dep-three-75cd684746-bfnpf   1/1     Running   0          36s
nginx-dep-three-75cd684746-nhgbq   1/1     Running   0          36s
nginx-dep-three-75cd684746-z7wjv   1/1     Running   0          36s
```

## 3. 扩缩容

- 可以手动的调整，k8s也能够做到动态的扩缩容

```bash
# 可以动态的调整副本数量
kubectl scale deploy/nginx-dep-three --replicas=5 -n erick-dev

# 打开对应的yaml
# 使用命令行创建的deploy，都会在底层生成一个yaml文件
kubectl edit deploy nginx-dep-three -n erick-dev
```

## 4. 自愈/故障转移

### 自愈

- deploy中，手动删除某个pod对应的下面的容器，
- k8会尝试重新启动该容器

### 故障转移

- 当某个node突然停机了，则过一段时间后，就会将该副本，重新在其他的node进行拉一份
- 会有一个默认的时间，默认5min
- 该值不能设置的太小，因为pod可能时不时因为网络故障，就会频繁的启停pod

## 5. 灰度发布

- 新启动一个pod，等其正常运行了，替换掉原来的pod

```bash
# 镜像的升级，代表着应用版本的升级
# --record        留下改动日志
# image名字=image新版本
kubectl set image deploy/nginx-dep-three nginx=nginx:1.21 --record -n erick-dev
```

## 6. 版本回退

``` bash
# 查看历史记录
kubectl rollout history deploy/nginx-dep-three -n erick-dev

# 查看某个revision的详细记录
kubectl rollout history deploy/nginx-dep-three --revision=1 -n erick-dev

# 回退到上个版本
kubectl rollout undo deploy/nginx-dep-three -n erick-dev

# 回退到指定版本
kubectl rollout undo deploy/nginx-dep-three --to-revision=2 -n erick-dev
```

# Service

- pod的服务发现与负载均衡

## 1. ClusterIP

- 一种k8s的资源将一组pods，公开为网络服务的抽象方法

```bash
# 1. 创建一个deploy，里面有三个nginx，进入每个nginx修改其中的index.html 为111，222，333

# 2. 将一组deploy部署的pod，对外进行公开
     # 会创建一个对应的service的资源，并且对该service分配一个内网ip，这个ip是在上面的安装步骤里面有规定范围
     # port:         service对外公开的端口
     # target-port： pod中容器的端口
     # type： 默认就是ClusterIP
     # service   ==  svc
kubectl expose deploy nginx-dep-three --port=9090 --target-port=80 --type=ClusterIP -n erick-dev


#  3. 查看当前部署的service
kubectl get service -n erick-dev

NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
nginx-dep-three   ClusterIP   10.96.172.7   <none>        9090/TCP   97s

# 4. 访问方式
    ## 方式一： serviceIp:port
    # 在集群中的任何一个node/pod中都可以访问
    # 在公网不能访问
    curl http://10.96.172.7:9090
    
    ## 方式二：deployName.namespace.svc
    # 只能在具体的某个pod内部访问
    curl http://nginx-dep-three.erick-dev.svc:9090
    
# 5. 删除service
kubectl delete service nginx-dep-three -n erick-dev
```

![image-20250129225725365](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250129225725365.png)

## 2. NodePort

```bash
# 创建一个nodeport类型的service
kubectl expose deploy nginx-dep-three --port=9090 --target-port=80 --type=NodePort -n erick-dev

# kubectl get service -n erick-dev
      # 30349: 为nodeport, k8s随机分配的
NAME              TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
nginx-dep-three   NodePort   10.96.99.254   <none>        9090:30349/TCP   28s

# 访问方式一：
   # serviceIp+端口
   # 在所有node和pod都可以访问
   curl http://10.96.99.254:9090
   
# 访问方式二： 在浏览器中访问, 会进行负载均衡，但是负载均衡具体机制不详
   # 任意一个node的Ip+nodeport
 http://101.200.150.208:30349
 http://101.200.181.171:30349
```

![image-20250129232137216](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250129232137216.png)

# Ingress

- Service的统一网关入口

## 1. 架构

- 所有的pod发送请求时，都会从Ingress进入，Ingress根据一定的规则，将请求发送给具体的某个service
- Service收到请求后，再通过负载均衡的方式，发送给该Service下对应的deploy的pod

![image-20250130094025175](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250130094025175.png)

## 2. 安装

- 通过配置Nginx的规则，来实现请求转发，限流等
