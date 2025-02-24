- [Vue 3 官网](https://vuejs.org/)
- [最新版本](https://vuejs.org/about/releases.html)

# 创建项目

## Vite

- [Vite官网](https://vite.dev/)
- 和Webpack类似，Vue官方开发的打包工具

```bash
- 支持TS, JSX, CSS等开箱即用
- 按需编译，不再等待整个应用编译完成
```

## 脚手架

```bash
# 安装create-vue脚手架，可以跟最新的版本
# 对应的Vue的版本是3.5.13
npm create vue@3.14.2

# 依次输入项目名称等其他配置
- 项目初始化完毕

npm i            # 安装对应的依赖
npm run dev     # 启动项目
```

## 开发者工具

![image-20250220130827125](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250220130827125.png)

## 基本结构

```vue
<!--template： 对应的html结构： 尽可能让里面的内容简单-->
<template>
  <div>你好</div>
</template>

<!--template: 对应的ts-->
<script lang="ts">

</script>

<!--style: 对应的样式-->
<style>
</style>
```

### scope

- 局部样式表，一般都必须加

```vue
<script lang="ts">
export default {
  name: "Dog",
}
</script>

<script setup lang="ts">

</script>

<template>
  <div class="erick">Hello World</div>
</template>

<!--scope： 局部样式表
1. 只能在当前Vue文件中引用,放置样式重名和污染
2. 如果不加scope，则该样式可以在任何地方使用-->
<style scoped>
.erick {
  background-color: blue;
}
</style>
```



## Composition API

- 组合式API
- Vue3新特性，用户组合data，methods, watch, computed等

# setup

### 入门DEMO

#### App.vue

```vue
<script setup lang="ts">

import MyPerson from "@/components/Person.vue";
</script>

<template>
<!--引入一个组件-->
  <MyPerson/>
</template>

<style>
</style>
```

#### Person.vue

```vue
<script lang="ts">
export default {
  /*组件的名字*/
  name: 'MyPerson',

  /*所有的TS代码集中在setup中*/
  setup() {
    /*setup中没有this*/
    /*这种定义的数据，和页面的template中不是响应式的*/
    let name = "Erick";
    let age = 20;
    let phone = "18892345621";

    function changeName() {
      console.log("name-changed");
      name = "Lucy";
    }

    function changeAge() {
      console.log("age-changed");
      age++;
    }

    function checkPhone() {
      alert(phone);
    }

    /*返回对应的变量和函数，能够在template中使用*/
    return {name, age, phone, changeName, changeAge, checkPhone};
  }
}
</script>

<template>
  <div>
    <div>姓名：{{ name }}</div>
    <div>年龄：{{ age }}</div>
    <div>电话：{{ phone }}</div>
    <button @click="changeName">修改姓名</button>
    <br>
    <button @click="changeAge">修改年龄</button>
    <br>
    <button @click="checkPhone">查看电话</button>
    <br>
  </div>
</template>

<style>

</style>
```

### 语法糖

```vue
<!--第一个script： 提供组件的名字-->
<script lang="ts">
export default {
  name: 'MyPerson',
}
</script>


<!--第二个script：定义好对应的setup-->
<script lang="ts" setup>
let name = "Erick";
let phone = "18892345621";

function changeName() {
  console.log("name-changed");
  name = "Lucy";
}

function checkPhone() {
  alert(phone);
}

/*不用再return了*/
</script>/

<template>
  <div>
    <div>姓名：{{ name }}</div>
    <div>电话：{{ phone }}</div>
    <button @click="changeName">修改姓名</button>
    <br>
    <button @click="checkPhone">查看电话</button>
    <br>
  </div>
</template>

<style>

</style>
```

# 响应式数据

## reactive

- 只能定义对象类型数据：对象，数组等
- 深层次比较
- 对原始数据进行了包裹

```vue
<script lang="ts">
export default {
  name: 'MyPerson',
}
</script>

<script lang="ts" setup>
import {reactive} from "vue";
import type {Student} from "@/entity/entity.ts";

let student = reactive<Student>({
  id: '1',
  name: 'Jack',
  address: 'Beijing',
});

function updateInfo() {
  student.id = '3';
  student.name = 'Lucy';
  student.address = 'New York';
}

</script>/

<template>
  <div>ID:{{ student.id }}</div>
  <div>ADDRESS:{{ student.address }}</div>
  <div>NAME:{{ student.name }}</div>
  <button @click="updateInfo">更新信息</button>
</template>

<style>
</style>
```

## ref

- 大部分场景推荐使用ref
- ref一把梭

### 基本类型

```vue
<script lang="ts">
export default {
  name: 'MyPerson',
}
</script>


<script lang="ts" setup>
import {ref} from "vue";

/*响应式数据类型*/
let name = ref<string>("Erick");
/*基本数据*/
let phone = "18892345621";


/*在js中必须 .value
* 在template中不用*/
function changeName() {
  name.value = 'Lucy';
  console.log(name.value);
}

function checkPhone() {
  alert(phone);
}
</script>/

<template>
  <div>
    <div>姓名：{{ name }}</div>
    <div>电话：{{ phone }}</div>
    <button @click="changeName">修改姓名</button>
    <br>
    <button @click="checkPhone">查看电话</button>
    <br>
  </div>
</template>

<style>
</style>
```

### 对象类型

- 底层使用的就是reactive

```vue
<script lang="ts">
export default {
  name: 'MyPerson',
}
</script>

<script lang="ts" setup>
import type {Student} from "@/entity/entity.ts";
import {ref} from "vue";

let student = ref<Student>({
  id: '1',
  name: 'Jack',
  address: 'Beijing',
});

function updateInfo() {
  student.value.id = '23';
  student.value.name = 'Lucy';
  student.value.address = 'New York';
}

</script>/

<template>
  <div>ID:{{ student.id }}</div>
  <div>ADDRESS:{{ student.address }}</div>
  <div>NAME:{{ student.name }}</div>
  <button @click="updateInfo">更新信息</button>
</template>

<style>
</style>
```

# Computed属性

- 计算属性的数据，只要他依赖的数据发生变化，计算属性就会变化
- 依赖的数据如果不变，computed属性调用多次时候，只计算一次(缓存)
- 方法是没有缓存的

```vue
<script lang="ts">
export default {
  name: 'MyPerson',
}
</script>

<script lang="ts" setup>
import {computed, ref} from "vue";

let firstName = ref<string>("张");
let lastName = ref<string>("三");

/*不能直接修改，但是通过firstName和lastName的变化引发的可以变*/
let fullName = computed(() => {
  console.log("calculating")
  return firstName.value + lastName.value;
})

function updateName() {
  firstName.value += '~';
  lastName.value += '*';
}
</script>/

<template>
  姓：<input type="text" v-model="firstName"/><br/>
  名：<input type="text" v-model="lastName"/><br/>
  <div>姓名：{{ fullName }}</div>
  <div>姓名：{{ fullName }}</div>
  <div>姓名：{{ fullName }}</div>
  <button @click="updateName">update</button>
</template>

<style>
</style>
```

# 监视

## Watch

- 监控一定的数据，从而触发对应的功能
- 监控场景

### ref-基本数据

```vue
<script lang="ts">
export default {
  name: "Dog",
}
</script>

<script setup lang="ts">
import {ref, watch} from "vue";

let sum = ref<number>(0);

function changeSum() {
  sum.value += 1;
}

// 监视： 可以通过返回值来结束监视
let stopWatch = watch(sum, (newValue, oldValue) => {
  /*结束监视*/
  if (newValue >= 5) {
    stopWatch();
  }

  console.log('sum 变化了', newValue, oldValue)
});
</script>

<template>
  <div>当前值{{ sum }}</div>
  <br/>
  <button @click="changeSum">点我加一</button>

</template>

<style scoped>

</style>
```

### ref-对象数据

```vue
<script lang="ts">
export default {
  name: "Dog",
}
</script>

<script setup lang="ts">
import {ref, watch} from "vue";
import "@/components/Dog.vue";
import type {People} from "@/entity/entity.ts";

let people = ref<People>({
  name: "Lucy",
  age: 18
});

/*改变属性*/
function changeName() {
  people.value.name += '~';
}

function changeAge() {
  people.value.age += 1;
}

function changePeople() {
  people.value = {
    name: "Tom",
    age: 20,
  }
}

// 监视： 监视的是对象的地址值，若想监视内部属性，需要手动开启deep
// 如果想上来先执行一次，immediate为true
// 有时候newValue和oldValue可能相同
watch(people, (newValue, oldValue) => {
  console.log('people changed', newValue, oldValue);
}, {deep: true, immediate: true});
</script>

<template>
  <div>当前值{{ people }}</div>
  <br/>
  <button @click="changeName">name属性</button>
  <br>
  <button @click="changeAge">age属性</button>
  <br>
  <button @click="changePeople">people地址值</button>
  <br>
</template>

<style scoped>

</style>
```

### reactive

```vue
<script lang="ts">
export default {
  name: "Dog",
}
</script>

<script setup lang="ts">
import {reactive, ref, watch} from "vue";
import "@/components/Dog.vue";
import type {People} from "@/entity/entity.ts";

let people = reactive<People>({
  name: "Lucy",
  age: 18
});

/*三种都能监视到*/
/*改变属性*/
function changeName() {
  people.name += '~';
}

function changeAge() {
  people.age += 1;
}

function changePeople() {
  Object.assign(people, {name: 'Tom', age: 20})
}

/*默认开启深度监视，不能关闭*/
watch(people, (newValue, oldValue) => {
  console.log('changed', newValue, oldValue);
})
</script>

<template>
  <div>当前值{{ people }}</div>
  <br/>
  <button @click="changeName">name属性</button>
  <br>
  <button @click="changeAge">age属性</button>
  <br>
  <button @click="changePeople">people地址值</button>
  <br>
</template>

<style scoped>

</style>
```

## watchEffect

- 用于监控属性，和watch类似，但是会动态分析需要监视的数据

```vue
<script lang="ts">
export default {
  name: "Dog",
}
</script>

<script setup lang="ts">

import {ref, watchEffect} from "vue";

let firstAge = ref<number>(10);
let secondAge = ref<number>(100);

function addFirst() {
  firstAge.value += 1;
}

function addSecond() {
  secondAge.value += 1;
}

/*1. 一上来就执行一次
* 2. 动态决定监控的数据类型，不用显示声明*/
watchEffect(() => {
  console.log("execute");
  if (firstAge.value >= 12 || secondAge.value >= 102) {
    console.log("age expired")
  }
})
</script>

<template>
  <div>{{ firstAge }}</div>
  <div>{{ secondAge }}</div>
  <button @click="addFirst">第一个加</button>
  <button @click="addFirst">第二个加</button>
</template>

<style scoped>

</style>
```

# ref-标签

## 普通html

- 获取到当前html的元素，且在不同组件中，ref的名字可以重名
- 通过原生js的代码通过id来获取时，在不同组件中如果有重名，则会发生异常

```vue
<script lang="ts">
export default {
  name: "Dog",
}
</script>

<script setup lang="ts">

import {ref} from "vue";

/*名字必须和下面的ref的名字对应*/
let erick = ref();
let lucy = ref();


/* <div>舒展</div>*/
function checkErick() {
  console.log(erick.value);
}

function checkLucy() {
  console.log(lucy.value);
}

</script>

<template>
  <div ref="erick">舒展</div>
  <div ref="lucy">露西</div>
  <button @click="checkErick">查看舒展</button>
  <button @click="checkLucy">查看露西</button>
</template>

<style scoped>

</style>
```

## 组件元素

- 获取到当前的子组件元素，子组件的数据和方法，必须显示暴露，父组件才能拿到

```vue
<script lang="ts">
export default {
  name: "Dog",
}
</script>

<script setup lang="ts">
import {ref} from "vue";

let age = ref<number>(0);
let name = ref<string>("erick");

/*子组件定义好需要暴露的元素*/
defineExpose(age, name);
</script>

<template>
</template>

<style scoped>

</style>
```

```vue
<script setup lang="ts">
import Dog from "@/components/Dog.vue";
import {ref} from "vue";

let dog = ref();

// 可以查看属性或和对应的方法 
function check() {
  dog.value.sayHello();
  console.log(dog.value.name);
  console.log(dog.value.age);
}
</script>

<template>
  <Dog ref="dog"/>
  <button @click="check">点我查看子组件</button>
</template>

<style>
</style>
```



# Props

- 父组件给子组件传递数据

```vue
<script lang="ts">
export default {
  name: "Dog",
}
</script>

<script setup lang="ts">


import "@/components/Dog.vue";
import type {People} from "@/entity/entity.ts";

/*定义数据
* 类型，必要性，默认值*/
withDefaults(defineProps<{ people?: People, pageSize?: number, isActive?: boolean, gender?: string }>(), {
  people: () => {
    return {
      name: 'Lucy',
      age: 18,
    }
  },
  pageSize: 10,
  isActive: false,
  gender: 'default'
})
</script>

<template>
  <div>people：{{ people }}</div>
  <div>pageSize：{{ pageSize }}</div>
  <div>isActive：{{ isActive }}</div>
  <div>gender：{{ gender }}</div>
</template>

<style scoped>

</style>
```

```vue
<script setup lang="ts">
import Dog from "@/components/Dog.vue";
import type {People} from "@/entity/entity.ts";

let people: People = {
  name: "张合适",
  age: 12,
}

</script>

<template>
  <!-- 1.     : 表示绑定的是变量，数字，
       2.      不加： ,绑定的是字符串-->
  <Dog :people="people" :page-size="20" :is-active="true" gender="male"/>
</template>

<style>
</style>
```

# 生命周期

- 先去渲染子组件，再去渲染父组件

```vue
<script lang="ts">
export default {
  name: "Dog",
}
</script>

<script setup lang="ts">

import {onBeforeMount, onBeforeUnmount, onBeforeUpdate, onMounted, onUnmounted, onUpdated, ref} from "vue";

let data = ref<string>("hello");

function updateData() {
  data.value += '~';
}

/*1*/
console.log("创建");

/*2*/
onBeforeMount(() => {
  console.log("挂载前")
});

/*3*/
onMounted(() => {
  console.log("挂载完毕")
});

/*4*/
onBeforeUpdate(() => {
  console.log("更新前")
});

/*5*/
onUpdated(() => {
  console.log("更新完毕")
});

/*6*/
onBeforeUnmount(() => {
  console.log("卸载前")
});

/*7*/
onUnmounted(() => {
  console.log("卸载完毕");
});

</script>

<template>
  <div>数据：{{ data }}</div>
  <button @click="updateData">更新数据</button>
</template>

<style scoped>

</style>
```

```vue
<script setup lang="ts">

import Dog from "@/components/Dog.vue";
import {ref} from "vue";

let flag = ref<boolean>(true);

</script>
<template>
  <button @click="flag=!flag">卸载子组件</button>
  <Dog v-if="flag"/>
</template>

<style>
</style>
```

# 自定义Hooks



