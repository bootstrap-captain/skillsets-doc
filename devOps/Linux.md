基于CentOS. 7.6

# 基本介绍

- Linux是一个内核，针对内核，衍生出了很多发行版
- 发行版：Ubuntu, RedHat, CentOS, Debain

## 1. 目录结构

- 层级的树状目录结构，根目录为/，然后在次目录下再创建其他的目录
- Linux中，一切皆文件

```bash
bin -> usr/bin                 # 经常使用的命令
sbin -> usr/sbin               # super bin  系统管理员使用的系统管理程序
home                           # 普通用户主目录，每个用户都有一个自己的目录，一般该目录名是用户的账户命名
root                           # 系统管理员的主目录
lib -> usr/lib                 # 系统开机所需要的基本的动态连接共享库
lib64 -> usr/lib64
lost+found                      # 一般为空的，系统非法关机后存放的文件
etc                             # 系统管理所需要的配置文件和子目录


boot                            # 启动时使用的核心文件，连接文件和镜像文件

dev                             # 把所有的硬件用文件形式存储
media                           # 自动识别U盘，灌区等，会把识别的设备挂载到这个目录下
mnt                             # 让用户临时挂载别的文件系统，将外部的存储挂载在里面

proc                            # 系统内存的映射
srv                             # service，服务启动后需要提取的数据
sys                             # linux2.6内核的一个很大变动，安装了一个文件系统sysfs


opt                             # 给主机安装软件所存放的目录(安装包)
tmp                             # 存放临时文件 
usr                             # 用户安装很多应用程序和文件都在这个目录
usr/local                       # 给主机安装额外软件的目录，一般是通过编译源码方式安装的程序


var                             # 存放不断扩充的东西，习惯将经常被修改的目录存放在这个目录下，包含各种日志文件
```

## 2. 关机重启

```bash
shutdown -h now       # 立刻关机 halt
shutdow -h 1          # 1分钟后关机
shutdown -r now       # 立刻关机 reboot
halt                  # 关机
reboot                # 重启
sync                  # 对内存中的数据同步到磁盘中

# 目前的shutdown/reboot/halt 等命令，都会先去执行sync
```

## 3. 用户管理

- 多用户，多任务的操作系统

### User

```bash
# 1.添加
# 默认该用户的家目录在/home/jack， 用jack登陆后，默认进入的界面就是/home/jack
useradd jack
passwd jack          # 设置密码: 为jack设定密码

# 手动指定用户的家目录为/home/test
useradd -d /home/nike-company tom

# 2.删除
userdel jack           # 删除用户，保留家目录
userdel -r jack       # 删除用户，删除家目录        recursion

# 3.查看某个用户
id jack
uid=1000(jack) gid=1000(jack) groups=1000(jack)

# 4. 查看当前用户
whoami         /   who am i

# 用户                   登陆的ssh客户端的时间和IP,以第一次登陆的用户为准
root     pts/0        Jan 24 01:44 (36.163.174.89)

# 5. 切换用户
# 如果用户权限不够，可以在当前用户切换到其他用户
su - 用户名

# 低-->高，需要输入密码;      高--->低，不需要输入密码

# 需要返回到原来用户时
exit     或      logout
```

### Group

- 系统可以对有相同权限的多个用户，进行统一管理

```bash
# 1. 添加组
groupadd nike-company
useradd -g nike-company lucy               # 创建lucy用户，并加入到nike组里
usermod -g citi-company lucy               # 将Lucy转移到citi-company组里    
useradd jack                               # 不指定组时，默认用该用户名创建一个组
 
# 2.删除组
groupdel nike
```

```bash
# user的配置文件，记录用户的各种信息
# jack:x:1000:1000::/home/jack:/bin/bash
/etc/passwd

# 登陆名，加密口令，最后一次修改时间等
/etc/shadow
jack:$6$biBDh3gP$76ZqtHL0GkCddOph4VOTmLZpJN7SP5e0f5NHgC1hnLMkP8Jh6ApHVcWrIBOKQWHUMOod4UlpFlPC7dTgXM6pQ/:20112:0:99999:7:::

# 组信息的文件
/ect/group
nike-company:x:1001:
```

# 进程管理

## 1. 基本介绍

- Linux中，每个执行的程序都称为一个进程，每个进程都会分配一个ID（PID进程号）

```bash
# 每个进程都可能以两种方式存在，前台和后台
- 前台： 用户可以在目前的屏幕上执行操作
- 后台： 实际在操作，但是屏幕上无法看到的进程

# 系统服务
- 以后台进程的方式存在，并且都会常驻在系统中，直到关机才结束
```

## 2. 进程查看

```bash
# 1. 查看所有
ps
        -a： 显示当前终端的所有进程信息
        -u： 以用户的格式显示进程信息
        -x： 显示后台进程运行的参数
# aux可以省略        
        
#用户       进程号 cpu memory                   运行状态 start 占用cpu的时间    启动进程的指令
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.2  51584  3760 ?        Ss   Jan24   0:08 /usr/lib/systemd/systemd
root         2  0.0  0.0      0     0 ?        S    Jan24   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    Jan24   0:00 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   Jan24   0:00 [kworker/0:0H]
root         7  0.0  0.0      0     0 ?        S    Jan24   0:00 [migration/0]
root         8  0.0  0.0      0     0 ?        S    Jan24   0:00 [rcu_bh]

# 2. 全格式显示  e：显示所有进程       f：全格式
ps -ef 
ps -ef|grep java

用户      pid     ppid(parent进程)
root     12188    11346           0 10:45 pts/0    00:00:00 grep --color=auto java      # ps本身的进程
```

```bash
# 1.查看java进程
ps -ef | grep replay-service-api-1.0-SNAPSHOT.jar

# 包含两个进程： 一个是运行的jar包对应的java进程， 一个是用jps查看的进程
root      5161     1  0 00:11 ?        00:00:07 java -jar /opt/erick/replay-service-api-1.0-SNAPSHOT.jar
root      5246  4503  0 00:26 pts/0    00:00:00 grep --color=auto replay-service-api-1.0-SNAPSHOT.jar

# 2. 过滤
# 2.1 过滤掉grep的进程: -v代表过滤
ps -ef | grep replay-service-api | grep -v grep
# 2.2  筛选出java和account的进程
ps -ef | grep replay-service-api | grep 'java -jar'

root      5161     1  0 00:11 ?        00:00:07 java -jar /opt/erick/replay-service-api-1.0-SNAPSHOT.jar

# 3. 获取端口： awk， 可以将对应的第二个字符获取，也就是30634， 游标是从1开始
ps -ef | grep replay-service-api | grep 'java -jar' | awk '{printf $2}'

5161[root@iZ2ze8y9odvmultgyoflt4Z erick]#
```

## 3. 杀死进程

```bash
kill -9 pid

-9: 强制停止
```

# 常用指令

## 1. Vi/Vim

- Vi是系统内置的文本编辑器
- Vim具有程序编辑的能力，是Vi的增强版本(代码补全，错误跳转，字体颜色等特性)

![image-20250124013409797](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250124013409797.png)

### 正常模式

- 用vim打开一个文本，就会进入该模式
- 使用上下左右移动光标，删除字符，删除整行，复制，粘贴

```bash
vim 1.txt         # 创建一个1.txt或者打开现有的1.txt 
yy                # 拷贝当前行
5yy               # 拷贝当前行向下的5行
p                 # 粘贴,粘贴位置通过光标移动
dd                # 删除当前行
5dd               # 删除当前行向下的5行
G                 # 光标移动到末行
gg                # 光标移动到gg
u                 # 撤销本次修改
```

### 编辑模式

- 可以在该模式下对内容进行输入

### 命令行模式

- 读取，存盘，替换，离开vim，显示行号等

```bash
:wq            # 保存退出  write & quit
:q             # 退出
:q!            # 强制退出不保存
:set nu        # 设置行号和取消行号
:set nonu 


/shuzhan        # 搜索对应的单词, 输入n切换到下一个搜索到的目标
```

## 2. 文件目录

```bash
pwd                  # 显示当前绝对路径            /usr/bin

# 列出显示文件和目录
ls                   
       -a ： 显示所有，包含隐藏
       -l：  以列表的方式
       -h:   显示时，对尺寸换算成k和m
ll


# cd
cd~   或   cd               # 家目录
cd ..                       # 上一级目录
cd /                        # 根目录
cd /opt                     # 进入opt目录

# 创建目录
mkdir erick                    # 创建一个目录
mkdir -p tom/peoples/china     # 创建多层目录
# 删除目录
rmdir tom/peoples/china                    # 删除china目录，必须一个空目录
rm -rf tom/peoples                         # 删除peoples目录和下面所有的东西
   -r： 递归
   -f：强制删除不提示

# 创建空文件
touch 1.txt

# 拷贝
cp source dest                  # 拷贝指定文件到指定地方
      -r:  递归复制整个文件夹
\cp -r source dest              # 递归拷贝并覆盖


# 移动
mv oldfileName newfileName                 # 重命名
mv /tmp/oldfileName /opt/newfileName       # 移动并重命名

# 查看
cat oldfileName
  -n: 显示行号
  
less fileName                       # 根据显示内容对文件加载，不是一次性加载完毕  
head -n 5 filename                    # 查看文件前5行，默认前10行
tail -n 5 filename                    # 查看文件最后5行，默认最后10行
tail -f file                          # 实时追踪文档的所有更新

# 输出到控制台
echo 'hello'

# 输出重定向
#  > 输出重定向(覆盖写)     >> 追加
# 如果 1.txt没有，则会创建
echo "hello" > 1.txt
cat 2.txt > 1.txt
ls -l > 1.txt
cal > 1.txt   # 日历
```

## 3. 搜索/查找

```bash
# find：  从指定目录，向下递归的遍历各个子目录，将满足条件的文件或目录显示
      -name 按照指定的文件名
      -user 查找属于指定用户
      -size 按照指定的文件大小  +(大于)  -(小于)    不写(等于)
      
 find -name java            # 从当前所在目录
 find /opt -name java
 find -size +10M
 
 # which  查看某个指令在什么地方保存的
 which ls
 
 alias ls='ls --color=auto'
	/usr/bin/ls
```

## 4. grep

```bash
# grep : 过滤查找
# ｜   ：将前一个命令的处理结果输出传递给后面的指令处理

# 1. 文本查找
       -n : 显示匹配及行号
       -i：  忽略大小写
       
cat 1.txt | grep -n January
grep -n January 1.txt
```

## 5. tar

```bash
# tar
# 最后打包的文件是 .tar.gz格式
  -v： 显示详细信息
  -f:  指定压缩后的文件名
  -z： 打包同时压缩
  -x： 解压 .tar文件
  -c: 产生 .tar打包文件
  
# 压缩  
tar -zcvf erick.tar.gz 1.txt 2.txt              # 将1.txt和2.txt压缩成erick.tar.gz
tar -zxvf erick.tar.gz                          # 解压到当前目录
tar -zxvf erick.tar.gz -C /opt                  # 解压到指定目录

```



# Shell

## 基本介绍

- Shell是一个命令行解释器，它接受应用程序/用户命令，然后调用操作系统内核
- Shell不仅仅可以是单纯的指令，可以是复杂的脚本
- 可以用IDEA对shell脚本进行编辑

![image-20250121160613601](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250121160613601.png)

```shell
# 查看当前Linux提供的shell解析器
cat etc/shells

/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash

# 查看默认shell解析器
echo $SHELL                       # /bin/bash
```



## 环境变量

- Linux开辟一片内存空间，用来保存一些变量

```bash
set            查看当前shell定义的所有变量
echo $HOME     查看某个变量
```

### 1. 父子SHELL

- 通过bash进入子shell
- 通过exit退出子shell

```bash
# 父shell
ps -f
root     12496 12494  0 10:09 pts/0    00:00:00 -bash        # 父shell
root     12670 12496  0 11:03 pts/0    00:00:00 ps -f

# 子shell
bash
ps -f
         # 当前pid  父pid
root     12496     12494  0 10:09 pts/0    00:00:00 -bash       # 父shell
root     12671     12496  0 11:04 pts/0    00:00:00 bash        # 子shell
root     12682     12671  0 11:04 pts/0    00:00:00 ps -f
```

### 2. 作用域

#### 全局

- 在所有的父shell和子shell都能够访问到的变量

![image-20250123113305928](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250123113305928.png)

```bash
# 查看所有的全局环境变量
env            
printenv

# 查看单独的全局环境变量
printenv USER
```

#### 局部

- 只在当前shell环境生效，父，子shell都不能访问
- 通过bash进入一个shell时候，退出后，那个子shell就消失了

![image-20250123113940283](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250123113940283.png)

### 3. 定义方式

#### 系统自定义

- Linux启动时，默认的一些变量，包含全局变量和局部变量

```bash
# $HOME    
# $PWD         当前目录
# $USER        当前用户
```

#### 用户自定义

- 用户可以自定义一些变量，包含全局变量和局部变量

### 4. 自定义

- 一般为小写

```bash
# 局部变量
# 定义或者更改      k=v             =前后不能有空格
# 
erick=19
erick="hello world"             # “” ‘’都可以


# 全局变量
# 只能在顶层shell中声明，提升为全局
# 在子shell中声明并提升后，父shell还是访问不到
export lucy=20

jack=21
export jack

# 只读变量
readonly job=12


# 撤销:   不能unset只读变量
unset k
```

## 基本语法

- 文件一般是 .sh为后缀，也可以不写，只要文本的规范是符合shell语法规范

### demo.sh

```shell
#!/bin/bash
# default shell interpreter, can be removed
echo 'hello world'
```

### 执行方式

```bash
# 方式一：  命令 + 脚本的相对路径或绝对路径  
# bash 和 sh都可以
# 不用给脚本赋权限
bash ./demo.sh                      # 当前目录
bash demo.sh                        # 当前目录
bash /opt/shell-demo/demo.sh        # 绝对路径
```

```bash
# 方式二： 输入脚本的绝对路径/相对路径
# 需要给脚本 + x 权限
chmod +x demo.sh 

./demo.sh                      # 相对路径-当前目录
shell-demo/demo.sh             # 相对路径-上层目录
/opt/shell-demo/demo.sh        # 绝对路径
```

```bash
# 方式三： 在脚本路径前加上 . 或者 source
source demo.sh                         # 相对路径-当前目录
source ./demo.sh                       # 相对路径-当前目录
source /opt/shell-demo/demo.sh         # 绝对路径
```

- 主要在于是否进入了子shell，从而对环境变量的继承关系会有影响

```bash
# bash + 脚本       或        ./demo.sh
- 打开了子shell

# source      .
- 直接在父shell中执行的， 不会进入到子shell
```

### PATH

```bash
# 如果进入对应的目录，想直接执行,就类似于直接使用命令
demo.sh

# Linux默认配置了一些可以直接直接命令的目录
# 默认不要修改这些目录
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin


# 修改环境变量并刷新
vim /etc/profile

# 将该目录下添加为PATH, 则该目录下的shell脚本可以直接执行
export PATH=$PATH:/opt/shell-demo

# 刷新环境变量
source /etc/profile

# 在任何目录下都可以执行
demo.sh
```

## 输入参数

```bash
# $n
- n为数字，$0代表脚本全路径，$0---$9代表第一到第九个参数。10以上表示${10}

# $#
- 获取所有输入参数个数，用于循环

# $?
- 最后一次执行的命令的放回状态
- 0:    上一个命令执行正确
- 非0:  (具体哪个数，由命令自己决定)，上一个命令执行失败
```

```bash
#!/bin/bash
echo "script name=" $0
echo $1
echo "second_var, $2"
```

## 高级语法

### 运算符

```bash
# + 左右有空格，得到结果
expr 5 + 4
expr 5 \* 2
lucy=$(expr 5 + 2)
tom=`expr 5 + 2`

# $[]  或   $((5*2))   常用
jack=$[5*2]
nancy=$((5*2))
```

### 条件判断

```bash
# condition前后要有空格
test 1 = 1
echo $?             # true为0

test 1 = 2    
echo $?             # false为非0

# [ condition ]  condition前后要有空格,  符号前后有空格
[ 1 = 1 ]
[ 1 = 2 ]

# 条件
[ 10 -ne 11 ]
[ -x demo.sh ]
```

```bash
# 整数之间
-eq        equal
-ne        not equal
-lt        less than
-le        less equal
-gt        greater than
-ge        greater than

# 文件权限
-r         read
-w         write
-x         execute

# 文件类型
-e        文件存在 existence
-f        文件存在且是一个常规文件file
-d        文件存在且是一个目录directory

# 字符串
=
!= 
```

### 流程控制

#### IF

```bash
#!/bin/bash
if [ 1 = 1 ]
  then echo 'hah'
fi
```

```bash
#!/bin/bash
age=${1}
if [ ${age} -le 10 ]
   then 
      echo "child"
elif [ ${age} -le 18 ] 
   then 
      echo "teenager"
else
      echo "adult"
fi
```

#### CASE

```bash
#!/bin/bash
case $1 in
1)
   echo "one"
;;

2)
   echo "two"
;;

10)
   echo "ten"
;;

*)
  echo "other"
;;

esac
```

#### FOR

```bash
#!/bin/bash
sum=0

for((i=0; i<=$1; i++))
do
  sum=$((sum+i))
done

echo $sum
```

```bash
#!/bin/bash
for i in {1,4,5,8,10} ;
do
  echo "$i"

done
```

- $*:  获取当前命令行的所有参数，把所有参数当作一个整体对待
- $@:  获取当前命令行的所有参数，每个参数区分对待

## 控制台输入输出

- -t: 用户输入等待几秒
- -p：用户输入提示信息

```bash
#!/bin/bash

read -t 10 -p "pls input your address：" address

echo "welcome, $address"
```

# 包管理工具

## RPM

- 用于互联网下载包的打包及安装工具
- RedHat Package Manger

```bash
# 查看版本
rpm

RPM version 4.11.3
Copyright (C) 1998-2002 - Red Hat, Inc.
This program may be freely redistributed under the terms of the GNU GPL

# 查看系统是否安装了rpm的包
rpm -qa|grep vim

      # vim-enhanced:        包名
      # 7.4.160-6：           版本号
      # el7_6.x86_64         操作系统
               
vim-enhanced-7.4.160-6.el7_6.x86_64

# 查看安装的某个软件的具体信息
rpm -qi vim-enhanced

# 安装的某个软件在具体哪些地方都创建了文件夹
rpm -ql vim-enhanced
```

```bash
# 安装
rpm -ivh RPM包全路径名称(互联网路径或者Linux宿主机全路径)
      # i： install
      # v： 提示verbose
      # h:  安装进度条hash
       
# 卸载
rpm -e firefox
```

## YUM

- 一个Shell前端软件管理包，基于RPM包管理，能够从指定的服务器自动下载RPM包并且自动安装
- 可以自动处理依赖性关系，并且一次安装所有依赖的软件包

```bash
# 自动查看远程的yum服务器是否包含某个软件
yum list|grep firefox

yum install firefox
```

