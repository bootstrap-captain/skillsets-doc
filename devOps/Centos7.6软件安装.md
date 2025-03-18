# Tomcat

## Docker

```bash
# 拉
docker pull tomcat

# 运行
docker run -d -p 8080:8080 --name my_tomcat tomcat

# 查看对应容器的目录: 修改文件，因为tomcat的docker版本存在的问题
ls -l
rm -rf webapps        
mv webapps.dist webapps

# 浏览器访问
http://39.105.210.163:8080/
```



# JDK17/8

## 1. 下载

- [官网下载](https://www.oracle.com/java/technologies/downloads/#java17)

![image-20220901003504136](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20220901003504136.png)

## 2. 安装

```bash
# 在阿里云服务器上
cd usr/local
mkdir java17

#  使用mac的sftp上传文件到
put /Users/shuzhan/Desktop/jdk-17_linux-x64_bin.tar.gz /usr/local/java17

# 解压文件
tar -zxvf jdk-17_linux-x64_bin.tar.gz 
```

## 3. 配置环境变量

```bash
vim /etc/profile

# 找到export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL，在下面写上：

#set java environment
export JAVA_HOME=/usr/local/java17/jdk-17.0.4.1
export CLASSPATH=.:$JAVA_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
```

## 4. 刷新环境变量

```bash
# 刷新环境变量
source /etc/profile

java -version

java version "17.0.4.1" 2022-08-18 LTS
Java(TM) SE Runtime Environment (build 17.0.4.1+1-LTS-2)
Java HotSpot(TM) 64-Bit Server VM (build 17.0.4.1+1-LTS-2, mixed mode, sharing)

# 如果切换了java 版本
- 需要重新打开terminal，不然可能出现查看java -version可能没改过来
```



# 安装Kafka

- 集群版本：1台zookeeper，3台kafka组成集群
- Kafka版本： 3.6.1

## 1. 启动zookeeper

```bash
# 1. 在服务器上139.224.249.175 安装zookeeper    2G内存
docker pull zookeeper:3.8
docker run --privileged --name zookeeper -p 2181:2181 --restart always -d 36c607e7b14d
```

## 2. Kafka集群安装

- 相同步骤，部署三台kafka，但保证对应的broker.id不同即可，本文分别为111， 222， 333
- 内存最少为1G

### 2.1 安装

- kafka的broker端是用scala写的
- 2.12/2.13代表的是scala的版本， 后面的3.6.1代表的是kafka版本

![image-20241109163233198](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20241109163233198.png)

### 2.2 上传

- Kafka的运行需要有java环境，选择安装java17
- 阿里云服务器     139.224.192.251    47.100.49.77       106.14.127.122

```bash
# 第一台服务器
cd usr/local
mkdir kafka

# 1. 上传文件并解压
put /Users/shuzhan/Desktop/kafka_2.13-3.6.1.tgz /usr/local/kafka
tar -zxvf kafka_2.13-3.6.1.tgz
```

```bash
# 目录解释
LICENSE 
NOTICE  
bin               # kafka执行时候的启动脚本
config            # kafka对应的参数
libs              # kafka内部运行所需要的jar包
licenses     
site-docs
```

### 2.3 修改配置文件

- /usr/local/kafka/kafka_2.13-3.6.1/config: Kafak的的所有的配置文件
- server.properties： 该台kafka server端对应的参数

```bash
# The id of the broker. This must be set to a unique integer for each broker.
# 该kafka在整个集群中的身份唯一表示
broker.id=111

# A comma separated list of directories under which to store log files
# 存储具体的kafka的message的地方，为自建目录
# 如果当前该目录不存在，启动的时候会自动创建
log.dirs=/usr/local/kafka/data

# 外部访问时候的ip和端口，对应的ip为本机的ip
advertised.listeners=PLAINTEXT://120.77.156.53:9092

# zookeeper的配置：zookeeper的ip和端口
zookeeper.connect=39.105.210.163:2181
```

### 2.4 环境变量

```bash
vim /etc/profile

export KAFKA_HOME=/usr/local/kafka/kafka_2.13-3.6.1
export PATH=$PATH:$KAFKA_HOME/bin

# 刷新环境变量
source /etc/profile
```

### 2.5 kafka启动

```bash
# 1. 启动
# 进入对应目录
cd usr/local/kafka/kafka_2.13-3.6.1

# kafka-server-start.sh： 启动的脚本
# -daemon： 后台启动
# config/server.properties: 按照指定的配置文件启动
bin/kafka-server-start.sh -daemon config/server.properties

# 查看启动进程, kafka也是java进程
jps
```

# Maven-3.9.9

- [官网下载](https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/)

- Maven依赖Java

```bash
# 上传到usr/local目录下并解压
put /Users/shuzhan/Desktop/apache-maven-3.9.9-bin.tar.gz /usr/local
tar -zxvf apache-maven-3.9.9-bin.tar.gz

# 查看版本， 出现下面代表安装成功
/usr/local/apache-maven-3.9.9/bin/mvn -v


Maven home: /usr/local/apache-maven-3.9.9
Java version: 17.0.4.1, vendor: Oracle Corporation, runtime: /usr/local/java17/jdk-17.0.4.1
Default locale: en_US, platform encoding: ANSI_X3.4-1968
OS name: "linux", version: "3.10.0-957.21.3.el7.x86_64", arch: "amd64", family: "unix"
```

-  修改maven的对应的setting.xml

```xml
<mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
 </mirror>
```

# Nexus私服

## 1. 安装

- Nexus的运行需要Java环境，先在Linux上安装Java(3.52.0.01要求安装Java8环境)

```bash
# 上传
put /Users/shuzhan/Desktop/nexus-3.52.0-01-unix.tar.gz /usr/local 
tar -zxvf nexus-3.52.0-01-unix.tar.gz

# 启动
cd /usr/local/nexus-3.52.0-01/bin
./nexus run       # 前台启动，能够实时查看日志，但是退出命令行就会终止
./nexus start     # 后台启动(推荐)，但不能看到日志
./nexus status    # 查看状态

# 参数设置
# 1. 参数一：内存: nexus-3.52.0-01/bin/nexus.vmoptions
- nexus中默认是2g
    # 修改前
    -Xms2703m
    -Xmx2703m
    -XX:MaxDirectMemorySize=2703m

    # 修改后
    -Xms528m
    -Xmx528m
    -XX:MaxDirectMemorySize=1024m
    
# 2. 参数二：浏览器端口：/nexus-3.52.0-01/etc/nexus-default.properties
- 默认端口： 8081
- application-port=8081

# error:  Detected execution as "root" user.  This is NOT recommended!
# 需要去/bin/nexus中修改配置,
- vi环境下可以使用/run_as_root搜索
- run_as_root=true   改为  run_as_root=false
```

## 2. 访问

```bash
# 1. 访问url
- http://60.205.229.31:8081/

# 2. 默认用户名和密码， 可以进行密码修改
- user：     admin
- password： 根据页面提示，在对应的linux的目录中去找password
# Disable anonymous access： # 选择禁用匿名访问
```

### 2.1 添加用户

- 添加一个用户： user:erick     password:19920507, 赋予admin权限

![image-20230424182949968](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230424182949968.png)

## 3. 仓库类型

- 拉取的jar包会保存在maven-central和maven-public中，我们项目中配置public的mirror即可

```bash
# maven-central
- Nexus对Maven中央仓库的代理，默认是取中央仓库拉的
- 设置镜像仓库，设置成阿里云的

# maven-public
- Nexus默认创建的，供开发下载使用的仓库
- 实际下载的jar包就会存在这里

# maven-release
- Nexus默认创建，供开发部署自己的jar，要求版本以release结尾

# maven-snapshots
- Nexus默认创建，供开发部署自己的jar，要求版本以snapshots结尾

# proxy
- 某个远程仓库的代理

# group
- 通过Nexus获取的第三方jar包

# hosted
- 本团队开发人员部署到Nexus的jar包
```

![image-20230424174044569](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230424174044569.png)

![image-20230424180729155](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230424180729155.png)

![image-20230424182322668](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230424182322668.png)

## 4. 公共jar包-本地拉取配置

- 配置setting.xml文件拉去公共的jar包
- 去对应的mirror拉代码时候，根据url找到对应的私服仓库，根据id找到对应的username和password来进行权限认证访问

```xml
  <servers>
    <server>
      <!--server的ID必须和对应的私服的mirror的ID相同-->
      <id>erick-nexus</id>
      <username>erick</username>
      <password>19920507</password>
    </server>
  </servers>

  <mirrors>
    <!--自己搭建的私服的repo-->
    <mirror>
      <id>erick-nexus</id>
      <name>erick-nexus</name>
      <url>http://60.205.229.31:8081/repository/maven-public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
```

## 5. 项目上传发布

- 通过IDEA来执行maven deploy

### member

- 如果配置了maven-public的member，则member添加jar包时，同时也会在**maven-public**存一份。如果取消了maven-public的member，则会立刻删除掉对应的jar包

![image-20230424221731984](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230424221731984.png)

### pom.xml

![image-20230424225135087](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230424225135087.png)

### 5.1  SNAPSHOTS

- pom中对应项目的version只要包含 SNAPSHOT都会发布到maven-snapshots仓库
- SNAPSHOTS发布时候，会自动加上时间戳作为备份

```bash
# Deployment policy: Allow redeploy
- 同一个jar包相同版本可以部署多次
```

![image-20230424214231603](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230424214231603.png)

### 5.2 RELEASES

- version如果不包含SNAPSHOT字符，都会自动发布到maven-releases仓库
- 建议release的jar包带上RELEASE，比如1.0.0-RELEASE

```bash
# Deploy Policy: Disable redeploy,  不允许deploy相同版本的jar包
status: 400 Repository does not allow updating assets: maven-releases
```

![image-20230424223749793](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230424223749793.png)



## 6. 项目拉取

### 6.1 maven-public

- 如果SNAPSHOT和RELEASE的jar包存在于maven-public，则直接可以去maven-public拉

#### setting.xml

```bash
# 在服务器上的maven和本地的maven配置，作为全局配置，项目pom.xml可以不加任何配置上面配置的已经生效
- 缺点： 强制配置setting.xml
- 优点： 较为安全，不会出现user和password暴露在pom.xml中
```

```xml
<servers>
    <server>
      <!--server的ID必须和对应的私服的mirror的ID相同-->
      <id>erick-nexus</id>
      <username>erick</username>
      <password>19920507</password>
    </server>
  </servers>

  <mirrors>
    <!--自己搭建的私服的repo-->
    <mirror>
      <id>erick-nexus</id>
      <name>erick-nexus</name>
      <url>http://60.205.229.31:8081/repository/maven-public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
```

```bash
# 引用时用原来项目的版本号，不能通过时间戳对应的版本来拉
- 会自动拉取maven-public中最新时间戳的jar包
- 如果jar包更新了，本地重新reimport即可，不用删除本地repo的包

        <dependency>
            <groupId>com.erick.daydreamer</groupId>
            <artifactId>erick-haha</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```

### 6.2 maven-xxx

- 如果maven-public没有对应的SNAPSHOT和RELEASE的member，则不能从maven-public拉，只能从对应的仓库去找

**setting.xml**

```xml
   <?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!--
 | This is the configuration file for Maven. It can be specified at two levels:
 |
 |  1. User Level. This settings.xml file provides configuration for a single user,
 |                 and is normally provided in ${user.home}/.m2/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -s /path/to/user/settings.xml
 |
 |  2. Global Level. This settings.xml file provides configuration for all Maven
 |                 users on a machine (assuming they're all using the same Maven
 |                 installation). It's normally provided in
 |                 ${maven.conf}/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -gs /path/to/global/settings.xml
 |
 | The sections in this sample file are intended to give you a running start at
 | getting the most out of your Maven installation. Where appropriate, the default
 | values (values used when the setting is not specified) are provided.
 |
 |-->
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 https://maven.apache.org/xsd/settings-1.2.0.xsd">

    <localRepository>/Users/shuzhan/Documents/repo</localRepository>

    <pluginGroups></pluginGroups>

    <proxies></proxies>

    <mirrors>
        <!--自己搭建的私服的repo，可以配置多个私服-->
        <mirror>
            <id>erick-nexus</id>
            <name>erick-nexus</name>
            <url>http://60.205.229.31:8081/repository/maven-public/</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>

    <servers>
        <server>
            <!--server的ID必须和对应的私服的mirror的ID相同-->
            <id>erick-nexus</id>
            <username>erick</username>
            <password>19920507</password>
        </server>
        <server>
            <!--snapshot的server-->
            <id>erick-nexus-public</id>
            <username>erick</username>
            <password>19920507</password>
        </server>

        <server>
            <!--snapshot的server-->
            <id>erick-nexus-snapshot</id>
            <username>erick</username>
            <password>19920507</password>
        </server>

        <server>
            <!--release的server-->
            <id>erick-nexus-release</id>
            <username>erick</username>
            <password>19920507</password>
        </server>
    </servers>

    <profiles>
        <profile>
            <!--id随意,后面要通过该id激活该profile-->
            <id>erick-repo</id>
            <!--有几个repo，就有几个repo的server验证信息
               1. repo的id不能重复
               2. 拉的时候遍历几个repo去拉-->
            <repositories>
                <!--配置snapshots的拉取权限-->
                <repository>
                    <!--id要和对应的server对应-->
                    <id>erick-nexus-snapshot</id>
                    <name>erick-snapshot-repo</name>
                    <url>http://60.205.229.31:8081/repository/maven-snapshots/</url>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </repository>
                <!--配置release的拉取权限-->
                <repository>
                    <!--id要和对应的server对应-->
                    <id>erick-nexus-release</id>
                    <name>erick-release-repo</name>
                    <url>http://60.205.229.31:8081/repository/maven-releases/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                </repository>
            </repositories>
        </profile>
    </profiles>

    <activeProfiles>
        <!--激活对应的profile-->
        <activeProfile>erick-repo</activeProfile>
    </activeProfiles>

</settings>
```

