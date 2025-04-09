# Promise

- ES6引入的，js中的异步编程的解决方案
- Promise是一个构造函数

## 1. 入门案例

- 将异步代码放在Promise函数中

### 1.1 定时器

```js
/*promise方式*/
let flag = true;

function saveDBWithPromise() {
    /*1. resolve：成功时候调用，函数类型的数据
    * 2. reject： 失败时候调用，函数类型的数据
    * 可以包裹一个异步任务*/
    let promise = new Promise((resolve, reject) => {
        /*异步任务*/
        setTimeout(function () {
            console.log(`flag=${flag}`);
            if (flag) {
                resolve(flag + '-erick-success'); // 成功,将promise对象的状态设置为成功，可以传递结果
            } else {
                reject(flag + '-erick-failure');  // 失败，将promise对象的状态设置为失败
            }
            flag = !flag;
        }, 2000);
    });

    /*resolve和reject的解析*/
    promise.then((value) => {
        alert(value);    // 成功
    }, (reason) => {
        alert(reason)   // 失败
    })
}
```

### 1.2 文件读取

- 直接以node.js来运行

```js
let fs = require('fs');

let promise = new Promise((resolve, reject) => {
    /*异步任务*/
    fs.readFile('erick.txt', (err, data) => {
        if (err) {
            reject(err);
        } else {
            resolve(data);
        }
    });
})

promise.then((value) => {
    console.log(`file content=${value}`)
}, (reason) => {
    console.log(`error reason=${reason}`)
})
```

### 1.3 Ajax请求

```js
function sendRequest() {
    let promise = new Promise((resolve, reject) => {
        let xhr = new XMLHttpRequest();
        xhr.open('GET', 'http://localhost:8080/erick/timeout');
        xhr.send();
        xhr.onreadystatechange = function () {
            if (xhr.readyState === 4) {
                if (xhr.status >= 200 && xhr.status < 300) {
                    resolve(xhr.response);
                } else {
                    reject(xhr.status);
                }
            }
        }
    });

    promise.then((value) => {
        console.log(value); // 成功回调
    }, (reason) => {
        console.log(reason); // 失败回调
    })
}
```

## 2. 封装Ajax请求Promise

- 将一个异步请求，进一步封装成，返回一个Promise对象的方法
- 调用这个方法时，只需要自定义resolve和reject处理方式

```js
/*封装好Promise函数*/
function sendRequest(method, url) {
    return new Promise((resolve, reject) => {
        let xhr = new XMLHttpRequest();
        xhr.open(method, url);
        xhr.send();
        xhr.onreadystatechange = function () {
            if (xhr.readyState === 4) {
                if (xhr.status >= 200 && xhr.status < 300) {
                    resolve(xhr.response);
                } else {
                    reject(xhr);
                }
            }
        }
    });
}

/*外面调用的就是这个方法*/
function createUser() {
    sendRequest('GET', 'http://localhost:8080/erick/timeout').then((value) => {
        console.log(value);
    }, (reason) => {
        console.log(reason);
    })
}
```

## 3. Promise流程

![image-20240526205322880](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240526205322880.png)

### 3.1 PromiseStatus

- promise的状态：promise实例对象中的一个属性

#### 状态转换

- 一个promise对象的状态只能转变一次

```bash
# 三种状态: pending, resolved(fulfilled), rejected, 并且只有两种转换方式
- pending ---> resolved             成功结果一般称为value
- pending ---> rejected             成功结果一般称为reason
```

#### 转换方式

```js
/*1. pending--->resolved/fulfilled*/
let p1 = new Promise((resolve, reject)=>{
    resolve('erick');
})
console.log(p1);


/*2. pending--->rejected
* 必须要进一步处理*/

/*2.1 : reject*/
let p2 = new Promise((resolve, reject)=>{
    reject('dam');
})

p2.catch(reason => {
    console.log(reason); // dam
})

/*2.2: 抛出异常*/
let p3 = new Promise((resolve, reject)=>{
   throw 'oh shit';
})

p3.catch(reason => {
    console.log(reason); // on shit
})
```

#### 转换回调

- 一旦状态发生转换，则该转换所对应的回调都会发生

```js
let p = new Promise((resolve, reject) => {
    resolve('erick');
});

/*两个回调都会发生*/
p.then(value => {
    console.log('first');
});

p.then(value => {
    console.log('second');
});
```

### 3.2 PromiseResult

- promise的结果：promise实例对象中的一个属性
- 存储异步任务，成功或失败的结果

## 4. API

```js
/**
 * executor: 执行器： 会在Promise内部立刻同步调用，异步操作在执行器中执行
 * 1. resolve函数： 内部定义成功时，回调的函数
 * 2. reject函数：  内部定义失败时，回调的函数
 */
let fs = require('fs');
let promise = new Promise((resolve, reject) => {
    /*2. 异步任务： 异步调用*/
    fs.readFile('erick.txt1', (err, data) => {
        if (err) {
            reject(err);
        } else {
            resolve(data);
        }
    });

    console.log('同步方法');  // 1: 同步执行：执行方法
})

promise.then((value) => {
    console.log(`value=${value}`)
}).catch(reason => { // 3. catch: 相当于把then中的第二个拆出来，单独做个语法糖
    console.log(`catch_reason=${reason}`)
})
```

## 5.静态方法

- 属于Promise构造函数的静态方法

### 5.1 resolve()

```js
// 1. 参数：非Promise类型对象，返回结果：PromiseState: fulfilled      PromiseResult:erick
let promise1 = Promise.resolve("erick");
console.log(promise1);

// 2.参数： Promise类型对象，则参数的结果决定了resolve的结果
//   2.1 PromiseState: resolved      PromiseResult:success
let promise2 = Promise.resolve(new Promise((resolve, reject) => {
    resolve('success');
}))
console.log(promise2);

//  2.2 PromiseState: rejected      PromiseResult:failure
let promise3 = Promise.resolve(new Promise((resolve, reject) => {
    reject('failure');
}));

// 如果是失败的结果，则需要处理
promise3.catch(reason => {
    console.log(reason);
})
```

### 5.2 reject()

- 返回一个失败的promise对象，不管传入的是什么类型的数据

```js
// 返回失败的
let promise = Promise.reject("erick");
console.log(promise);

// 必须处理
promise.catch(reason => {
    console.log(reason);
});
```

### 5.3 all()

- 包含n个promise的数据，只有所有的promise成功，返回的promise才成功
- 如果全部成功，则PromiseResult中是一个成功结果的数组['p1-success',521,'hello']

#### 全部成功

- PromiseResult中是一个成功结果的数组['p1-success',521,'hello']
- PromiseState: fulfilled
- 在node项目中不可以

```js
// 全部成功
let p1 = Promise.resolve('p1-success');
let p2 = Promise.resolve(521);
let p3 = new Promise((resolve, reject) => {
    resolve('hello');
})

const promise = Promise.all([p1, p2, p3]);

console.log(promise)
```

#### 部分成功

- 部分成功，则返回的是一个失败的promise对象，要做异常处理

```js
// 全部成功
let p1 = Promise.resolve('p1-success');
let p2 = Promise.reject(521);
let p3 = new Promise((resolve, reject) => {
    resolve('hello');
})

const promise = Promise.all([p1, p2, p3]);

promise.catch(reason => {
    console.log(reason);
})
```

### 5.4 race()

- 包含n个promise的数组，返回一个新的promise
- 第一个完成的promise的结果状态就是最终的状态

## 6. then()

- 返回一个新的Promise对象
- 返回结果：then指定的回调函数执行的结果决定

### 6.1 抛出异常

```js
let p = new Promise((resolve, reject) => {
    resolve('success');
})

let p2 = p.then(value => {
    //  p2就是rejected
    throw 'fuck';
}, reason => {

});

console.log(p2)
```

### 6.2 非Promise对象

- 成功的promise，状态是fulfilled，且PromiseResult是返回的值521

```js
let p = new Promise((resolve, reject) => {
    resolve('success');
})

let p2 = p.then(value => {
    return 521;
}, reason => {

});

console.log(p2)
```

### 6.3 Promise对象

- 返回包含的Promise的对象结果

```js
let p = new Promise((resolve, reject) => {
    resolve('success');
})

let p2 = p.then(value => {
   //  根据这个来决定失败和成功
    return new Promise((resolve, reject)=>{
        resolve('hahah');
    });
}, reason => {

});

console.log(p2)
```

## 7. 任务串联

### 7.1 链式调用

```js
/*异步任务一*/
let p = new Promise((resolve, reject) => {
    resolve('firstJob');
})

p.then(value => {
    /*异步任务二*/
    return new Promise((resolve, reject) => {
        resolve(value + '-secondJob')
    });
}, reason => {
}).then(value => {
    console.log(value);     // firstJob-secondJob
}).then(value => {
    console.log(value);      // undefined,如果需要值，则要在上面的then中将结果返回
})
```

### 7.2. 异常穿透

```js
/*异步任务一*/
let p = new Promise((resolve, reject) => {
    resolve('firstJob');
})

p.then(value => {
    /*异步任务二*/
    return new Promise((resolve, reject) => {
        reject(value + '-secondJob')
    });
}, reason => {
}).then(value => {
    /*异步任务三*/
    return new Promise((resolve, reject) => {
        resolve(value + '-thirdJob')
    });
}).catch(reason => {
    console.log(reason); // firstJob-secondJob
})
```

### 7.3 任务中断

- 链式调用如何中断

```js
/*异步任务一*/
let p = new Promise((resolve, reject) => {
    resolve('firstJob');
    console.log('first coming'); // 先执行
})

p.then(value => {
    /*异步任务二*/

    /*中断任务的唯一方式*/
    if (value === 'firstJob') {
        return new Promise(() => {
        })
    }
    
    return new Promise((resolve, reject) => {
        resolve(value + '-secondJob')
        console.log('second coming'); // 先执行
    });
}, reason => {
}).then(value => {
    /*异步任务三*/
    return new Promise((resolve, reject) => {
        resolve(value + '-thirdJob')
        console.log('third coming'); // 先执行
    });
}).catch(reason => {
    console.log(reason); // firstJob-secondJob
})
```

## 8. async/await

### 8.1 async

- 用来标记一个函数
- 返回结果是一个Promise对象，promise对象的结果由async函数执行的返回值决定

```js
/*返回一个普通数据*/
async function createUser() {
    return 'shuzhan';
}

/*是一个成功的promise对象： fulfilled*/
let promise = createUser();
console.log(promise)
```

```js
/*返回一个Promise对象*/
async function createUser() {
    return new Promise((resolve, reject) => {
        resolve('success');
    })
}

/*是一个成功的promise对象： fulfilled*/
let promise = createUser();
console.log(promise)
```

### 8.2 await

```bash
# await右侧表达式
- promise对象： await返回的是promise成功的值
- 其他值：      可以将此值直接作为await的返回值

# 注意
- await必须写在asynch函数中，但 async函数中可以没有await
- 如果await的promise失败了，就会抛出异常，需要try。。catch来处理
```

```js
async function createUser() {
    let promise = new Promise((resolve, reject) => {
        // 成功
        resolve('shuzhan');
    });

    /* 右侧为promise*/
    let val = await promise;

    console.log(val);
}

createUser();
```

```js
async function createUser() {
    let promise = new Promise((resolve, reject) => {
        // 失败
        reject('shuzhan');
    });

    /* 右侧为promise*/
    let val;
    try {
        val = await promise;
    } catch (e) {
        console.log('failure')
    }
}

createUser();
```

```js
async function createUser() {
    /* 右侧为普通数据*/
   let result = await 10;
    console.log(result);
}

createUser();
```

### 8.3. Ajax综合案例

```js
function sendRequest(method, url) {
    return new Promise((resolve, reject) => {
        let xhr = new XMLHttpRequest();
        xhr.open(method, url);
        xhr.send();
        xhr.onreadystatechange = function () {
            if (xhr.readyState === 4) {
                if (xhr.status >= 200 && xhr.status < 300) {
                    resolve(xhr.response);
                } else {
                    reject(xhr);
                }
            }
        }
    });
}

async function callApi() {
    const METHOD = 'GET';
    const URL = 'http://localhost:8080/erick/timeout'
    try {
        let result = await sendRequest(METHOD, URL);
        console.log(result)
    } catch (e) {
        console.log(e)
    }
}
```

