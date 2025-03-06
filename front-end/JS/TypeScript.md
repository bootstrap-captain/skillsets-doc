# 简介

## 1. 介绍

- TS是由微软开发的，是基于JS的一个扩展语言
- TS需要编译为JS，然后交给浏览器或者其他JS运行环境执行

```bash
# 静态类型检查
- 在代码运行前进行检查，发现代码的错误喝不合理之处，减少运行时异常的出现的几率
- 把运行时的错误前置

# TS代码量
- TS的代码量是大于JS的，但是TS的代码结构更加清晰，在后期代码维护中，较为方便
```

## 2. 安装

```bash
1. 下载node.js
2. 安装typescript:    npm i -g typescript        # ts解析器
```

## 3. 编译模式

### 3.1  单个文件

- tsc demo01.ts -w        热更新编译
- tsc        单次编译

```typescript
/* 1. tsc erick.ts -w    :  监视文件，发生变化后,ctrl+s 重新编译
*       1.1 命令行进行监视：只能监视一个文件，多个文件要开多个窗口
*       1.2 关闭窗口后，监视就会中止
*       1.3 ts erick.ts -watch
*  2. tsc erick -w
*  3. tsc erick */
/**
 *
 */
let address :string = 'zhangsan';
console.log(address);
```

### 3. 2. 批量编译

- 在项目根目录下，执行tsc -init，就会生成一个tsconfig.json
- 进入项目目录，执行tsc或tsc -w，就会编译所有的ts文件为js文件

### 3.3 参数配置

![image-20241026133735800](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20241026133735800.png)

```json
{
  /*编译的详细参数*/
  "compilerOptions": {
    /*模块化规范: commonJs， ES6*/
    "module": "ES6",
    /*编译后的js的版本， 默认是ES3*/
    "target": "ES6",
    /*"lib": []   ts中引入的js的内置的库，如DOM，在写ts代码时，会有提示
                  一般不写，会有默认值
                  一旦写上，就会替换 */

    /*编译后的 js 文件的存放位置, 会自动在当前文件夹创建dist文件夹*/
    "outDir": "./dist",
    /*是否编译js文件: 默认false, 设计到文件的移动 文件从 src 移动到 dist中*/
    "allowJs": false,
    /*是否用ts的规范去检测js文件: 默认false*/
    "checkJs": false,
    /*是否在js文件中包含注释， 默认false*/
    "removeComments": false,
    /*当ts有语法错误时，不会生成js*/
    "noEmitOnError": true,
    /*是否开启js的严格模式： 严格模式性能更好， 会在生成的js文件中，带上 "use strict" 的头*/
    "alwaysStrict": true,
    /*不允许隐式的 any类型，也就是隐式的不定义类型*/
    "noImplicitAny": true,
    /*严格检查空值*/
    "strictNullChecks": true,
    /*所有严格检查的总开关*/
    "strict": true
  },
  /*哪些文件不需要编译
   默认值：["node_modules", "bower_components","jspm_packages"]*/
  "exclude": [
  ],
  /*哪些文件需要实时编译
        两个*： 任意目录
        一个*： 任意文件
   */
  "include": [
    "./src/**/*"
  ]
}
```

# 基本语法

## 1. 入门案例

### 1.1 使用方式

- 一般对变量，如果声明后直接赋值，可以不指定类型，因为ts文件会默认进行类型推荐

```ts
/*1. 先声明后赋值*/
let info: string;
info = 'haha';

/*2. 声明后直接赋值*/
let flag: boolean = false;
let age: number = 12;

/*3. 类型推断: 先赋值，不声明类型，就会产生类型推断： 推荐！！！
    因为是在ts文件中，所以第一次赋值后，就会对该变量产生类型推荐，后续不能再次重新赋值其他类型的*/
let isEnabled = false;

/* 就会报错，因为是ts文件*/
isEnabled = 'erick';
```

### 1.2 函数

- 对函数作用更大，因为函数的形参和返回值肯定是先声明，后使用

```ts
/*4. 类型声明：对函数作用更大*/
function sum01(a, b) {
    return a + b;
}

console.log(sum01(1, 2));

function sum02(a: number, b: number) {
    return a + b;
}

console.log(sum02(1, 4));
```

```bash
# 编译
- 进入该文件目录，执行  tsc erick.ts 
- 会在当前文件夹，产生一个新的同名的js文件

# 执行， 不能直接执行 node erick.ts
- node erick.js
```

## 2. 变量类型

### JS-数据类型

```bash
# JS中的内置构造函数： Number, String, Boolean等，用于创建对应对象的包装对象
- 日常开发中很少使用

# TS进行类型声明的时候，通常都是用小写的number，string， boolean
```

#### number/boolean/string

```ts
let a: number = 19;
let b: boolean = false;
let c: string = 'erick';
```

```ts
let str1: string;
let str2: String;

str1 = 'erick';
// 报错 str1 = new String('adf');

/*可以赋值为基本数据类型string和包装类型*/
str2 = 'afaf';
str2 = new String('adfa');
```

### TS-新增类型

#### any

- 相当于关闭了ts类型检测，如果有可能，把这个东西忘掉
- any的变量可以赋值给其他任何变量，哪怕转换失败也不会报错

```ts
/* 1. 显示声明为any的变量， 可以对该变量任何属性值*/
let a: any;

a = true;
console.log(a);
a = 'nice';
console.log(a);

/*2. any变量，可以给任何其他变量， 会将其他变量也变成any
*    2.1 编译不报错
*    2.2 运行不报错
* */
let b: number;
b = 1;
console.log(b);
b = a;
console.log(b);
```

```ts
/* 先声明变量，但不指定类型，后赋值的变量
  - ts会默认给一个any属性*/
let a;
a = 1;
console.log(a);
a = 'hello';
console.log(a);
```

#### unknown

- 任意类型，但是是一个类型安全的any
- unknown 只能自己变化类型，不能赋值给任何其他显示声明的类型(编译错误)

```ts
let a: unknown;
/*1.可以自己变换类型*/
a = 1;
console.log(a);
a = 'haha';
console.log(a);

/*2. unknown 赋值给其他显示声明的，会编译错误
* 哪怕上面的a，已经是string*/
let b: string;
// b = a;
```

- 如果要将unknown  赋值给其他的，可以通过下面三种方式

```ts
let a: unknown;
a = 'erick';

/*赋值给其他声明了类型的*/
let b: number;


/* 1. 方式一：类型检查: 只有符合时候才会转换*/
if (typeof a === "number") {
    b = a;
    console.log(`1===${b}`)
}

/*2. 方式二：强制转换: 解决了编译问题，但是最终b='erick*/
b = a as number;
console.log(`2===${b}`)

/*3. 方式三：强制转换*/
b = <number>a;
console.log(`3===${b}`)
```

#### never

- 任何值都不是，不能有值，undefined, null, '',0都不行
- 不会有返回值结果，用于专门来抛出异常的方法，不要去限制变量
- 用的很少

```ts
// 不要去限制变量
let a: never;

// 编译错误 a = 'adf';
function say(flag: boolean): void {
    if (flag) {
        fn03();
    } else {
        console.log('hi');
    }
}

/*异常的处理*/
function fn03(): never {
    throw new Error("failure");
}

say(true);
```

#### void

- 函数返回值类型

```ts
// 默认返回值：undefined
function fn01(): void {

}

function fn02(): void {
    return;
}

function fn03(): void {
    return undefined;
}
```

#### 数组

- js中默认数组中元素可以存放不同类型数据

```ts
/*声明数组变量的两种方式*/
let arr1: string[];
let arr2: Array<number>;

arr1.push("erick");
arr2.push(2);
// arr1.push(true); 报错
```

#### tuple

- 元组:  固定长度的数组，相比js，ts新加入的数据类型

```ts
let arr: [string, number, boolean?];
/*元组在赋值时候，个数和类型必须严格和声明时候一样*/
arr = ['df', 12, true];
console.log(arr)
```

#### enum

- 数字枚举

```ts
/*定义一个枚举*/
enum Brand {
    APPLE,
    XIAOMI,
    HUAWEI,
    OPPO
}

/*1.赋值*/
let phone: Brand = Brand.APPLE;
console.log(phone); // 0
console.log(phone.valueOf()); // 0

let res: string = Brand[phone]; // APPLE
console.log(res);
```

- 字符串枚举

```ts
/*定义一个枚举*/
enum Brand {
    APPLE = 'iphone-15',
    XIAOMI = 'xiaomi-14',
    HUAWEI = 'huiwei-12',
    OPPO = 'oppo-10'
}

let phone: Brand = Brand.APPLE;
console.log(phone); // iphone-15
```

- 常量枚举
- 只需要在enum前加关键字const，生成的js代码会小很多，提升js运行时性能

```ts
/*定义一个枚举*/
const enum Brand {
    APPLE = 'iphone-15',
    XIAOMI = 'xiaomi-14',
    HUAWEI = 'huiwei-12',
    OPPO = 'oppo-10'
}

let phone: Brand = Brand.APPLE;
console.log(phone); // iphone-15
```

### TS-类型进阶

#### 联合类型

- 一个变量为多种类型中，定义参数时，可以指定多种不同的数据类型

```ts
/*将入参：限定为某几种参数*/
function say(info: string | number | boolean) {
    console.log(info)
}

say('hello');
say(1);
say(true);
```

#### 字面量

- 一般用来定义枚举类型的

```ts
/*定义一个枚举类型的字面量
*  1. 后续使用该值时，只能从这里定义的去选 */
let fruit: 'apple' | 'banana' | boolean;

fruit = 'banana';
console.log(fruit);

fruit = "banana";
console.log(fruit);

fruit = false;
console.log(fruit);

// fruit = "water";  就会编译报错
```

```ts
export type Student = {
    // 如果字段类型是枚举性质的，则可以
    address: 'beijing' | 'nanjing' | 'shenzhen',
}

let firstStudent: Student = {
    address: 'beijing',
}

let secondStudent: Student = {
    address: 'nanjing',
}

/*就会报错*/
/*
let third:Student = {
    address: 'adf',
}*/
```

#### type

- 为一个现有的类型取一个别名

```ts
/*1. 联合类型*/
type Fruit = string | number | boolean;

let a: Fruit;
a = 'daf';

/*2. 字面量类型*/
type Gender = 'boy' | 'female' | 'male';
let b: Gender;
b = 'female';
```

- 声明一个对象类型的

```ts
type Fruit = {
    name?: string;
    address: string;
}

let fruit: Fruit = {
    name: 'apple',
    address: 'xian'
}
```

## 3. 对象类型

- 可以先声明变量为一个对象类型，并定义对象中的属性。后面为变量赋值

### 严格匹配

- 在赋值的时候，不能多，不能少，严格一致

```ts
/*  变量声明
* 1. 定义一个名字为person的变量
* 2. 该变量的类型限制为一个对象
* 3. 对象中所有属性都必须给*/
let person: {
    name: string,
    address: string,
    age: number
}

/* 变量赋值
* 少一个属性，都会报错*/
person = {
    name: 'erick',
    address: 'beijing',
    age: 10
}

console.log(person);
```

### 可选属性

```ts
/*  变量声明
* 1. 定义一个名字为person的变量
* 2. 该变量的类型限制为一个对象*/
let person: {
    name: string,
    address?: string
    age: number
}

/* 变量赋值*/
person = {
    name: 'erick',
    address: 'beijing',
    age: 10
}

console.log(person);

person = {
    name: 'nancy',
    age: 12
}
console.log(person);
```

### 多余属性

```ts
/*  变量声明
* 1. 定义一个名字为person的变量
* 2. 该变量的类型限制为一个对象
* 3. 允许传递多余的属性，属性名为string，属性value为any*/
let person: {
    name: string,
    [fieldName: string]: any;
}

/* 变量赋值*/
person = {
    name: 'erick',
    address: 'beijing',
    age: 10
}
```

### 解构赋值

- 可以正常使用解构赋值

```ts
let person: {
    userName: string,
    address: string,
    age: number
}

person = {
    userName: 'erick',
    address: 'beijing',
    age: 10
}

let {userName, address, age} = person;
console.log(userName);
console.log(address);
console.log(age)
```

## 4. 函数类型

```ts
/*定义一个函数的结构，定义函数参数类型，返回值类型*/
let say: (address: string, age: number) => string;

/*实际声明函数的实现*/
say = function (address: string, age: number) {
    return address + age;
}
```

# 常用技巧

## 1. 非空断言

```ts
// 1. 返回值是null
let a: string | null = null;

// ! : 跟在变量后面 非null断言
let b: string = a!;


if (a!) {
    console.log(a);
}
```

# 面向对象

## 1. 属性简写

```ts
class ErickService {
    name: string;
    address: String;

    public constructor(name: string, address: string) {
        this.name = name;
        this.address = address;
    }
}

let erickService = new ErickService('shuzhan', 'beijing');
console.log(erickService.address);
console.log(erickService.name);
```

```ts
class ErickService {
    /*简写形式：需要一定的修饰符号配合：这里的public，就是该类上对应属性的修饰符号*/
    public constructor(public name: string, public address: string) {
    }
}

let erickService = new ErickService('shuzhan', 'beijing');
console.log(erickService.address);
console.log(erickService.name);
```

## 2. 修饰符

```bash
# public
- 类内部，子类，类外部访问

# protected
- 类内部，子类

# private 
- 类内部

# readyonly
- 只读属性，一旦初始化完成后，不能更改
```

```ts
class ErickService {
    public constructor(public name: string, public readonly address: string) {
    }
}

let erickService = new ErickService('shuzhan', 'beijing');
console.log(erickService.address);
console.log(erickService.name);
erickService.name = 'lisi';
// 编译错误
// erickService.address='127.0.0.1';
```

## 3. Abstract

- 无法被实例化，定义类的结构和行为，可包含抽象方法和具体实现
- 为派生类提供一个基础结构，派生类必须实现其中的抽象方法
- 实现一些共有的方法

```ts
abstract class SearchService {
    /*1. 可以定义了一些公共属性*/
    public constructor(public input: string, public output: string) {
    }

    public abstract callApi(): number;

    public search = () => {
        console.log('start');
        this.callApi();
        console.log('end');
    }
}

class HomeSearchService extends SearchService {
    /*初始化抽象类的变量
    * 1.建议加上 override， 表示是为了抽象类的狗早起*/
    public constructor(public override input: string,
                       public override output: string,
                       public homeUrl: string) {
        super(input, output);
    }


    /*override: 可以不写，但是建议写*/
    override callApi(): number {
        console.log(this.homeUrl);
        return 1111;
    }
}

let homeSearchService = new HomeSearchService('in', 'out', '___');
homeSearchService.search();
```

## 4. Interface

- 接口只能定义格式，不能包含任何实现
- 接口多实现，接口可以继承一个接口

```ts
interface ISearchService {
    input: string,
    output: string

    callApi(): number;

}

class HomeSearchService implements ISearchService {
    /*上面定义好了对应的field*/
    public constructor(public input: string, public output: string,) {
    }

    callApi(): number {
        return 0;
    }
}

let homeSearchService: ISearchService = new HomeSearchService('in', 'out');
console.log(homeSearchService.input);
```

- 定义对应的的格式：比如返回的具体的数据

```ts
interface Result {
    input: string,
    output?: string
}
```

## 5. 范型

### 方法级别

```ts
class Erick {

    /*普通函数*/
    say<T>(input: T) {
        console.log(input)
    }

    /*箭头函数*/
    work = <L, M>(first: L, second: M): void => {
        console.log(first);
        console.log(second);
    }
}

let erick = new Erick();
/*方法级别*/
erick.say<string>('Hello World!');
erick.work<number, boolean>(1, true);
```

### 类级别

```ts
class Erick<T, L, M> {

    /*普通函数*/
    say(input: T) {
        console.log(input)
    }

    /*箭头函数*/
    work = (first: L, second: M): void => {
        console.log(first);
        console.log(second);
    }
}

/*类级别*/
let erick = new Erick<string, number, boolean>();
erick.say('Hello World!');
erick.work(1, true);
```

### 范型基类

```ts
interface SearchCondition {
    pageNo: number,
    pageSize: number;
}

interface HomeSearchCondition extends SearchCondition {
    name: string;
}

class Erick<T extends SearchCondition> {
    /*普通函数*/
    say(input: T) {
        /*可以获取基类的一些信息*/
        console.log(input.pageNo);
        console.log(input)
    }
}


let erick: Erick<HomeSearchCondition> = new Erick();
```

###  单个参数

```ts
/*默认就是any*/
export class FirstService<T> {
  // @ts-ignore
  data: T;
}

/*1. 可以不指定：则为 unknown*/
let firstService_1: FirstService<unknown> = new FirstService();

/*2. 指定：则为指定的类型*/
let firstService_2: FirstService<number> = new FirstService<number>();
```

```ts
/*显示标注为any*/
export class SecondService<T = any> {
  // @ts-ignore
  data: T;
}

let secondService_1: SecondService<any> = new SecondService();
let thirdService_2: SecondService<number> = new SecondService<number>();
```

```ts
let secondService_1: SecondService<any> = new SecondService();
let secondService_2: SecondService<number> = new SecondService<number>();


/*指定类型是某个的基类*/
export class ThirdService<T extends CommonSearchRequest> {
  // @ts-ignore
  data:T;
}

/*类型推断*/
let thirdService_1:ThirdService<CommonSearchRequest> = new ThirdService();
let thirdService_2:ThirdService<HomeSearchRequest> = new ThirdService<HomeSearchRequest>();
```

### 多个参数

- 通过=限定后，表明该参数是个可选参数，可以在后面使用时候可传可不传

```ts
/*通过=限定后，为可选参数*/
export class SixService<T = any, D = CommonSearchRequest> {
  // @ts-ignore
  data_1: T;
  // @ts-ignore
  data_2: D;
}

/*不传递*/
let sixService = new SixService();

/*传递一个*/
let sixService1 = new SixService<number>();

/*传递两个*/
let sixService2 = new SixService<number,HomeSearchRequest>();
```

- 不做类型限制时，参数如果全部传递，则必须全部传递

```ts
export class FiveService<T, D> {
  // @ts-ignore
  data_1: T;
  // @ts-ignore
  data_2: D;
}

let fiveService_1: FiveService<unknown, unknown> = new FiveService();
let fiveService_2: FiveService<number, string> = new FiveService<number, string>();
```

- extend的，必须全部传递

```ts
export class FourthService<T extends CommonSearchRequest, D extends CommonSearchRequest> {
  // @ts-ignore
  data_1: T;
  // @ts-ignore
  data_2: D;
}

let fourthService_1: FourthService<CommonSearchRequest, CommonSearchRequest> = new FourthService();
let fourthService_2: FourthService<CommonSearchRequest, HomeSearchRequest> = new FourthService<HomeSearchRequest, HomeSearchRequest>();
```

### 默认类型

```ts
function test<T = string>(name: T) {
    console.log(name);
}

/*使用默认类型*/
test("erick");

/*根据类型自定义推断*/
test(123);

/*显示指定类型*/
test<string>("erick");
```

