- 阿里Centos 7.6安装

# 安装Docker

- Docker并非一个通用的容器工具，依赖于已经存在并运行的Linux内核环境，在Linux下制造了一个隔离的文件环境

```bash
# 1. 安装 Docker 依赖的软件包
sudo yum install -y yum-utils

# 2. 设置Docker远程仓库


# 2. 查看版本
docker -v
docker version
```

# 安装JDK17/8

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

![image-20220901004521435](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20220901004521435.png)

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

# 安装Redis

## 单机版-7.2.4

- [官网下载](https://redis.io/download)一个redis
- 版本地址：https://download.redis.io/releases/

```bash
# 1. 上传文件到linux
put /Users/shuzhan/Desktop/redis-7.2.4.tar.gz /opt
tar -zxvf redis-7.2.4.tar.gz

# 2. . 安装
cd /opt/redis-7.2.4
make && make install
# 提示。Hint: It's a good idea to run 'make test' ;  表示安装成功

# 3. 最终安装位置 ： /usr/local/bin/
cd /usr/local/bin/
ll

-rwxr-xr-x 1 root root 5092624 Mar 29 22:42 redis-benchmark                   # 性能测试工具
lrwxrwxrwx 1 root root      12 Mar 29 22:42 redis-check-aof -> redis-server   # 修复有问题的aof
lrwxrwxrwx 1 root root      12 Mar 29 22:42 redis-check-rdb -> redis-server   # 修复有问题的dump.rdb文件
-rwxr-xr-x 1 root root 5188792 Mar 29 22:42 redis-cli                         # 客户端，操作入口
lrwxrwxrwx 1 root root      12 Mar 29 22:42 redis-sentinel -> redis-server    # redis集群使用
-rwxr-xr-x 1 root root 8916584 Mar 29 22:42 redis-server                      # redis服务器启动命令

# 4. 修改配置文件, 必须重启redis服务才能生效
#    自定义配置文件
cd /opt/redis-7.2.4
mkdir erick_redis
cp redis.conf erick_redis/erick_redis.conf
vim erick_redis.conf

bind 127.0.0.1 -::1     # 默认redis只能本地cli访问。注释掉该行，或者改成本机IP,否则会影响远程ip链接
protected-mode no       # 默认yes，开启保护模式，限制为本地访问
daemonize yes           # 守护进程，默认no，改为yes
databases 10            # 默认为16，数据库个数，可以改成10验证配置文件是否生效
requirepass 123456      # 添加密码

# 5. 启动服务, 在任意目录下，通过配置文件启动服务。 redis-server是一个系统级别的指令
#    启动的是redis-server
cd /opt/redis-7.2.4/erick_redis
redis-server erick_redis.conf
ps -ef|grep redis|grep -v grep     # 验证是否成功
root     30490     1  0 23:08 ?        00:00:00 redis-server *:6379

# 6. 查看redis版本
redis-server -v
Redis server v=7.2.4 sha=00000000:0 malloc=jemalloc-5.3.0 bits=64 build=a203381984fddfe4

# 7. 直接打开一个redis client
 redis-cli
 ping  # 验证是否链接成功
 auth 123456

# 7. 用IDEA配置的用户名和密码进行登录
username: default
password: 123456
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

# MongoDB

- MongoDB Community Server

## 1. 单机版

- [官方安装文档](https://www.mongodb.com/try/download/community)

![image-20250107195332229](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250107195332229.png)

### 1.1 上传文件

```bash
# 上传
put /Users/shuzhan/Desktop/mongodb-linux-x86_64-rhel70-7.0.16.tgz /opt

# 解压
tar -xvf mongodb-linux-x86_64-rhel70-7.0.16.tgz

# 目录结构
-rw-r--r-- 1 root root  30608 Apr 24 23:50 LICENSE-Community.txt
-rw-r--r-- 1 root root  16726 Apr 24 23:50 MPL-2
-rw-r--r-- 1 root root   1978 Apr 24 23:50 README
-rw-r--r-- 1 root root 122512 Apr 24 23:50 THIRD-PARTY-NOTICES
drwxr-xr-x 2 root root   4096 May 18 11:10 bin


# 移动解压后的文件夹到指定目录
mv mongodb-linux-x86_64-rhel70-7.0.16 /usr/local/mongodb

# 新建几个目录，用来存储日志和数据
cd /usr/local/mongodb
mkdir -p erick/data/db                # 存储数据
mkdir -p erick/log                    # 存储日志
```

### 1.2 mongodb.conf 

- [配置文件](https://www.mongodb.com/docs/manual/reference/configuration-options/)

```conf
systemLog:
   destination: file                                  # 以文件的形式输出日志
   path: "/usr/local/mongodb/erick/log/mongod.log"    # 具体的绝对路径
   logAppend: true                                    # mongo重启时，日志附加到原文件末尾

storage:
   dbPath: "/usr/local/mongodb/erick/data/db"         # 存储数据的目录

processManagement:
   fork: true                                          # 后台启动

net:
   bindIp: 172.17.56.24                               # 局域网ip，非公网IP
   port: 27017
```

### 1.3 启动

```bash
# 上传配置文件
put /Users/shuzhan/Desktop/mongodb.conf /opt

# 启动
/usr/local/mongodb/bin/mongod -f /opt/mongodb.conf 

- 如果启动失败，一般是配置文件出错，去对应的log里面查找即可

# 查看
ps -ef | grep mongod

# 远程DataGrip连接
- 连接的IP为公网IP
```

![image-20240518120043762](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240518120043762.png)

## 2. 安全认证

- 默认的mongodb启动后，是不需要安全认证的
- mongodb提供了基于用户的权限认证
- 通过datagrip登陆进去

```bash
use admin;                 #  切换到admin
show collections;          #  包含 system.version 的这个collection
db.system.version.find();   

# 结果
[
  {
    "_id": "featureCompatibilityVersion",
    "version": "7.0"
  }
]
```

### 2.1 管理员用户

```bash
# root: 超级权限
db.createUser({user:"erickroot", pwd:"123456", roles:[{"role":"root", "db":"admin"}]});
show collections;           # 在admin库创建了一个system.users的collection
db.system.users.find();

# userAdminAnyDatabase: 在指定数据库创建和修改用户(除了config和local库存)
db.createUser({user:"erickadmin", pwd:"123456", roles:[{"role":"userAdminAnyDatabase","db":"admin"}]});
show collections;           # 在admin库的system.users添加用户
db.system.users.find();

# 删除用户
db.dropUser("erickadmin");

# 修改密码
db.changeUserPassword("erickadmin","123456");
```

### 2.2 关闭服务端

```bash
ps -ef |grep mongo         # 查看端口

kill -9 1741               # 杀死进程

# 在配置文件中：开启认证
```

### 2.3 修改配置文件

```bash
systemLog:
   destination: file                                  # 以文件的形式输出日志
   path: "/usr/local/mongodb/erick/log/mongod.log"    # 具体的绝对路径
   logAppend: true                                    # mongo重启时，日志附加到原文件末尾

storage:
   dbPath: "/usr/local/mongodb/erick/data/db"         # 存储数据的目录

processManagement:
   fork: true                                          # 后台启动

net:
   bindIp: 172.17.56.24                               # 局域网ip，非公网IP
   port: 27017

security:
  authorization: enabled                             # 开启权限认证
```

### 2.4 重启服务端

```bash
# 上传配置文件
put /Users/shuzhan/Desktop/mongodb.conf /opt

# 启动
/usr/local/mongodb/bin/mongod -f /opt/mongodb.conf 
```

# MYSQL

## 1. 单机版

- 环境：CentOS7.6，MySQL 8.0.27
- [MySQL社区版](https://downloads.mysql.com/archives/community/)

![image-20230731163537301](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230731163537301.png)

### 1.1 上传文件

```bash
# mac本地, 将文件上传到linux服务器的/opt目录下
put /Users/shuzhan/Desktop/mysql-8.0.27-1.el7.x86_64.rpm-bundle.tar /opt

# 解压对应的文件得到多个文件,标注1的是需要的
tar -xvf mysql-8.0.27-1.el7.x86_64.rpm-bundle.tar

mysql-community-client-8.0.27-1.el7.x86_64.rpm            # 1
mysql-community-client-plugins-8.0.27-1.el7.x86_64.rpm    # 1
mysql-community-common-8.0.27-1.el7.x86_64.rpm            # 1
mysql-community-devel-8.0.27-1.el7.x86_64.rpm
mysql-community-embedded-compat-8.0.27-1.el7.x86_64.rpm
mysql-community-libs-8.0.27-1.el7.x86_64.rpm              # 1
mysql-community-libs-compat-8.0.27-1.el7.x86_64.rpm
mysql-community-server-8.0.27-1.el7.x86_64.rpm            # 1 
mysql-community-test-8.0.27-1.el7.x86_64.rpm

# 在当前目录下，创建目录
mkdir erick_mysql

# 将上面标注了1的文件，都拷贝到erick_mysql下面
cp mysql-community-c* erick_mysql/
cp mysql-community-libs-8.0.27-1.el7.x86_64.rpm erick_mysql
cp mysql-community-server-8.0.27-1.el7.x86_64.rpm erick_mysql
```

### 1.2 CentOS7检查MySQL依赖

```bash
# 检查/tmp临时目录权限(必不可少)
- mysql在安装过程中，会通过mysql用户在/tmp目录下新建 tmp_db文件，所以请给/tmp较大权限
cd /
chmod -R 777 /tmp

# 检查存在libaio依赖, 存在net-tools依赖
# 存在时候的结果： libaio-0.3.109-13.el7.x86_64,     net-tools-2.0-0.25.20131004git.el7.x86_64
rpm -qa | grep libaio 
rpm -qa | grep net-tools

# 不存在的话安装
yum install -y libaio
```

### 1.3 安装

- 严格按照下面rpm顺序

```shell
cd opt/erick_mysql/
rpm -ivh mysql-community-common-8.0.27-1.el7.x86_64.rpm

rpm -ivh mysql-community-client-plugins-8.0.27-1.el7.x86_64.rpm

# 执行时候会
# warning: mysql-community-libs-8.0.27-1.el7.x86_64.rpm: Header V3 DSA/SHA256 Signature, key ID 5072e1f5:  # # NOKEY
# error: Failed dependencies:
# 	mariadb-libs is obsoleted by mysql-community-libs-8.0.27-1.el7.x86_64
# 先删除之前安装的lib
yum remove mysql-libs
rpm -ivh mysql-community-libs-8.0.27-1.el7.x86_64.rpm

rpm -ivh mysql-community-client-8.0.27-1.el7.x86_64.rpm

rpm -ivh mysql-community-server-8.0.27-1.el7.x86_64.rpm

# 安装完毕后，检查安装是否成功
# mysql  Ver 8.0.27 for Linux on x86_64 (MySQL Community Server - GPL)
mysql --version
mysqladmin --version
```

### 1.4 服务初始化

- 为了保证数据库目录与文件的所有者为mysql登录用户，如果是以root身份运行mysql服务，需要执行命令初始化

```bash
# 默认为root用户生成一个密码并将该密码标记为过期，登录后需要设置一个新的密码
# 生成的临时密码会保存在 /var/log/mysqld.log
mysqld --initialize --user=mysql
cat /var/log/mysqld.log   # -CaBeiZa>1)k

# 启动mysql
systemctl start mysqld
systemctl stop mysqld
systemctl restart mysqld

# 查看是否启动
systemctl status mysqld

# 查看虚拟机启动后，mysql是否自启动：  enabled代表是
systemctl list-unit-files | grep mysqld.service

# 如果不是，设置为自启动
systemctl enable mysqld.service

# 关闭自启动
systemctl disable mysqld.service
```

### 1.5 登录

```bash
# 输入密码
mysql -u root -p

# 执行任意sql，则报错
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.

# 先修改密码
alter user 'root'@'localhost' identified by 'erick_123456';

# 退出mysql，重新使用新密码登录
show databases;
```

### 1.6 远程连接

```sql
# 默认的root用户，只能是localhost来访问
USE mysql;
select host, user from user;

# 修改一下
UPDATE user SET host='%' WHERE user='root';

# 一定要刷新一下
FLUSH privileges;
```

# 安装**Maven3.6.3**

## 1. 下载

- [官网下载](https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/)

## 2. 上传安装

- Maven依赖Java环境，因此必须先下载Java并配置环境变量

```bash
# 上传到usr/local目录下并解压
put /Users/shuzhan/Desktop/apache-maven-3.6.3-bin.tar.gz /usr/local
tar -zxvf apache-maven-3.6.3-bin.tar.gz

# 查看版本， 出现下面代表安装成功
/usr/local/apache-maven-3.6.3/bin/mvn -v


Maven home: /usr/local/apache-maven-3.6.3
Java version: 17.0.4.1, vendor: Oracle Corporation, runtime: /usr/local/java17/jdk-17.0.4.1
Default locale: en_US, platform encoding: ANSI_X3.4-1968
OS name: "linux", version: "3.10.0-957.21.3.el7.x86_64", arch: "amd64", family: "unix"
```

## 3. 镜像配置

-  修改maven的对应的setting.xml

```
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

# Jenkins安装

- Jenkins是一个war包，因此服务器需要先安装JDK

- Jenkins需要用Maven进行打包，所以服务器需要先下载Maven

- Jenkins需要用到git拉代码，因此需要先下载Git

- CICD: continuous integration, continuous deploy 

## 1.  下载

- [官网war包](https://www.jenkins.io/download/)
- version：2.361.2

```bash
# 将对应war包上传到Linux,
put /Users/shuzhan/Desktop/jenkins.war /usr/local
```

## 2. 启动

```bash
# 后台启动Jenkins
#  & 表示后台启动     nohup 是centos的后台启动的指令
# >jenkins.log： 在命令执行的目录下，生成jenkins.log文件，并将启动日志写入
# >  >> 文件写入流的两种方式    > 覆盖              >> 追加
#  2>&1     2表示错误输出，1表示标准输出      错误输出和标准输出都写进来
# 如果不写输出日志，则命令会卡住，按回车后会将日志默认写入到nohup.log中
nohup java -jar jenkins.war --httpPort=8081 >jenkins.log 2>&1 &

# jps查看是否启动成功
jps
```

```bash
#  如果出现下面错误，是因为缺少字体的原因
yum install fontconfig
fc-cache --force
```

![image-20220901014435411](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20220901014435411.png)

## 3. 访问

```bash
# 通过浏览器访问, 第一次进入需要admin的密码
http://119.23.242.170:8081/

# 根据红字提示查看密码
# 输入密码，等待响应，第一次加载时间会比较慢

# 其他安装
- 按照提示安装指定插件
- 创建admin用户
```

![image-20220901015413967](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20220901015413967.png)

#  Git安装

```bash
yum install git

git --version

git version 1.8.3.1
```
