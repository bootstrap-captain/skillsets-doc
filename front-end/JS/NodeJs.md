# 简介

## 1. 基本介绍

- node.js 是 一个javascript的一个运行时环境，可以用来做后端开发
- 基于chrome v8的，一个javascript的环境

![image-20240513162027211](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240513162027211.png)

## 2. MAC-安装

- [官网下载](https://nodejs.org/en): 下载最新版本，傻瓜式安装即可
- 版本：v20.13.1
- 安装后， node -v查看版本号
- 在终端，进入文件指定目录，node erick.js ，即可执行对应的js代码

```bash
	•	Node.js v20.13.1 to /usr/local/bin/node
	•	npm v10.5.2 to /usr/local/bin/npm
	
	# 卸载
	sudo rm -rf /usr/local/{bin/{node,npm},lib/node_modules/npm,lib/node,share/man/*/node.*}
```



# 内置模块

- 内置模块在使用前，需要先导入

## fs

- 文件处理

### 读

```js
// 导入fs模块
let fs = require('fs');

fs.readFile('./book/1.txt', 'utf-8', function (err, success) {
    if (err) {
        console.log(err)
        return;
    }

    if (success) {
        console.log(success)  //读取的文本
        return;
    }
})
```

### 写

#### 覆盖写

```js
let fs = require('fs');

fs.writeFile('./book/1.txt', 'Hello word',
    'utf-8', function (error) {
        if (error) {
            console.log("写入文件失败： ", error.message);
            return;
        }

        console.log("写入成功");
    })
```

### 路径问题

- 读取文件时用的是相对路径 ./.   或者 ../ 等，可能出现问题
- node执行命令时，会将当前文件所在目录，拼接上js中的相对路径
- 如果用 node ./book 1.js，就可能存在问题(路径动态拼接错误问题)
- 可以直接用绝对路径来解决，但是不好移植

```bash
_ _ dirname: 当前js文件所处的目录： 通过这种方式动态拼接，
不管那个目录运行js文件，保证不会出错

- 但是只能拼接当前目录，无法拼接上一级目录
```

```js
let fs = require('fs');

/*__dirname: 当前js文件所处的目录*/
fs.writeFile(__dirname + 'read.txt', 'Hello Node',
    'utf-8', function (error) {

        if (error) {
            console.log(error.path)
            console.log("写入文件失败： ", error.message);
            return;
        }
        console.log("写入成功");
    })
```

## path

- 处理路径常用的一些方法

```js
const path = require("path");

/*将多个路径拼接，一种更加正规的方式: /a/b*/
let filePath = path.join('/a', '/b');
console.log(filePath);

/* ../ 会抵消一级路径，最终结果为 /a/c */
let secondPath = path.join('/a/b', '../', '/c');
console.log(secondPath);

/*  输出结果：index.html */
let fullName = path.basename('/a/b/c/index.html');
console.log(fullName);

/*  输出结果： index*/
let nameWithoutExtension = path.basename('/a/b/c/index.html', '.html');
console.log(nameWithoutExtension);

console.log(path.extname('/a/b/c/index.html')); /*文件扩展名： .html*/
console.log(path.dirname('/a/b/c/index.html'))  /*/a/b/c*/
```
