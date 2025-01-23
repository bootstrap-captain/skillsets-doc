**基于CentOS. 7.6**

# 基本介绍







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



# 进程查看

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

