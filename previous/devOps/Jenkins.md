**CICD: continuous integration, continuous deploy** 

# 安装

- war包，需要用Maven打包，git拉代码
- JDK(17)，Maven(3.9.9)，Git

## 单节点

### 安装包

- [官网war包](https://www.jenkins.io/download/)
- version：2.479.3

```bash
# 将对应war包上传到Linux
put /Users/shuzhan/Desktop/jenkins.war /usr/local

#  安装必要的字体，否则会出现下面的问题
yum install fontconfig
fc-cache --force

# 后台启动Jenkins
#  &: 后台启动     nohup 是centos的后台启动的指令
# >jenkins.log： 在命令执行的目录下，生成jenkins.log文件，并将启动日志写入
# >  >> 文件写入流的两种方式    > 覆盖              >> 追加
#  2>&1     2表示错误输出，1表示标准输出      错误输出和标准输出都写进来
# 如果不写输出日志，则命令会卡住，按回车后会将日志默认写入到nohup.log中
cd /usr/local
nohup java -jar jenkins.war --httpPort=8081 >jenkins.log 2>&1 &

# jps查看是否启动成功
jps
```

![image-20250118211801867](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250118211801867.png)

```bash
# 通过浏览器访问, 第一次进入需要admin的密码
http://101.200.241.88:8081/

# 根据红字提示查看密码
# 输入密码，等待响应，第一次加载时间会比较慢

# 其他安装
- 按照提示安装指定插件
- 创建admin用户
```

# 安装插件

## Maven

### 1. 安装

![image-20250118215802317](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250118215802317.png)

- 安装完成后，就会在Item中有Maven项目(默认没有)

![image-20250118220419399](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250118220419399.png)

### 2. 配置路径

- 需要手动在Jenkins中配置上，本地Linux的Maven的路径
- Dashboard--Manage Jenkins--Tools

![image-20250118223455217](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250118223455217.png)

## GIT-配置

### HTTPS

- 国内GIT仓库，可以直接拉代码

![image-20250120212019086](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250120212019086.png)

### SSH

- 后面代码就可以通过这种方式拉代码

```bash
# 1.在Jenkins服务器，已经安装了git，生成公私钥对
ssh-keygen -t rsa

# 2.在github上公钥地方添加 id_rsa.pub对应的内容

# 在jenkins上添加私钥的内容 id_rsa
- 用户名为生成公私钥的时候的root账户
```

![image-20250119092440933](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250119092440933.png)

## Publish Over SSH

- 通过SSH将文件(比如打包完成后的jar)发送到测试环境服务器上

### 1. 安装

![image-20250120233338940](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250120233338940.png)

### 2. SSH配置

- Dashboard--->Mange Jenkins ---> System
- 可以通过用户名密码来链接，也可以通过其他证书类的链接
- 新建后，可以通过test connection来验证链接是否成功
- Remote Directory: 默认发送文件过去是到root目录下

![image-20250120234233399](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250120234233399.png)

### 3. 超时机制

- 可以修改

![image-20250121111235028](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121111235028.png)

# Item-Maven

## 1. 项目构建

-  构建对应的Maven项目--- New Item

![image-20250119083047472](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250119083047472.png)

```bash
# /root/.jenkins/
- 存放对应数据

# 构建完成后，会在下面创建目录
# replay-service-api:  项目名称，也是对应下面文件目录名,可以改名
# job下存放不同的item,  包含 builds, config.xml
# 存放的比如构建结构，下次build的number，本次build的相关参数等

/root/.jenkins/jobs/replay-service-api
```

## 2. General

![image-20250119083141594](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250119083141594.png)

## 3. Source Code

- 可集成gitee，gitlab，github等代码托管仓库， 使用国内开源gitee，避免网络问题
- 和本地IDEA拉取代码类似
- Repository URL: 会通过添加的git的地址，来测试是否拉取代码
- 该item需要执行的哪个branch，根据自己的github分支来决定

![image-20250119104417663](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250119104417663.png)

## 4. Pre Steps

- 在Build对应的jar包前可执行的步骤

![image-20220831230008051](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20220831230008051.png)

## 5. Build

- 是以对应的git url打开后的路径作为根路径识别pom，如果pom存在嵌套，则更改即可
- 如果是聚合项目，则只要提供父项目的pom即可

![image-20250120210522748](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250120210522748.png)

## 6. Post Steps

- 可以把构建完成后的jar包，发送到对应的服务器上，并运行

![image-20250120210551237](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250120210551237.png)

## 7. Build

- 上述步骤构建完毕后，可以手动在Item中点击build
- 也可以通过配置的方式，来自动化构建

```bash
# src文件：会将src代码拉取到该目录，并类似本地IDEA，将该文件通过maven打包生成对应的jar
/root/.jenkins/workspace/replay-service

# replay-service: 根据item的名称来创建
- 该目录下，就是对应的src
# replay-service-api目录下
- pom.xml  src  target

# 构建记录
/root/.jenkins/jobs/项目名
```

# 微服务-Jar包

## Post Steps

### 1. 发送文件

- 打包完成后的后置操作，通过SSH发送文件到指定服务器
- 指定服务器上要安装Java17

![image-20250120234605225](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250120234605225.png)

```bash
# source files:  会将target/replay-service-api-1.0-SNAPSHOT.jar发送过去
- 以workspace作为根目录，可以通过匹配方式来获取到指定的文件
**/replay-service-api*.jar

#  Remove Prefix: 默认前缀是workspace的项目名下的目录结构： 比如上面就是 replay-service/replay-service-api/target
replay-service-api/target
 
# Remote directroy: 默认发送到测试服务器的root目录, 可以在对应的ssh-server中配置为opt
# 如果没有erick，则会去创建这个目录
- erick
- 传输过去的目录路径： /opt/erick

# Exec command:
- 文件发送完成后，执行的对应操作

# 最终发送到测试服务器的目录
/opt/erick/replay-service-api-1.0-SNAPSHOT.jar
```

![image-20250120234854336](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250120234854336.png)

### 2. 启动服务

#### 手动执行

- 在目标服务器中，手动启动jar包

```bash
# 1.直接启动， 但是关闭terminal或者 ctrl+c就会终止进程
java -jar replay-service-api-1.0-SNAPSHOT.jar

# 2.后台启动      & 表示在后台执行
nohup java -jar /opt/erick/*.jar &

# 执行上面指令，会卡住，按回车才会退出，并把日志输出到nohup.out中
# 会在执行该指令的目录下，创建一个新的日志文件 nohup.out
nohup: ignoring input and appending output to 'nohup.out'

# 3. 后台启动，输出日志到指定目录，就不需要按回车
nohup java -jar /opt/erick/*.jar  >service.log 2>&1 &
```

#### Jenkins执行

```bash
# 指定文件输出日志，不然需要按enter，所以Jenkins会认为任务还在执行， 就会任务一直没完成，最终导致任务超时
nohup java -jar /opt/erick/*.jar  >service.log 2>&1 &

#  对应的脚本： 必须加上上面一行，否则shell命令不会执行
. /etc/profile 
nohup java -jar /opt/erick/*.jar  >service.log 2>&1 &
```

![image-20250121001113225](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121001113225.png)

## Pre Steps

- 先杀死掉之前的应用，再启动新的应用，否则就引发端口冲突问题

### 清理脚本

- 在jar包运行服务器上，创建好清理脚本

```bash
# 在/opt/erick下创建 clear.sh
touch clear.sh

#  编写对应的脚本，授权
chmod 777 clear.sh
```

```bash
# optional
#!/bin/bash
# delete jar files
rm -rf *.jar

# get the first arguments passed
projectname=$1

# get java pid
pid=`ps -ef | grep $projectname | grep 'java -jar' | awk '{printf $2}'`

if [ -z $pid ];
    then 
         echo "$projectname not started"
    else
         kill -9 $pid
         echo "$projectname stoping..."
fi
```

### Jenkins配置

- 在构建Jar包之前，传输文件之前，先去执行的操作
- 通过Publish Over SSH插件来完成

![image-20250121003631335](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121003631335.png)

```bash
./etc/profile
cd /opt/erick                      # 需要先进入到这个目录，才能删除对应下面的文件， 但是其中的ps进程不进入也是可以的
./clear.sh replay-service-api       # replay-service-api 是传入的项目名称
```

![image-20250121003823552](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121003823552.png)

# 微服务-Docker

- dev环境安装Docker

## Post Steps

- Dockerfile发送和构建

### 1. 发送文件

- 按照上面方式发送Jar文件
- 在发送Dockerfile文件，保证两个文件在同一个目录下

![image-20250121012013812](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121012013812.png)

### 2. 启动服务

```bash
. /etc/profile 
cd /opt/erick
docker build -t replay-service-api:1.0.0.snapshot .
docker run --name myservice -d -p 8080:8080 replay-service-api:1.0.0.snapshot
```

![image-20250121013623144](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121013623144.png)

## Pre-Step

```bash
. /etc/profile 
cd /opt/erick 
rm -rf *                            
docker rm -f myservice             
docker rmi replay-service-api:1.0.0.snapshot       
```

![image-20250121014414578](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121014414578.png)

# Pipeline

- 使用Pipeline， 可以将上面所有的UI操作，转换为代码，就像docker的dockerfile一样
- Jenkinsfile作为源代码提交到git仓库

```bash
# pipeline组成
- pipeline： 整条流水线
- agent： 指定执行的哪个jenkins node
- stages： 所有阶段
- stage： 某一阶段，可以有多个
- steps： 阶段内的每一步，可执行命令
```

## 项目构建

### 1. Pipeline

![image-20250121130733834](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121130733834.png)

### 2. 脚本构建

- 可直接在console里面写pipeline的脚本
- 需要勾选Use Groovy Sandbox，否则该脚本执行时会有Jenkins的安全验证问题

![image-20250121134930557](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121134930557.png)

```bash
// 可以在外面定义变量
pipeline {
    // 任意使用一个agent
    agent any
 
    // 定义多个步骤
    stages {
        stage('pull code') {
            steps {
                echo 'pull code starting'
            }
        }

        stage('build jar') {
            steps {
                echo 'build jar starting'
            }
        }
    }
}
```

### 3. 脚本批准

![image-20250121134852898](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121134852898.png)

### 4. 启动

- 先安装插件

![image-20250121145549989](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121145549989.png)

![image-20250121145759667](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121145759667.png)

### 5. Blue Ocean

- 安装Blue Ocean 插件

![image-20250121135643902](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121135643902.png)

## 脚本构建

### 1. 代码生成器

- 下拉选项，是根据装的插件来决定的， 使用Pipeline Syntax来快速生成代码

![image-20250121140500571](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121140500571.png)

![image-20220904144911431](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20220904144911431.png)

### 2. 代码拉取

- 利用代码生成器生成

```bash
# 对应的pipeline的脚本
git branch: 'main', url: 'https://gitee.com/daydreamer9451/replay-service.git'
```

![image-20250121140738990](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121140738990.png)

### 3. Maven构建

- clear之前的版本

![image-20250121142839558](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121142839558.png)

```bash
# 目录结构问题：
- Maven build的时候，默认是去/root/.jenkins/workspace/replay-service-pipeline找的
  因此要去执行的时候，必须先进入到cd replay-service-pipeline找到对应的pom文件
```

### 4. SSH-Pubish

- 发送测试服务器，启动服务，前期清理工作

### 5. 最终脚本

```bash
pipeline {
    agent any

    // 定义maven配置的名字，具体使用哪个Maven，在Jenkins中已经配置过了
    tools {
        maven 'Linux-local-maven'
    }

    stages {
        stage('Pull Source Code') {
            steps {
                git branch: 'main', url: 'https://gitee.com/daydreamer9451/replay-service.git'
                echo 'Pull Code successfully'
            }
        }

        stage('Clean Jar and Stop the service'){
             steps {
                       sshPublisher(publishers: [sshPublisherDesc(configName: 'DEV-SERVER', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''./etc/profile
                        cd /opt/erick
                        ./clear.sh replay-service-api''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])

             }
        }

        stage('Build Jar By Maven') {
            steps {
              // 运行多行指令
               sh """
               cd replay-service-api
               mvn clean
               mvn package
               """
            }
        }

        stage('Send Jar to DEV-SERVER'){
           steps {
               sshPublisher(publishers: [sshPublisherDesc(configName: 'DEV-SERVER', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'erick', remoteDirectorySDF: false, removePrefix: 'replay-service-api/target', sourceFiles: '**/replay-service-api*.jar')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
           }
        }

        stage('Start Service in DEV-SERVER'){
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'DEV-SERVER', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''. /etc/profile
                       nohup java -jar /opt/erick/*.jar  >service.log 2>&1 &''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
    }
}
```

## Jenkinsfile构建

- 在代码中创建Jenkinsfile文件，上面的脚本都写在里面
- 是使用Grovvy语法编写的脚本语言

![image-20250121150659204](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121150659204.png)

![image-20250121150714118](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121150714118.png)

