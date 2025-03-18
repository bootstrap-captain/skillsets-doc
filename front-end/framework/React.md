# 基本入门

## 1. Vite脚手架

```bash
npm create vite@latest
# 依次输入  项目名， React, TypeScript
```

![image-20250317112745263](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250317112745263.png)

## 2. 文件

### index.html

- SPA：single page application

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <title>Erick React</title>
</head>
<body>
<!--主容器：只会放一个组件-->
<div id="root"></div>
<!--引入main.tsc-->
<script src="/src/main.tsx" type="module"></script>
</body>
</html>
```

### main.tsc

- 入口文件，将APP组件渲染到页面

```js
import {StrictMode} from 'react'
import {createRoot} from 'react-dom/client'
import App from './App.tsx'
/*获取页面节点,并转换为虚拟DOM*/
createRoot(document.getElementById('root')!).render(
    /*将组件渲染到目标结点的页面上
    * 使用React严格语法，比如react过时的api*/
    <StrictMode>
        <App/>
    </StrictMode>,
)
```

### App.tsc

- 导出APP组件
- 一般不会在这里直接写组件，而是利用app.tsc来统一导入其他的组件

```js
function App() {
    return (
        <>
            <div>hello world</div>
        </>
    )
}

export default App
```

## 3. 开发规范

```bash
# 具体的模块，建在src/components/xxx中
- tsx：React的模块组件代码        rfc快速生成函数式组件            rf
- ts：ts功能代码 
- css：样式代码

# 导出的模块，可以在App.tsx中统一配置
```

### Food.tsx

```tsx
import {sayHello} from "./sayHello.ts";
import './food.css'

function Food() {
    return (
        <>
            <div className='header'>我是食物</div>
            <button onClick={sayHello}>点我</button>
        </>

    );
}

export default Food;
```

### food.css

```css
.header {
    height: 200px;
    width: 500px;
    background-color: gold;
}
```

### Food.ts

```ts
export function sayHello() {
    console.log("food");
}
```

### APP.tsx

```tsx
import Food from "./component/food/Food.tsx";

function App() {
    return (
        <>
            <Food/>
        </>
    )
}

export default App
```

## 4. TSX语法

- React的语法，TSX最终也会被翻译为js

### 基本规则

```tsx
import {sayHello} from "./sayHello.ts";
import './food.css'
import Cat from "../cat/Cat.tsx";

function Food() {
    const name: string = 'erick';

    /*1. 虚拟DOM, 必须有一个根标签，且根标签内的标签必须闭合，因此用<div>包裹起来
    * 2. 绑定样式时，不要用class, 要使用className,避免和ES6中的class关键字引发冲突
    * 3. 使用内联样式时，用style={{key:value}}方式
    * 4. 使用js表达式时，使用{}来获取
    * 5. 标签首字母
                 如果是小写开头，比如<div>，则将该标签转换为html中同名元素。若html中无该标签同名元素，则报错
                 如果是大写开头，比如<Nike/>，则就去渲染对应的组件，若组件没定义，则报错*/
    return (
        <>
            <div className='header'>我是食物</div>
            <div style={{background: "blue"}}>我是第二个</div>
            <button onClick={sayHello}>点我</button>
            {name}
            <Cat/>
        </>
    );
}

export default Food;
```

### 数组/对象

```tsx
function Cat() {
    /*数组，React会自动遍历*/
    const cat: string[] = ['狸花', '美短', '大橘'];
    /*对象：报错： Objects are not valid as a React child */
    const people = {name: 'erick', age: 12};
    return (
        <>
            <div>{cat}</div>
            {/*<div>{people}</div>*/}
        </>

    );
}

export default Cat;
```

### js表达式

- jsx中，标签中内容需要引入js表达式，则需要用{}来使用

```bash
# js表达式： 可以用一个变量来接收的js代码
- 可以产生一个值，则可以放在{}中
         1. a
         2. a+b
         3. demo()                - 调用函数
         4. arr.map()
         5. function test(){}     - 定义一个函数
         
# js代码： 不产生值的
         1. if
         2. for
         3. switch
```

```jsx
function Cat() {
    const cat: string[] = ['狸花', '美短', '大橘'];
    return (
        /*li中必须指定key，从而让DOM在比较时可以使用，可以用index*/
        <div>
            {cat.map((item, index) => (
                <li key={index}>
                    {item}
                </li>
            ))}
        </div>

    );
}

export default Cat;
```

## 5. Fragment

- tsx中，函数式组件和类式组件，返回值都必须用一个闭合标签来包裹，因此多了很多不需要的div

### 闭合标签

```tsx
const Cat = () => {
    return (
        <div>
            <div>哈哈</div>
            <div>嘿嘿</div>
        </div>
    )
}

export default Cat;
```

### 空标签

- 空标签不能指定任何属性，会不在加对应的标签

```tsx
const Cat = () => {
    return (
        <>
            <div>哈哈</div>
            <div>嘿嘿</div>
        </>
    )
}

export default Cat;
```

### Fragment

- 如果不想嵌套不必要的div，则可以使用React提供的Fragment
- React在解析时候，会自动把Fragment去掉
- 相比空标签，Fragment可以指定key，只能指定key，可以参与唯一标识的遍历

```tsx
import {Fragment} from "react/jsx-runtime";

const Cat = () => {
    return (
        <Fragment key={0}>
            <div>哈哈</div>
            <div>嘿嘿</div>
        </Fragment>
    )
}

export default Cat;
```





## 5. 虚拟DOM

- 本质是Object类型的对象(一般对象)
- 虚拟DOM轻，真实DOM比较重。虚拟DOM是React内部使用，不需真实DOM那么多的属性
- 虚拟DOM最终会被React转换为真实DOM，呈现在页面上

```js
import {StrictMode} from 'react'
import {createRoot} from 'react-dom/client'
import App from './App.tsx'

const root = createRoot(document.getElementById('root')!);

root.render(
    <StrictMode>
        <App/>
    </StrictMode>,
);

console.log("Virtual DOM", root);
console.log("Virtual DOM Type", typeof root); // object

const realDom = document.getElementById('root');
console.log('Real Dom', realDom);
```

## 6. React开发工具

- 在谷歌浏览器中加入，后面可以在Component中，查看每个组件的具体的属性，组件树之间的关系

![image-20240531135737923](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240531135737923.png)

![image-20240531140028704](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240531140028704.png)

# 函数组件

- 模块化：js的模块化，对js的拆分
- 组件：对js，html，css，image，video，font等的进行拆分
- 一个大的应用，所有的js，html等代码，都会划分为多个组件，多个组件拼接起来，就是一个完整的页面
- React18后，新特性可以使用组件的一些特性，官方比较推荐，代码复用率高

## 基本写法

### 普通函数

```jsx
/*  渲染组件到页面
   * 1. React解析组件标签，找到了Cat组件
   * 2. 发现是组件是使用函数定义的，调用该函数
   * 3. 将返回的该组件的虚拟DOM转换为真实DOM,随后呈现在页面*/
export default function Father() {
    /*1. 必须要有返回值，返回值就是该组件的虚拟DOM
    * 2. 首字母不要用小写，否则会当作html标签渲染
         首字母大写，就会当作组件来渲染*/
    return (
        /*li中必须指定key，从而让DOM在比较时可以使用，可以用index*/
        <div>
            Hi, 我是函数式组件
        </div>

    );
}
```

### 箭头函数

```jsx
export const Father = () => {
    return (
        <div>
            我是箭头函数组件
        </div>
    )
}
```

## useState

- 当前组件的一些状态，属性，
- 属性如果需要在页面显示，那么使用state。否则就考虑使用ref，避免页面的重复渲染问题

### 1. 单一属性

```tsx
import React from "react";
import {nanoid} from "nanoid";

export default function Food() {
    /*Food函数执行n+1次
    *   1. 初次渲染执行一次
    *   2. 后续每次任意一个state属性改变，执行一次*/
    console.log("Food executing");

    /*初始值
    * 返回一个数组，第一个是属性，第二个是修改属性的方法
    * 数组的解构赋值*/
    const [name, setName] = React.useState<string>("erick");
    const [address, setAddress] = React.useState<string>(nanoid(5));

    /*不能直接调用上面的setName和setAddress，必须用一个函数包裹起来*/

    /*写法一：传入一个函数*/
    const changeName = () => {
        setName((previous) => {
            return previous + "~";
        })
    };

    /*写法二：传入一个更新后的值*/
    const changeAge = () => {
        setAddress(nanoid(5));
    }

    return (
        <>
            <h2>姓名{name}</h2>
            <button onClick={changeName}>改变姓名</button>
            <h2>地址{address}</h2>
            <button onClick={changeAge}>改变地址</button>
        </>
    );
}
```

### 2. 对象属性

```tsx
import {BaseSyntheticEvent, useState} from "react";

export interface People {
    username: string;
    password: string;
    address: string;
}

export default function Food() {
    console.log("Food executing");
    const [people, setPeople] = useState<People>({} as People);

    const peopleChange = (type: string) => {
        return (event: BaseSyntheticEvent) => {
            setPeople({
                /*原对象解构*/
                ...people,
                /* 新属性覆盖*/
                [type]: event.target.value,
            })
        }
    }

    return (
        <>
            <div>{people.username}===={people.address}===={people.password}</div>
            姓名:<input onChange={peopleChange('username')}/><br/>
            密码:<input onChange={peopleChange('password')}/><br/>
            地址:<input onChange={peopleChange('address')}/><br/>
        </>
    );
}
```

### 3. 更新方式

- 方法调用是同步的，更新操作是异步更新

```bash
1. state更新操作按钮
2. state发生了改变
3. 函数组件重新调用，从而触发整个页面重新render
```

```tsx
import {useState} from "react";

export default function Food() {
    console.log('Food executing')
    const [count, setCount] = useState(0);

    const updateCount = () => {
        setCount((previous) => {
            return previous + 1;
        })
        /*异步更新：原来是0，异步更新，这里打印出来就还是0*/
        console.log('当前数据', count);
    }

    return (
        /*进行到return的时候，已经异步更新完毕了*/
        <div>
            <h2>当前计数{count}</h2>
            <button onClick={updateCount}>改变计数器</button>
        </div>
    )
}
```

## useEffect

- 在组件的生命周期内，进行一些操作，整个过程，没有发生任何的用户操作事件

```bash
# 场景
- 发送ajax请求
- 手动更改真实DOM
- 设置订阅，启动定时器

# 可以把useEffect Hook看成如下三个函数的组合
- componentDidMount:  组件加载完毕后
- componentDidUpdate: 组件更新后
- componentWillMount: 组件卸载前
```

### 1. 空监听state

- 依赖项数据为空数组：不监测任何属性，只会在组件整个渲染完毕后，执行一次

#### 不修改state

- 在副作用钩子中，不修改state属性：只会在组件第一次渲染完毕后，调用该钩子
- 组件因为其他方式更新后，不会再次触发该钩子函数

```tsx
import {useEffect, useState} from "react";

export default function App() {
    console.log('render');

    const [data, setData] = useState<string>('');

    /*1. 检测空数组
    * 2. 在hook中不修改state属性*/
    useEffect(() => {
        console.log("副作用hook");
    }, [])
    return (
        <>
            <div>{data}</div>
            <button onClick={() => {
                setData(previous => previous + '~')
            }}>修改数据
            </button>
        </>
    )
}
```

#### 修改state

```bash
1. 页面初次渲染，                                                      render --- 1
2. 渲染完毕后，调用useEffect钩子，                                       副作用hook
3. useEffect钩子改变了state，触发页面更新                                render --- 2
4. 后续任何state更新， useEffect不再调用
```

```tsx
import {useEffect, useState} from "react";

export default function App() {

    const [data, setData] = useState<number>(1);

    console.log('render', data);

    /*1. 检测空数组
    * 2. 在hook中修改state属性*/
    useEffect(() => {
        setData((previous) => {
            return previous + 1;
        });
        console.log("副作用hook");
    }, [])
    return (
        <>
            <div>{data}</div>
            <button onClick={() => {
                setData(previous => previous + 1)
            }}>修改数据
            </button>
        </>
    )
}
```

### 2. 指定监听state

- 页面初始化渲染完毕后调用一次钩子，state改变时继续调用钩子

```tsx
import {useEffect, useState} from "react";

export default function App() {

    const [data, setData] = useState<number>(1);

    console.log('render', data);

    /*1. 检测state属性-data
    * 2. 在hook中修改state属性*/
    useEffect(() => {
        console.log("副作用hook");
    }, [data]);
    return (
        <>
            <div>{data}</div>
            <button onClick={() => {
                setData(previous => previous + 1)
            }}>修改数据
            </button>
        </>
    )
}
```

### 3. 监听所有state

- 如果不指定监听的state，则默认会监听当前组件的所有state

```tsx
import {useEffect, useState} from "react";

export default function App() {

    const [data, setData] = useState<number>(1);

    console.log('render', data);

    /*1. 不指定监听的state*/
    useEffect(() => {
        console.log("副作用hook");
    });
    return (
        <>
            <div>{data}</div>
            <button onClick={() => {
                setData(previous => previous + 1)
            }}>修改数据
            </button>
        </>
    )
}
```

### 4. 死循环

- 不要在副作用中，监听某个state，同时改变某个state，否则就会触发页面的无限渲染

```tsx
import {useEffect, useState} from "react";

export default function App() {

    const [data, setData] = useState<number>(1);

    console.log('render', data);

    /*1. 不指定监听的state*/
    useEffect(() => {
        setData((prev) => prev + 1);
    }, [data]);
    return (
        <>
            <div>{data}</div>
        </>
    )
}
```



## Props

- 组件之间的状态传递，父组件向子组件传递属性，方法

### 1. 值/函数

#### 普通函数

```tsx
import {BaseSyntheticEvent, useState} from "react";
import Son from "./Son.tsx";

export default function Father() {
    const [age, setAge] = useState<number>(0);
    const [username, setUsername] = useState<string>("lucy");

    const firstMethod = () => {
        console.log("first method")
    }

    /*普通写法*/
    const secondMethod = (city: string, year: number): string => {
        console.log("second method", city, year);
        return `${city}, ${year}`;
    }

    /*函数柯里化写法*/
    const thirdMethod = (city: string, year: number) => {
        return (event: BaseSyntheticEvent) => {
            console.log(event)
            console.log("third method", city, year);
        }
    }

    return (
        <div>
            <Son age={age} username={username} say={firstMethod} work={secondMethod} hardWork={thirdMethod}/>
        </div>
    );
}
```

```tsx
import {BaseSyntheticEvent} from "react";

interface SonProps {
    age: number;
    username?: string; /*可选属性*/
    /*无参无返回值: 普通函数即可*/
    say: () => void;
    /*有参有返回值*/
    work: (city: string, year: number) => string;
    /*柯里化*/
    hardWork: (city: string, year: number) => (event: BaseSyntheticEvent) => void;
}

export default function Son(props: SonProps) {
    /*结构赋值*/
    const {age, username, say, work, hardWork} = props;

    return (
        <>
            <h2>年龄{age}</h2>
            <h2>姓名{username}</h2>
            {/*不带括号： 将一个函数的指向作为回调*/}
            <button onClick={say}>无参数</button>

            {/*非柯里化写法*/}
            <button onClick={() => {
                return work('北京', 2025);
            }}>有参数普通写法
            </button>

            {/*柯里化写法*/}
            <button onClick={
                hardWork("南京", 2024)
            }>柯里化写法
            </button>
        </>
    );
}
```

#### 箭头函数

```tsx
import {BaseSyntheticEvent, FC} from "react";

interface SonProps {
    age: number;
    username?: string;
    say: () => void;
    work: (city: string, year: number) => string;
    hardWork: (city: string, year: number) => (event: BaseSyntheticEvent) => void;
}

export const Son: FC<SonProps> = (props) => {
    const {age, username, say, work, hardWork} = props;

    return (
        <>
            <h2>年龄{age}</h2>
            <h2>姓名{username}</h2>
            <button onClick={say}>无参数</button>

            <button onClick={() => {
                return work('北京', 2025);
            }}>有参数普通写法
            </button>

            <button onClick={
                hardWork("南京", 2024)
            }>柯里化写法
            </button>
        </>
    );
}
```

### 2. children

#### Html标签

- 组件嵌套的内容，如果是html标签，标签体内部的东西则可以直接渲染

```tsx
import Son from "./Son.tsx";

export default function Father() {
    return (
        <div>
            <div>我是父组件</div>
            {/*组件标签内的东西：是普通的html*/}
            <Son><h1>Hi Son</h1></Son>
        </div>
    )
}
```

```tsx
import * as React from "react";

interface SonProps {
    /*children的类型*/
    children: React.ReactNode;
}

export default function Son(props: SonProps) {
    /*从props中接收*/
    return (
        <>叫我？{props.children}</>
    )
}
```

#### 组件标签

```tsx
import Father from "./component/cat/Father.tsx";
import Son from "./component/cat/Son.tsx";

function App() {
    return (
        <>
            <Father>
                <Son/>
            </Father>
        </>
    )
}

export default App
```

```tsx
import * as React from "react";

interface FatherProps {
    children?: React.ReactNode
}

export default function Father(props: FatherProps) {
    return (
        <div>
            <div>我是父组件</div>
             {/*子组件的渲染位置*/}
            <div>{props.children}</div>
        </div>
    )
}
```

```tsx
export default function Son() {
    return (
        <>我是儿子</>
    )
}
```

### 3. 父子组件

- 可以通过逐层嵌套的方式，父子组件的关系不太好判断

#### 标签嵌套

- 一种父子组件的嵌套方式

```jsx
import Father from "./Father";
import {Son} from "./Son";

export default function Customer() {
    return (
        <div>
            <h1>我是customer</h1>
            {/*通过标签体的方式
             1. 可以更加清晰的查看父子组件的嵌套关系
             2. 在Father中必须显示调用 props.children才能渲染son
             3. Father的标签体：<Son>*/}
            <Father>
                <Son>Hi, Son</Son>
            </Father>
        </div>
    );
}
```

```jsx
import React from "react";

interface FatherProps {
    children?: React.ReactNode;
}

export default function Father(props: FatherProps) {
    return (
        <div>
            <h1>父亲组件</h1>

            {/*1. 显示调用
               2. 指定子组件的渲染的位置,并渲染子组件
               3. 并不会解析Son的标签体内容：'Hi, Son'，交给Son去解析*/}
            {props.children}
        </div>
    );
}
```

```jsx
import React from "react";

interface SonProps {
    children?: React.ReactNode;
}

export function Son(props: SonProps) {
    return (
        <div>
            <h1>儿子组件</h1>
            {props.children}
        </div>
    );
}
```

#### renderProps

- 通过标签嵌套方式，形成的父子组件，并且可以传递状态

```jsx
import React from "react";
import {Father} from "./Father";
import {Son} from "./Son";

export default function Mall() {

    /*顶层组件的属性*/
    const [mall, setMall] = React.useState<string>('mall');

    /*顶层组件的方法*/
    const logTop = () => {
        console.log('我是top的函数')
    }

    return (
        <div>
            <h1>我是customer</h1>

            {/*显示组件嵌套关系*/}
            {/*1. mall：顶层组件的属性，  logTop: 顶层组件的方法
               2. 可以跨越父组件，直接传递给子组件*/}
            <Father
                /* 1.erickRender: 给Father提供一个回调方法
                   2. 将Father的属性或者方法传递给Son*/
                erickRender={(name, address, age) => {
                    return <Son name={name} address={address} age={age} mall={mall} logTop={logTop}/>
                }}/>
        </div>
    );
}
```

```tsx
import React from "react";

interface FatherProps {
    erickRender: (name: string, address: string, age: string) => React.ReactNode;
}

export function Father(props: FatherProps) {
    const [data, setData] = React.useState({
        name: 'shuzhan',
        address: 'xian',
        age: '20'
    });

    const {name, address, age} = data

    return (
        <div>
            <h1>父亲组件开始</h1>

            {/*子组件
            1. 预留位置
            2. 显示调用props.children进行解析子组件*/}
            {props.erickRender(name, address, age)}

            <h1>父亲组件结束</h1>
        </div>
    );
}
```

```tsx
type SonProps = {
    name: string;
    address: string;
    age: string;
    mall: string;
    logTop: () => void;
}

export function Son(props: SonProps) {
    return (
        <div>
            <h1>儿子组件开始</h1>
            {props.name} {props.age} {props.address} {props.mall}
            <button onClick={props.logTop}>点击</button>
            <h1>儿子组件结束</h1>
        </div>
    );
}
```

## useRef

- 维护组件的属性，状态，如果这种属性不直接被页面渲染所需要，则可以考虑使用ref，比如页面的查询条件

### 1. HTML标签

```tsx
import {useRef} from "react";

export default function App() {
    // 专人专用，保存一个数据
    const userNameRef = useRef<HTMLInputElement>({} as HTMLInputElement);
    const ageRef = useRef<HTMLInputElement>({} as HTMLInputElement);

    const check = () => {
        console.log(userNameRef.current.value);
        console.log(ageRef.current.value);
    }

    return (
        <>
            姓名： <input ref={userNameRef} type={'text'} placeholder={'请输入姓名'}/>
            年龄: <input ref={ageRef} type={"number"} placeholder={'请输入年龄'}/>
            <button onClick={check}>点击查看</button>
        </>
    )
}
```

- 尽量少用ref，发生事件的事件源和输入框如果是一个，使用event

```tsx
import {BaseSyntheticEvent} from "react";

export default function App() {

    const check = (event: BaseSyntheticEvent) => {
        console.log(event.target.value);
    }

    return (
        <>
            姓名： <input type={'text'} placeholder={'请输入姓名'} onBlur={check}/>
        </>
    )
}
```

### 2. 保存数据

- 存放不用于触发页面重新渲染的可变数据
- 数据会进行缓存。如果页面因为state触发了重新渲染，Ref之前保存的数据也不会丢失

```tsx
import {useRef} from "react";

export default function App() {

    console.log("render...")

    const counter = useRef<number>(0);

    /*ref数据改变后，页面不会重选渲染*/
    const changeCounter = () => {
        counter.current += 1;
        console.log(counter.current);
    }

    return (
        <>
            <button onClick={changeCounter}>加一</button>
        </>
    )
}
```

### 3. 使用场景

- 组件需要存储一些值，但当前值不会进行页面渲染，请选择 ref

```tsx
import {useRef} from "react";
import Son from "./Son.tsx";

export interface People {
    name: string;
}

export default function Father() {
    console.log("父页面渲染")

    const queryCondition = useRef<People>({} as People);

    const updateQueryCondition = (people: People) => {
        queryCondition.current = people;
    }

    return (
        <>
            <Son updateQueryCondition={updateQueryCondition}/>
        </>
    )
}
```

```tsx
import {People} from "./Father.tsx";
import {useState} from "react";

interface SonProps {
    updateQueryCondition: (people: People) => void;
}

export default function Son(props: SonProps) {
    console.log("子页面渲染");

    /*state属性修改后，子页面就会重新渲染，但是并不想让父页面也被渲染*/
    const [name, setName] = useState<string>("");

    const handleClick = () => {
        props.updateQueryCondition({
            name: name,
        })
    }

    return (
        <>
            姓名: <input type="text" onBlur={(e) => setName(e.target.value)}/>
            {/*点击后，父页面也不会重选渲染*/}
            <button onClick={handleClick}>提交</button>
        </>
    )
}
```

## Render顺序

### 父子组件

- 渲染父组件，接着渲染子组件，不管子组件有没有使用父组件的状态(DFS)
- 父组件一旦重新渲染，就会触发所有的子组件渲染，不管子组件是否用到父组件的数据

```tsx
import {useState} from "react";
import FirstSon from "./FirstSon.tsx";
import SecondSon from "./SecondSon.tsx";

export default function Father() {
    const [name, setName] = useState('lucy');

    const changeName = () => {
        setName((prevState) => {
            return prevState + '~';
        });
    }
    return (
        <>
            <FirstSon/>
            <SecondSon name={name}/>
            {/*父组件一旦重新渲染，就会触发所有的子组件渲染，不管子组件是否用到父组件的数据*/}
            <button onClick={changeName}>父组件state改变</button>
        </>
    )
}
```

```tsx
export default function FirstSon() {
    console.log('FirstSon Render');
    return (
        <div>first son</div>
    )
}
```

```tsx
interface SecondSonProps {
    name: string;
}

export default function SecondSon(props: SecondSonProps) {
    console.log('SecondSon Render');
    return (
        <div>{props.name}</div>
    );
}
```

### 单一state改变

- 父组件的state改变后，就会触发对应的重新渲染

```tsx
import {useState} from "react";
import {FirstSon} from "./FirstSon";
import {SecondSon} from "./SecondSon";

export function Father() {
    const [address, setAddress] = useState('XIAN');

    /*方式一：给了新属性，但值一样，不会重新render*/
    function changeName() {
        setAddress('XIAN');
    }

    /*方式二：给了新属性，但值变了，会重新render父组件及所有子组件*/
    function changeName01() {
        setAddress('BEIJING');
    }

    /*方式三：直接返回了旧的值，不重新render*/
    function changeName02() {
        setAddress(previous => {
            return previous;
        })
    }

    /*方式四：旧值更新*/
    function changeName03() {
        setAddress(previous => {
            return previous + 'adf';
        })
    }

    console.log('Father-Render')
    return (
        <div>
            <h2>父亲：{address}</h2>

            <button onClick={changeName03}>更换姓名</button>
            <br/>
            <FirstSon/>
            <SecondSon address={address}/>
        </div>
    )
}
```

### 对象state改变

```tsx
import {useState} from "react";
import {FirstSon} from "./FirstSon";
import {SecondSon} from "./SecondSon";

export function Father() {
    const [data, setData] = useState({
        name: 'erick',
        age: 20
    })

    /*方式一：重新赋值，不管值是否相同，都会重新render*/
    function changeInfo01() {
        setData({
            name: 'erick',
            age: 20
        })
    }

    /*方式二：返回原对象，不会render*/
    function changeInfo02() {
        setData((previous) => {
            return previous;
        })
    }

    /*方式三：返回原对象，对象值变了，不会render*/
    function changeInfo03() {
        setData((previous) => {
            previous.name = 'lucy';
            return previous;
        })
    }

    /*方式四：返回新对象，哪怕新对象值和之前不变，也会render*/
    function changeInfo04() {
        setData((previous) => {
            let newVal = {
                name: previous.name,
                age: previous.age
            }
            return newVal;
        })
    }

    console.log('Father-Render')
    return (
        <div>
            <h2>父亲：{data.name}{data.age}</h2>

            <button onClick={changeInfo04}>更换信息</button>
            <br/>
            <FirstSon/>
            <SecondSon address={data.address} age={data.age}/>
        </div>
    )
}
```

# 受控组件

## 高阶函数

### 1. 定义

```bash
# 高阶函数：    如果一个函数满足下面两条任意一个
- A函数，接收的参数是一个函数，       那么A就是高阶函数
- A函数，调用的返回值依然是一个函数，  那么A就是高阶函数
       #    Promise  setTimeout, arr.map

# 函数柯里化
- 通过函数调用继续返回函数的方式，实现多次接受参数，最后统一处理的函数编码方式
```

### 2. 非柯里化

```ts
function sum(a, b, c) {
    return a + b + c;
}

let sum1 = sum(1, 3, 5);
console.log(sum1);
```

### 3. 柯里化

```ts
/*因为调用参数时，不一定一下子都能拿到所有参数*/
function sum(a) {
    console.log(`a=${a}`);

    return (b) => {
        console.log(`b=${b}`)

        return (c) => {
            console.log(`c=${c}`)
            return a + b + c;
        }
    }
}

/*a=1
 b=3
 c=5
*/
sum(1)(3)(5);
```

## 函数回调

- 如果在回调方法中，需要传递除了event之外的参数，可以考虑柯里化和非柯里化两种方式

### 1. 调用方式

```tsx
export default function App() {

    const work = (): string => {
        console.log("App work");
        return "Hello World!";
    }

    /*内容： 该函数的返回值*/

    /*1. 不带(), 函数不会执行，该变量的返回值就是一个函数*/
    console.log(work);
    /*2. 带()，该函数执行后，返回值：Hello World!*/
    console.log(work());

    return (
        <>

        </>
    )
}
```

```tsx
export default function App() {

    const work = (): string => {
        console.log("App work");
        return "Hello World!";
    }

    return (
        <>
            {/*1. 不带()，该变量的返回值是一个函数，一开始不会执行，事件触发后，会调用该函数*/}
            邮箱：<input onBlur={work} type="text" placeholder="邮箱"/>

            {/*2. 带(), 该变量的返回值是一个string, 一上来就会执行该方法，并将返回值作为回调函数*/}
            姓名：<input onBlur={work()} type="text" placeholder="姓名"/>
        </>
    )
}
```

### 2. 非柯里化

```tsx
import {BaseSyntheticEvent} from "react";

export default function App() {

    const work = (keyName: string, event: BaseSyntheticEvent) => {
        console.log(keyName);
        console.log(event.target.value);
    }


    return (
        <>
            {/*用函数包装，再去调用*/}
            邮箱：<input onBlur={(event: BaseSyntheticEvent) => {
            work('email', event);
        }} type="text" placeholder="邮箱"/>
        </>
    )
}
```

### 3. 柯里化

```tsx
import {BaseSyntheticEvent} from "react";

export default function App() {

    const work = (keyName: string) => {
        console.log(keyName);
        return (event: BaseSyntheticEvent) => {
            console.log(event.target.value);
        }
    }

    return (
        <>
            {/*用函数包装，再去调用
             1. 一上来就调用一次
             2. 后续每次点击，就会调用其内层函数
             3. dataType可以自己传递，但是event是React帮忙传递的*/}
            邮箱：<input onBlur={work('email')} type="text" placeholder="邮箱"/>
        </>
    )
}
```

## 非受控组件

- 收集表单中的数据：页面内所有输入类的DOM，现用现取

```tsx
import {BaseSyntheticEvent, useRef} from "react";

export default function App() {

    const nameRef = useRef<HTMLInputElement>({} as HTMLInputElement);
    const addressRef = useRef<HTMLInputElement>({} as HTMLInputElement);
    const emailRef = useRef<HTMLInputElement>({} as HTMLInputElement);

    /*用的时候再去取*/
    const savePeople = (event: BaseSyntheticEvent) => {
        console.log(event);
        event.preventDefault();
        alert(nameRef.current.value + addressRef.current.value + emailRef.current.value);
    }

    return (
        <div>
            <form action={""} onSubmit={savePeople}>
                用户名：<input type={"text"} ref={nameRef}/>
                地址：<input type={"text"} ref={addressRef}/>
                邮箱：<input type={"text"} ref={emailRef}/>
                <button>提交</button>
            </form>
        </div>
    )
}
```

##  受控组件

- 每次改变输入框的值，就将其值放入state中

```tsx
import {BaseSyntheticEvent, useState} from "react";

export interface People {
    name: string;
    address: string;
    email: string;
}

export default function App() {
    console.log('render');

    const [people, setPeople] = useState<People>({} as People);

    const savePeople = (keyName: string) => {
        return (event: BaseSyntheticEvent) => {
            setPeople({
                ...people,
                [keyName]: event.target.value,
            })
        }
    }

    const log = (event: BaseSyntheticEvent) => {
        event.preventDefault();
        console.log(people);
    }

    return (
        <div>
            <form action={""} onSubmit={log}>
                用户名：<input type={"text"} onBlur={savePeople('name')}/>
                地址：<input type={"text"} onBlur={savePeople('address')}/>
                邮箱：<input type={"text"} onBlur={savePeople('email')}/>
                <button>提交</button>
            </form>
        </div>
    )
}
```



# 类组件

## 1. State

- 当前组件的一些状态，属性，等
- 组件实例对象上的，因为类组件继承了React.Component，React.Component中预先定义了一些属性
- React.Component中的state默认给null
- state是readOnly，组件初始化state属性后，后续修改只能调用setState()方法

### 1.1 普通函数

- 必须要使用bind，否则存在this丢失问题

```jsx
import {Component} from "react";

export default class Citi extends Component {
    constructor(props) {
        super(props);
        this.change = this.changeWhether.bind(this);
    }

    /*1. 初始化状态，初始完毕后，state不能再重新赋值*/
    state = {
        isHot: true,
        address: '西安'
    }

    changeWhether() {
        /*1. 获取原来的状态*/
        let isHot = this.state.isHot;
        /*2. 状态只能通过setState修改：merge更新*/
        this.setState({isHot: !isHot})
    }

    render() {
        /*解构数据*/
        const {isHot, address} = this.state;
        return (
            <div>
                <h2>今天{address}天气很{isHot ? '炎热' : '凉爽'}</h2>
                <button onClick={this.change}>改变天气</button>
            </div>
        )
    }
}
```

### 1.2 箭头函数

- 不用关注this问题
- 推荐的一种方式

```jsx
import {Component} from "react";

export default class Citi extends Component {
    state = {
        isHot: true,
        address: '西安'
    }

    /*功能函数：箭头函数+赋值语句*/
    changeWhether = () => {
        let isHot = this.state.isHot;
        this.setState({isHot: !isHot})
    }

    render() {
        /*解构数据*/
        const {isHot, address} = this.state;
        return (
            <div>
                <h2>今天{address}天气很{isHot ? '炎热' : '凉爽'}</h2>
                <button onClick={this.changeWhether}>改变天气</button>
            </div>
        )
    }
}
```

### 1.3 setState

```bash
# setState()
- 方法调用是同步的，但是执行的具体动作是异步的
```

#### 对象写法

- setState(stateChange, [callback]) : 对象写法

```jsx
import {Component} from "react";

export class SetState extends Component {
    state = {
        count: 0
    }

    addCount = () => {
        this.setState({
            count: this.state.count + 1
        }, () => {
            // callback是可选的回调函数，它在状态更新完毕，界面也更新后(render调用后)，才被调用
            console.log('callback', this.state.count);
        });

        console.log('code', this.state.count);  // 是0
    }

    render() {
        return (
            <div>
                <button onClick={this.addCount}>数字：{this.state.count}</button>
            </div>
        )
    }
}
```

#### 函数写法

```bash
# setState(updater, [callback]): 函数写法
- updater 为返回stateChange对象的函数
- updater可以接收到state和props属性
```

```jsx
import {Component} from "react";

export class SetState extends Component {
    state = {
        count: 0
    }

    addCount = () => {
        this.setState((state, props) => {
            return {count: this.state.count + 1}
        }, () => {
            console.log('callback', this.state.count)
        })

        console.log('code', this.state.count)
    }
    
    render() {
        return (
            <div>
                <button onClick={this.addCount}>数字：{this.state.count}</button>
            </div>
        )
    }
}
```

## 2. Props

- 组件之间的状态传递，父子组件传递属性，方法等
- 组件实例对象上的，因为类组件继承了React.Component
- props是Readonly

```bash
# props作用：
- 在初始化一个组件的时候，可以给该实例对象传递一些属性，这些属性会封装到props属性中，用于父子组件中信息传递
- 还可以传递将父组件的函数，传递到子组件

# props限制
- 对应的props，是子组件身上的，子组件来使用的，子组件可以声明需要的props的具体的类型
- 父组件给子组件传递数据时，就知道子组件需要的类型
- 相当于给A方法定义形参类型，这样其他方法调用A方法时，知道需要传递的数据类型
```

### 2.1 传值

```jsx
import React from "react";
import Citi from "./component/Citi/Citi";


export default class App extends React.Component {

    /* 父子组件：别和原型链扯关系
     父组件的函数，也可以传递到子组件中*/

    state = {
        count: 1
    }

    update = () => {
        this.setState({
            count: this.state.count + 1
        })
    }

    render() {
        const person = {name: "shuzhan", age: 20, address: "beijing"};
        return (
            <div>
                <h2>count={this.state.count}</h2>

                {/*1. 传递方式一: 直接写*/}
                <Citi name="erick" address="xian" age={13} updateInfo={this.update}/>

                {/*2 传递方式二：解析对象*/}
                <Citi name={person.name} address={person.address} age={person.age} updateInfo={this.update}/>

                {/*3. 传递方式三：属于方式二的语法糖，在jsx中批量传递对象的，这种解构赋值，只能用来传递props*/}
                <Citi {...person} updateInfo={this.update}/>
            </div>)
    }
}
```

```jsx
import {Component} from "react";
import PropTypes from "prop-types";

export default class Citi extends Component {

    /*定义数据规范*/
    static propTypes = {
        /*数据类型，是否必传*/
        name: PropTypes.string.isRequired,
        address: PropTypes.string,
        age: PropTypes.number,
        updateInfo: PropTypes.func  /*规定必须是一个函数*/
    }

    /*默认值*/
    static defaultProps = {
        address: 'NAN'
    }

    render() {
        /*组件使用*/
        const {name, address, age, updateInfo} = this.props;
        return (
            <div>
                <ul>
                    <li>{name}</li>
                    <li>{address}</li>
                    <li>{age + 1}</li>
                    <button onClick={updateInfo}>点击</button>
                </ul>
            </div>)
    }
}
```

### 2.2 props.chiledren

- 组件嵌套的内容，如果是html标签，标签体内部的东西则可以直接渲染
- 如果是组件标签，标签体内部的东西需要子组件从props.children中接收才能渲染

```jsx
import {Component} from "react";
import {Son} from "./Son";

export class Father extends Component {
    render() {
        return (
            <>
                {/*1. 如果是html标签，则标签体内部的东西可以自动渲染
                   2. 如果是组件标签，则标签体内部的东西，需要从children中接收*/}
                <h5>我是父组件</h5>
                <Son>父亲</Son>
            </>
        )
    }
}
```

```jsx
import {Component} from "react";

export class Son extends Component {
    render() {
        /*会封装到props属性中的children： 标签体内容从this.props中去取*/
        let children = this.props.children;
        console.log(children);
        return (
            <>我是子组件</>
        )
    }
}
```

## 3.Ref

- 用来获取jsx中带值标签的值
- 也是React.Component的属性
- ref尽量少用

### 3.1 字符串ref

- React已经慢慢不支持了，效率低，不推荐使用

```jsx
import {Component} from "react";

export default class Citi extends Component {

    getName = () => {
        // 1. 从refs中拿到对应的真实DOM
        const {nameInput} = this.refs;
        // 2. 解析DOM的value
        alert(nameInput.value);
    }

    getAddress = () => {
        const {addressInput} = this.refs;
        alert(addressInput.value);
    }

    render() {
        return (
            <div>
                {/*1. 获取当前的input整个获取到真实DOM的结点
                   2. 将该结点放在组件的refs属性上 */}
                <input ref="nameInput" type="text" placeholder="请输入姓名"/>
                <button onClick={this.getName}>查看姓名</button>

                <input ref="addressInput" type="text" placeholder="请输入地址" onBlur={this.getAddress}/>
            </div>)
    }
}
```

### 3.2 回调函数ref

- 将当前结点，放在当前实例组件的属性上

#### 内联形式

```jsx
import {Component} from "react";

export default class Citi extends Component {

    getName = () => {
        /*获取真实DOM,然后获取到该值*/
        alert(this.nameInput.value);
    }

    getAddress = () => {
        alert(this.addressInput.value);
    }

    render() {
        return (<div>

            {/*将当前节点的真实DOM，传递给Citi组件的一个新的属性nameInput*/}
            <input ref={(currentNode) => {
                this.nameInput = currentNode;
            }} type="text" placeholder="请输入姓名"/>
            <button onClick={this.getName}>查看姓名</button>


            <input ref={(currentNode) => {
                this.addressInput = currentNode;
            }} type="text" placeholder="请输入地址" onBlur={this.getAddress}/>
        </div>)
    }
}
```

#### 类绑定形式

```jsx
import {Component} from "react";

export default class Citi extends Component {

    getName = () => {
        /*获取真实DOM,然后获取到该值*/
        alert(this.nameInput.value);
    }

    getAddress = () => {
        alert(this.addressInput.value);
    }

    saveName = (currentNode) => {
        this.nameInput = currentNode;
    }

    saveAddress = (currentNode) => {
        this.addressInput = currentNode;
    }

    render() {
        return (<div>
            <input ref={this.saveName} type="text" placeholder="请输入姓名"/>
            <button onClick={this.getName}>查看姓名</button>
            
            <input ref={this.saveAddress} type="text" placeholder="请输入地址" onBlur={this.getAddress}/>
        </div>)
    }
}
```

### 3.3 createRef

- React最推荐的一种

```jsx
import {Component, createRef} from "react";

export default class Citi extends Component {

    /*ref容器： 只能保存一个，后加入的会对之前的进行覆盖*/
    nameRef = createRef();

    addressRef = createRef();

    getName = () => {
        alert(this.nameRef.current.value);
    }

    getAddress = () => {
        alert(this.addressRef.current.value);
    }
    
    render() {
        return (<div>
            {/*将当前结点存在ref容器中*/}
            <input ref={this.nameRef} type="text" placeholder="请输入姓名"/>
            <button onClick={this.getName}>查看姓名</button>

            <input ref={this.addressRef} type="text" placeholder="请输入地址" onBlur={this.getAddress}/>
        </div>)
    }
}
```

### 3.4 事件

- 不要过度的使用ref，发生事件的事件源和输入框如果是一个，使用event

```jsx
import {Component} from "react";

export default class Citi extends Component {

    getAddress = (event) => {
        /*event.target: 真实结点*/
        alert(event.target.value);
    }

    render() {
        return (<div>
            <input type="text" placeholder="请输入地址" onBlur={this.getAddress}/>
        </div>)
    }
}
```

## 4. 父子组件

### 4.1 多层嵌套

- 形成了TopParent-->Father-->Son的父子关系

```jsx
import {Component} from "react";
import {Father} from "./Father";

export class TopParent extends Component {
    render() {
        return (
            <>
                <Father/>
            </>
        )
    }
}
```

```jsx
import {Component} from "react";
import {Son} from "./Son";

/*父组件*/
export class Father extends Component {
    render() {
        return (
            <>
                <h5>我是父组件</h5>
                {/*儿子组件*/}
                <Son>父亲</Son>
            </>
        )
    }
}
```

```jsx
import {Component} from "react";

export class Son extends Component {
    render() {
        return (
            <>我是子组件</>
        )
    }
}
```

### 4.2 标签嵌套

- 形成了TopParent-->Father-->Son的父子关系

```jsx
import {Component} from "react";
import {Father} from "./Father";
import {Son} from "./Son";

export class TopParent extends Component {
    render() {
        return (
            <>
                {/*父子关系*/}
                <Father>
                    <Son/>
                </Father>
            </>
        )
    }
}
```

```jsx
import {Component} from "react";

/*父组件*/
export class Father extends Component {
    render() {
        return (
            <>
                <h5>我是父组件</h5>
                {/*加上后才可以渲染Son组件*/}
                {this.props.children}
            </>
        )
    }
}
```

```jsx
import {Component} from "react";

export class Son extends Component {
    render() {
        return (
            <>我是子组件</>
        )
    }
}
```

### 4.3 renderProps

- 通过标签嵌套方式，形成的父子组件，如果有状态需要传递，则可以使用下面方式

```jsx
import {Component} from "react";
import {Father} from "./Father";
import {Son} from "./Son";

export class TopParent extends Component {
    render() {
        return (
            <>
                {/*父子关系
                 1. 父包裹子时，提供一个回调函数：props属性
                    1.1 render可以随意起名字*/}
                <Father render={(name, address, age) => {
                    return <Son name={name} address={address} age={age}/>
                }}/>
            </>
        )
    }
}
```

```jsx
import {Component} from "react";

/*父组件*/
export class Father extends Component {

    state = {
        name: '爸爸',
        age: 30,
        address: '西安'
    }

    render() {
        const {name, address, age} = this.state;
        return (
            <>
                <h5>我是父组件</h5>

                {/* 2. 预留空间*/}
                {this.props.render(name, address, age)}
            </>
        )
    }
}
```

```jsx
import {Component} from "react";

export class Son extends Component {
    render() {
        return (
            <>
                <h5>叫我？{this.props.name}{this.props.age}{this.props.address}</h5>
            </>
        )
    }
}
```

## 5.PureComponent

- 父组件只要更新了state或者props属性，所有的子组件都会重新渲染，可能存在效率低的问题

### 5.1 state更新

- 第一次加载：渲染父组件，接着渲染子组件
- 后续每次更新父组件中的state属性，都会重新渲染父--->子组件
- 不管子组件有没有使用父组件的状态

```jsx
import {Component} from "react";
import {FirstSon} from "./FirstSon";
import {SecondSon} from "./SecondSon";

/*父组件*/
export class Father extends Component {

    state = {
        name: 'Erick'
    }

    changeName = () => {
        this.setState({name: 'Lucy'})
    }

    render() {
        console.log('Father render')
        return (
            <>
                <h5>我是父组件{this.state.name}</h5><br/>
                <button onClick={this.changeName}>更换姓名</button>
                <br/>
                {/*属性有依赖*/}
                <FirstSon name={this.state.name}/><br/>
                {/*属性无依赖*/}
                <SecondSon/>
            </>
        )
    }
}
```

```jsx
import {Component} from "react";

export class FirstSon extends Component {
    render() {
        console.log('FirstSon render')
        return (
            <>我是第一个子组件{this.props.name}</>
        )
    }
}
```

```jsx
import {Component} from "react";

export class SecondSon extends Component {
    render() {
        console.log('SecondSon render')
        return (
            <>我是第二个子组件{this.props.name}</>
        )
    }
}
```

### 5.2 state假装更新

- 只要父组件调用了setState方法，不管到底有没有更新，都会从父到子全部render一次

```bash
# 父组件中    shouldComponentUpdate
- 不做特殊处理，总是返回true
```

```jsx
import {Component} from "react";
import {FirstSon} from "./FirstSon";
import {SecondSon} from "./SecondSon";

/*父组件*/
export class Father extends Component {

    state = {
        name: 'Erick'
    }

    changeName = () => {
        this.setState({})
    }

    render() {
        console.log('Father render')
        return (
            <>
                <h5>我是父组件{this.state.name}</h5><br/>
                <button onClick={this.changeName}>更换姓名</button>
                <br/>
                {/*属性有依赖*/}
                <FirstSon name={this.state.name}/><br/>
                {/*属性无依赖*/}
                <SecondSon/>
            </>
        )
    }
}
```

### 5.3 条件render-自定义

- 需要在父子组件中，重写shouldComponentUpdate，如果组件的state属性和props的确更改了，再让它去render

```jsx
shouldComponentUpdate(nextProps, nextState, nextContext) {

    if (this.state.name === nextState.name) {
        return false;
    }

    if (this.props.name === nextProps.name) {
        return false;
    }

    return true;
}
```

### 5.4 PureComponent

- React中提供了一个类，可以实现上面逻辑
- 让父子组件，都继承PureComponent, PureComponent重写了上面shouldComponentUpdate方法

#### 基本使用

```jsx
import {PureComponent} from "react";

export class FirstSon extends PureComponent {

    render() {
        console.log('FirstSon render')
        return (
            <>我是第一个子组件{this.props.name}</>
        )
    }
}
```

#### 浅对比

```bash
# 比较state属性时候，浅对象
- 比较的是引用地址，而非引用地址的具体的值
- 在setState时候，不要和原来的对象发生关系
```

```jsx
import {PureComponent} from "react";
import {FirstSon} from "./FirstSon";
import {SecondSon} from "./SecondSon";

/*父组件*/
export class Father extends PureComponent {

    state = {
        name: 'Erick'
    }

    /*没有改变指针，通过指针改变了堆中的对象
    * 不会触发render*/
    changeName = () => {
        const obj = this.state;
        obj.name = 'lucy';
        this.setState(obj)
    }

    render() {
        console.log('Father render')
        return (
            <>
                <h5>我是父组件{this.state.name}</h5><br/>
                <button onClick={this.changeName}>更换姓名</button>
                <br/>
                {/*属性有依赖*/}
                <FirstSon name={this.state.name}/><br/>
                {/*属性无依赖*/}
                <SecondSon/>
            </>
        )
    }
}
```

# Context

- 多层父子组件之间通信的一种方式
- 开发中一般不用context，一般都用它来封装react插件
- Props传递：传递数据可以使用props，但是层级多了后，必须一层层传递props，比较麻烦
- 而且props传递时，层级多时，如果从顶层组件到底层组件，中间组件并不想获取到这个数据

## 2.1 公共缓存

```js
import React from "react";
/*定义一个Context，所有组件都可以访问到*/
export const ErickContext = React.createContext();
```

## 2.2 顶组件

- 包裹了Second组件，则Second组件及Second组件的子组件系列，都可以访问到顶组件的数据

```jsx
import {Component} from "react";
import {Second} from "./Second";
import {ErickContext} from "./ErickContext";

export class First extends Component {

    state = {
        name: 'first',
        address: 'xian',
    }

    render() {
        return (
            <>
                <div>我是First组件{this.state.name}{this.state.address}</div>
                {/*必须是value*/}
                <ErickContext.Provider value={
                    {
                        name: this.state.name,
                        address: this.state.address
                    }
                }>
                    <Second/>
                </ErickContext.Provider>

            </>
        )
    }
}
```

## 2.3 中间组件

- 需要就声明，否则不用

```jsx
import {Component} from "react";
import {Third} from "./Third";

export class Second extends Component {
    render() {
        return (
            <>
                <div>我是Second组件</div>
                <Third/>
            </>
        )
    }
}
```

## 2.4 底层组件

### 类式组件

```jsx
import {Component} from "react";
import {ErickContext} from "./ErickContext";

export class Third extends Component {

    /*static属性，举手：要从顶层中去取数据*/
    static contextType = ErickContext;

    render() {
        return (
            <>
                <div>
                    <div>我是Third组件{this.context.name}{this.context.address}</div>
                </div>
            </>
        )
    }
}
```

### 类式/函数都可

```jsx
import {Component} from "react";
import {ErickContext} from "./ErickContext";

export class Third extends Component {

    render() {
        return (
            <>
                <ErickContext.Consumer>
                    {
                        (value) => {
                            const {name, address} = value;
                            return <div>我是Third组件{name}{address}</div>
                        }
                    }
                </ErickContext.Consumer>
            </>
        )
    }
}
```

```jsx
import {ErickContext} from "./ErickContext";

export function Third() {
    return (
        <>
            <ErickContext.Consumer>
                {
                    (value) => {
                        const {name, address} = value;
                        return <div>我是Third组件{name}{address}</div>
                    }
                }
            </ErickContext.Consumer>
        </>
    )

}
```

# 项目运行

## 1. 网络代理

![image-20240603095750770](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240603095750770.png)

### 1.1 配置方式一

- 在package.json中添加，React启动后，就会在3000端口启动一个Proxy，后序请求就会将数据发送到Server端的8080端口
- 3000端口只要没有的资源，都会去通过proxy找8080要
- 只能配置一个后台的端口

```bash
"proxy": "http://localhost:8080"
```

![image-20240603102230316](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240603102230316.png)

```jsx
import React from "react";
import axios from "axios";

export default class App extends React.Component {

    getInfo = () => {
        axios({
            method: 'GET',
            url: 'http://localhost:3000/erick/sleep'
        }).then(value => {
            console.log(value.data);
        }).catch(reason => {
            console.log(reason)
        })
    }

    render() {
        return (
            <div>
                <button onClick={this.getInfo}>点击获取数据</button>
            </div>)
    }
}
```

### 1.2 配置方式二

- 在React的src目录下，建一个名字必须为setupProxy.js的文件
- 请求不同服务，不同服务的不同转发
- React在执行时，会自动使用配置的setupProxy.js配置的规则

#### setupProxy.js

```js
const {createProxyMiddleware} = require('http-proxy-middleware');

module.exports = function (app) {
    app.use(
        '/firstserver',/*请求如果带这个前缀*/
        createProxyMiddleware({
            target: 'http://localhost:8080', // 目标服务器地址
            changeOrigin: true,              // 是否改变源地址
            pathRewrite: {
                '^/firstserver': '',                   // 重写路径
            },
        })
    );

    app.use(
        '/secondserver',
        createProxyMiddleware({
            target: 'http://localhost:9090', // 目标服务器地址
            changeOrigin: true,              // 是否改变源地址
            pathRewrite: {
                '^/secondserver': '',                   // 重写路径
            },
        })
    );
};
```

```jsx
import React from "react";
import axios from "axios";

export default class App extends React.Component {

    getFromFirstServer = () => {
        axios({
            method: 'GET',
            url: 'http://localhost:3000/firstserver/erick/sleep'
        }).then(value => {
            console.log(value.data);
        }).catch(reason => {
            console.log(reason)
        })
    }

    getFromSecondServer = () => {
        axios({
            method: 'GET',
            /*带前缀*/
            url: 'http://localhost:3000/secondserver/erick/sleep'
        }).then(value => {
            console.log(value.data);
        }).catch(reason => {
            console.log(reason)
        })
    }

    render() {
        return (
            <div>
                <button onClick={this.getFromFirstServer}>第一个服务</button>
                <button onClick={this.getFromSecondServer}>第二个服务</button>
            </div>)
    }
}
```

## 2. 打包

```bash
# 编译打包
npm run build
```

