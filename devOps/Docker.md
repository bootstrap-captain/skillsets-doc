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

# 设置稳定仓库
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
- 可以配置阿里云镜像作为仓库

```bash
# 登陆阿里云 --> 容器镜像服务 --> 镜像工具 --> 镜像加速器
# 拷贝对应的镜像地址
vim /etc/docker/daemon.json

{
  "registry-mirrors": ["https://xxxxxr.mirror.aliyuncs.com"]
}

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
docker rmi -f bd54813ab48c                        # rm images

# 先获取所有的imageid，然后全部强制删除
docker rmi -f $(docker images -qa)
```

## 容器命令

### 后台启动

- 后台启动的容器，为守护使容器

```bash
# 启动
#   -d： 以后台模式打开一个容器，不打开命令终端
#   -p： 端口映射 9001:9000,  左边=linux宿主机端口   右边=docker内部，该容器的端口
#   --name: 自定义名字，否则为系统随机分配
#   镜像： 可以用镜像id/镜像名称(加tag)
docker run -d --name erick_redis -p 6379:6379 41de2cc0b30e
docker run -d --name lucy_redis -p 6380:6379 redis:6.0.7
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

### docker--linux文件拷贝

```bash
# 将容器内的文件，拷贝到linux的宿主机上
docker cp c0bd7fd3f2e8:/data/dump.rdb /tmp

# 将linux宿主机上的东西拷贝到容器内
docker cp /tmp/erick.txt c0bd7fd3f2e8:/data
```

### 容器备份

```bash
# 将整个容器进行导出到本地为tar包，会导出到命令所在目录
docker export c0bd7fd3f2e8 > erick.tar

# 将tar包解压为新的image镜像
cat erick.tar | docker import - daydreamer/redis:0.0.1.snapshot
```

![image-20250110113304777](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250110113304777.png)

# 自定义镜像





# Dockerfile

- 用来构建docker镜像的文本文件，是由一条条构建镜像所需要的指令和参数构成的脚本文件

## 1. 简介

### 1.1 基础知识

- 每条保留字指令必须为大写字母，后面要至少跟随一个参数
- 指令按照从上到下，顺序执行
- \# 表示注释
- 每条指令都会创建一个新的镜像层并对镜像进行提交

### 1.2 构建流程

- docker从基础镜像运行一个容器
- 执行一条指令并对容器做出修改
- 执行类似docker commit的操作提交一个新的镜像层
- docker再基于刚提交的镜像运行一个新容器
- 执行dockerfile中的下一条指令直到所有指令执行完毕

### 1.3 角色

- Dockerfile：软件的原材料， 面向开发
- Docker镜像： 软件的交付品，交付标准
- Docker容器：软件镜像的运行台，部署与运维

## 2. 保留字

```bash
# 基础镜像：当前新镜像是基于哪个镜像的，指定一个已经存在的镜像作为模板
# 第一条必须是FROM
FROM

# 镜像维护者的姓名和邮箱地址
MAINTAINER

# 等同于在终端操作的shell命令，容器构建时需要运行的命令
# 两种格式： shell格式和exec格式
# RUN 是在docker build时运行
RUN<命令行命令>

# 当前容器对外暴露出的端口
EXPOSE

# 指定在创建容器后，终端默认登录的进来工作目录，一个落脚点
WORKDIR

# 指定该镜像是以什么样的用户去执行，默认为root，权限相关
USER

# 配置环境变量
# 可以在后续的任何RUN指令中使用，
ENV

# 将宿主机目录下的文件拷贝进镜像，且会自动处理URL和解压tarr压缩包
ADD

# 将宿主机目录下的文件拷贝进镜像
COPY

# 容器数据卷，用于数据保存和持久化工作
VOLUME

# 指定容器启动后要做的事情
# Dockerfile中可以有多个CMD指令，但只有最后一个生效
# CMD会被docker run之后的参数替换， 为用户提供了自定义参数
CMD

# 指定容器启动时需要的命令
# 类似CMD，但ENTRYPOINT不会被docker run后面的命令覆盖
# 可变参数时： CMD就是给ENTRYPOINT传参
ENTRYPOINT
```

# Docker微服务

## 1. 本地服务打包

### 1.1 服务依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.erick</groupId>
    <artifactId>guli-mall</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <spring-boot>2.7.3</spring-boot>
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
        <!-- ......用于扫描 dao 文件下的mapper 文件................. start -->
        <resources>
            <!-- 编译 src/main/java 目录下的 mapper 文件 -->
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>

            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*</include>
                </includes>
            </resource>
        </resources>
        <!-- ......用于扫描 dao 文件下的mapper 文件................. end -->

        <plugins>
            <!--maven package的插件-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!--添加配置跳过测试-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.1</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
            <!--添加配置跳过测试-->
        </plugins>
    </build>

</project>
```

### 1.2  maven打包

- 打包得到对应的jar包
- 打包后后，通过java -jar 来验证是否可以本地正常启动
- 默认端口为8080

![image-20220831170740333](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20220831170740333.png)

### 1.3 Dockerfile

```bash
# 具体的版本信息，可以从dockerhub镜像上找
# 基础镜像是该jdk,将该jdk安装在docker中
FROM openjdk:17.0-jdk

# 将该jar包文件，拷贝到容器中，并更名
# 是因为后续会将Dockerfile和对应java包放在服务器同一个目录下
COPY guli-mall-1.0-SNAPSHOT.jar guli-erick.jar

# 按照java -jar guli-erick.jar来运行
ENTRYPOINT ["java","-jar","guli-erick.jar"]
```

## 2. 服务器构建

### 2.1 上传文件

- 将对应的jar包和Dockerfile上传到阿里云服务器的同一个目录下

```bash
 # mac的sftp功能
 - put source target   # 上传
 - get source target   # 下载
```

### 2.2 构建镜像

```bash
# 进入到阿里云服务器Dockerfile和jar包的目录
# nike-guli:     对应的docker image的名字
# 1.0:           版本号
# .  :           不能缺少, 代表在当前目录下
docker build -t nike-guli:1.0 .
```

![image-20220831181610088](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20220831181610088.png)

### 2.3 运行容器

```bash
# 容器里面启动的是8080端口，9090是对外访问的端口
docker run --name balance-service -d -p 9090:8080 51b6019f5c15

# 访问路径
http://60.205.229.31:9090/user/date
```

# Docker-Compose

## 1. 简介

- Docker官方的开源项目，负责实现对Docker容器集群的快速编排
- 管理多个Docker容器，组成一个应用。同时要注意各个组件的启动顺序
- 定义一个docker.compose.yaml， 写好多个容器之间的调用关系。通过一个命令，同时启动/关闭这些容器

## 2. 常用命令

```bash
docker-compose -version               # 查看版本

docker-compose up                     # 启动所有docker-compose服务
docke-compose up -d                   # 启动所有docker-compose服务并后台运行
dokcer-compose down                   # 停止并删除容器，网络，卷，镜像
docker-compose restart/start/stop     # 重启/启动/停止
```

