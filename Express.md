# 简介

- express是基于node.js平台，快速，开放，极简的web开发框架
- express和node.js内置的http模块类似，用来专门创建web服务器的
- express是第三方包，基于http封装，提供简单易用的api
- npm i express

```js
/*导入*/
let express = require('express');

/*创建一个server*/
let server = express();

/*端口，ip，服务器启动后的回调函数*/
server.listen(8080, 'localhost', function () {
    console.log('Server Start Successfully');
})
```

# 路由

- 类似springboot项目中的controller

## 1. 请求参数

- 路径匹配，获取请求体中的一些参数：路径，自定义header等

```js
let express = require('express');
let server = express();
server.listen(8080, 'localhost', function () {
    console.log('Server Start Successfully');
})

/* http://localhost:8080/ */
server.get('/', function (request, response) {
    response.end('hi');
});

server.get('/test', function (request, response) {
    // 原生操作
    console.log(request.method);         /* GET */
    console.log(request.url);            /* /test */
    console.log(request.httpVersion);    /* 1.1 */
    console.log(request.headers);        /*所有 header */

    // express操作
    console.log(request.path)           /* /test */
    console.log(request.get('address'));   /*获取请求头的名字: 可以为自定义，也可以为系统自带*/
    response.end('test');
});
```

## 2. 响应

### 2.1 原生API

```js
let express = require('express');
let server = express();
server.listen(8080, 'localhost', function () {
    console.log('Server Start Successfully');
})

/* http://localhost:8080/ */
server.get('/', function (request, response) {
    response.statusCode = 404;
    response.statusMessage = 'love';
    response.setHeader('xxx', 'yyy');
    response.write('hell express');
    response.end('hi');   // 中文会乱码
});
```

### 2.2 express API

```js
let express = require('express');
let server = express();
server.listen(8080, 'localhost', function () {
    console.log('Server Start Successfully');
})

/* http://localhost:8080/ */
server.get('/', function (request, response) {
    response.status(500);
    response.set('aaa', 'bbb'); // header
    response.send('你好啊shuzhan');  // 中文不会乱码
});

/*链调用*/
server.get('/info', function (request, response) {
    response.status(500).set('aaa', 'bbb').send('睡了没shuzhan');
});
```

### 2.3 其他响应

```js
let express = require('express');
let server = express();
server.listen(8080, 'localhost', function () {
    console.log('Server Start Successfully');
})

/* 重定向：http://localhost:8080/other */
server.get('/other', function (request, response) {
    response.redirect('https://www.baidu.com');
});

/* 下载响应 http://localhost:8080/download*/
server.get('/download', function (request, response) {
    let filePath = __dirname + '/package.json';  // 绝对路径
    response.download(filePath);
});

/*json响应 : http://localhost:8080/people */
server.get('/people', function (request, response) {
    response.json({
        name: 'erick',
        age: 18
    })
});
```

## 3. GET参数

- 通过request.query或者request.params获取

```js
let express = require('express');
let server = express();
server.listen(8080, 'localhost', function () {
    console.log('Server Start Successfully');
})

/* http://localhost:8080/user */
server.get('/user', function (request, response) {
    /*api请求结果: 可以以json返回数据，也可以以普通文本返回*/
    response.send({"name": "erick", "age": 18});
});


/* 通过 query来获取参数：  http://localhost:8080/name?name=shuzhan&age=10*/
server.get('/name', function (request, response) {
    let query = request.query;
    console.log(query.name);
    console.log(query.age);
    console.log(query)
    response.send(query);
});

/*get请求，通过动态参数拼接：  http://localhost:8080/queryById/2*/
server.get('/queryById/:id', function (request, response) {
    let idParma = request.params;
    console.log(idParma) //{ id: '2' }

    /*解构赋值*/
    let {id} = idParma;
    console.log(id) //
    response.send(id);
})

/*get请求多个参数： http://localhost:8080/query/2/shuzhan*/
server.get('/query/:id/:name', function (request, response) {
    let params = request.params;
    console.log(params);  /*{ id: '2', name: 'shuzhan' }*/
    
    /*解构赋值*/
    let {id, name} = params;
    console.log(id);
    console.log(name);
    response.send(params);
})
```

## 4. POST

- npm i body-parser

```js
let express = require('express');
let bodyParser = require('body-parser');

let server = express();
server.listen(8080, 'localhost', function () {
    console.log('Server Start Successfully');
})

/*解析json格式的请求体*/
let bodyJson = bodyParser.json();


/*http://localhost:8080/info */
server.post('/info', function (request, response) {
    response.send("Hello Erick");
});

/* 带请求体的： http://localhost:8080/queryInfo */

/*需要借助 body-parser 包: 采用路由中间件的方式引入
* 执行完毕后，会在response中修改其中的body属性*/
server.post('/queryInfo', bodyJson, function (request, response) {
    let body = request.body;
    console.log(body);  // { name: 'erick', age: 20, address: 'xian' }
    // 下面可以通过解构函数进行解构

    response.send('erick');
});
```

![image-20240513221217345](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240513221217345.png)

## 三、express路由
### 1. 直接挂载，如上所属
### 2. 路由模块化
![在这里插入图片描述](https://img-blog.csdnimg.cn/ee40b5ff531b4e26bf9088af83f53ade.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aOe57-U55qE6I-c6bif5Y-U,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
**userRouter.js**

```javascript
let express = require('express');

let userRouter = express.Router();

/*定义多个路由*/
userRouter.get('/name', function (request, response) {
    response.send('User name : erick');
});

userRouter.get('/age', function (request, response) {
    response.send('User age : 20');
});

/*不要用{}方式导出*/
module.exports = userRouter;
```
**index.js**

```javascript
let express = require('express');
let userRouter = require('../router/userRouter');

let server = express();

server.listen(8080, 'localhost', function () {
    console.log('Server Start Successfully');
})

/*使用当前导入的router
 1. 可以挂载多个路由
 2. 声明顺序决定路由加载顺序
 3. 如果一个路由匹配到，后续则不再匹配*/
server.use(userRouter);
```
### 3. 路由模块添加统一前缀

```javascript
let express = require('express');
let userRouter = require('../router/userRouter');

let server = express();

server.listen(8080, 'localhost', function () {
    console.log('Server Start Successfully');
})

/*路由模块添加统一前缀
* http://localhost:8080/api/name*/
server.use('/api',userRouter);

```
# 中间件
- Middleware
- 请求进入express服务后，对请求和响应预处理，类似filter
- 可以处理request或者response，链式处理或者自定义属性

## 1. 全局中间件

- 所有的请求进来后，都会从全局中间件经过

###  1.1 一个
```javascript
let express = require('express');
let app = express();
app.listen(8080, 'localhost', function () {
    console.log('server start successfully');
});


/*定义一个middleware
* 1. 一定是有next函数的，才是一个middleware
* 2. 请求进来后，先进入对应的middleware，再去执行对应的get或者post请求*/
function firstMiddleWare(request, response, next) {
    console.log('First Middle Ware is coming!!!');
    /*next：表示当前中间件执行完毕了，继续执行后续操作*/
    next();
}

/*将中间件注册为全局middleware*/
app.use(firstMiddleWare);

app.get('/user', function (request, response) {
    console.log('User Get method is executed');
    response.send('hi');
})
```
###  1.2 多个

```javascript
let express = require('express');

let app = express();

app.listen(8080, 'localhost', function () {
    console.log('server start successfully');
});

function firstMiddleWare(request, response, next) {
    /*在中间件内部，可以自定义属性或者字段，后面的中间件，可以获取改属性*/
    request.startTime = Date.now();
    console.log('First Middle Ware is coming!!!');
    next();
}

function secondMiddleWare(request, response, next) {
    console.log(request.startTime);
    console.log('Second Middle Ware is coming!!!');
    next();
}

/*多个中间件执行顺序： 根据中间件的挂载顺序依次执行*/
app.use(firstMiddleWare);
app.use(secondMiddleWare);


app.get('/user', function (request, response) {
    console.log('User Get method is executed');
    response.send('hi');
})
```

## 2. 路由中间件

- 只能挂载在具体的某个路由路径下，才会进行

### 2.1 一个

```javascript
let express = require('express');

let app = express();

app.listen(8080, 'localhost', function () {
    console.log('server start successfully');
});

function firstMiddleWare(request, response, next) {
    request.startTime = Date.now();
    console.log('First Middle Ware is coming!!!');
    next();
}

/*局部中间件，不要在app上挂载，而是挂载到具体的方法里面*/
app.get('/user', firstMiddleWare, function (request, response) {
    console.log(request.startTime);
    console.log('User Get method is executed');
    response.send('hi');
})
```
### 2.2  多个
- [firstMiddleWare, secondMiddleWare]
- 或者firstMiddleWare，secondMiddleWare
```javascript
let express = require('express');

let app = express();

app.listen(8080, 'localhost', function () {
    console.log('server start successfully');
});

function firstMiddleWare(request, response, next) {
    console.log('First Middle Ware is coming!!!');
    next();
}

function secondMiddleWare(request, response, next) {
    console.log('Second Middle Ware is coming!!!');
    next();
}

/*局部中间件，不要在app上挂载，而是挂载到具体的方法里面*/
app.get('/user', [firstMiddleWare, secondMiddleWare], function (request, response) {
    console.log('User Get method is executed');
    response.send('hi');
})
```
## 3. 静态资源中间件

- 一个express的内置的全局中间件
- express.static()：将目录下的img,css,js对外公开访问
- 静态文件的目录名不会出现在url中
- 目录名和当前启动服务的js代码平级

```javascript
let express = require('express');

let server = express();

server.listen(8080, 'localhost', function () {
    console.log('Server Start Successfully');
})

/*将和本js文件平级目录下的，public目录下的img，css，js，html对外暴露
  1.可以修改目录，需要注意找到对应的public路径
  2.图片访问地址： http://localhost:8080/1.jpg
  3.可以定一个或者多个静态资源路径
  4. 从上往下，根据静态资源加载目录来查看对应的文件*/
server.use(express.static(__dirname + '/public'));
server.use(express.static(__dirname + '/static'));

/*静态资源访问时候，可以指定路径前缀
* 1. http://localhost:8080/nike/2.jpg*/
server.use('/nike', express.static(__dirname + '/entry'));
```

## 4. 注意事项

- 中间件要在路由挂载前
- 可以声明多个中间件，多个中间件共享request和response
- 中间件必须要调用next()
- 避免代码逻辑混乱，不要在next()后加入其他代码

