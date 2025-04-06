# 基本入门

## 1. Vite脚手架

```bash
npm create vite@latest
# 依次输入  项目名， React, TypeScript+SWC
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

### main.tsx

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

### App.tsx

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
- tsx：React的模块组件代码        rfc快速生成函数式组件          
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

- tsx中，组件返回值都必须用一个闭合标签来包裹，因此多了很多不需要的div

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
- React在解析时，会自动把Fragment去掉
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

## 6. 虚拟DOM

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

## 7. React开发工具

- 在谷歌浏览器中加入，后面可以在Component中，查看每个组件的具体的属性，组件树之间的关系

![image-20240531135737923](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240531135737923.png)

![image-20240531140028704](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240531140028704.png)

# 函数组件

- 模块化：js的模块化，对js的拆分
- 组件：对js，html，css，image，video，font等的进行拆分
- 一个大的应用，所有的js，html等代码，都会划分为多个组件，多个组件拼接起来，就是一个完整的页面

## 基本写法

### 普通函数

```jsx
/*  渲染组件到页面
   * 1. React解析组件标签，找到了Father组件
   * 2. 发现是组件是使用函数定义的，调用该函数
   * 3. 将返回的该组件的虚拟DOM转换为真实DOM,随后呈现在页面*/
export default function Father() {
    /*1. 必须要有返回值，返回值就是该组件的虚拟DOM
    * 2. 首字母不要用小写，否则会当作html标签渲染
         首字母大写，就会当作组件来渲染*/
    return (
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

# React-Router

```bash
# SPA: Single Page Application: 单页面Web应用
- 整个页面只有一个完整的页面
- 点击页面中的链接，不会刷新页面，只会做页面的局部更新
- 数据都需要通过Ajax请求获取，并在前端异步呈现
- 单页面，多组件，通过点击，路由跳转到指定组件
```

## 基本介绍

- react的一个插件库，专门用来实现一个SPA应用，基于React的项目基本都会用到该插件
- [官网](https://reactrouter.com/home)

```bash
# react-router-dom
- 为web打造

# react-router-native
- 为react原生应用开发的

# react-router-any
- 全部都能用，通用型强，api能复杂点
```

```bash
npm i react-router-dom@7.5.0
```

## 路由规则

- 只能在url中进行跳转，没有页面上的link可以点
- 匹配成功后，不再向下遍历匹配

### serviceRouter.tsx

- element：都是路由组件，懒加载，只有点击触发了某个路由，才会去调用对应的函数式路由组件

```tsx
import {createBrowserRouter, createRoutesFromElements, Route} from "react-router-dom";
import Welcome from "../pages/welcome/Welcome.tsx";
import ErickError from "../pages/error/ErickError.tsx";
import Home from "../pages/home/Home.tsx";
import Good from "../pages/good/Good.tsx";
import Cart from "../pages/cart/Cart.tsx";
import Admin from "../pages/admin/Admin.tsx";
import Customer from "../pages/admin/customer/Customer.tsx";
import Guest from "../pages/admin/guest/Guest.tsx";

export const router = createBrowserRouter(createRoutesFromElements(
    /*顶层路由: 需要在Welcome中设置Outlet
      element: 配置的组件都是懒加载的*/
    <Route path={'/'} element={<Welcome/>} errorElement={<ErickError/>}>
        {/*二级路由*/}
        <Route path={'home'} element={<Home/>}/>
        <Route path={'good'} element={<Good/>}/>
        <Route path={'cart'} element={<Cart/>}/>
        {/*三级路由: 需要在Admin中设置Outlet*/}
        <Route path={'admin'} element={<Admin/>}>
            <Route path={'customer'} element={<Customer/>}/>
            <Route path={'guest'} element={<Guest/>}/>
        </Route>
    </Route>
));
```

### App.tsx

```tsx
import {RouterProvider} from "react-router-dom";
import {router} from "./routes/serviceRouter.tsx";

export default function App() {
    return (
        <RouterProvider router={router}></RouterProvider>
    )
}
```

### Welcome.tsx

```tsx
import {Outlet} from "react-router-dom";

export default function Welcome() {
    return (
        <>
            <div>我是欢迎页面</div>
            {/*声明下一级别的路由的渲染的地方*/}
            <Outlet/>
        </>
    );
}
```

![image-20250406142512635](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250406142512635.png)

## 页面点击

- 路由页面中，定义一些页面元素链接
- 可以定义在父路由页面中，可以识别到当前的路由，子路由，父路由
- 点击后，浏览器url进行跳转，从而触发之前定义的路由规则，装填不同的组件

### Welcome.tsx

```tsx
import {createBrowserRouter, createRoutesFromElements, Route} from "react-router-dom";
import Welcome from "../pages/welcome/Welcome.tsx";
import ErickError from "../pages/error/ErickError.tsx";
import Home from "../pages/home/Home.tsx";
import Good from "../pages/good/Good.tsx";
import Cart from "../pages/cart/Cart.tsx";
import Admin from "../pages/admin/Admin.tsx";
import Customer from "../pages/admin/customer/Customer.tsx";
import Guest from "../pages/admin/guest/Guest.tsx";

export const router = createBrowserRouter(createRoutesFromElements(
    /*顶层路由: 需要在Welcome中设置Outlet*/
    <Route path={'/'} element={<Welcome/>} errorElement={<ErickError/>}>
        {/*二级路由*/}
        <Route path={'home'} element={<Home/>}/>
        <Route path={'good'} element={<Good/>}/>
        <Route path={'cart'} element={<Cart/>}/>
        {/*三级路由: 需要在Admin中设置Outlet*/}
        <Route path={'admin'} element={<Admin/>}>
            <Route path={'customer'} element={<Customer/>}/>
            <Route path={'guest'} element={<Guest/>}/>
        </Route>
    </Route>
));
```

### Admin.tsx

```tsx
import {NavLink, Outlet} from "react-router-dom";

export default function Admin() {
    return (
        <>
            <div>用户管理</div>
            {/*全路径*/}
            <NavLink to={'/admin/customer'}>普通用户</NavLink><br/>
            {/*子路由名字*/}
            <NavLink to={'guest'}>游客模式</NavLink><br/>
            <Outlet/>
        </>
    );
}
```

## 编程式路由

- 当触发某个函数的时候，在ts的代码中，让浏览器中的url发生变化，从而再根据路由规则显示某些组件

```tsx
import {NavigateFunction, useNavigate} from "react-router-dom";

export default function Guest() {

    /*钩子函数*/
    const navigate: NavigateFunction = useNavigate();

    const handleGoToCart = () => {
        /*全路径*/
        navigate('/cart', {
            replace: false,
            state: {}
        });
    }

    return (
        <>
            <div>游客模式</div>
            <button onClick={handleGoToCart}>回到Cart</button>
        </>

    );
}
```

## 跳转模式

### push

- 默认方式

```bash
# 默认push，使用压栈操作，在浏览器页面
- 回退或者前进，可以进行

http://localhost:3000/home             # 3
http://localhost:3000/about/detail     # 2     # 当前元素
http://localhost:3000/about            # 1
```

### replace

```bash
# 在Navlink中开启 
 <NavLink replace={true}
 
# 替换操作， 如果下一个是替换模式，则会替换掉栈顶元素
http://localhost:3000/about/detail     # 2     
http://localhost:3000/about            # 1
```

## 传参

- 从一个路由跳转到其他路由时，可以携带数据传递到目标路由
- params参数，search参数，state参数

### 路由规则

- 在浏览器中直接输入对应的链接，即可
- 必须匹配好，不然后面的页面中的点击就会不生效

```tsx
import {createBrowserRouter, createRoutesFromElements, Route} from "react-router-dom";
import Welcome from "../pages/welcome/Welcome.tsx";
import ErickError from "../pages/error/ErickError.tsx";
import Home from "../pages/home/Home.tsx";
import Good from "../pages/good/Good.tsx";
import Cart from "../pages/cart/Cart.tsx";

export const router = createBrowserRouter(createRoutesFromElements(
    <Route path={'/'} element={<Welcome/>} errorElement={<ErickError/>}>

        {/*1. 既可以传参数，也可以不传参
           2. 如果没有外层的包裹，则只能使用带参数的url访问
           */}

        {/*1. params传递参数：
        页面链接： http://localhost:5173/home/123/erick_params*/}
        <Route path={'home'} element={<Home/>}>
            <Route path={':id/:type'} element={<Home/>}/>
        </Route>

        {/*2. search传递参数
        页面链接: http://localhost:5173/good?id=999&type=erick_search*/}
        <Route path={'good'} element={<Good/>}>
            <Route path={'good'} element={<Good/>}></Route>
        </Route>

        {/*3. state传参数： 不能通过页面链接的方式
        页面链接：http://localhost:5173/cart*/}
        <Route path={'cart'} element={<Cart/>}/>
    </Route>
));
```

### 页面点击

```tsx
import {NavLink, Outlet} from "react-router-dom";

export default function Welcome() {
    const params = {
        id: '123',
        type: 'erick_params',
    }

    const search = {
        id: '999',
        type: 'erick_search',
    }

    const state = {
        id: '666',
        type: 'erick_state',
    }

    return (
        <>
            <div>我是欢迎页面</div>
            {/*1. param 传参数*/}
            <NavLink to={`/home/${params.id}/${params.type}`}>Home页面</NavLink><br/>

            {/*2. search传参*/}
            <NavLink to={`/good?id=${search.id}&type=${search.type}`}>Good页面</NavLink><br/>

            {/*3. state传参数*/}
            <NavLink to={'/cart'} state={{
                id: state.id,
                type: state.type,
            }}>Cart页面</NavLink><br/>
            <Outlet/>
        </>
    );
}
```

### 数据接收

- 目标路由页面，接收传递的参数

```tsx
import {useParams} from "react-router-dom";

export default function Home() {
    /*useParams钩子函数*/
    const params = useParams();
    const {id, type} = params;
    return (
        <div>我是家{id} ===== {type}</div>
    );
}
```

```tsx
import {useLocation, useSearchParams} from "react-router-dom";

export default function Good() {
    /*方式一：searchParams*/
    const searchParams = useSearchParams();
    const [search] = searchParams;

    /*方式二：从location中获取：
    结果：   ?id=999&type=erick_search*/
    const location = useLocation();
    console.log(location.search);

    return (
        <>
            <div>{search.get('id')} ===== {search.get('type')}</div>
        </>
    )
}
```

```tsx
import {useLocation} from "react-router-dom";

export default function Cart() {
    /*钩子函数*/
    const location = useLocation();

    return (
        <div>购物车{location.state.id} === {location.state.type}</div>
    );
}
```

# 状态管理

## Context

- 多层父子组件之间通信的一种方式
- 开发中一般不用context，一般都用它来封装react插件
- Props传递：传递数据可以使用props，但是层级多了后，必须一层层传递props，比较麻烦
- 而且props传递时，层级多时，如果从顶层组件到底层组件，中间组件并不想获取到这个数据

### 定义

```ts
import {createContext} from "react";

export interface FoodContextConfig {
    name: string;
    version: string;
    appId: string;
    work: () => void;
}

/*定义context*/
export const FoodContext = createContext<FoodContextConfig>({} as FoodContextConfig);
```

### 顶层组件

- 包裹了Father组件，则Father组件及Father组件的子组件系列，都可以访问到顶组件的数据

```tsx
import {FoodContext} from "./component/context/FoodContext.ts";
import Father from "./component/cat/Father.tsx";

export default function App() {

    return (
        <div>
            <FoodContext.Provider value={{
                name: '顶部',
                version: '1.0.0',
                appId: 'random',
                work: () => {
                    console.log("hello")
                }
            }}>
                {/*子组件*/}
                <Father/>
            </FoodContext.Provider>
        </div>
    )
}
```

### 中间组件

```tsx
import FirstSon from "./FirstSon.tsx";
import SecondSon from "./SecondSon.tsx";

export default function Father() {

    /*当前组件用不到对应的属性，就什么也不做*/
    return (
        <>
            <FirstSon/>
            <SecondSon/>
        </>
    )
}
```

### 消费组件

- 消费组件可以通过顶层组件提供的方法，来对其中的内容进行修改

```tsx
import {useContext} from "react";
import {FoodContext, FoodContextConfig} from "../context/FoodContext.ts";

export default function FirstSon() {
    /*直接使用*/
    const {appId} = useContext<FoodContextConfig>(FoodContext);

    return (
        <div>first son{appId}</div>
    )
}
```

```tsx
import {FoodContext} from "../context/FoodContext";

export default function SecondSon() {
    console.log('SecondSon Render');
    return (
        <div>
            {/*复杂逻辑用这种*/}
            <FoodContext.Consumer>
                {context => (<div>{context.name}</div>)}
            </ FoodContext.Consumer>

        </div>
    );
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

