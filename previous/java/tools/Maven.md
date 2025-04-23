# 父子工程

- 父工程打包为pom，不写业务代码，只提供版本管理机制对应的pom.xml
- 子工程打包是jar
- 打包的时候，只需要在父工程一键install即可

```bash
# 父子项目最大优点
- 可以直接执行父工程的maven命令，父工程会自动根据项目模式，找到对应的其他子工程
- 执行maven命令时，比如package，只需在父工程中进行，聚合项目会自动对该父工程下所有项目进行一键package(父项目是自己的)
- 如果父项目是公司级别的其他项目，则该优点就失效了，只能进行版本控制
- 版本锁定也可以通过DM引入其他的对应的DM来进行管理，优先级比parent高

# 尽量使用parent类型引入pom
- 这样可以使用到 properties
- DM可以进行依赖传递
```

## 1. 单继承-parent

- 一个子工程只能包含一个parent工程
- 子工程的version和groupId如果和父工程一样，则可以省略
- 父工程version一处修改，各个子工程全部生效，子工程可单独定义version，优先级比父工程高
- 子工程要使用时，无需定义对应version，只需引入GA坐标，然后再通过版本仲裁来确定引入的依赖的深层依赖的version

![image-20221028110938184](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20221028110938184.png)

## 2. 多继承-DM

- 解决pom的parent单继承问题
- import必须作用于pom类型的dependency，必须放在dependencyManagement中
- DM和parent同时存在时候，DM优先级高

### 2.1 单体项目-import

- 单体项目依赖多个parent
- 可以将父项目通过import来引入，也可作为DM来引入

#### a. 多pom相同依赖

- 多个pom级中如果包含相同依赖，声明优先，不管路径深度
- 嵌套pom-声明优先

![image-20230423150224529](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230423150224529.png)

![image-20230423152552841](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230423152552841.png)

#### b. 一级import：显式优先

![image-20230423150757244](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230423150757244.png)

### 2.2 父子项目-parent

- 在父工程中通过import的方式导入多个其他父工程，然后让子项目去依赖
- 如果父工程和子工程都定义了一个pom类型的父工程，则选用子项目的版本
- 父工程可以不断向上追溯

![image-20221030163207215](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20221030163207215.png)

## 3. 项目抽离

- 可以对父工程下各个子工程需要的公共的服务，抽取成common三方包，install到本地仓库，并上传到私服
- 然后让各个子模块去依赖

## 4. 标签属性

- 标签属性： 可以在父工程中定义，也可以在子工程中定义
- 标签在引用时候，只能存在于子父工程，不能存在于通过DependencyManagement来使用

### 4.1 自定义标签

- 可以自己为标签取名字，然后在工程中通过${}来进行获取对应的值

![image-20221028111835450](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20221028111835450.png)

# Build

## 1. 基本设置

```bash
# finalName
- 默认打包后的文件名是： artifactId-version.jar
- 可以通过该标签个性化名字
```

## 2. 微服务打包

- maven的直接打包，不能生成直接可以运行的微服务jar包

```bash
# 微服务jar包
- 当前微服务本身jar包
- 当前微服务所依赖的jar包
- 内置可以直接运行的tomcat
- jar包可以通过 java -jar方式直接启动

# 添加对应的plugin，最终会生成两个jar
erick-customization.jar                # fat jar，即可以直接运行的jar
erick-customization.jar.original       # 去掉 .original, 就是maven默认的打包后的jar
```

![image-20230424132142527](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230424132142527.png)

```xml
<build>
        <plugins>
            <!--如果不加该插件，则只包含当前服务的代码-->

            <!--满足微服务打包的插件
            版本信息： 1. 如果parent是springboot的版本，则不配置可以直接使用
                      2. 也可以自定义
                      3. 不能直接通过DM来进行传递，只能通过parent来传递-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>3.0.5</version>
            </plugin>
        </plugins>

        <!--自定义打包后的名字: erick-customization.jar -->
        <finalName>erick-customization</finalName>
    </build>
```
