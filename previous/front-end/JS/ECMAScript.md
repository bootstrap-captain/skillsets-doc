# 数据类型

## 4. Object

```js
let people = {
    username: 'erick',
    age: 20,
}

let keys = Object.keys(people);  // 数组
let values = Object.values(people); // 数组
console.log(keys)      // [ 'username', 'age' ]
console.log(values)     // [ 'erick', 20 ]

let p = {};
Object.assign(p, people);  // 对象拷贝
console.log(p)
```

## 5. 展开运算符

```js
// 1. 展开一个数组 1 2 3
let arr1 = [1, 2, 3];
console.log(...arr1);

// 2. 合并两个数组
let arr2 = [4, 5, 6];
let arr3 = [7, 8, 9];
const num = [...arr2, ...arr3]; // 合并两个数组
console.log(...num);

// 3.深克隆对象
let person = {name: 'erick', age: 20};
let person_1 = {...person};
person_1.name = 'jack';
console.log(person_1, person);

// 4.克隆同时进行修改
let dog = {name: 'mimi', age: 2};
let dog2 = {...dog, name: 'xiaohua'};
console.log(dog, dog2);
```



# 对象



## 2. 构造函数

- 也是一种创建对象的方式

### 2.1 实例属性/方法

- 公共的属性和方法，可以封装在构造函数中
- 封装的方法，存在浪费内存问题，不同的对象new出来后，所指向的方法，都是一个单独的内存，不能复用

```js
function People(username, address) {
    this.username = username;
    this.address = address;
    /*匿名函数， 具名函数， 箭头函数 都可以放*/
    this.sayHello = function () {
        console.log('hello')
    }
}

let people = new People('erick', 'xian'); // 使用new关键字调用函数

people.sayHello();
```

### 2.2 静态属性/方法

- 只能通过类名调用，不能通过实例调用

```js
function People(username, address) {
    this.username = username;
    this.address = address;
    this.workHard = function work() {
        console.log('work hard');
    }
}

/*定义静态方法*/
People.staticMethod = function () {
    console.log('静态方法')
}

/*定义静态成员*/
People.info = '静态属性'

let people = new People('erick', 'xian'); // 使用new关键字调用函数

console.log(people.info); // undefined， 不能通过实例调用
console.log(People.info);
People.staticMethod()
```

## 3. 原型

### 3.1 原型对象

- js中，每个构造函数都有一个prototype属性，指向另一个对象，称prototype为原型对象
- 可以把不变的方法，直接定义在prototype对象上，这样所有对象的实例可以共享这些方法

```bash
# 公共的属性，写到构造方法中
# 公共的方法，写到原型对象中
```

```js
function People(username, address) {
    /*公共的属性*/
    this.username = username;
    this.address = address;
}

/*公共的方法*/
People.prototype.workHard = function () {
    console.log('公共的方法')
}

let people1 = new People('erick', 'xian');
let people2 = new People('lucy', 'beijing');

people1.workHard();

console.log(people1.workHard === people2.workHard); // true
```

- 构造函数和原型对象中的this，都指向实例化的对象

```js
let thisConstructor;
let thisPrototype;

function People(username, address) {
    /*公共的属性*/
    this.username = username;
    this.address = address;
    thisConstructor = this;
}

/*公共的方法*/
People.prototype.workHard = function () {
    console.log('公共的方法')
    thisPrototype = this;
}

let people = new People('erick', 'xian');

people.workHard();

/*都指向当前对象*/
console.log(people === thisConstructor && people === thisPrototype);
```

-  扩展js本身的库，自定义函数

```js
let arr = [1, 2, 3];

/*给array加了一个方法*/
Array.prototype.getSum = function () {
    let length = this.length;
    let result = 0;
    for (let i = 0; i < length; i++) {
        result += this[i];
    }
    return result;
}

console.log(arr.getSum());
```

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

## 4. ES6-class

### 4.1 基本语法

```js
class People {
    /*1. 必须要写，不然传入的参数接收不到*/
    constructor(name, address) {
        /*this: 类的实例对象*/
        this.name = name;
        this.address = address;
    }

    /*2. 一般方法
    *  speak():
    *     2.1 并不是在实例对象上，而是放在了类的原型对象上
    *     2.2 供实例使用
    *     2.3 this:
    *              - 如果使用people1实例调用speak, this就是people1实例
    *              - call调用，更改this指向，传什么，this就是什么
     */
    speak() {
        console.log(this);
        console.log(this.name + this.address);
    }
}

let people1 = new People('tom', 'xian');
let people2 = new People('lucy', 'nanjing');

console.log(people1);
people1.speak();

console.log("===========")

console.log(people2);
people1.speak.call({a: 1, b: 2});  /*call调用，更改this指向，传什么，this就是什么*/
people2.speak();
```

```js
class People {
    say() {
        console.log('say---')
    }

    speak() {
        console.log('speak---');
        // 必须加this，不然会报错
        this.say();
    }
}

let people = new People();
people.speak();
```



```js
class People {

    /*构造函数*/
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    /*函数必须是这种写法*/
    say() {
        console.log('hello');
    }

    /*静态方法和属性: 只能通过类调用，不能通过实例对象调用*/
    static region = 'cn';

    static work() {
        console.log('work')
    }
}

let people = new People('erick', 20);

console.log(people.age + people.name);
people.say();
People.work();
```

### 4.2 继承

```js
class People {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    say() {
        console.log('people')
    }

    work() {
        console.log('people work hard')
    }

}

class Boy extends People {
  // 如果不写该构造器，则会自动调用父类构造器，传递name和age，其他参数就会丢失
    constructor(name, age, sex, address) {
        super(name, age); // 对父类的属性进行初始化
        this.sex = sex;
        this.address = address;
    }

    /*自定义方法*/
    fish() {
        console.log('boy fishing')
    }

    /*方法重写
    * 方法名相同：就会认为是重写*/
    say() {
        console.log('boy dancing')
    }
}

let boy = new Boy('erick', 20, 'boy', 'xian');
boy.fish();
boy.say();
boy.work();  // 子类可直接调用父类的方法和属性
console.log(boy.age);
```

### 4.2 私有属性

```js
class People {
    // 公有属性
    name;
    age;

    // 私有属性
    #info;

    constructor(name, age, info) {
        this.name = name;
        this.age = age;
        this.#info = info;
    }
}

class Boy extends People {
    sex;
    address;

    constructor(name, age, info, sex, address) {
        super(name, age, info); // 对父类的属性进行初始化
        this.sex = sex;
        this.address = address;
    }
}


let boy = new Boy('erick', 20, 'boy', 'xian');
console.log(boy.sex);
console.log(boy.info); // 获取不到
```

