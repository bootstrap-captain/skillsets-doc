# 简介

- Node Package Manager:  npm.inc公司的一个，关于node 的包管理工具
- [npm第三方依赖](https://www.npmjs.com/)
- 随着node的安装，会一起安装在客户机上
- npm -v可以查看对应的npm的版本: 10.5.2

#  项目构建

## 1. 初始化

- package.json来定义依赖的包，类似maven中的pom.xml
- 创建空目录
- npm init -y： 会在执行命令的目录下，创建一个package.json来管理，可以创建多个

```yaml
{
  "name": "node_learn",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "lodash": "^4.17.21"
  }
}
```

## 2. 安装三方包

- npm install lodash
- 会在项目根路径下，创建node_moudles模块，并将第三方包放进去
- 修改对应的package.json，更新对应依赖
- 创建package.lock.json文件，会详细定义具体的依赖的版本，组织等信息
- 有时候可能package.lock.json和package.json不匹配，删除package.lock.json重新生成

![image-20240513180746201](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240513180746201.png)

## 3. 项目代码

- 不需要上传node_modules模块代码，因为体积太大
- 源代码拿到，直接 npm install 就会把配置文件中指定的第三方包下载到本地项目中
- 有时候第三方包加载出现问题，可以直接删除node_modules，然后 npm install

# 配置

- 第一次执行任何一个config set的命令后，就会在本地/Users/shuzhan/.npmrc创建一个配置文件

## 1. 查看配置

```bash
# npm config list
```

## 2. 镜像配置

- 默认npm包是从海外服务器拉的：https://registry.npmjs.org

### 2.1 直接修改

- 修改后会在上面的配置文件中生效，可以直接查看文件

```bash
# 使用淘宝镜像
npm config set registry https://registry.npmmirror.com/

#  切回默认地址
npm config set registry https://registry.npmjs.org/

# 验证是否成功
npm config get registry
```

### 2.2 nrm

```bash
# 下载对应的工具
npm install -g nrm
sudo npm install -g nrm  # 如果权限失败

# 查看所有镜像
nrm ls

# 选择哪个镜像： 会修改配置文件
nrm use taobao

# 验证是否成功
npm config get registry
```

```bash
* npm ---------- https://registry.npmjs.org/
  yarn --------- https://registry.yarnpkg.com/
  tencent ------ https://mirrors.cloud.tencent.com/npm/
  cnpm --------- https://r.cnpmjs.org/
  taobao ------- https://registry.npmmirror.com/
  npmMirror ---- https://skimdb.npmjs.com/registry/
```

## 3. repo设置

```bash
# 默认repo
npm config get prefix            # 结果： /usr/local，后面下载的所有的g包都在/usr/local/lib/node_modules
```

## 4. 热部署

- 热部署，不用每次进行部署
- webstorm中，编辑完文件后，可以通过 ctrl+来手动触发热部署

```bash
npm install -g nodemon

# 执行对应的js
nodemon index.js
```

# 常用指令

```bash
# 在项目文件中，生成一个package.json
npm init -y

# 安装： 
npm install lodash
npm i lodash                  # 默认最新版本
npm i lodash@4.17.17          # 指定版本
npm install lodash moment     # 多个同时安装

# 卸载
npm uninstall lodash
npm uni lodash


#一次性加载: node_moudle和项目package.json 不匹配时，运行会更新
#当删除node_moudle目录时候，运行就会在项目中下载对应的包
#一般node_moudle会占项目的90%的大小
npm install
npm i


# 只有开发时需要的依赖, 卸载时候的语法同上
# -D 和具体包名顺序可颠倒
npm install moment --save-dev
npm install moment -D

npm install lodash -g
npm uninstall lodash -g

# 2. 检查当前项目软件哪些依赖已经过时
npm outdated
```

# 包分类

## 1. 项目包

- 安装在项目中的node_modules模块下的包
- 分为开发依赖时包和开发运行依赖包

```bash
## 依赖类型
- DevDepenency: 只会在开发时候用到，项目实际运行时候不需要
- Dependency： 开发和运行时候都需要
- npm install moment --save-dev
- 如何判断： 去npm来看就可以了
```

## 2. 全局包

- 安装到本地的node/npm/node_modules模块
- 只有工具一类的包，才需要安装全局包

## 3. 自定义包

### 3.1 创建项目

- name: 希望后面从npm官方仓库被搜索的名字，不能和npm上其他包名重复， 也是npm install时候名字
- main: 程序的入口文件，别人需要导入时的文件
- license: npm 遵循的协议
- keywords:  npm 中搜索的时候的关键字

```yaml
{
  "name": "erick_log",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": ["erick","daydreamer"],
  "author": "Erick Shu",
  "license": "ISC",
  "description": "This is just a test",
  "dependencies": {
    "lodash": "^4.17.21"
  }
}
```

### 3.2 目录结构

![image-20240522155217315](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240522155217315.png)

#### 项目代码

```javascript
let name = 'cat';
let food = 'fish meat';

function play() {
    console.log('Cat Play with Fur Ball');
}

function eat() {
    console.log(`Cat eat ${food}`);
}

module.exports = {name, food, play, eat}
```

```javascript
let name = 'dog';
let food = 'pork meat';

function play() {
    console.log('Dog Love Swiming');
}

function eat() {
    console.log(`Dog eat ${food}`);
}

module.exports = {name, food, play, eat}
```

#### index.js

- 一般情况，index.js作为程序入口，只是对外导出对应的文件
- 尽可能少的涉及具体的业务逻辑

```javascript
let info = 'This is introduction of this project';

let cat = require('./src/cat');
let dog = require('./src/dog');

module.exports = {
    info,
    cat,
    dog
}
```

### 3.3 使用

```javascript
/*本项目内引用*/
/*let data = require('./index');*/

/* 别人调用本项目中的程序入口  方式一：*/
/*let data = require('../erick_log/index')*/

/* 别人调用本项目中的程序入口  方式二：*/
/* 只传递文件夹，就回根据package中的main属性，去找当前目录的，对应的那个文件*/
/*let data = require('../erick_log')*/

let index = require('./index');

console.log(index.info)
console.log(index.dog.name);
```

### 3.4 npm包发布

#### 注册npm

-   需要注册对应的npm账户(daydreamer111)，然后发布项目

#### 终端

- 必须切换到npm官方的包地址，不能使用镜像

```bash
# 1. 根据第一步注册的账号，进行登陆
npm login

# 2.  切换到项目所在目录, 发布项目
npm publish 

# 3. 删除已经发布的包
# 3.1 只能删除 72h内发布的包
# 3.2 删除的包，在24h内不允许再次发布
npm unpublish 包名 --force
```

## 



