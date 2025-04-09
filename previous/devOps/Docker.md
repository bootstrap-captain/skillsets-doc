# 简介

- [Docker官网](https://www.docker.com/)
- [Docker Hub官网：仓库](https://hub.docker.com/)

## 基本介绍

### Before

```bash
# 1. 环境不一致
- 一个项目，需要部署在不同的服务器上，每个服务器得单独安装对应的jdk，mysql等
- 多个服务器如果安装的软件版本不一致，可能出现异常问题

# 2. 安装麻烦
- 集群安装比较麻烦

# 3. 动态扩容缩容
- 需要运维手动进行，比较麻烦
```

![image-20241125162140040](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20241125162140040.png)

### After

- 将项目源码，项目所需的其他软件，打包统一称为一个docker-image，然后将该image放在装有docker的linux运行
- 一次镜像，处处运行

![image-20241125162802740](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20241125162802740.png)

## 三要素

 - docker host：Linux 服务器

```bash
# docker image
- docker类软件的镜像，比如QQ安装包

# docker repository
- 镜像的存放空间，包含公共库和私有库

# docker container
- 镜像启动后的一个实例就是一个容器
- 一个镜像可以启动多个容器，各个容器通过不同的宿主机端口号独立运行
    # 每个容器可以看作一个简易版的linux
     - 包含root权限，进程空间，用户空间，网络空间等
     - 运行在其中的应用程序，如redis
```

## 安装-Docker Engine

- 不是一个通用的容器工具，依赖于已经存在并且运行的Linux内核环境
- Docker是在已经运行的Linux下制造了一个隔离的文件环境，因此它执行的效率及户等同于所部署的Linux主机
- 在CentOS 7.6上安装

### 卸载

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 前提安装

```bash
# 基本的包安装：linux通用的
yum -y install gcc
yum -y install gcc-c++

# 安装yum-utils
sudo yum install -y yum-utils

# 设置稳定仓库: 告诉yum源，去哪里下载docker这个软件 
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 重建yum索引
yum makecache fast
```

### docker engine

```bash
sudo yum -y install docker-ce docker-ce-cli containerd.io
```

### 启动

```bash
# 启动/停止/重启
systemctl start docker
systemctl stop docker
systemctl restart docker
systemctl status docker    # docker的状态
systemctl enable docker    # 开机启动Docker

# 查看版本
docker -v
docker version

# docker里面的容器等信息
docker info
```

### 镜像配置

- docker镜像默认从海外拉取
- 有些版本的docker，必须先启动以后，才会有docker这个目录

```bash
# 配置镜像地址，
# 配置cggroup，对后面用k8s安装很重要

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://dockerhub.azk8s.cn",
    "https://mirror.ccs.tencentyun.com",
    "https://registry.cn-hangzhou.aliyuncs.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.m.daocloud.io",   
    "https://noohub.ru", 
    "https://huecker.io",
    "https://dockerhub.timeweb.cloud" 
  ],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

# 重启docker
sudo systemctl daemon-reload
sudo systemctl restart docker

# 重启docker
# 测试
docker run hello-world
```

## 镜像命令

```bash
# 1. 查看本地查看所有镜像
docker images  

REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
hello-world   latest    feb5d9fea6a5   3 years ago   13.3kB

- a：   # 列出本地所有镜像(含历史映像层)
- q：   # 只显示IMAGE ID
docker images -a
docker images -q    
docker images -qa
docker images -aq

# 2. 所有容器及镜像信息
docker system df

TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          1         1         13.26kB   0B (0%)
Containers      1         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B

# 3. 拉取镜像，在镜像名后可以加标签，默认为latest，即为该软件的最新版本
docker pull tomcat
docker pull tomcat:7.0

# 4. 删除软件镜像(IMAGE_ID),该镜像的所有实例都已经删除时,才可以删除镜像
# 加上- f ，即使该image有对应的container，强制删除image
docker rmi -f bd54813ab48c                       

# 先获取所有的imageid，然后全部强制删除
docker rmi -f $(docker images -qa)
```

## 容器命令

### 后台启动

- 后台启动的容器，为守护使容器

```bash
# 启动
#   -d： 以后台模式打开一个容器，不打开命令终端
#   -p： 端口映射 9001:9000,  左边=linux宿主机端口   右边=docker内部，该容器开放的端口
#   --name: 自定义名字，否则为系统随机分配
#   镜像： 可以用镜像id/镜像名称(加tag)
#    --restart=always    docker启动后，自动启动该容器

# 如果没有对应的容器，就会先去执行docker pull
docker run -d --name erick_redis -p 6379:6379 41de2cc0b30e
docker run -d --name lucy_redis -p 6380:6379 redis:6.0.7

# 针对已经启动的容器，修改参数
docker update 41de2cc0b30e --restart=always
```

### 前台启动

- 有些容器必须是-d，有些容器必须是-it

```bash
# -i ： interactive     以交互模式运行容器，通常与 -t同时使用
# -t：  tyy            为容器分配一个伪输入终端，通常与 -i同时使用
docker run -it 
```

### 查看

```bash
# 查看运行中的镜像/所有的容器
# -a : 当前所有正在运行的容器+历史运行过的
# -l： 显示最近创建的一个容器
# -n： 显示最近n个创建的容器
# -q： 只显示容器编号
docker ps
docker ps -a
docker ps -n 2
docker ps -l
docker ps -q

# 查看docker的某个容器日志 （容器id）
docker logs 89ce823c53b9
docker logs --since 30m 89ce823c53b9
docker top c0bd7fd3f2e8      # 查看某个容器里面的进程:比如包含jvm，mysql，redis三个进程
docker inspect c0bd7fd3f2e8
```

![image-20250110101758400](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250110101758400.png)

### 停止/删除

```bash
# container_id 或者 names

# 停止/启动/重启 创建好的容器； 
docker stop/start/restart containerId
docker kill containerId  # 强制删除

# 删除容器  
docker rm 073fe4390fa0              # 容器必须关闭才能删除
docker rm -f 073fe4390fa0           # 强制删除正在运行中的容器
docker rm -f $(docker ps -a -q)     # 删除所有容器
docker ps -a -q|xargs docker rm -f  # 删除所有容器
```

### 进出容器

```bash
# 进入docker中的 容器ID 或 容器名称, 进入后的容器，也是一个小型的linux
# -it：打开命令行
# exec是在容器中打开新的终端，并且可以启动新的进程，用exit退出后，不会导致容器的停止
docker exec -it c0bd7fd3f2e8 /bin/bash

# 直接进入容器，启动命令的终端，不会启动新的进程，用exit推出后，会导致容器的停止
docker attach c0bd7fd3f2e8

# 退出容器
exit
```

## 容器备份

### docker--linux文件拷贝

```bash
# 将容器内的文件，拷贝到linux的宿主机上
docker cp c0bd7fd3f2e8:/data/dump.rdb /tmp

# 将linux宿主机上的东西拷贝到容器内
docker cp /tmp/erick.txt c0bd7fd3f2e8:/data
```

### 数据备份

- 用于备份或迁移容器的文件系统，而不包括 Docker 镜像的所有层和元数据
- 导出容器的时候，不能对现有容器进行再次的添加镜像分层

```bash
# 容器备份： 容器id
# 将整个容器进行导出到本地为tar包
# 会导出到命令所在目录
docker run -d --name erick_redis -p 6379:6379 4075a3f8c3f8
# 进入容器
touch 1.txt
echo "shuzhan" > 1.txt 
# 退出容器


# 导出容器
docker export f4cb760eeaa4 > erick.tar 
# 将tar包解压为新的image镜像  
cat erick.tar | docker import - redis:10

# 运行新的image后，会带有vim和上面的1.txt文件
docker run -d --name second_redis -p 6380:6379 ba65741c9416
```

## 容器数据卷

- CentOS7安全模块的作用，因此在挂载目录时可能存在权限问题
- --privileged=true， 容器内的root拥有linux的真正的root权限，否则容器内的root只是外部的一个普通用户权限
- 实现容器内的数据和宿主机的数据共享，数据备份

```bash
# 卷：docker容器中的目录或文件
# --privileged=true ：                  权限问题
# -v /宿主机绝对路径:/容器内目录            数据映射
docker run -it --privileged=true -v /宿主机绝对路径:/容器内目录 镜像名
```

- 卷中的更改，可以直接实时生效同步到linux宿主机上
- 卷中的更改，不会包含在镜像的更新中
- 卷的生命周期，一直持续到没有容器使用它为止
- 以redis为例，验证，默认docker中启动redis后，数据保存在容器的 /data下

### 数据互通

- 数据卷挂载后，两个目录数据会进行实时同步更新
- 主机写，容器跟进。容器写，主机跟进。

```bash
# -v /tmp/redis/data:/data 7614ae9453d1
# /opt/redis/data:       宿主机的目录,没有对应的目录时，会自动创建
# /data：                 容器内的目录
docker run -d --name erick_redis -p 6379:6379 --privileged=true -v /opt/redis/data:/data 7614ae9453d1

# 在redis中执行写操作，并且手动bgsave,就会由redis在容器的 data目录中产生dumb.rdb
- 容器中：     存在dumb.rdb文件
- 宿主机目录： /opt/redis/data，也存在对应的dumb.redis文件

# 在redis的data目录中，手动创建1.txt文件
- 容器中：     存在1.txt文件
- 宿主机目录： /opt/redis/data，也存在对应的1.txt文件

# 在宿主机目录中创建2.txt
- 宿主机目录： /opt/redis/data：     存在2.txt文件
- 容器中  ： data目录，               也存在对应的2.txt文件

# 是否挂载成功
docker inspect containerId
```

```bash
# 挂载数据卷后，假如容器突然停了，再往linux宿主机中添加数据时
# 容器再次启动后，linux宿主机刚才添加的数据，会再次同步到容器内部中
```

### 读写规则

- 上面的数据卷挂载，为可读可写

```bash
# 默认是rw
docker run -d --name erick_redis -p 6379:6379 --privileged=true -v /opt/redis/data:/data 7614ae9453d1

# ro： 容器内部被限制，只读不写 readonly
# reids不能设置成这种，redis默认就是给容器内写数据的
docker run -d --name erick_redis -p 6379:6379 --privileged=true -v /opt/redis/data:/data:ro 7614ae9453d1
```

### 卷的继承

- 容器A和宿主机建立了某种映射规则，容器B可以继承这种规则
- 后续即使容器A挂了，这种映射规则并不会受到影响

```bash
# 容器A创建映射规则
docker run -d --name first_redis -p 6379:6379 --privileged=true -v /opt/redis/data:/data 7614ae9453d1

# 容器B继承容器A的映射规则
docker run -d --name second_redis -p 6380:6379 --privileged=true --volumes-from first_redis 7614ae9453d1
```

# 自定义镜像

## 镜像分层
- 联合文件系统：Union FS， 支持对文件系统的修改作为一次提交来进行层层叠加
- 镜像可以通过分层来进行继承。基于基础镜像，可以制作具体的各种应用的镜像
- 一个镜像，相当于一个简易版本的linux内核，比如常见就不包含vim
- 镜像分层：可以共享

## Vim安装
- 给某个不具备vim指令的docker容器中添加该功能，比如redis中的容器中不包含vim指令
- 将具备vim指令的docker容器做成新的镜像

```bash
# 1. 在某个容器中安装新的软件
- docker pull redis
- docker run -d --name erick_redis -p 6379:6379 4075a3f8c3f8
- docker exec -it containerId /bin/bash  # 进入容器
- touch 1.txt                       # 创建了一个文件
- apt-get  update                   # 更新包管理工具
- apt-get install -y vim            # 下载vim

# 2. 退出容器，将当前容器提交打包，做成新的镜像存储在本地
exit
# 2.1   -m：评论
# 2.2   -a: 作者
# 2.3   容器的id
# 2.4   新镜像的名字及版本号
# 做成的新镜像，因为安装了vim，相对会大70m，直接运行，就会带有vim
# commit: 将镜像存储到本地, 本地就会多了一个image
docker commit -m 'redis with vim ' -a Erick 1010aa9e7184 redis_vim_1.txt:0.0.1.snpshot

# 3. 验证：
# 应用程序: 上面安装的vim生效了
# data：创建的1.txt文件并不会复制
- docker run -d --name copy_redis -p 6380:6379 838b957f7f60
```

![image-20250116003622913](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250116003622913.png)

## 推送hub

- 将linux服务器上自己创建的新的镜像，推送到阿里云

![image-20250112101504862](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250112101504862.png)

```bash
# 在linux服务器中： 登陆
docker login --username=daydreamer002 registry.cn-beijing.aliyuncs.com

# 输入密码

# 将本地的image和阿里云镜像对应起来
REPOSITORY   TAG             IMAGE ID       CREATED        SIZE
redis_vim    0.0.1.snpshot   dcaee69013da   43 hours ago   156MB

# 1. tag dcaee69013da: 本地的imageId打上标签
# 2. 远程仓库的地址
# 3. redis-vim-erick:1.0.0.release :  本地镜像的名字及版本号
docker tag dcaee69013da registry.cn-beijing.aliyuncs.com/erick-prod/redis_vim:0.0.1.snpshot

# 本地image上传
docker push registry.cn-beijing.aliyuncs.com/erick-prod/redis_vim:0.0.1.snpshot

# 拉取远程镜像
docker pull registry.cn-beijing.aliyuncs.com/erick-prod/redis_vim:0.0.1.snpshot
```

# Dockerfile

- 构建docker镜像的文本文件，由多条构建镜像所需要的指令和参数构成的脚本文件

## 语法

- 指令必须为大写字母，后面至少跟随一个参数
- 指令按照从上到下，顺序执行
- 每条指令都会创建一个新的镜像层并对镜像进行提交

```bash
# 构建流程
- docker从基础镜像运行一个容器
- 执行一条指令并对容器做出修改
- 执行类似docker commit的操作提交一个新的镜像层
- docker再基于刚提交的镜像运行一个新容器
- 执行dockerfile中的下一条指令直到所有指令执行完毕

# 角色
- Dockerfile：软件的原材料， 面向开发
- Docker镜像： 软件的交付品，交付标准
- Docker容器： 软件镜像的运行台，部署与运维
```

## 关键字

```bash
# 基础镜像：当前新镜像是基于哪个镜像的，指定一个已经存在的镜像作为模板
# 第一条必须是FROM
FROM

# 镜像维护者的姓名和邮箱地址
MAINTAINER

# 等同于在终端操作的shell命令，容器build时需要运行的命令
# 两种格式： shell格式:在终端中的指令                exec格式
# RUN 是在docker build时运行
RUN<命令行命令>

# 当前容器对外暴露出的端口
EXPOSE

# 指定在创建容器后，终端默认登录的进来工作目录，一个落脚点
WORKDIR

# 指定该镜像是以什么样的用户去执行，默认为root，权限相关(用的较少)
USER

# 配置环境变量
# 可以在后续的任何RUN指令中使用，
ENV

# 将宿主机目录下的文件拷贝进镜像，且会自动处理URL和解压tar压缩包
ADD

# 将宿主机目录下的文件拷贝进镜像
COPY

# 容器数据卷，用于数据保存和持久化工作
VOLUME
```

```bash
# 容器启动后要做的事情

# Dockerfile中可以有多个CMD指令，但只有最后一个生效
# CMD会被docker run之后的参数替换， 为用户提供了自定义参数
# shell格式 和 exec格式
CMD

# 类似CMD，但ENTRYPOINT不会被docker run后面的命令覆盖
# 可变参数时： CMD就是给ENTRYPOINT传参
ENTRYPOINT
```

## 微服务-基本

### 依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.daydreamer</groupId>
    <artifactId>nike-mall</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.3</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <!--maven package的插件-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### maven打包

- 打包得到对应的fat-jar包
- 打包后，通过java -jar 来验证是否可以本地正常启动

![image-20250113145738662](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250113145738662.png)

### Dockerfile

```bash
# 基础镜像是该jdk,将该jdk安装在docker中
FROM openjdk:17.0-jdk

# 将该jar包文件，拷贝到容器中，并更名
# 后续会将Dockerfile和对应java包放在目标服务器同一个目录下
COPY nike-mall-1.0-SNAPSHOT.jar nike-mall.jar

# 按照java -jar nike-mall.jar来运行
ENTRYPOINT ["java","-jar","nike-mall.jar"]
```

### 上传文件

- 将对应的jar包和Dockerfile上传到阿里云服务器的同一个目录下

### 镜像Build

- 进入到阿里云服务器Dockerfile和jar包的目录
- 目标服务器已经安装了docker

```bash
# nike-mall:     对应的docker image的名字
# 1.0:           版本号
# .  :           不能缺少, 代表在当前目录下查找对应的dockerfile以及jar文件
docker build -t nike-mall:1.0.0.snapshot .

# 构建成功后，通过docker images查看镜像

# 容器里面启动的是8080端口，9090是对外访问的端口
docker run --name myproject  -d -p 9090:8080 9b0033c1747b

# 访问路径
http://39.105.210.163:9090/home/test
```

![image-20250113154209673](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250113154209673.png)

## 微服务-进阶

### Dockerfile

```bash
# 基础镜像是该jdk,基于jdk该版本
FROM openjdk:17.0-jdk
# 维护作者
MAINTAINER ErickShu-1037289945@qq.com

# 在容器中，循环该目录
RUN mkdir -p opt/erick/service
# 将Dockerfile所在路径的jar包，拷贝到容器指定的目录中，并更名
COPY nike-mall-1.0-SNAPSHOT.jar /opt/erick/service/nike-mall.jar

# 设置一个Linux的环境变量, 可以通过spring-java代码获取拿到该值
ENV Base_Url 'nike.com'
# 获取并打印Linux的该环境变量
RUN echo ${Base_Url}

# 按照java -jar nike-mall.jar来运行
# 用全路径，否则要设置jdk的环境变量
ENTRYPOINT ["java","-jar","/opt/erick/service/nike-mall.jar"]
```

![image-20250113222815022](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250113222815022.png)

### 运行

```bash
# 进入对应的Dockerfile和jar包的路径
docker build -t nike-mall:2.0.0.snapshot .

# 运行 9090: 上面expose的端口，    9091：linux的宿主机端口
docker run --name second-project  -d -p 9091:8080 67a5b91c0c0d

# 查看
http://39.105.210.163:9091/home/test

# 结果
@GetMapping("/test")
public String getName() {
    System.out.println(env);
    return env.getProperty("Base_Url");
}
// nike.com
```

# Network



# Docker-Compose

## 简介

- Docker官方的开源项目，负责实现对Docker容器集群的快速编排
- 管理多个Docker容器，组成一个应用。同时注意各个组件的启动顺序
- 定义一个docker-compose.yaml， 配置好多个容器之间的调用关系

```bash
# 一个容器，一个服务
- docker容器本身占用资源极少，所以最好是将每个服务单独的分割开来

# docker-compose
- 假如一个项目用到了多个微服务，mysql，redis， 统一编排一个docker-compose.yaml文件，定义一组相互关联的容器编排

# 安装： 安装上面的docker-engine的时候已经自己带了
docker compose version          # Docker Compose version v2.27.1
docker -v                       # Docker version 26.1.4, build 5650f9b

# 两要素
- 文件：     docker-compose.yml
- service：  一个个应用容器实例，比如不同的多个微服务
- project：  一组关联的应用容器，组成的一个完整业务单元

# 常用命令
docker-compose up                     # 启动所有docker-compose服务
docke-compose up -d                   # 启动所有docker-compose服务并后台运行
dokcer-compose down                   # 停止并删除容器，网络，卷，镜像
docker-compose restart/start/stop     # 重启/启动/停止
```

## 微服务编排

# Portainer

- 轻量级的docker可视化工具
- [官网安装](https://docs.portainer.io/start/install-ce/server/docker/linux)

```bash
# 数据卷
docker volume create portainer_data

# 安装
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:2.21.5

# 登陆
https://39.105.210.163:9443

# 初始化用户名和密码
username：erick
password：shuzhan19970701
```

![image-20250114203305246](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250114203305246.png)

![image-20250114203546242](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250114203546242.png)

