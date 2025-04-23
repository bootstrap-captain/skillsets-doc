# 依赖管理

## 3. version conflict

- 同一个项目中使用了同一个jar包的不同版本，导致冲突，使用的时候可能出现问题

```bash
# 抛异常
- java.lang.ClassNotFoundException:   编译过程找不到类
- java.lang.NoClassDefFoundError：    运行过程找不到类
- java.lang.LinkageError:             不同类加载器分别加载的多个类有相同的全限定类名

# 抛异常
- java.lang.NoSuchMethodError:  程序找不到预期的方法，多见于通过反射调用方法

# 没报错但结果不对
- 两个jar包中的类分别实现了同一个接口，但是因为没有注意命名规范，两个不同的类刚好是同一个名字
```

![](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230423141604705.png)

### 3.1 Maven仲裁机制

- Maven本身提供了仲裁机制，决定到底该采用哪个版本

#### a.最短路径

- 路径优先：依赖中出现相同的GAV，资源层级越深，优先级越低
- 最终选择lombok2.0

![image-20221030195638273](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20221030195638273.png)

#### b.路径相同，声明优先

- 一个项目依赖其他几个项目，其他几个项目中都包含某个jar包，那么引入其他几个项目，先引用的生效

![image-20221030170215889](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20221030170215889.png)

#### c.覆盖优先

- 同一个文件中配置了一个资源的不同版本，后配置的覆盖前面配置的

![image-20221030165348830](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20221030165348830.png)

### 3.2 手动解决

- 多个三方包引入的不同版本，版本仲裁会指定一个版本的jar。

```bash
# 仲裁的最终结果：
- JVM在加载一个jar包的时候，如果加载了其中一个jar包，其他不同版本的该jar就不会再次加载

# 低版本： 
- 低版本可能存在安全漏洞，因此需要指定一个高版本

# 高版本：
- 高版本中可能删除了一些类，导致某个三方包运行不起来。 ClassNotFound Error, MethodNotFound

# 因此，在经过maven仲裁机制过后的某个jar版本，如果项目运行不起来，就需要手动改变该jar的版本
```

![image-20221030203750528](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20221030203750528.png)

#### a.exclude

- 可以借助Maven Helper插件帮我们快速排除依赖

![image-20221030203856638](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20221030203856638.png)

#### b.版本锁定

- 若某个包被很多个框架引入，那么exclude时，就需要一个个去排除，影响代码美观性能，因此可以在本工程中使用版本锁定
- **锁定版本法可以打破依赖传递的原则，优先级为最高**
- 也可以在父工程中来控制版本锁定
- 若父工程和子工程同时进行了版本锁定，子项目优先级高

![image-20230423131605590](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230423131605590.png)

- 版本锁定只针对传递的依赖，直接依赖如果指定版本，则版本锁定失效

![image-20230423131700641](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230423131700641.png)

### 3.3 依赖隐藏

- optional属性：B工程可以隐藏自己的依赖，不让A工程看到，不传递该依赖
- 默认为false，表示可以传递，true代表不传递
- 被依赖方解决依赖冲突

![image-20221028094740815](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20221028094740815.png)

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

# 版本信息

```bash
# SNAPSHOT
- 快照版本
- 每个模块进行的临时版本(测试版本)

# RELEASE 
- 发布版本
- 稳定版本

# 工程版本号: 主版本  次版本  增量版本   里程碑版本
- 主版本：     项目重大架构的变更
- 次版本：     较大的功能增加和变化， 或者全面系统的修复漏洞
- 增量版本：   重大漏洞的修复
- 里程碑版本：  版本内部。同下一个正式版本相比，相对来说不是很稳定，有待更多测试

5.1.9 RELEASE
```
