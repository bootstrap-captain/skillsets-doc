# 对象



### 3.2 constructor属性

#### 介绍

- 每个构造函数中，都包含一个原型对象，原型对象中，包含一个constructor属性，constructor属性再指向构造函数

![image-20240513104520047](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240513104520047.png)

```js
function People(username, password) {
    this.username = username;
    this.password = password;
}

console.log(People.prototype.constructor === People); // true
```

#### 场景

```js
function People(username, password) {
    this.username = username;
    this.password = password;
}

/*可以把公共的方法，都放在原型链中*/
People.prototype = {

    /*必须要加，不然这个原型链就是把之前的原型链替换掉了*/
    constructor: People,

    sing: function (){
        console.log('sing');
    },

    work: function (){
        console.log('work')
    }
}

console.log(People.prototype)
```

### 3.3 对象原型

- 属于每个实例对象的，指向构造函数的prototype原型对象

```bash
# 对象原型
- 每个实例对象，都有一个属性 __proto__,指向构造函数的prototype原型对象
- 因此实例对象，可以通过属性 __proto__，访问到prototype原型对象
```

 ![image-20240513112648429](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240513112648429.png)

```js
function People(username, password) {
    this.username = username;
    this.password = password;
}

let people = new People('erick', 'xian');

console.log(people.__proto__ === People.prototype); //true
```

### 3.4 原型继承

- js中大多是借助原型对象来实现对象的继承

```js
/*父类*/
function People() {
    this.eye = 'blue';
    this.mouth = 'open';
}


/*子类1*/
function Boy() {
}

Boy.prototype = new People(); // 继承
Boy.prototype.constructor = Boy; // 指回去
Boy.prototype.fish = function () {
    console.log('go fishing')
}

/*子类2*/
function Girl() {
}

Girl.prototype = new People(); // 继承
Girl.prototype.constructor = Girl; // 指回去
Girl.prototype.dance = function () {
    console.log('go dancing')
}

console.log(new Boy())
console.log(new Girl())
```





