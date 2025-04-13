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



