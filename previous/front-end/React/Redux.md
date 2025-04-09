# 简介

## 1. 简介

- Redux是一个专门用于做状态管理的JS库，不是react插件库
- 作用：集中式管理react应用中多个组件共享的状态

## 2. 适用场景

- 很多数据随时间而变化
- 希望状态有一个唯一确定的来源（single source of truth）
- 将所有状态放在顶层组件中管理已不可维护
- 总体原则：能不用就不用，如果不用比较吃力，才考虑使用

## 3. 原理图

![image-20240609151028634](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240609151028634.png)

#  Redux核心-基础

- 基于浏览器内存的存储方式，刷新页面后，会丢失原来的state
- 当前组件对一些数据进行读写，同时将这些数据也暴露给其他组件

```bash
- 包含全局状态的单一仓库
- 当应用中发生某些事情时，分发普通对象(plain object) 动作(action)给仓库
- Pure reducer 函数查看这些动作(action)并且返回不可更新的状态
```

```bash
- npm i redux                     #  "redux": "^5.0.1"
```

## 1. store.ts

- 对外暴露一个store，整个应用只有一个Store对象

```ts
import {createStore} from "redux";
import {rootReducer} from "./CombinedReducer";

/*根组件*/
export default createStore(rootReducer);
```

## 2. rootReducer.ts

- 统一暴露多个不同的reducer
- 所有的redux-state属性，都会以树的形式存放在redux中
- 不同功能的Reducer，建议分开，这样state属性好维护

```ts
import counterReducer from "./reducer/CounterReducer";
import {combineReducers} from "redux";
/*会将多个Reducer合并成一个根组件树
* state的状态{"k1","v1","k2","v2 }
* */
export const rootReducer = combineReducers({
    calculator: counterReducer
});
```

### 2.1 CounterReducer.ts

```ts
import {CounterActionType} from "../action/CounterAction";
import {DECREMENT, INCREMENT} from "../constants";

/*该Reducer存储的数据对象*/
type CalculatorResult = {
    curVal: number;
    calculateTimes: number;
    lastOperationName: string;
}

/*1 专门为某个组件服务的reducer
* 2.本质是一个函数
*   2.1 参数一： 之前的状态： preState,
*   2.2 参数二： 动作对象 action*/
/*实际存储的结果*/
const initialState: CalculatorResult = {
    curVal: 0,
    calculateTimes: 0,
    lastOperationName: 'Erick'
}

export default function counterReducer(prevState: CalculatorResult, action: CounterActionType): CalculatorResult {
    /*第一次
     prevState: undefined
     action: "@@redux/INIT   c.y.w.x.b(随机字符串，防止和下面的判断对上)" */
    if (prevState === undefined) {
        console.log('initial-state', prevState)
        prevState = initialState;
    }
    console.log('state', prevState)

    const {type, data} = action;

    /*获取到原来的数据*/
    let nextState: CalculatorResult = {
        ...prevState
    }

    /*根据数据操作类型，进行匹配*/
    switch (type) {
        case INCREMENT:
            nextState.curVal = prevState.curVal + data.value;
            break;
        case DECREMENT:
            nextState.curVal = prevState.curVal - data.value;
            break;
        default:
            return prevState;
    }

    nextState.lastOperationName = data.name;
    nextState.calculateTimes = prevState.calculateTimes + 1;
    return nextState;
}
```

### 2.2 其他的reducer

## 3. Action

### 3.1 CounterAction.ts

```ts
/*为某个组件生成对应的Action对象*/
import {CalculatorInput} from "../../component/Counter";
import {DECREMENT, INCREMENT} from "../constants";

export type CounterActionType = {
    type: string,
    data: CalculatorInput;
}

export function createIncrementAction(data: CalculatorInput): CounterActionType {
    return {
        type: INCREMENT,
        data: data
    }
}

export function createDecrementAction(data: CalculatorInput): CounterActionType {
    return {
        type: DECREMENT,
        data: data
    }
}
```

### 3.2 其他action

### 3.3 constant.ts

```ts
export const INCREMENT = 'INCREMENT';
export const DECREMENT = 'DECREMENT';
```

## 4. Counter.tsx

- 容器组件
- 之前怎么向redux-state中存的，就怎么取

```tsx
/*value：具体的数据
* name：操作人的信息*/
import React, {BaseSyntheticEvent} from "react";
import store from "../redux/store";
import {createDecrementAction, createIncrementAction} from "../redux/action/CounterAction";

export type CalculatorInput = {
    value: number;
    name: string;
}

export function Counter() {

    let valRef = React.useRef(1);
    let nameRef = React.useRef('');

    const handleClickIncrement = () => {
        const calculatorInput: CalculatorInput = {
            value: valRef.current * 1,
            name: nameRef.current
        }
        /*向store发送任务*/
        store.dispatch(createIncrementAction(calculatorInput));
    }

    const handleClickDecrement = () => {
        const calculatorInput: CalculatorInput = {
            value: valRef.current,
            name: nameRef.current
        }
        /*向store发送任务*/
        store.dispatch(createDecrementAction(calculatorInput));
    }

    const handleClickIncrementIfOdd = () => {
        const calculatorInput: CalculatorInput = {
            value: valRef.current * 1,
            name: nameRef.current
        }
        /*查找出之前的数据*/
        let previous = store.getState().calculator;
        if (previous?.curVal % 2 === 0) {
            return;
        } else {
            store.dispatch(createIncrementAction(calculatorInput))
        }
    }


    /*页面初次加载完毕后，开启监控:监测整个state树
    * 1. 如果发现变化了，就会手动设置一个React的state属性，触发整个页面重新渲染*/
    const [flagCount, setFlagCount] = React.useState(100);

    React.useEffect(() => {
        store.subscribe(() => {
            setFlagCount((previous => {
                return previous + 1;
            }));
        })
    }, [])

    return (
        <div>
            {/*redux中state变化后，不会触发页面重新render*/}
            <h3>{flagCount}</h3>
            <h3>当前值：{store.getState().calculator?.curVal}</h3>
            <h3>计算次数：{store.getState().calculator?.calculateTimes}</h3>
            <h3>上次操作人姓名：{store.getState().calculator?.lastOperationName}</h3>

            <select onChange={(event: BaseSyntheticEvent) => {
                console.log(event.target.value)
                valRef.current = event.target.value;
            }}>
                <option value={1}>1</option>
                <option value={2}>2</option>
                <option value={3}>3</option>
            </select>
            &nbsp;&nbsp;&nbsp;
            <button onClick={handleClickIncrement}>+</button>
            &nbsp;&nbsp;&nbsp;
            <button onClick={handleClickDecrement}>-</button>
            &nbsp;&nbsp;&nbsp;
            <button onClick={handleClickIncrementIfOdd}>奇数+</button>
            &nbsp;&nbsp;&nbsp;

            <div>
                提交人：<input type='text' onChange={(event: BaseSyntheticEvent) => {
                nameRef.current = event.target.value;
            }}/>
            </div>
        </div>
    );
}
```

# Redux核心-异步

```bash
npm i redux-thunk  # "redux-thunk": "^3.1.0",
```

## 1. store.ts

```ts
import {applyMiddleware, createStore} from "redux";
import {rootReducer} from "./CombinedReducer";
/*支持异步任务*/
import {thunk} from "redux-thunk";

/*根组件*/
// @ts-ignore
export default createStore(rootReducer, applyMiddleware(thunk));
```

## 2. Action.ts

```ts
/*为某个组件生成对应的Action对象*/
import {CalculatorInput} from "../../component/Counter";
import {INCREMENT} from "../constants";

export type CounterActionType = {
    type: string,
    data: CalculatorInput;
}

/*同步Action:
* 1. 返回值是一个对象*/
export function createIncrementAction(data: CalculatorInput): CounterActionType {
    return {
        type: INCREMENT,
        data: data
    }
}

/*异步Action:
* 1. 返回值是一个函数，
* 2. store会调用该函数，调用时会传递一个store.dispatch的对象
* 3. 异步Action最终创建的还是一个同步Action*/
export function createAsyncIncrementAction(data: CalculatorInput, timeSeconds: number) {
    return (dispatch: any) => {
        setTimeout(() => {
            dispatch(createIncrementAction(data));
        }, timeSeconds * 1000)
    }
}
```

## 3. Counter.tsx

```tsx
/*value：具体的数据
* name：操作人的信息*/
import React, {BaseSyntheticEvent} from "react";
import store from "../redux/store";
import {createAsyncIncrementAction, createIncrementAction} from "../redux/action/CounterAction";

export type CalculatorInput = {
    value: number;
    name: string;
}

export function Counter() {

    let valRef = React.useRef(1);
    let nameRef = React.useRef('');

    const handleAsynClickByReact = () => {
        const calculatorInput: CalculatorInput = {
            value: valRef.current * 1,
            name: nameRef.current
        }
        setTimeout(() => {
            store.dispatch(createIncrementAction(calculatorInput));
        }, 2000)
    }

    const handleAsyncClickByRedux = () => {
        const calculatorInput: CalculatorInput = {
            value: valRef.current * 1,
            name: nameRef.current
        }
        store.dispatch(createAsyncIncrementAction(calculatorInput, 2))
    }


    const [flagCount, setFlagCount] = React.useState(100);

    React.useEffect(() => {
        store.subscribe(() => {
            setFlagCount((previous => {
                return previous + 1;
            }));
        })
    }, [])

    // @ts-ignore
    const {calculateTimes, curVal, lastOperationName} = store.getState().calculator;
    return (
        <div>
            {/*redux中state变化后，不会触发页面重新render*/}
            <h3>{flagCount}</h3>

            <h3>计算次数：{calculateTimes}</h3>
            <h3>当前值：{curVal}</h3>
            <h3>上次操作人姓名：{lastOperationName}</h3>

            <select onChange={(event: BaseSyntheticEvent) => {
                console.log(event.target.value)
                valRef.current = event.target.value;
            }}>
                <option value={1}>1</option>
                <option value={2}>2</option>
                <option value={3}>3</option>
            </select>
            &nbsp;&nbsp;&nbsp;
            <button onClick={handleAsynClickByReact}>组件异步加</button>
            &nbsp;&nbsp;&nbsp;
            <button onClick={handleAsyncClickByRedux}>Redux异步加</button>
            &nbsp;&nbsp;&nbsp;
            <div>
                提交人：<input type='text' onChange={(event: BaseSyntheticEvent) => {
                nameRef.current = event.target.value;
            }}/>
            </div>
        </div>
    );
}
```

# Redux- devtools

- 浏览器中下载Redux Devtools

![](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240628145857931.png)

```bash
npm i @redux-devtools/extension    #  "@redux-devtools/extension": "^3.3.0"
```



## 1. Store.ts

```ts
import {applyMiddleware, createStore} from "redux";
import {rootReducer} from "./CombinedReducer";
/*支持异步任务*/
import {thunk} from "redux-thunk";
import {composeWithDevTools} from "@redux-devtools/extension";

/*
* 1. 没有异步任务：composeWithDevTools()
* 2. 有异步任务： composeWithDevTools(applyMiddleware(thunk))*/
// @ts-ignore
export default createStore(rootReducer, composeWithDevTools(applyMiddleware(thunk)));
```

![image-20240628150706951](/Users/shuzhan/Library/Application Support/typora-user-images/image-20240628150706951.png)

# Redux-Tookit

- Redux官方推荐的，使用Redux的方式

```bash
# 官方包
npm install @reduxjs/toolkit   #  "@reduxjs/toolkit": "^2.2.5"

# React的UI包
npm install react-redux         # "react-redux": "^9.1.2"
```

## 1. store.ts

```ts
import {configureStore} from "@reduxjs/toolkit";
import {counterReducer} from "./reducer/counterSlice";

/*1. 引入了reducer
* 2. 自动配置了Redux-Devtools扩展*/
export const store = configureStore({
    reducer: {
        /*以k，v的方式，存放对应的state属性，最终合并成一个根state*/
        calculator: counterReducer
        /*定义其他的reducer*/
    }
})

/*从store本身，推断出 'RootState' 和 'AppDispatch类型'*/
export type RootState = ReturnType<typeof store.getState>;

/*推断出类型：{calculator:CalculatorState}*/
export type AppDispatch = typeof store.dispatch;
```

## 2. counterslice.ts

```ts
import {createSlice, PayloadAction} from "@reduxjs/toolkit";
import {CalculatorInput} from "../../component/Counter";

/*该reducer的state*/
type CalculatorState = {
    curVal: number,
    operationTimes: number,
    lastUpdatedName: string,
}

const initialState: CalculatorState = {
    curVal: 0,
    operationTimes: 0,
    lastUpdatedName: 'Erick'
}

export const counterSlice = createSlice({
    name: 'counter',                 /*作为action的字符串的一部分 counter/increment*/
    initialState: initialState,      /*初始值*/
    reducers: {
        /*1. 内部使用了Immer库，可以允许写'可变逻辑'
        * 2. 每个case reducer函数，会生成对应的Action,必须用PayloadAction来包装传入的action*/
        increment: (state: CalculatorState, action: PayloadAction<CalculatorInput>) => {
            console.log(action.type) //  'counter/increment'
            state.curVal += action.payload.value;
            state.lastUpdatedName = action.payload.name;
            state.operationTimes += 1;
            /*只能在createSlice和createReducer中写上面的可变代码*/
        },

        decrement: (state: CalculatorState, action: PayloadAction<CalculatorInput>) => {
            console.log(action.type) //  'counter/decrement'
            state.curVal -= action.payload.value;
            state.lastUpdatedName = action.payload.name;
            state.operationTimes += 1;
        },
    }
})

/*导出对应的Action*/
export const {increment, decrement} = counterSlice.actions;

/*外部使用的reducer，导出对应的reducer*/
export let counterReducer = counterSlice.reducer;
```

## 3. hooks.ts

```ts
import {TypedUseSelectorHook, useDispatch, useSelector} from 'react-redux'
import type {AppDispatch, RootState} from './store'

// 在整个应用程序中使用，而不是简单的 `useDispatch` 和 `useSelector`
export const useAppDispatch: () => AppDispatch = useDispatch
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector
```

## 4. Counter.tsx

```tsx
import React, {BaseSyntheticEvent} from "react";
import {decrement, increment} from "../redux/reducer/counterSlice";
import {useAppDispatch, useAppSelector} from "../redux/hooks";

export type CalculatorInput = {
    value: number,
    name: string,
}

export function Counter() {
    let valRef = React.useRef(1);
    let nameRef = React.useRef('');

    /*使用自定义的钩子*/
    const dispatch = useAppDispatch();

    /*获取到当前的state，提取数据*/
    let curState = useAppSelector((state => {
        return state.calculator
    }));

    const handleClickIncrement = () => {
        const calculatorInput: CalculatorInput = {
            value: valRef.current * 1,
            name: nameRef.current
        }

        /*向store分发任务*/
        dispatch(increment(calculatorInput))
    }

    const handleClickDecrement = () => {
        const calculatorInput: CalculatorInput = {
            value: valRef.current * 1,
            name: nameRef.current
        }

        /*向store分发任务*/
        dispatch(decrement(calculatorInput))
    }

    const handleClickIncrementIfOdd = () => {
        const calculatorInput: CalculatorInput = {
            value: valRef.current * 1,
            name: nameRef.current
        }

        if (curState.curVal % 2 !== 0) {
            dispatch(increment(calculatorInput));
        }
    }

    return (
        <div>

            <h3>当前值：{curState.curVal}</h3>
            <h3>当前计算次数：{curState.operationTimes}</h3>
            <h3>最后更新人：{curState.lastUpdatedName}</h3>

            <select onChange={(event: BaseSyntheticEvent) => {
                valRef.current = event.target.value;
            }}>
                <option value={1}>1</option>
                <option value={2}>2</option>
                <option value={3}>3</option>
            </select>
            &nbsp;&nbsp;&nbsp;
            <button onClick={handleClickIncrement}>+</button>
            &nbsp;&nbsp;&nbsp;
            <button onClick={handleClickDecrement}>-</button>
            &nbsp;&nbsp;&nbsp;
            <button onClick={handleClickIncrementIfOdd}>奇数+</button>
            &nbsp;&nbsp;&nbsp;

            <div>
                提交人：<input type='text' onChange={(event: BaseSyntheticEvent) => {
                nameRef.current = event.target.value;
            }}/>
            </div>

        </div>
    );
}
```

# Tips

## 1. 刷新丢失

- 是记录浏览器内存的一种存储方式，刷新页面时，状态会恢复成初始值
- 可以结合localstorage实现持久化

```ts
import {createSlice, PayloadAction} from "@reduxjs/toolkit";
import {LoginResponse} from "../../component/Login/Login";
import {PASSWORD, TOKEN, USERNAME} from "../../component/constant/Constants";

type LoginState = {
    username: string,
    password: string,
    token: string,
}

/*初始化时，先从localstorage中拿,避免页面刷新时，redux中数据丢失问题*/
const initialState: LoginState = {
    username: localStorage.getItem(USERNAME) || '',
    password: localStorage.getItem(PASSWORD) || '',
    token: localStorage.getItem(TOKEN) || '',
}


export const loginSlice = createSlice({
    name: 'login',
    initialState: initialState,
    reducers: {
        saveLoginUser: (state: LoginState, action: PayloadAction<LoginResponse>) => {
            const {username, password, token} = action.payload;
            state.username = username;
            state.password = password;
            state.token = token;

            /*保存时候，localStorage也存一份*/
            localStorage.setItem(USERNAME, username);
            localStorage.setItem(PASSWORD, password);
            localStorage.setItem(TOKEN, token);
        }
    }
})

export const {saveLoginUser} = loginSlice.actions;

export let loginReducer = loginSlice.reducer;
```

