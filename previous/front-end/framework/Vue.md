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

## toRefs

- 对于解构赋值出来的数据，也做成响应式的，原数据必须是reactive

```vue
<script setup lang="ts">
import {reactive, toRefs} from "vue";
import type {Student} from "@/component/entity.ts";

let data = reactive<Student>({
  name: 'erick',
  address: "xian",
});

/*解构赋值出来的数据，必须加toRef才能变成响应式*/
let {name, address} = toRefs(data);

function updateName() {
  data.address += '~';
}

function updateAddress() {
  data.name += '~';
}
</script>

<template>
  <div>{{ data.address }}======{{ data.name }}</div>
  <div>{{ name }}$$$$$$$$${{ address }}</div>
  <button @click="updateAddress">更新地址</button>
  <button @click="updateName">更新姓名</button>
</template>

<style scoped>
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

# 组件通信

## Props

### 父-->子

- 父传递给子，属性必须是非函数

#### 父

```vue
<script setup lang="ts">
import type {People} from "@/entity/entity.ts";
import Son from "@/components/Son.vue";

let people: People = {
  id: 123,
  name: '父亲',
  address: '北京'
}
</script>

<template>
  <div>我是父亲{{ people }}</div>
  <!--1.    :   表示绑定的是变量，数字，boolean值
         不加:   当作字符串解析-->
  <Son :people="people" :pageSize="10" gender="male" :flag="true"/>
</template>

<style scoped>

</style>
```

#### 子

- 不加任何限制的

```vue
<script setup lang="ts">

/*定义接受数据的名称，没任何限制*/
defineProps(['people', 'pageSize', 'flag', 'gender']);
</script>

<template>
  <div>儿子{{ people }}{{ pageSize }}{{ flag }}{{ gender }}</div>
</template>

<style scoped>

</style>
```

- 定义接受数据类型

```vue
<script setup lang="ts">

import type {People} from "@/entity/entity.ts";

/*定义接受数据的类型*/
defineProps<{ people: People, pageSize: number, flag: boolean, gender: string }>();
</script>

<template>
  <div>儿子{{ people }}{{ pageSize }}{{ flag }}{{ gender }}</div>
</template>

<style scoped>

</style>
```

- 定义数据类型，是否比选，默认值

```vue
<script setup lang="ts">
import type {People} from "@/entity/entity.ts";

/*定义接收的数据类型*/
withDefaults(defineProps<{
      people?: People,
      pageSize?: number,
      flag: boolean,
      gender?: string
    }>(),
    {
      people: () => {
        return {
          id: 999,
          name: 'default',
          address: 'default',
        }
      },
      pageSize: 100,
      gender: "default"
    })
</script>

<template>
  <div>儿子{{ people }}{{ pageSize }}{{ flag }}{{ gender }}</div>
</template>

<style scoped>

</style>
```

### 子-->父

- 子传递数据给父，属性值必须是函数

#### 子

```vue
<script setup lang="ts">

import type {People} from "@/entity/entity.ts";

let people: People = {
  id: 999,
  name: 'Lucy',
  address: 'Beijing'
}

defineProps<
    /*定义数据类型为函数*/
    {
      sendPeople: (name: string, people: People) => void
    }
>();

</script>

<template>
  <!--调用方式一-->
  <button @click="sendPeople('张三',people)">调用</button>
</template>

<style scoped>

</style>
```

#### 父

```vue
<script setup lang="ts">
import Son from "@/components/Son.vue";
import type {People} from "@/entity/entity.ts";

function getPeople(val: string, people: People) {
  console.log('父接受到数据了');
  console.log(val);
  console.log(people);
}
</script>

<template>
  <Son :sendPeople="getPeople"/>
</template>

<style scoped>

</style>
```

## 自定义事件

- 典型的子传递父

### 子

```vue
<script setup lang="ts">

import type {Dog, People} from "@/entity/entity.ts";
import {onMounted} from "vue";

let people: People = {
  id: 999,
  name: 'Lucy',
  address: 'Beijing'
}

let dog: Dog = {
  brand: '博美',
  address: '南京'
}

let gender = 'male';

/*自定义多个事件
* e:事件名字
* 其他参数：参数类型*/
const emit = defineEmits<{
  (e: 'send-people', gender: string, people: People): void
  (e: 'send-dog', dog: Dog): void
}>();

/*触发方式一：组件挂载后触发一次*/
onMounted(() => {
  setTimeout(() => {
    emit("send-people", gender, people);
  }, 3000);
})

</script>

<template>
  <!--触发方式二:点击调用一-->
  <button @click="emit('send-dog',dog)">子组件按钮</button>
</template>

<style scoped>

</style>
```

### 父

```vue
<script setup lang="ts">
import Son from "@/components/Son.vue";
import type {Dog, People} from "@/entity/entity.ts";

function getPeople(gender: string, people: People) {
  console.log('被触发了', gender, people);
}

function getDog(dog: Dog) {
  console.log(dog)
}

</script>

<template>
  <Son @send-people="getPeople" @send-dog="getDog"/>
</template>

<style scoped>

</style>
```

## MITT

- 第三方的，用于所有组件互相通信的一种事件机制

```bash
npm i mitt
```

### 组件声明

```ts
import mitt from 'mitt';

let emitter = mitt();

export default emitter;
```

### 事件发布方

```vue
<script setup lang="ts">
import emitter from "@/util/emitter.ts";
import type {People} from "@/entity/entity.ts";

let people: People = {
  id: 999,
  name: 'Lucy',
  address: 'Beijing'
}

function triggerEvent(people: People) {
  /*触发事件
  * 1. 参数一：事件名
  * 2. 参数二：事件参数*/
  emitter.emit('people-event', people);
}

</script>

<template>
  <button @click="triggerEvent(people)">发送数据</button>
</template>

<style scoped>

</style>
```

### 事件接收方

```vue
<script setup lang="ts">

import emitter from "@/util/emitter.ts";
import {onUnmounted} from "vue";

/*订阅事件*/
emitter.on('people-event', (value) => {
  console.log('接受到的事件：', value)
})

/*页面卸载时，最好能取消当前订阅*/
onUnmounted(() => {
  emitter.off('people-event')
})

</script>

<template>

</template>

<style scoped>

</style>
```

## provide-inject

- 父组件，跨层给后代所有组件传递参数
- 父组件声明哪些参数需要传递，后代组件各取所需

### 父

```vue
<script setup lang="ts">
import Son from "@/components/Son.vue";
import type {People} from "@/entity/entity.ts";
import {provide, ref} from "vue";

/*响应式数据*/
let people = ref<People>({
  id: 123,
  name: 'Father',
  address: '南京'
});

let gender = ref<string>('男');

function updateGender(): void {
  gender.value += '~';
}

provide<People>('k-people', people);
provide<string>('k-gender', gender);

</script>

<template>
  <div>父组件{{ people }}{{ gender }}</div>
  <button @click="updateGender">父组件更新</button>
  <Son/>
</template>

<style scoped>

</style>
```

### 子

```vue
<script setup lang="ts">
import {inject} from "vue";
import type {People} from "@/entity/entity.ts";

let people = inject<People>('k-people');
let gender = inject<string>('k-gender');
</script>

<template>
  <div>子组件{{ people.address }}{{ gender }}</div>
</template>

<style scoped>

</style>
```

## Slot

### 默认slot

- 声明子组件时，可以使用双标签，并在标签中携带内容

#### 父

```vue
<script setup lang="ts">
import Son from "@/components/Son.vue";
</script>

<template>
  <!--son中的内容，就是slot-->
  <Son>
    <h2>哈哈哈哈哈哈</h2>
  </Son>
</template>

<style scoped>

</style>
```

#### 子

- 通过插槽进行占位

```vue
<script setup lang="ts">

</script>

<!--渲染子组件时，可以通过slot获取到其中的内容，并且可以定义默认值-->
<template>
  <div>start</div>
  <slot>默认值</slot>
  <div>end</div>
</template>

<style scoped>

</style>
```

### 具名slot

- 最好可以借助到template

#### 父

```vue
<script setup lang="ts">
import Son from "@/components/Son.vue";
</script>

<template>
  <Son>
    <template v-slot:s2>
      <h2>哈哈哈哈哈哈哈</h2>
    </template>

    <!--语法糖-->
    <template #s1>
      <h2>嘿嘿嘿嘿嘿嘿嘿嘿</h2>
    </template>
  </Son>
</template>
```

#### 子

```vue
<script setup lang="ts">

</script>

<template>
  <slot name="s1">默认值----s1</slot>
  <div>start</div>
  <slot name="s2">默认值----s2</slot>
  <div>end</div>
</template>
```

### 作用域slot

- 数据在子组件中，但是数据的渲染方式却是需要父来决定

#### 子

```vue
<script setup lang="ts">

import {ref} from "vue";

let address = ref<string>('ERICK');
let age = ref<number>(0);

</script>

<template>
  <!--将子组件的数据传输到父组件中-->
  <slot :addressParam=address :ageParam=age></slot>
  <div>start</div>
  <div>end</div>
</template>
```

#### 父

```vue
<script setup lang="ts">
import Son from "@/components/Son.vue";
</script>

<template>
  <!--父组件通过 v-slot拿到对应的子组件的数据，进行个性化渲染-->
  <Son>
    <template v-slot="erickParams">
      <h1>{{ erickParams.ageParam }}{{ erickParams.addressParam }}</h1>
    </template>
  </Son>

  <Son>
    <template v-slot="erickParams">
      <h3>{{ erickParams.ageParam }}{{ erickParams.addressParam }}</h3>
    </template>
  </Son>

</template>
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

# Pinia

- 集中式状态管理： 状态管理的工具，对应Vue2之前的Vuex
- [官网](https://pinia.web3doc.top/)
- 在创建项目的时候就指定需要使用

```bash
 "pinia": "^3.0.1",
```

## 基本结构

### main.ts

```ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)

/*引入了pinia*/
app.use(createPinia())

app.mount('#app')
```

![image-20250224203033209](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250224203033209.png)

### counter.ts

```ts
import {ref} from 'vue'
import {defineStore} from 'pinia'
import type {People} from "@/entity/entity.ts";

export const useCounterStore = defineStore('counter', () => {

    /*state属性*/
    let sum = ref<number>(666);

    let people = ref<People>({
        id: 1,
        name: '张三',
        address: '西安'
    })

    return {sum, people}
})
```

## 读取数据

- 可以有两种方式进行读取数据

```vue
<script setup lang="ts">

import {useCounterStore} from "@/stores/counter.ts";

/*获取到对应的状态
* 1. 一个reactive包裹的对象*/
let counterStore = useCounterStore();

/* 2. 获取数据的方式*/
/*2.1: 直接获取*/
console.log("read-data-001", counterStore.sum, counterStore.people);

/*2.2：从$state中去解析*/
console.log("read-data-002", counterStore.$state.sum, counterStore.$state.people);

</script>

<template>
  <div>{{ counterStore.sum }}</div>
</template>

<style scoped>
</style>
```

## 修改数据

### 直接修改

```vue
<script setup lang="ts">

import {useCounterStore} from "@/stores/counter.ts";

let counterStore = useCounterStore();


/*方式一：就会对对应的count的状态进行两次修改：就是两次set*/
function firstChange() {
  counterStore.sum += 1;
  counterStore.people.address += '~';
}

/*方式二：*/
function secondChange() {
  counterStore.$patch({
    sum: counterStore.sum + 1,
    people: {
      address: counterStore.people.address + '~',
    }
  })
}

</script>

<template>
  <div>{{ counterStore.sum }}</div>
  <div>{{ counterStore.people }}</div>
  <button @click="firstChange">方式一</button>
  <button @click="secondChange">方式二</button>
</template>

<style scoped>
</style>
```

### action

```ts
import {ref} from 'vue'
import {defineStore} from 'pinia'
import type {People} from "@/entity/entity.ts";

export const useCounterStore = defineStore('counter', () => {

    /*state属性*/
    let sum = ref<number>(666);

    let people = ref<People>({
        id: 1,
        name: '张三',
        address: '西安'
    })

    /*action方法*/
    function changeData() {
        sum.value += 1;
        people.value.address += '~';
    }

    return {sum, people, changeData}
})
```

```vue
<script setup lang="ts">

import {useCounterStore} from "@/stores/counter.ts";

let counterStore = useCounterStore();

function update() {
  counterStore.changeData();
}

</script>

<template>
  <div>{{ counterStore.sum }}</div>
  <div>{{ counterStore.people }}</div>
  <button @click="update">方式三</button>
</template>

<style scoped>
</style>
```



# 路由

- 创建项的时候，就自动把路由选项勾选上

## 基本切换

- 路由组件通常保存在pages或views文件夹，一般组件通常放在components文件夹
- 通过点击导航，消失了的组件，默认是被销毁的，需要的时候再去挂载

![image-20250224144652835](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250224144652835.png)

### index.ts

```ts
import {createRouter, createWebHistory} from 'vue-router'
import HomeView from '../views/HomeView.vue'
import AboutView from "@/views/AboutView.vue";
import WelcomeView from "@/views/WelcomeView.vue";

/*创建路透*/
const router = createRouter({
    history: createWebHistory(),
    routes: [
        {
            path: '/home',
            component: HomeView,
        },
        {
            path: '/welcome',
            /*name：可以通过name跳转*/
            name: 'erickWelcome',
            component: WelcomeView
        },
        {
            path: '/about',
            component: AboutView
        },
        // 再定一个一个可以重定向的
    ],
})

/*导出路由*/
export default router
```

### main.ts

```ts
import {createApp} from 'vue'
import App from './App.vue'
import router from './router'

const app = createApp(App)

/*挂载路由*/
app.use(router)

app.mount('#app')
```

### 三个路由组件

```vue
<template>
  <div>About Page</div>
</template>
```

```vue
<template>
  <div>Home Page</div>
</template>
```

```vue
<template>
  <div>我是欢迎页面</div>
</template>
```

### App

```vue
<script setup lang="ts">
</script>

<template>

  <div>我是主页面</div>
  <div>
    <h2>导航栏目</h2>
    <div>
      <!--to: 三种写法-->
      <RouterLink to="/home">Home</RouterLink>
      <RouterLink :to="{path:'/about'}">About</RouterLink>
      <RouterLink :to="{name: 'erickWelcome'}">Welcome</RouterLink>
    </div>
  </div>
  <!-- 占位符，路由的页面在什么地方引入 -->
  <RouterView/>
</template>

<style scoped>
</style>
```

## 嵌套路由

```ts
import {createRouter, createWebHistory} from 'vue-router'
import HomeView from '../views/home/HomeView.vue'
import Erick from "@/views/home/Erick.vue";
import Lucy from "@/views/home/Lucy.vue";

const router = createRouter({
    history: createWebHistory(),
    routes: [
        {
            path: '/home',
            component: HomeView,
            children: [
                {
                    path: 'lucy',
                    component: Lucy,
                },
                {
                    path: 'erick',
                    component: Erick,
                }
            ]
        }
    ],
})

export default router
```

```vue
<template>
  <div>Home Page</div>
  <RouterLink to="/home/erick">Erick</RouterLink>
  <RouterLink to="/home/lucy">Lucy</RouterLink>
  <RouterView/>
</template>
<script setup lang="ts">
</script>
```

## 路由传参

### query

```bash
# 路由地址： 
http://localhost:5173/home/lucy?name=lucy&gender=female
```

```vue
<template>
  <div>Home Page</div>

  <RouterLink :to="{
    path: '/home/erick',
    query: {
      name: erick.name,
      gender: erick.gender,
    }
  }">Erick
  </RouterLink>

  <RouterLink :to="{
    path: '/home/lucy',
    query: {
      name: lucy.name,
      gender: lucy.gender,
    }
  }">Lucy
  </RouterLink>

  <RouterView/>

</template>
<script setup lang="ts">
let erick = {
  name: 'erick',
  gender: 'male',
}

let lucy = {
  name: 'lucy',
  gender: 'female',
}
</script>
```

```vue
<template>
  <div>我是舒展的家</div>
  <div>{{route.query.name}}</div>
  <div>{{route.query.gender}}</div>
</template>

<script setup lang="ts">

import {useRoute} from "vue-router";

let route = useRoute();
</script>
```

### params

- 需要在路由中提前预定义好占位符
- 路由匹配只能使用name，不能使用path

```bash
url
http://localhost:5173/home/lucy/lucy/female
```

```ts
import {createRouter, createWebHistory} from 'vue-router'
import HomeView from '../views/home/HomeView.vue'
import Erick from "@/views/home/Erick.vue";
import Lucy from "@/views/home/Lucy.vue";

const router = createRouter({
    history: createWebHistory(),
    routes: [
        {
            path: '/home',
            component: HomeView,
            children: [
                {
                    /*只能根据name匹配*/
                    name: 'mylucy',
                    /*需要提前占位*/
                    path: 'lucy/:name/:gender',
                    component: Lucy,
                },
                {
                    name: 'myerick',
                    path: 'erick/:name/:gender',
                    component: Erick,
                }
            ]
        }
    ],
})

export default router
```

```vue
<template>
  <div>Home Page</div>

  <RouterLink :to="{
   name: 'myerick',
   params: {
     name: erick.name,
     gender: erick.gender,
   }
  }">Erick
  </RouterLink>

  <RouterLink :to="{
   name:'mylucy',
   params: {
     name: lucy.name,
     gender: lucy.gender,
   }
  }">Lucy
  </RouterLink>
  <RouterView/>

</template>
<script setup lang="ts">
let erick = {
  name: 'erick',
  gender: 'male',
}

let lucy = {
  name: 'lucy',
  gender: 'female',
}
</script>
```

```vue
<template>
  <div>我是舒展的家</div>
  <div>{{route.params.name}}</div>
  <div>{{ route.params.gender}}</div>
</template>

<script setup lang="ts">

import {useRoute} from "vue-router";

let route = useRoute();
</script>
```

## 编程式

```vue
<script setup lang="ts">

import {useRouter} from "vue-router";

let router = useRouter();

function gotoNew() {
  router.push("/home");
}
</script>

<template>
  <div>我是主页面</div>
  <div>
    <RouterLink to="/home">Home</RouterLink>
  </div>

  <button @click="gotoNew">点击跳转</button>
  <!-- 占位符，路由的页面在什么地方引入 -->
  <RouterView/>
</template>

<style scoped>
</style>
```

