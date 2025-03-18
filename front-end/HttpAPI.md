# Ajax

- Asynchronous JavaScript And XML
- 在网页不刷新的情况下，向服务端发送请求
- 属于基于浏览器执行引擎的，发送http服务的工具

```bash
# 优点
- 可以无需刷新页面，与服务端进行通信
- 允许根据用户事件，更新部分页面内容

# 缺点
- 没有浏览历史，不能回退
- 存在跨域问题（同源）
- SEO不友好(响应体不包含ajax的返回数据)
```

## 1. GET

### 服务端接口

```java
package com.citi.erick.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/erick")
@CrossOrigin()
@Slf4j
public class StudentController {

    @GetMapping("/sleep")
    public String sleep() {
        log.info("sleep coming");
        return "hello erick";
    }

    @GetMapping("/eat")
    public String eat(@RequestParam(name = "name") String name,
                      @RequestParam(name = "age") String myName) {
        log.info("name={},age={}", name, myName);
        return name + myName;
    }

    /*响应Json*/
    @GetMapping("/getAll")
    public People getAll(@RequestParam(name = "prefix") String prefix) {
        People people = new People();
        people.setAddress(prefix + "xian");
        people.setAge(prefix + "45");
        people.setName(prefix + "shuzhan");
        return people;
    }
}
```

### html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <script src="GetRequest.js"></script>
</head>
<body>

<button onclick=getWithoutArgs()>GET不带参数</button>
<br>
<button onclick=getWithArgs()>GET带参数</button>
<br>
<button onclick=getWithJsonResponseFirst()>GET返回json01</button>
<br>
<button onclick=getWithJsonResponseSecond()>GET返回json02</button>
</body>
</html>
```

### js

```js
/*1. get请求，不带参数*/
function getWithoutArgs() {
    // 1. Ajax 对象
    let xhr = new XMLHttpRequest();
    // 2. 请求方法和路径
    xhr.open('GET', 'http://localhost:8080/erick/sleep');
    // 请求头: 可以自定义
    xhr.setRequestHeader('k1', 'v1');
    xhr.setRequestHeader('k2', 'v2');
    // 3. 发送
    xhr.send();
    // 4. 返回数据
    /* 4.1 readstate: 是xhr对象中的属性，一共会触发四次，表示xhr的状态变化
    *   0: 初始化
    *   1: open方法调用完毕
    *   2： send方法调用完毕
    *   3： 服务端返回了部分结果
    *   4.  服务端返回了所有结果
    *  */
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) { // 服务端返回了所有结果
            if (xhr.status >= 200 && xhr.status < 300) { // 判断响应码
                // 200
                console.log('状态码=', xhr.status);
                // 
                console.log('状态字符串=', xhr.statusText);
                // content-length: 11 content-type: text/plain;charset=UTF-8
                console.log('header =', xhr.getAllResponseHeaders());
                // 响应体= hello erick
                console.log('响应体=', xhr.response);
            }
        }
    }
}

/**
 * 2. get请求带参数
 */
function getWithArgs() {
    let xhr = new XMLHttpRequest();
    xhr.open('GET', 'http://localhost:8080/erick/eat?name=shuzhan&age=20'); // 带参数的
    xhr.send();
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            if (xhr.status >= 200 && xhr.status < 300) {
                console.log('result', xhr.response); // shuzhan20
            }
        }
    }
}


/**
 *  3. Get返回json: 手动转换
 */
function getWithJsonResponseFirst() {
    let xhr = new XMLHttpRequest();
    xhr.open('GET', 'http://localhost:8080/erick/getAll?prefix=test');
    xhr.send();
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            if (xhr.status >= 200 && xhr.status < 300) {
                // {"name":"testshuzhan","age":"test45","address":"testxian"}
                console.log(xhr.response);
                // 1. 手动转换为json
                let data = JSON.parse(xhr.response);
                console.log(data);
            }
        }
    }
}

/**
 *  4. Get返回json：自动转换json
 */
function getWithJsonResponseSecond() {
    let xhr = new XMLHttpRequest();
    xhr.open('GET', 'http://localhost:8080/erick/getAll?prefix=test');
    xhr.responseType = 'json';
    xhr.send();
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            if (xhr.status >= 200 && xhr.status < 300) {
                // 直接就是jspn
                console.log(xhr.response);
            }
        }
    }
}
```

## 2.POST

```bash
# 发送post请求时，需要设置该header
# 一般情况，一个请求应该只包含一种数据类型

# params
"content-Type", "application/x-www-form-urlencoded"

# body： json
"content-Type", "application/json"
```

### 服务端接口

```java
package com.citi.erick.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/erick")
@Slf4j
@CrossOrigin
public class PostController {

    @PostMapping("/noargs")
    public String noargs() {
        log.info("post1 coming");
        return "erick";
    }

    @PostMapping("/withargs")
    public People savePeople(@RequestParam(name = "address") String myAddress,
                             @RequestParam(name = "name") String myName,
                             @RequestParam(name = "age") String myAge) {
        People people = new People();
        people.setAddress("param_" + myAddress);
        people.setName("param_" + myName);
        people.setAge("param_" + myAge);
        return people;
    }

    @PostMapping("/withRequestBody")
    public People savePeopleBody(@RequestBody People people) {
        people.setAddress("body_" + people.getAddress());
        people.setAge("body_" + people.getAge());
        people.setName("body_" + people.getName());
        return people;
    }
}
```

### html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <script src="../post/PostRequest.js"></script>
</head>
<body>

<button onclick=postWithArgs()>POST不带参数</button>
<br>

<button onclick=postWithParamArgs()>POST带参数</button>
<br>

<button onclick=postWithRequestBodyArgs()>POST带Body</button>
<br>

</body>
</html>
```

### js

```js
function postWithArgs() {
    let xhr = new XMLHttpRequest();
    xhr.open('POST', 'http://localhost:8080/erick/noargs');
    xhr.send();

    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) { // 服务端返回了所有结果
            if (xhr.status >= 200 && xhr.status < 300) { // 判断响应码
                console.log('响应体=', xhr.response);
            }
        }
    }
}

function postWithParamArgs() {
    let xhr = new XMLHttpRequest();
    xhr.open('POST', 'http://localhost:8080/erick/withargs'); // 带参数的
    let param = "name=shuzhan&age=12&address=xian";
    //  POST传递参数的必须的header：application/x-www-form-urlencoded
    xhr.setRequestHeader("content-Type", "application/x-www-form-urlencoded");
    xhr.send(param);
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            if (xhr.status >= 200 && xhr.status < 300) {
                console.log('result', xhr.response);
            }
        }
    }
}

function postWithRequestBodyArgs() {
    let xhr = new XMLHttpRequest();
    xhr.open('POST', 'http://localhost:8080/erick/withRequestBody');

    // POST传递Body的必须的header：application/json
    xhr.setRequestHeader("content-Type", "application/json");
    let data = JSON.stringify({
        name: 'erick',
        age: '30',
        address: 'pk'
    });

    xhr.send(data);
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            if (xhr.status >= 200 && xhr.status < 300) {
                console.log(xhr.response);
            }
        }
    }
}
```



## 3. 超时/异常

### 服务端接口

```java
package com.citi.erick.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.TimeUnit;

@RestController
@RequestMapping("/erick")
@CrossOrigin()
@Slf4j
public class TimeoutController {

    @GetMapping("/timeout")
    public String sleep() throws InterruptedException {
        TimeUnit.SECONDS.sleep(4);
        log.info("timeout coming");
        return "hello erick";
    }
}
```

### html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="timeout.js"></script>
</head>
<body>
<button onclick=testTimeOut()>超时</button><br>
<button onclick=testNetWorkError()>网络异常</button><br>

</body>
</html>
```

### js

```js
function testTimeOut() {
    let xhr = new XMLHttpRequest();
    // 设置超时
    xhr.timeout = 2000;
    // timeout 超时回调
    xhr.ontimeout = function () {
        alert('超时')
    }

    xhr.open('GET', 'http://localhost:8080/erick/timeout');
    xhr.send();
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            if (xhr.status >= 200 && xhr.status < 300) {
                console.log('响应体=', xhr.response);
            }
        }
    }
}

function testNetWorkError() {
    let xhr = new XMLHttpRequest();
    // 网络异常回调
    xhr.onerror = function () {
        alert('网络异常')
    }
    xhr.open('GET', 'http://localhost:8080/erick/random');
    xhr.send();
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            if (xhr.status >= 200 && xhr.status < 300) {
                console.log('响应体=', xhr.response);
            }
        }
    }
}
```

## 4. 取消请求

- 当一个请求，处于pending状态时候，可以取消请求
- 取消时候，后端可能已经收到请求了，只是本次ajax和后端的连接断开了

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="cancel.js"></script>
</head>
<body>
<button onclick=sendRequest()>发送请求</button><br>
<button onclick=cancelRequest()>取消请求</button><br>

</body>
</html>
```

```js
let xhr = new XMLHttpRequest();

/**
 * 发送请求
 */
function sendRequest() {
    xhr.open('GET', 'http://localhost:8080/erick/timeout');
    xhr.send();
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            if (xhr.status >= 200 && xhr.status < 300) {
                console.log('响应体=', xhr.response);
            }
        }
    }
}

/* 取消请求*/
function cancelRequest() {
    xhr.abort();
}
```

## 5. 重复请求

- 用户多次点击同一个按钮，则取消当前请求，避免多次请求

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="cancel.js"></script>
</head>
<body>
<button onclick=sendRequest()>发送请求</button><br>

</body>
</html>
```

```js
let isSending = false;

function sendRequest() {
    if (isSending) {
        // 取消当前请求
       return;
    }

    let xhr = new XMLHttpRequest();
    isSending = true; // 创建好xhr之后，就代表已经开始发送请求了

    xhr.open('GET', 'http://localhost:8080/erick/timeout');
    xhr.send();
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            // 这个时候请求已经处理完毕了
            isSending = false;

            if (xhr.status >= 200 && xhr.status < 300) {
                console.log('响应体=', xhr.response);
            }
        }
    }
}
```

```java
package com.citi.erick.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.TimeUnit;

@RestController
@RequestMapping("/erick")
@CrossOrigin()
@Slf4j
public class TimeoutController {

    @GetMapping("/timeout")
    public String sleep() throws InterruptedException {
        log.info("timeout coming");
        TimeUnit.SECONDS.sleep(4);
        log.info("timeout ending");
        return "hello erick";
    }
}
```

## 6. 同源策略

- 浏览器的一种安全策略
- 浏览器发送的请求，和后台的服务，协议，域名，端口号必须完全相同，否则报错
- 违反同源策略的就是跨域
- Ajax发送请求时，默认要遵循同源策略

```bash
# 跨域问题：
- 前端发送数据了，后端也会接受到请求
- 但是数据回不来
```

### 6.1 CORS

- 官方的跨域解决方案：Cross-Origin Resource Sharing
- 在浏览器客户端不做任何特殊操作，完全在服务端中进行处理
- 在服务端设置一个响应头，告知浏览器，该请求允许跨域。浏览器收到响应头后就会响应

```java
package com.citi.erick.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.TimeUnit;

@RestController
@RequestMapping("/erick")
@CrossOrigin() // 就会添加一个响应头： Access-Control-Allow-Origin: *
@Slf4j
public class TimeoutController {

    @GetMapping("/timeout")
    public String sleep() throws InterruptedException {
        log.info("timeout coming");
        TimeUnit.SECONDS.sleep(4);
        log.info("timeout ending");
        return "hello erick";
    }
}
```

# Axios

## 1. 基本介绍

- 基于Promise的Http客户端，可以在浏览器和Node.js环境中使用
- [官方DOC](https://axios-http.com/docs/intro)
- npm install axios

```bash
# 浏览器端： ajax请求
- axios使用XMLHttpRequest对象发送HTTP请求
- XMLHttpRequest是浏览器提供的内置对象，将请求发送到服务端，并处理服务器的响应
- axios创建和配置XMLHttpRequest对象

# Node端： http模块
- axios使用http模块发送HTTP请求
- http模块是Node.js的内置模块，将请求发送到服务端，并处理服务器的响应
- axios创建和配置http.ClientRequest对象
```

## 2. axios(config)

- 传入具体的请求的详细信息

### GET

```java
package com.citi.erick.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/get")
@CrossOrigin()
@Slf4j
public class StudentController {

    @GetMapping("/sleep")
    public String sleep() {
        log.info("sleep coming");
        return "hello erick";
    }

    @GetMapping("/eat")
    public String eat(@RequestParam(name = "name") String name,
                      @RequestParam(name = "age") String myName) {
        log.info("name={},age={}", name, myName);
        return name + myName;
    }

    /*响应Json*/
    @GetMapping("/getAll")
    public People getAll(@RequestParam(name = "prefix") String prefix) {
        People people = new People();
        people.setAddress(prefix + "xian");
        people.setAge(prefix + "45");
        people.setName(prefix + "shuzhan");
        return people;
    }
}
```

```js
const axios = require('axios');

/*1. 创建这个对象后，就已经发送了请求
* 2. axios()返回的是一个Promise对象*/

/*get无参请求*/
axios({
    method: 'GET',
    url: 'http://localhost:8080/get/sleep',
}).then(value => {
    console.log(value.data);     // 响应体
    console.log(value.status)    // 响应码
}).catch(error => {
    console.log(error)
})

/*get带参数*/
axios({
    method: 'GET',
    url: 'http://localhost:8080/get/eat?name=shuzhan&age=12',
}).then(value => {
    console.log(value.data);
    console.log(value.status)
}).catch(error => {
    console.log(error)
})

/*get带参数*/
axios({
    method: 'GET',
    url: 'http://localhost:8080/get/eat',
    params: {
        name: 'zhangsan',
        age: 70
    }
}).then(value => {
    console.log(value.data);
    console.log(value.status)
}).catch(error => {
    console.log(error)
});


/*get带参数返回json*/
axios({
    method: 'GET',
    url: 'http://localhost:8080/get/getAll',
    params: {
        prefix: 'shshs'
    }
}).then(value => {
    console.log(value.data);     // json数据
    console.log(value.status)
}).catch(error => {
    console.log(error)
});
```

### POST

```java
package com.citi.erick.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/post")
@Slf4j
@CrossOrigin
public class PostController {

    @PostMapping("/noargs")
    public String noargs() {
        log.info("post1 coming");
        return "erick";
    }

    @PostMapping("/withargs")
    public People savePeople(@RequestParam(name = "address") String myAddress,
                             @RequestParam(name = "name") String myName,
                             @RequestParam(name = "age") String myAge) {
        People people = new People();
        people.setAddress("param_" + myAddress);
        people.setName("param_" + myName);
        people.setAge("param_" + myAge);
        return people;
    }

    @PostMapping("/withRequestBody")
    public People savePeopleBody(@RequestBody People people) {
        people.setAddress("body_" + people.getAddress());
        people.setAge("body_" + people.getAge());
        people.setName("body_" + people.getName());
        return people;
    }
}
```

```js
const axios = require("axios");

/*无参post*/
axios({
    method: 'POST',
    url: 'http://localhost:8080/post/noargs',
}).then(value => {
    console.log(value.data);     // 响应体
}).catch(error => {
    console.log(error)
})


/*带参post*/
axios({
    method: 'POST',
    url: 'http://localhost:8080/post/withargs',
    params: {
        name: 'erick',
        age: 12,
        address: 'xian',
    }
}).then(value => {
    console.log(value.data);     // 响应体
}).catch(error => {
    console.log(error)
})

/*带参post*/
axios({
    method: 'POST',
    url: 'http://localhost:8080/post/withargs?name=erick&age=12&address=beijing'
}).then(value => {
    console.log(value.data);     // 响应体
}).catch(error => {
    console.log(error)
})

/*带参post*/
axios({
    method: 'POST',
    url: 'http://localhost:8080/post/withRequestBody',
    // 请求体,和后台数据一样的话，可以简写为data：data
    data: {
        address: 'nanjing',
        age: 30,
        name: 'lucy',
    }
}).then(value => {
    console.log(value.data);     // 响应体
}).catch(error => {
    console.log(error)
})
```

### 其他方式

- axis.get
- axis.post

```js
const axios = require("axios");

/*request*/
axios.request({
    method: 'GET',
    url: 'http://localhost:8080/get/sleep',
}).then(value => {
    console.log(value.data);
}).catch(error => {
    console.log(error)
});

/*get*/
axios.get('http://localhost:8080/get/eat?name=shuzhan&age=12')
    .then(value => {
        console.log(value.data);
        console.log(value.status)
    }).catch(error => {
    console.log(error)
});

/*post*/
axios.post('http://localhost:8080/post/withRequestBody', { //  // 请求体
    address: 'nanjing',
    age: 30,
    name: 'lucy',
}).then(value => {
    console.log(value.data);     // 响应体
}).catch(error => {
    console.log(error)
})
```

### 默认配置

- 可以配置一些全局的axios的默认配置

```js
const axios = require('axios');

axios.defaults.baseURL = 'http://localhost:8080';
axios.defaults.method = 'GET';

/*get无参请求*/
axios({

    url: 'get/sleep',
}).then(value => {
    console.log(value.data);
}).catch(error => {
    console.log(error)
})

/*get带参数*/
axios({
    url: 'get/eat?name=shuzhan&age=12',
}).then(value => {
    console.log(value.data);
}).catch(error => {
    console.log(error)
})
```

###  AxiosRequestConfig

- 传入到axios的参数

```java
interface AxiosRequestConfig<D = any> {
  url?: string;
  method?: Method | string;
  baseURL?: string;
  transformRequest?: AxiosRequestTransformer | AxiosRequestTransformer[];
  transformResponse?: AxiosResponseTransformer | AxiosResponseTransformer[];
  headers?: (RawAxiosRequestHeaders & MethodsHeaders) | AxiosHeaders;
  params?: any;
  paramsSerializer?: ParamsSerializerOptions | CustomParamsSerializer;
  data?: D;
  timeout?: Milliseconds;
  timeoutErrorMessage?: string;
  withCredentials?: boolean;
  adapter?: AxiosAdapterConfig | AxiosAdapterConfig[];
  auth?: AxiosBasicCredentials;
  responseType?: ResponseType;
  // 其他略
}
```

## 3. 创建axios实例

- 两个创建自定义axios的方法

```js
const axios = require("axios");

/*自己创建的axios对象，和axios默认的差不多*/

/*调用a服务的一个接口*/
let firstService = axios.create();
firstService.defaults.baseURL = 'http://localhost:8080';
firstService.defaults.method = 'GET';

firstService({
    url: 'get/sleep',
}).then(value => {
    console.log(value.data);
}).catch(error => {
    console.log(error)
})

/*调用b服务的一个接口*/
let secondService = axios.create({
    baseURL: 'http://localhost:8080',
    method: 'GET'
});
secondService({
    url: 'get/sleep',
}).then(value => {
    console.log(value.data);
}).catch(error => {
    console.log(error)
})
```

## 4. Interceptors

```bash
# 成功响应
2-request-success
1-request-success
1-response-success
2-response-success
hello erick

# request处理错误时
2-request-success
1-request-failure
1-response-failure
2-response-failure
cuole

# response处理错误时
2-request-success
1-request-success
1-response-failure
2-response-failure
AxiosError: Request failed with status code 404
```

```js
const axios = require('axios');

/**
 * 在发送请求时候，加入一些自定义修改或者验证
 * config: InternalAxiosRequestConfig，
 */
axios.interceptors.request.use(function (config) {
    // request发送之前的设置，必须返回config
    console.log('1-request-success');
    config.headers.set('k1', 'v1');
    return config;
}, function (error) {
    // request处理错误后，就会走到这里
    console.log('1-request-failure');
    return Promise.reject(error);
});

axios.interceptors.request.use(function (config) {
    console.log('2-request-success')
    throw 'cuole';
}, function (error) {
    // request发送错误后的设置
    console.log('2-request-failure');
    return Promise.reject(error);
});


/**
 * 在返回请求的时候，加入一些自定义验证
 * response：AxiosResponse
 *
 */
axios.interceptors.response.use(function (response) {
    // 2xx的回调，处理完响应结果后,必须return
    console.log('1-response-success');
    return response;
}, function (error) {
    // 其他所有状态码的回调
    console.log('1-response-failure');
    return Promise.reject(error);
});

axios.interceptors.response.use(function (response) {
    // 2xx的回调，处理完响应结果后,必须return
    console.log('2-response-success');
    return response;
}, function (error) {
    console.log('2-response-failure');
    return Promise.reject(error);
});


/*拦截器必须放在上面*/
axios.defaults.baseURL = 'http://localhost:8080';
axios.defaults.method = 'GET';

/*get无参请求*/
axios({
    url: 'get/sleep/fh',
}).then(value => {
    console.log(value.data);
}).catch(error => {
    console.log(error)
})
```

