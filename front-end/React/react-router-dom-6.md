- 6版本和5版本差距比较大
- React明确推荐：函数式组件
- [官网](https://reactrouter.com/en/main)

```bash
npm i react-router-dom                           # "react-router-dom": "^6.24.0",
npm i --save-dev @types/react-router-dom         # "@types/react-router-dom": "^5.3.3"
```

# 函数式

- 更受欢迎的一种路由配置方式

## 1. 路由规则

- 只能在url中进行跳转，没有页面上的link可以点

### serviceRouters.tsx

```tsx
import {createBrowserRouter, createRoutesFromElements, Route} from "react-router-dom";
import Home from "../pages/Home";
import Cart from "../pages/cart/Cart";
import Good from "../pages/good/Good";
import Admin from "../pages/admin/Admin";
import {User} from "../pages/admin/user/User";
import {Customer} from "../pages/admin/customer/Customer";
import {ErrorPage} from "../pages/errorPages/ErrorPage";

export const router = createBrowserRouter(createRoutesFromElements(
    /*顶层路由*/
    <Route path='/' element={<Home/>} errorElement={<ErrorPage/>}>
        {/*二级路由，需要在Home组件中使用outlet定位*/}
        <Route path='cart' element={<Cart/>}/>

        {/*匹配成功后，不再向下遍历匹配*/}
        <Route path='good' element={<Good/>}></Route>
        <Route path='good' element={<Cart/>}></Route>

        {/*三级路由，需要在admin下面设置Outlet*/}
        <Route path='admin' element={<Admin/>}>
            <Route path='user' element={<User/>}></Route>
            <Route path='customer' element={<Customer/>}></Route>
        </Route>
    </Route>
));
```

### App.tsx

```tsx
import React from 'react';
import {RouterProvider} from "react-router-dom";
import {router} from "./routes/serviceRouters";

function App() {
    return (
        <RouterProvider router={router}></RouterProvider>
    );
}

export default App;
```

### Home.tsx

- 在任何需要使用到二级路由的地方，需要使用\<Outlet/>来指定渲染地方

```tsx
import {Outlet} from "react-router-dom";

export default function Home() {
    return (

        <div>
            {/*显示二级路由,确定二级路由的内容在什么地方显示*/}
            <Outlet/>
            <h3>我是Home</h3>
        </div>
    );
}
```

![image-20240630103554886](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240630103554886.png)

## 2. 页面点击

- 在主页面的pages中，可以定义路由规则，并装填不同的component

### 2.1 Home.tsx

```tsx
import {Link, NavLink, Outlet} from "react-router-dom";

export default function Home() {
    return (

        <div>
             {/*装填其他的内容组件*/}
            <div>欢迎来到home界面</div>
        
            {/*方式一：全路径*/}
            <NavLink to='/good'>Good商品</NavLink><br/>

            {/*方式二：当前子路由的名字*/}
            <Link to='./cart'>Cart购物车</Link><br/>

            {/*方式三：方式二的简写*/}
            <NavLink to='admin'>Admin界面</NavLink>

            {/*显示二级路由,确定二级路由的内容在什么地方显示*/}
            <Outlet/>
        </div>
    );
}
```

### 2.2 Admin.tsx

```tsx
import {Link, NavLink, Outlet} from "react-router-dom";

export default function Admin() {
    return (
        <div>
            {/*定义路由规则*/}
            {/*方式一：全路径*/}
            <NavLink to='/admin/user'>User界面</NavLink><br/>

            {/*方式二：当前子路由的名字*/}
            <Link to='./customer'>Customer界面</Link><br/>

            <h3>我是admin</h3>
            <Outlet/>
        </div>
    );
}
```

## 3. 路由传参

### 3.1 路由规则中

- 在浏览器中直接点击

```tsx
import {createBrowserRouter, createRoutesFromElements, Route} from "react-router-dom";
import Home from "../pages/Home";
import Cart from "../pages/cart/Cart";
import Good from "../pages/good/Good";
import {ErrorPage} from "../pages/errorPages/ErrorPage";
import Manage from "../pages/admin/manage/Manage";

export const router = createBrowserRouter(createRoutesFromElements(
    /*顶层路由*/
    <Route path='/' element={<Home/>} errorElement={<ErrorPage/>}>

        {/*1. 既可以传参数，也可以不传参
           2. 如果没有外层的包裹，则只能使用带参数的url访问*/}

        {/*1 .params传递参数
              页面链接： http://localhost:3000/cart/15/xian  */}
        <Route path='cart' element={<Cart/>}>
            <Route path=':id/:address' element={<Cart/>}/>
        </Route>


        {/*2. search传递参数
              页面链接：http://localhost:3000/good?id=15&address=xian&name=shuzhan*/}
        <Route path='good' element={<Good/>}>
            <Route path='good' element={<Good/>}></Route>
        </Route>


        {/*3. state传参
              不能通过页面链接的方式*/}
        <Route path='manage' element={<Manage/>}/>
    </Route>
))
```

### 3.2 页面点击时

- 通过页面点击的方式跳转

```tsx
import {Link, NavLink, Outlet} from "react-router-dom";

export default function Home() {

    const params = {
        id: '123',
        address: 'beijing',
    }

    const searchs = {
        id: 'adf',
        name: 'erick',
        address: '西安',
    }

    const states = {
        id: '34',
        name: 'lucy',
        address: '北京'
    }

    return (

        <div>
            <div>欢迎来到home界面</div>

            {/*1. param传参数*/}
            <NavLink to={`cart/${params.id}/${params.address}`}>Cart购物车</NavLink><br/>

            {/*2. search传参*/}
            <Link to={`good?id=${searchs.id}&name=${searchs.name}&address=${searchs.address}`}>Good商品</Link><br/>

            {/*3. state传参*/}
            <NavLink to='manage' state={{
                id: states.id,
                name: states.name,
                address: states.address,
            }}>Manage界面</NavLink><br/>

            {/*显示二级路由,确定二级路由的内容在什么地方显示*/}
            <Outlet/>
        </div>
    );
}
```

### 3.3 数据接收

```tsx
import {useParams} from "react-router-dom";

export default function Cart() {
    /*接收params参数*/
    const params = useParams();
    console.log(params);
    const {id, address} = params;

    return (
        <div>
            我是cart {id} === {address}
        </div>
    );
}
```

```tsx
import {useLocation, useSearchParams} from "react-router-dom";

export default function Good() {
    /*获取search参数*/
    /*获取方式一： search为传递的参数，setSearch为改变search的方法*/
    const [search, setSearch] = useSearchParams();
    const id = search.get('id');
    const name = search.get('name');
    const address = search.get('address');

    /*获取方式二：可以从location里面获取到
    *  ?id=15&address=xian&name=shuzhan
    * */
    let location = useLocation();

    console.log(location.search)
    return (
        <div>
            我是Goods {id} === {name} === {address}
        </div>
    );
}
```

```tsx
import {useLocation} from "react-router-dom";

export default function Manage() {
    let location = useLocation();
    const state = location.state;

    let id = '';
    let name = '';
    let address = '';
    if (state !== undefined && state !== null) {
        id = state.id;
        name = state.name;
        address = state.address
    }
    console.log(state)
    return (
        <div>
            我是Manage {id}==={name}==={address}
        </div>
    );
}
```

# 对象式

- 另外一种写法，和上面使用方式一样
- 定义好路由表，一般在src/routes目录下
- 定义好路由规则，React-Router帮忙渲染
- 一般在src/routes目录下

```tsx
import {createBrowserRouter, RouteObject} from "react-router-dom"
import Home from "../pages/Home";
import Cart from "../pages/cart/Cart";
import Good from "../pages/good/Good";
import Manage from "../pages/admin/manage/Manage";
import Admin from "../pages/admin/Admin";
import {User} from "../pages/admin/user/User";
import {Customer} from "../pages/admin/customer/Customer";

const rootTables: RouteObject[] = [
    {
        path: '/',
        element: <Home/>,
        children: [
            {
                path: 'cart',
                element: <Cart/>,
                children: [
                    {
                        path: ':id/:address',
                        element: <Cart/>
                    }
                ]
            },
            /* search传递参数
              页面链接：http://localhost:3000/good?id=15&address=xian&name=shuzhan*/
            {
                path: 'good',
                element: <Good/>
            },
            {
                path: 'manage',
                element: <Manage/>
            },
            {
                path: 'admin',
                element: <Admin/>,
                children: [
                    {
                        path: 'user',
                        element: <User/>
                    },
                    {
                        path: 'customer',
                        element: <Customer/>
                    }
                ]
            }
        ]
    }
]

/*外部使用的*/
export const rooter = createBrowserRouter(rootTables);
```

# 编程式路由

```tsx
import {NavigateFunction, useNavigate} from "react-router-dom";

export function User() {
    /*类似5中的props中取history的api*/
    let navigate: NavigateFunction = useNavigate();

    const jumpToCart = () => {
        navigate('/cart', {
            replace: false,
            state: {}
        })
    }

    const jumpToGood = () => {
        navigate('/good', {
            replace: false,
            state: {}
        })
    }

    return (
        <div>
            <button onClick={jumpToCart}>跳转到Cart</button>
            <button onClick={jumpToGood}>跳转到Good</button>
        </div>
    );
}
```

