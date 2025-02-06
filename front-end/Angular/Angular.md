# Startup

[Angular Cli依赖](https://www.npmjs.com/package/@angular/cli?activeTab=versions)

- Angular Cli的版本，决定了通过该脚手架搭建的项目的Angularx版本

```bash
# 1. 安装Angular CLI
sudo npm install -g @angular/cli@19.0.0

# 2. 创建项目: 不要用sudo来创建项目，不然文件就只是可读
# 创建好之后，本地依赖安装完成，删除.vscode文件
ng new erick-mall

# 3. 进入项目
cd erick-mall

# 4. 安装开发者工具
npm install @angular-devkit/build-angular --save-dev 

# 5. 开启热更新启动项目
ng serve --open
ng serve 
npm start
```

# Signal

## 可读可写

```ts
import {signal, WritableSignal} from '@angular/core';

/*可写可读 : WritableSignal<T>*/
export const age: WritableSignal<number> = signal(0);

/*1.读：调用 count()是获取对应的值*/
export const readeAge = () => {
  console.log("count-signal", age());
}

/*2.写：通过set方法*/
export const writeAgeBySet = () => {
  age.set(10);
}

/*3.写：通过update方法*/
export const writeAgeByUpdate = () => {
  age.update((originalValue) => {
    return originalValue + 1;
  })
}
```

## Computed

- 带缓存的，直到下一次依赖的signal变化后，才会重新计算
- 适合用来处理复杂的数据，起到懒加载的效果
- 只读属性
- 如果依赖多个可读可写的signal，则对应的依赖，可能是动态变化的

```ts
import {computed, Signal} from '@angular/core';
import {age} from './globalSignal';

/*依赖于其他属性
* age变化的时候，erickAge才会变化*/
export const erickAge: Signal<number> = computed(() => {
  return age() * 2;
})

/*读取*/
export const readErickAge = () => {
  console.log('computed,', erickAge());
}
```

## Equallity

- 默认基础类型比较值，引用类型比较引用

```ts
/*自定义是否变化的标准：即使后面signal变化了，也不会触发*/
export const fruits = signal(['apple', 'peach'], {
  equal: (a, b) => {
    return a.length === b.length;
  }
});
```

# Component

## 创建组件

```bash
# ng generate component home
# ng g component home
```

```ts
import {Component} from '@angular/core';

@Component({
  selector: 'erick-cart', /*在html中，可以用该名称来实例化该组件*/
  imports: [], /* 在该component对应的html中，使用到其他的组件的时候需要导入*/
  templateUrl: './cart.component.html', /*对应的html，两种方式: templateUrl或者template*/
  standalone: true, /*true代表其他组件可以直接在imports中导入*/
  styleUrl: './cart.component.css', /*对应的cs,styleUrl或者style，默认组件的样式仅影响该组件模版定义的元素*/
})
export class CartComponent {

}
```

- 一个独立组件，是在组件元数据中设置 standalone: true, 官方推荐，新开发的项目都使用独立组件
- true：表示在其他组件中使用该component中时，可以直接在目标component中的imports中导入
- 会为每个遇到的组件创建组件一个实例对象

## 输入属性-input

- 将特定属性标记为可绑定，该带有输入属性的组件，可以从其他使用该组件的html中进行获取到对应的值
- 一般用来封装公共组件

### 公共组件

```ts
import {Component, input} from '@angular/core';
import {JsonPipe} from '@angular/common';

@Component({
  selector: 'app-common-table',
  imports: [
    JsonPipe
  ],
  templateUrl: './common-table.component.html',
  standalone: true,
  styleUrl: './common-table.component.css'
})
/* input: 1. 大小写敏感
*         2. 可以被子类继承
*         3. 类型为InputSignal, 读取时候要用 tableName()*/

export class CommonTableComponent {
  /*1. 输入的值： 默认值为对应类型的 空*/
  tableName = input<string>();

  /*2. 输入的值：带默认值*/
  columnName = input<string>('email');

  /*number类型*/
  columnWidth = input<number>(10);

  /*boolean类型*/
  flag = input<boolean>(false);

  /*自定义类型*/
  person = input<Person>({firstName: '', secondName: 'e', address: 'e'});

  /*2. 输入的别名*/
  tablePurpose = input<string>('', {alias: 'myTablePurpose'});

  /*3. transform
  * 范型一：原来的值
  * 范型二：返回的值*/
  brand = input<string, string>("cart", {transform: toUpperCase});

  /*4. required:强制必须输入*/
  price = input.required<number>();
}

function toUpperCase(title: string): string {
  return title.toUpperCase();
}

export type Person = {
  firstName: string,
  secondName: string,
  address: string,
}
```

```html
<!--读取时候：要带()-->
<p>{{ tableName() }}</p>
<p>{{ columnName() }}</p>
<p>{{ columnWidth() }}</p>
<p>{{ flag() }}</p>
<p>{{ person() | json }}</p>
<p>{{ tablePurpose() }}</p>
<p>{{ brand() }}</p>
<p>{{ price() }}</p>
```

### 使用方

```html
<!--address：直接传递一个static的值
   [value]: 可以传递一个该父组件的值：
       static值：
       address：只适合于string
       [address]: 其他所有类型必须都用这种方式
   -->
<app-common-table tableName="cart-table"
                  columnName="address"
                  [columnWidth]="12"
                  [flag]="true"
                  [person]="{firstName: '舒', secondName: '展', address: '西安'}"
                  myTablePurpose="cart"
                  brand="xiaomi"
                  [price]="20">
</app-common-table>
```

## 输出属性-output

### 公共组件

```ts
import {Component, output} from '@angular/core';

@Component({
  selector: 'app-common-box',
  imports: [],
  templateUrl: './common-box.component.html',
  standalone: true,
  styleUrl: './common-box.component.css'
})
export class CommonBoxComponent {

  /*基本数据类型*/
  boxSize = output<string>();

  /*引用数据类型*/
  dog = output<Dog>();

  /*方法调用*/
  boxClosed = output<void>();
  
  /*子组件中：对应的点击时间，触发对应的event*/
  boxSizeChange = (size: string) => {
    this.boxSize.emit(size);
  }

  dogChange = (dog: Dog): void => {
    this.dog.emit(dog);
  }

  boxClosedChange = (): void => {
    console.log("子方法被调用了")
    this.boxClosed.emit();
  }
}

export type Dog = {
  brand: string,
  size: string
}
```

```html
<button (click)="boxSizeChange('250')">子基本数据</button><br/>
<button (click)="dogChange({brand:'china',size:'10'})">子引用数据</button><br/>
<button (click)="boxClosedChange()">子方法调用</button><br/>
```

### 使用方

```html
<!--左边：公共组件中的event名字
    右边：使用方如何处理冒泡出来的数据-->
<app-common-box (boxSize)="homeBoxSize($event)"
                (dog)="homeDog($event)"
                (boxClosed)="homeClosed()"></app-common-box>

```

```ts
import {Component} from '@angular/core';
import {CommonBoxComponent, Dog} from '../common-box/common-box.component';

@Component({
  selector: 'app-home',
  imports: [
    CommonBoxComponent
  ],
  templateUrl: './home.component.html',
  standalone: true,
  styleUrl: './home.component.css'
})
export class HomeComponent {

  homeBoxSize = (event: string) => {
    console.log(event);
  }

  homeDog = (event: Dog) => {
    console.log(event);
  }

  homeClosed = () => {
  }
}
```

## Reference Query

- 介于不同的component之间
- 包含View Query和Content Query两种方式，区别在于在html中定义的方式稍微不同

### View Query

#### 公共组件

```ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-header',
  imports: [],
  templateUrl: './header.component.html',
  standalone: true,
  styleUrl: './header.component.css'
})
export class HeaderComponent {
  headerTitle: string = 'I am header';
}
```

#### 使用方

```html
<!--必须在这里引用-->
<app-header></app-header>
<button (click)="check()">打印</button>
```

```ts
import {Component, viewChild} from '@angular/core';
import {HeaderComponent} from '../header/header.component';

@Component({
  selector: 'app-page',
  imports: [
    HeaderComponent
  ],
  templateUrl: './page.component.html',
  standalone: true,
  styleUrl: './page.component.css'
})
export class PageComponent {
  /*获取到其对应的引用*/
  header = viewChild(HeaderComponent);

  check = () => {
    console.log(this.header()?.headerTitle);
  }
}
```

### Content Query

#### 第三方

```html
<!--引用方式：放在插槽里面-->
<app-page>
  <app-header></app-header>
</app-page>
```

#### 公共组件

```ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-header',
  imports: [],
  templateUrl: './header.component.html',
  standalone: true,
  styleUrl: './header.component.css'
})
export class HeaderComponent {
  headerTitle: string = 'I am header';
}
```

#### 使用方

```ts
import {Component, contentChild, Signal} from '@angular/core';
import {HeaderComponent} from '../header/header.component';

@Component({
  selector: 'app-page',
  imports: [
    HeaderComponent
  ],
  templateUrl: './page.component.html',
  standalone: true,
  styleUrl: './page.component.css'
})
export class PageComponent {
  /*获取到其对应的引用*/
  header: Signal<HeaderComponent | undefined> = contentChild(HeaderComponent);

  check = () => {
    console.log(this.header()?.headerTitle);
  }
}
```

```html
<button (click)="check()">打印</button>
```

### 特性

#### Required

- 某些时候，知道肯定某个引用不是undefined的，则可以使用required属性
- 适用于viewChild和contentChild

```ts
import {Component, contentChild, Signal} from '@angular/core';
import {HeaderComponent} from '../header/header.component';

@Component({
  selector: 'app-page',
  imports: [
    HeaderComponent
  ],
  templateUrl: './page.component.html',
  standalone: true,
  styleUrl: './page.component.css'
})
export class PageComponent {
  /*获取到其对应的引用*/
  header: Signal<HeaderComponent> = contentChild.required(HeaderComponent);

  check = () => {
    console.log(this.header().headerTitle);
  }
}
```

#### viewChildren + contentChildren

- 得到的是一个数组的signal，用法和上面差不多

### 变量名获取

```html
<app-header #firstHeader></app-header>
<app-header #secondHeader></app-header>
<button (click)="check()">打印</button>
```

```ts
import {Component, viewChild} from '@angular/core';
import {HeaderComponent} from '../header/header.component';

@Component({
  selector: 'app-page',
  imports: [
    HeaderComponent
  ],
  templateUrl: './page.component.html',
  standalone: true,
  styleUrl: './page.component.css'
})
export class PageComponent {
  /*获取到其对应的引用*/
  firstHeader = viewChild.required<HeaderComponent>('firstHeader');
  secondHeader = viewChild.required<HeaderComponent>('secondHeader');

  check = () => {
    console.log(this.firstHeader().headerTitle);
    console.log(this.secondHeader().headerTitle);
  }
}
```

# Template

## 文本/HTML属性

```ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-erick',
  imports: [],
  templateUrl: './erick.component.html',
  standalone: true,
  styleUrl: './erick.component.css'
})
export class ErickComponent {
  /*text数据的双向绑定*/
  title: string = 'Erick';
  age: number = 10;

  /*properties绑定*/
  isButtonDisabled: boolean = false;

  change = () => {
    this.title = 'Lucy';
    this.age++;
    this.isButtonDisabled = !this.isButtonDisabled;
  }
}
```

```html
<!--1.文本绑定-->
{{ title }}
{{ age }}

<!--2. 用 [] :
     2.1 html原生properties绑定-->
<button [disabled]="isButtonDisabled">测试button</button> <br/>

<button (click)="change()">改变</button>
```

## 事件监听

```html
<!--1. 原生事件监听： 输入按键抬起时调用-->
<input type="text" (keyup)="updateFirstField()"><br>

<!--2. 获取到事件本身: $event-->
<input type="text" (keyup)="updateSecondField($event)"><br>

<!--3. 直接根据key本身-->
<input type="text" (keyup.enter)="updateThirdField($event)"><br>
```

```ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-erick',
  imports: [],
  templateUrl: './erick.component.html',
  standalone: true,
  styleUrl: './erick.component.css'
})
export class ErickComponent {

  updateFirstField = () => {
    console.log('updateField');
  }

  updateSecondField = (event: KeyboardEvent) => {
    console.log(event);
    if (event.key === 'Enter') {
      console.log('enter');
    }
  }

  updateThirdField = (event: KeyboardEvent) => {
    console.log(event);
  }
}
```

## 3. 双向绑定

### 3.1 表单元素-[(ngModel)]

```ts
import {Component} from '@angular/core';
/*会跟着[(ngModel)] 导入*/
import {FormsModule} from '@angular/forms';

@Component({
  selector: 'app-erick',
  imports: [
    FormsModule
  ],
  templateUrl: './erick.component.html',
  standalone: true,
  styleUrl: './erick.component.css'
})
export class ErickComponent {

  address: string = '';

  /*清空数据的时候，input框也会被清空*/
  reset = ()=>{
    this.address = '';
  }
}
```

```html
<input type="text" [(ngModel)]="address"><br>

{{ address }}
<button (click)="reset()">清空</button>
```

### 3.2 父子通信

- 父子共享数据，在父组件和子组件中同步更新值
- 父为子提供初始值，子组件修改数据，父组件能及时的获取到更新后的最新值

#### 子

```html
<h2>son === {{money}}</h2>
<button (click)="updateMoney()">花钱</button>
```

```ts
import {Component, EventEmitter, Input, Output} from '@angular/core';

@Component({
  selector: 'app-son',
  imports: [],
  templateUrl: './son.component.html',
  standalone: true,
  styleUrl: './son.component.css'
})
export class SonComponent {

  /*接收负组件传值*/
  @Input() money: number = 0;

  /*子组件改变该值后向父组件传值*/
  @Output() moneyChange = new EventEmitter<number>();

  updateMoney(): void {
    /*改变值*/
    this.money--;
    /*传递值*/
    this.moneyChange.emit(this.money);
  }
}
```

#### 父

```html
<!--缩写形式-->
<h1> father === {{ initialMoney }}</h1>
<app-son [(money)]="initialMoney"></app-son>
```

```html
<!--展开形式-->
<h1> father === {{ initialMoney }}</h1>
<!--函数接收： initialMoney=$event-->
<app-son [money]="initialMoney" (moneyChange)="initialMoney=$event"></app-son>
```

```ts
import {Component} from '@angular/core';
import {SonComponent} from '../son/son.component';

@Component({
  selector: 'app-father',
  imports: [
    /*在html中使用了子组件，就会在这里import*/
    SonComponent
  ],
  templateUrl: './father.component.html',
  standalone: true,
  styleUrl: './father.component.css'
})
export class FatherComponent {
  initialMoney: number = 20;
}
```

## 4. 流程控制

- 展示，隐藏，重复指定的html元素

### 4.1 @If

```html
<!--1.基本流程控制-->
@if (firstNumber > secondNumber) {
  <h2>{{ firstNumber }} is greater {{ secondNumber }}</h2>
} @else if (firstNumber < secondNumber) {
  <h1>{{ firstNumber }} is less than {{ secondNumber }}</h1>
} @else {
  {{ firstNumber }} equals {{ secondNumber }}
}

<!--2.变量取别名：如果不为空，则展示-->
@if (address; as addressDisplay) {
  {{ addressDisplay }}
}
```

```ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-erick',
  imports: [],
  templateUrl: './erick.component.html',
  standalone: true,
  styleUrl: './erick.component.css'
})
export class ErickComponent {
  firstNumber: number = 10;
  secondNumber: number = 12;

  address: string = "beijing";
}
```

### 4.2 @for

```html
<!--需要id作为key，提升性能-->
@for (item of fruits; track item.id) {
  <h2>{{ item.name }}</h2>
} @empty {
  <!--如果为空，则渲染下面的-->
  <h3>no data</h3>
}
```

```ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-erick',
  imports: [],
  templateUrl: './erick.component.html',
  standalone: true,
  styleUrl: './erick.component.css'
})
export class ErickComponent {
  fruits: Fruit[] = [{name: 'apple', id: 1}, {name: 'peach', id: 2}, {name: 'lemon', id: 3}]
}

export type Fruit = {
  name: string,
  id: number,
}
```

### 4.3 @switch

```html
@switch (address) {
  @case ('beijing') {
    <h2>北京</h2>
  }
  @case ('xian') {
    <h2>西安</h2>
  }
  @default {
    <h2>默认地区</h2>
  }
}
```

```ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-erick',
  imports: [],
  templateUrl: './erick.component.html',
  standalone: true,
  styleUrl: './erick.component.css'
})
export class ErickComponent {
  address: string = 'beijing';
}
```

## 5. Pipes

- 用 ｜ 连接

### 5.1 built-in

```html
<!--首字母转大写-->
<h3>{{ company | titlecase }}</h3>
<!--转大写-->
<h3>{{ company | uppercase }}</h3>
<!--转小写-->
<h3>{{ company | lowercase }}</h3>
<!-- 转换为货币-->
<h3>{{ price | currency }}</h3>
<!-- 转换为日期-->
<h3>{{ createTime | date }}</h3>
<!--可多个pipe-->
<h3>{{ createTime | date | uppercase}}</h3>
<!--object转json-->
<h3>{{ fruits | json }}</h3>
```

```ts
import {Component} from '@angular/core';
import {CurrencyPipe, DatePipe, JsonPipe, LowerCasePipe, TitleCasePipe, UpperCasePipe} from '@angular/common';

@Component({
  selector: 'app-erick',
  imports: [
    /*组件导入*/
    TitleCasePipe,
    CurrencyPipe,
    DatePipe,
    UpperCasePipe,
    LowerCasePipe,
    JsonPipe
  ],
  templateUrl: './erick.component.html',
  standalone: true,
  styleUrl: './erick.component.css'
})
export class ErickComponent {
  company: string = "nike";
  price: number = 17.2;
  createTime: string = '2024-09-01';

  fruits = {
    name: 'apple',
    brand: 'china',
  }
}
```

### 5.2 自定义

- 对应的数据发生变化的时候，可以检测到
- 一般只用在基本数据类型中，又称为pure的pipe

#### 定义pipe

```ts
import {Pipe, PipeTransform} from '@angular/core';

@Pipe({
  /*名字：其他地方使用的时候用，驼峰命名*/
  name: 'peopleTransformer',
  standalone: true,
})
export class ErickPipe implements PipeTransform {
  /*1. value：可以自定义类型
  * 2. 传递参数*/
  transform(value: string, prefix: string, suffix: string): string {
    return prefix + value + suffix;
  }
}
```

#### 使用pipe

```html
<!--传递多个参数-->
<h2>{{ name | peopleTransformer:'PREFIX':'SUFFIX' | uppercase }}</h2>
<button (click)="name='nancy'">修改</button>
```

```ts
import {Component} from '@angular/core';
import {ErickPipe} from '../pipes/customizedPipes';
import {JsonPipe, UpperCasePipe} from '@angular/common';

@Component({
  selector: 'app-erick',
  imports: [
    ErickPipe,
    JsonPipe,
    UpperCasePipe
  ],
  templateUrl: './erick.component.html',
  standalone: true,
  styleUrl: './erick.component.css'
})
export class ErickComponent {
  name: string = 'erick';
}
```

## ng-content

- 作用域插槽
- 在一个html中，可以定义一个ng-content

### 单独插槽

- 在运行时，不能对ng-content的东西进行写操作，也不能进行条件渲染操作

#### 插槽定义

```html
<h2>顶部导航栏</h2>
<ng-content/><!--placeholder-->
<h3>底部导航栏</h3>
```

```ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-erick',
  imports: [],
  templateUrl: './erick.component.html',
  standalone: true,
  styleUrl: './erick.component.css'
})
export class ErickComponent {
}
```

#### 插槽使用

```html
<div>
  <!--回自动用中间的东西，替换掉ng-content中的东西-->
  <app-erick>
    <h1>我是中间的东西</h1>
  </app-erick>
</div>
```

### 多插槽

- 基于CSS的选择器

#### 插槽定义

```html
<h2>顶部导航栏</h2>
<!--类选择器-->
<ng-content select=".left-dom"></ng-content>

<!--id选择器-->
<ng-content select="#middle-dom"></ng-content>

<h3>底部导航栏</h3>
```

```ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-erick',
  imports: [],
  templateUrl: './erick.component.html',
  standalone: true,
  styleUrl: './erick.component.css',
})
export class ErickComponent {
}
```

#### 插槽使用

```html
<div>
  <app-erick>
    <!--类选择器-->
    <div class="left-dom">左边辅导费</div>

    <!--id选择器-->
    <div id="middle-dom">中间发放</div>

  </app-erick>
</div>
```

```ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';
import {ErickComponent} from './erick/erick.component';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet, ErickComponent],
  templateUrl: './app.component.html',
  standalone: true,
  styleUrl: './app.component.css'
})
export class AppComponent {
}
```

## ng-template

### 6.1 声明

- 该标签中的内容不会被渲染到真实的DOM中

```html
<!--1. 只是起到一个声明的作用，并不会将该模块的内容渲染到DOM中
    2. 有该内容对应的component-->
<ng-template>
  <h3>我是一个自定义的模块</h3>
</ng-template>
```

```ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-common-table',
  imports: [],
  templateUrl: './common-table.component.html',
  standalone: true,
  styleUrl: './common-table.component.css'
})
export class CommonTableComponent {
}
```

### 6.2 使用

## ng-container

- 将一组元素包裹起来

### 1. 基本使用

```html
<div>
  <!--1. 将一组标签包裹起来，ng-container不会在UI中显示-->
  <ng-container>
    <h3>hello</h3>
    <input type="text">
  </ng-container>
</div>
```

### 2. structure-directive

```html
<ng-container *ngIf="flag">
  <h2>hello</h2>
  <h3>world</h3>
</ng-container>

<ng-container *ngFor="let item of fruits;">
  <h2>{{item}}</h2>
  <h3>次数</h3>
</ng-container>
```



# 指令

## 1. 内置指令

### 1.1 ngIf

- 用来向宿主元素田间或删除元素
- 为true时候，会加载该组件。为false时，将该组件及其所有子组件从DOM中移除，从而释放内存和资源

#### 组件

```html
<button (click)="changeFlag()">切换显示子组件</button>

<app-child *ngIf="isChildValid"></app-child>
<router-outlet/>
```

```ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';
import {NgIf} from "@angular/common";
import {ChildComponent} from "./child/child.component";

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, NgIf, ChildComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  isChildValid = true;

  changeFlag() {
    this.isChildValid = !this.isChildValid;
  }
}
```

#### HTML元素

```html
<button (click)="changeFlag()">切换显示子组件</button>

<div *ngIf="isValid">你好</div>
<router-outlet/>
```

```ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';
import {NgIf} from "@angular/common";
import {ChildComponent} from "./child/child.component";

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, NgIf, ChildComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  isValid = true;

  changeFlag() {
    this.isValid = !this.isValid;
  }
}
```

### 1.2 ngFor

- 动态生成html的数据

#### 组件

- 可以用来多次渲染某个子组件

```html
<!--将fruit传递给子组件-->
<app-child *ngFor="let fruit of fruits" [fruit]="fruit"></app-child>
<router-outlet/>
```

```ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';
import {NgForOf, NgIf} from "@angular/common";
import {ChildComponent} from "./child/child.component";

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, NgIf, ChildComponent, NgForOf],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  fruits: string[] = ["apple", "peach", "lemon"];
}
```

```ts
import {Component, Input} from '@angular/core';
import {NgForOf} from "@angular/common";

@Component({
  selector: 'app-child',
  standalone: true,
  imports: [
    NgForOf
  ],
  templateUrl: './child.component.html',
  styleUrl: './child.component.css'
})
export class ChildComponent {
  @Input() fruit: string = '';
}
```

#### HTML元素

```html
<!--获取到对应的元素和索引-->
<div *ngFor="let fruit of fruits; let i = index">{{ i }}==={{ fruit }}</div>
<router-outlet/>
```

```ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';
import {NgForOf, NgIf} from "@angular/common";
import {ChildComponent} from "./child/child.component";

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, NgIf, ChildComponent, NgForOf],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  fruits: string[] = ["apple", "peach", "lemon"];
}
```

# DI

## 1. 基本使用

- service：用来执行特定功能的一个class，用@Injectable()来标注
- component：UI对应的组件对应的数据，专注为对应的html服务。可以将具体的任务下放到serivce中，实现代码分离

### service

- 提供特定功能的，需要被注入的类
- service中可以继续去依赖其他的service
- service最好放在单独的包中

```bash
# 创建一个名字为api.service.ts的文件，一般是以service结尾
ng generate service api
ng g service api

# 在/src/app目录下，在对应的cart包中，创建一个service
ng g service cart/phone

# 默认root级别，可以调整
```

```ts
import {Injectable} from '@angular/core';

@Injectable({
  /*整个应用级别的*/
  providedIn: 'root'
})
export class ApiService {

  /*也可以注入其他类型的Service*/
  constructor() {
  }

  /*提供特定功能*/
  say() {
    console.log('hello');
  }
}
```

### component

- 使用定义好的依赖，一般是以component结尾
- component可以注入多个不同的service

```ts
import {Component} from '@angular/core';
import {ApiService} from "../api.service";

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [],
  templateUrl: './app.component.html',
  styleUrl:
    './app.component.css'
})

export class AppComponent {

  /*注入*/
  constructor(private apiService: ApiService) {
  }

  /*使用*/
  use() {
    this.apiService.say();
  }
}
```

### registry

- 连接service的提供方和消费方的桥梁，类似spring中的容器

## 2. 作用域

### Root Level

- service在整个应用中可以直接使用
- 只有一个实例对象，可以用来存储整个project级别的信息

#### 方式1-- tree-shaking

- 在service上标记，在component中直接注入
- 如果项目中没有用到，则不会注入到项目中

```ts
import {Injectable} from '@angular/core';

@Injectable({
  /*整个应用级别的*/
  providedIn: 'root',
})
export class ApiService {

  say() {
    console.log('hello');
  }
}
```

#### 写法2--app.config.ts

- 不在service上标记，在app.config.ts中通过provide识别
- 这种方式注入的，对应的Service不管用不用到，都会在registry中存在

```ts
import {Injectable} from '@angular/core';

@Injectable()
export class ApiService {

  say() {
    console.log('hello');
  }
}
```

```ts
import {ApplicationConfig, provideZoneChangeDetection} from '@angular/core';
import {provideRouter} from '@angular/router';

import {routes} from './app.routes';
import {provideAnimationsAsync} from '@angular/platform-browser/animations/async';
import {ApiService} from './api.service';

export const appConfig: ApplicationConfig = {
  providers: [provideZoneChangeDetection({eventCoalescing: true}), provideRouter(routes), provideAnimationsAsync(),
    /*在appconfig中提供*/
    {provide: ApiService}]
};
```

### Component Level

- 使用时候，每次都是新的实例，相当于spring的多例模式
- 不能保存跨component的信息，因为是多个实例
- 这种方式注入的，对应的Service不管用不用到，都会在registry中存在

```ts
import {Injectable} from '@angular/core';

@Injectable()
export class ApiService {

  /*不能保存跨component的信息*/
  name: string = ''

  constructor() {
  }

  sayHello() {
    console.log("hello")
  }
}
```

```ts
import {Component} from '@angular/core';
import {ApiService} from '../api.service';

@Component({
  selector: 'app-cart',
  imports: [],
  templateUrl: './cart.component.html',
  standalone: true,
  styleUrl: './cart.component.css',
  /*在当前component中注入：必须通过这种方式提供，不提供就会报错
   NullInjectorError： No provider for _ApiService!*/
  providers: [ApiService]
})
export class CartComponent {

  /*public还是private，取决于是否在对应的html中调用service的方法*/
  constructor(public apiService: ApiService) {
  }
}
```

### TIPS

- 不推荐在root级别的service中，注入另外一个component级别的service

## 3. 注入方式

### construcotr

- 最常用

```ts
import {Component} from '@angular/core';
import {ApiService} from '../api.service';

@Component({
  selector: 'app-cart',
  imports: [],
  templateUrl: './cart.component.html',
  standalone: true,
  styleUrl: './cart.component.css',
})
export class CartComponent {
  constructor(public apiService: ApiService) {
  }
}
```

### inject

```ts
import {Component, inject} from '@angular/core';
import {ApiService} from '../api.service';

@Component({
  selector: 'app-cart',
  imports: [],
  templateUrl: './cart.component.html',
  standalone: true,
  styleUrl: './cart.component.css',
})
export class CartComponent {

  /*通过inject注入*/
  public apiService = inject(ApiService);

  constructor() {
  }
}
```

## 4. provide

- 进行DI的时候，注入的方式

### useClass

-  指定注入的Service的类，根据类名，创建一个新的类
- 可以用在UT中

#### 同名实现

```ts
import {Injectable} from '@angular/core';
import {AbstractSearchService} from './abstract.search.service';

@Injectable()
export class HomeSearchService extends AbstractSearchService {

  constructor() {
    super();
  }

  /*对某些方法进行重写*/
  override sayHello() {
    console.log("Home Search Service")
  }
}
```

```ts
import {Component, inject} from '@angular/core';
import {HomeSearchService} from '../home-search.service';

@Component({
  selector: 'app-cart',
  imports: [],
  templateUrl: './cart.component.html',
  standalone: true,
  styleUrl: './cart.component.css',
  /*
  * provide:  需要注入的类的类名
  * useClass: 具体用的类*/
  providers: [
    {
      provide: HomeSearchService,
      useClass: HomeSearchService // 可以省略，默认写法
    }],
})
export class CartComponent {

  /*provide对应的类*/
  public searchService = inject(HomeSearchService);

  constructor() {
  }

  check = () => {
    this.searchService.sayHello();
    this.searchService.work();
  }
}
```

#### 抽象类-接口

```ts
export abstract class AbstractSearchService {
  name: string = '';

  /*抽象方法*/
  abstract sayHello(): void

  /*具体方法*/
  work = () => {
    console.log("Base Work");
  }
}
```

```ts
import {Injectable} from '@angular/core';
import {AbstractSearchService} from './abstract.search.service';

@Injectable()
export class HomeSearchService extends AbstractSearchService {

  constructor() {
    super();
  }

  /*对某些方法进行重写*/
  override sayHello() {
    console.log("Home Search Service")
  }
}
```

```ts
import {Component, inject} from '@angular/core';
import {HomeSearchService} from '../home-search.service';
import {AbstractSearchService} from '../abstract.search.service';

@Component({
  selector: 'app-cart',
  imports: [],
  templateUrl: './cart.component.html',
  standalone: true,
  styleUrl: './cart.component.css',
  /*
  * provide:  注入的类的基类，可以为抽象类，无需@Injectable()修饰
  * useClass: 具体实现类，必须 @Injectable()修饰
  * 一个component中，只能使用一次继承同一个类的实现类*/
  providers: [
    {
      provide: AbstractSearchService,
      useClass: HomeSearchService
    }
  ],
})
export class CartComponent {

  /*provide对应的类：基类*/
  public searchService = inject(AbstractSearchService);

  constructor() {
  }

  check = () => {
    this.searchService.sayHello();
    this.searchService.work();
  }
}
```

### useExisting



### useValue

#### non-class依赖

- 场景一：可以注入interface，等，利用@InjectionToken

```ts
import {InjectionToken} from '@angular/core';

/*要依赖的non-class的类型:  定义一个类*/
export interface UI_CONFIG {
  host: string;
  port: string;
  address: string;
}

/*具体的token定义的值
* 1.不通过DEV_UI_CONFIG.xxx直接引入，而是通过token的方式
* 2.方便测试，不然后续测试不方便*/
export const DEV_UI_CONFIG: UI_CONFIG = {
  host: '192.168.134',
  port: '9090',
  address: 'NEW YORK',
}

export const UAT_UI_CONFIG: UI_CONFIG = {
  host: '9999999',
  port: '8080',
  address: 'BEIJING',
}

/* 变量名：token名
 * 参数一：token的类型
* 参数二：token的描述*/
export const UI_CONFIG_TOKEN = new InjectionToken<UI_CONFIG>('erick_config desc token');
```

```ts
import {Component, Inject} from '@angular/core';
import {UAT_UI_CONFIG, UI_CONFIG, UI_CONFIG_TOKEN} from '../UI_CONFIG';

@Component({
  selector: 'app-cart',
  imports: [],
  templateUrl: './cart.component.html',
  standalone: true,
  styleUrl: './cart.component.css',

  providers: [
    { 
       // 必须是wrapper类型的
      provide: UI_CONFIG_TOKEN,
      useValue: UAT_UI_CONFIG
    }
  ],
})
export class CartComponent {
  host: string = '';
  port: string = '';
  address: string = '';

  /*这种方式的数据不能通过this获取到，需要先在constructor中做转化*/
  constructor(@Inject(UI_CONFIG_TOKEN) config: UI_CONFIG) {
    this.host = config.host;
    this.port = config.port;
    this.address = config.address;
  }

  check = () => {
    console.log(this.host);
    console.log(this.port);
    console.log(this.address);
  }
}
```

#### HTML中使用的变量

- 对于一些单独的类或者变量，在一个独立文件中
- 在HTML中如果要使用这些变量，必须在html中对应的component中，创建一个新的变量来接收
- 使用useValue的方式，可以将这种变量或者类注入进来，并且在html中可以使用

```ts
  /*通过这种方式注入的常量依赖，可以在html中直接使用*/
  constructor(@Inject(GLOBAL_COLUMNS_TOKEN) public globalColumns: ColumnInfo[]) {
  }
```



#### UT中

- 被测试的类，依赖于其他service的类的时候，可以使用jasmi创造的mock的类去替换

## 5. Inject Context

### 构造器+字段

- 可以在@Component或@Injectable修饰的class的，
- 构造器中或者方法中直接注入

```ts
import {Component, inject} from '@angular/core';
import {HomeSearchService} from '../home-search.service';
import {NG_VALUE_ACCESSOR} from '@angular/forms';
import {ApiService} from '../api.service';

@Component({
  selector: 'app-cart',
  imports: [],
  templateUrl: './cart.component.html',
  standalone: true,
  styleUrl: './cart.component.css',

  providers: [
    {
      provide: ApiService,
    },
    {
      provide: HomeSearchService,
      useClass: HomeSearchService
    }
  ],
})
export class CartComponent {

  /*1.在类初始化的时候*/
  private apiService: ApiService = inject(ApiService);

  /*2. 在constructor中注入*/
  constructor(private homeSearchService: HomeSearchService) {
  }

  check = () => {
    this.homeSearchService.work();
    this.apiService.sayHello();
  }
}
```

### Stack-Frame

- 在Angular中，有一些特殊的函数，可以进行DI，比如AuthGurard

```ts
import {CanActivateFn} from '@angular/router';
import {inject} from '@angular/core';
import {UpdateService} from './update.service';

export const basicGuard: CanActivateFn = (route, state) => {
  // 注入，必须是root级别的
  let updateService = inject(UpdateService);
  updateService.update();
  return true;
};
```

# 独立组件

- 通过standalone标志和imports
- 独立组件可以直接导入另一个独立组件

```ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';
import {NgForOf, NgIf} from "@angular/common";
import {ChildComponent} from "./child/child.component";

@Component({
  selector: 'app-root',
  standalone: true,/* 独立组件*/
  imports: [RouterOutlet, NgIf, ChildComponent, NgForOf],/*导入的其他的独立组件*/
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  /*DI注入*/
  constructor(private childComponent: ChildComponent) {
  }

  fruits: string[] = ["apple", "peach", "lemon"];

  say() {
    this.childComponent.say();
  }
}
```

# Lifecycle

### 2.3 ngOnChanges()监听

- Angular的生命周期钩子

####  父

```ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';
import {ChildComponent} from "./child/child.component";
import {NgForOf} from "@angular/common";

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, ChildComponent, NgForOf],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  /*父组件的属性*/
  major = 1;
  minor = 23;

  addMinor = () => {
    this.minor++;
  }

  addMajor = () => {
    this.major++;
  }
}
```

```html
<h2>Parent Source Version</h2>
<button type="button" (click)="addMajor()">Major加</button><br/>
<button type="button" (click)="addMinor()">Minor加</button><br/>

<app-child [major]="major" [minor]="minor"></app-child>
<router-outlet/>
```

#### 子

```ts
import {Component, Input, OnChanges, SimpleChanges} from '@angular/core';
import {NgForOf} from "@angular/common";

@Component({
  selector: 'app-child',
  standalone: true,
  imports: [
    NgForOf
  ],
  templateUrl: './child.component.html',
  styleUrl: './child.component.css'
})
export class ChildComponent implements OnChanges {
  /*接收父组件的传递值*/
  @Input() major = 0;
  @Input() minor = 0;

  /*子组件自己的值*/
  changeLogs: string[] = [];

  /*监听父组件传递过来的值
  * 1. 组件首次加载时，调用一次
  * 2. 后续父组件每次改变任意属性，也会调用一次
  * 3. 改变了几个属性，SimpleChanges就会有几个对象*/
  ngOnChanges(changes: SimpleChanges): void {
    const logs: string[] = [];
    for (const propName in changes) {
      console.log(propName) // 属性名
      const propEntity = changes[propName];
      console.log(propEntity); //属性具体的值： SimpleChange {previousValue: undefined, currentValue: 1, firstChange: true}

      if (propEntity.isFirstChange()) {/*第一次加载时候的*/
        logs.push(`Initial value of ${propName} set to ${propEntity.currentValue}`);
      } else {
        logs.push(`${propName} changed from ${propEntity.previousValue} to ${propEntity.currentValue}`);
      }
    }

    this.changeLogs.push(...logs);
  }
}
```

```html
<h3> Version:{{ major }}.{{ minor }}</h3>
<h4>Change Logs:</h4>
<ul>
  <li *ngFor="let log of changeLogs">{{ log }}</li>
</ul>
```

# 路由

## 基本路由

### 1. 两个路由页面

- 创建两个页面的 component的组件

```html
<p>Hi,I am home</p>
```

### 2. app.routes.ts

- 在该文件中定义路由

```ts
import {Routes} from '@angular/router';
import {FirstComponent} from './first/first.component';
import {SecondComponent} from './second/second.component';

/*定义好两个路由*/
export const routes: Routes = [
  {
    path: 'erick-first', component: FirstComponent
  },
  {
    path: 'erick-second', component: SecondComponent
  }
];
```

### 3. app.config.ts

- 不用做任何变化

```ts
import {ApplicationConfig, provideZoneChangeDetection} from '@angular/core';
import {provideRouter} from '@angular/router';

import {routes} from './app.routes';

/*自动集成了router*/
export const appConfig: ApplicationConfig = {
  /*providerRouter(routes)：导入后续的路由*/
  providers: [provideZoneChangeDetection({eventCoalescing: true}), provideRouter(routes)]
};
```

### 4. app页面

```html
<h1>Common Title</h1>

<!--路由定义的地方：定位到具体元素，存放位置-->
<router-outlet/>
```

### 访问

#### URl访问

- 现在可以通过在浏览器中输入地址，来访问不同的页面

```bash
# http://localhost:4200/
- 显示title

# http://localhost:4200/erick-first
- 显示title和first界面

# http://localhost:4200/erick-second
- 显示title和second界面
```

#### UI元素访问

- 上面的方式，只能让用户在浏览器中的地址栏中手动输入路径才能改变
- 通过在页面创建link，button类似的东西，从而点击后跳转到指定路由

```html
<h1>Common Title</h1>

<!-- a标签：用来跳转路由
     routerLink: Angular的组件,需要自动导入到app.component.ts中-->
<nav>
  <a routerLink="/erick-first">first</a><br/>
  <a routerLink="/erick-second">second</a>
</nav>
<!--路由定义的地方：定位到具体元素，存放位置-->
<router-outlet/>
```

### 5. 匹配顺序 &  重定向 & 统配符 & Title

- 第一个匹配上后，后面就不再匹配了
- 跳转到不同的页面时候，浏览器中的标签，会跟随着变
- 使用title属性，应用中的每个页面都应该有一个唯一的标题

```ts
import {Routes} from '@angular/router';
import {FirstComponent} from './first/first.component';
import {SecondComponent} from './second/second.component';
import {PageNotFoundComponent} from './page-not-found/page-not-found.component';

/*定义好两个路由*/
export const routes: Routes = [

  /*指定的路由*/
  {
    path: 'erick-first', component: FirstComponent, title: 'first-page'
  },
  {
    path: 'erick-second', component: SecondComponent, title: 'second-page'
  },

  /* redirect
  * 1. 跳转前： http://localhost:4200/first
  * 2. 跳转后： http://localhost:4200/erick-first */
  {
    path: 'first', redirectTo: 'erick-second', pathMatch: 'full',
  },

  /*默认跳转： http://localhost:4200/*/
  {
    path: '', component: FirstComponent
  },

  /*统配符： 1. 如果任何路径都没匹配上，则使用该叶敏啊
   *        2. 通配符一般放在数组的末尾，因为一般找到就不再向后匹配了
   *        3. 可以展示一个自定义的404页面，或者redirect到其他页面/
  {
    path: '**', component: PageNotFoundComponent, title: 'not-found-page'
  }
];
```

![image-20241121160151171](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20241121160151171.png)

## Nesting Routes

### 1. app.routes.ts

```ts
import {Routes} from '@angular/router';
import {HomeComponent} from './home/home.component';
import {CartComponent} from './cart/cart.component';
import {ErickComponent} from './home/erick/erick.component';
import {LucyComponent} from './home/lucy/lucy.component';
import {TomComponent} from './home/tom/tom.component';

/*定义好两个路由*/
export const routes: Routes = [
  {
    path: 'home', component: HomeComponent, children: [
      {
        path: 'erick', component: ErickComponent
      },
      {
        path: 'lucy', component: LucyComponent
      },
      {
        path: 'tom', component: TomComponent
      }
    ]
  },
  {
    path: 'cart', component: CartComponent
  }
];
```

### 2. app页面

```html
<h1>Common Title</h1>

<router-outlet/>
```

### 3. 一级路由页面

```html
<!--在子路由中的位置-->
<router-outlet></router-outlet>

<p>home works!</p>
```

### 访问

#### URL访问

```bash
# http://localhost:4200/home

# http://localhost:4200/home/lucy

# http://localhost:4200/home/tom
```

#### UI元素访问

- 定义在对应的父级路由中：home.component.html

```html
<!--在子路由中的位置-->
<router-outlet></router-outlet>

<nav>
  <a routerLink="erick">erick</a><br/>
  <a routerLink="lucy">lucy</a><br/>
  <a routerLink="tom">tom</a><br/>

</nav>

<p>home works!</p>
```

## AuthGuard

```bash
ng g guard basic
```

### basicGuard

```ts
import {CanActivateFn, Router} from '@angular/router';
import {inject} from '@angular/core';

let login = true;
export const basicGuard: CanActivateFn = (next, state) => {
  /*导入Router，操作路由*/
  const router = inject(Router);

  if (login) {
    /*返回true，则到对应页面*/
    return true
  } else {
    /*返回false，则需要异常处理，或者对应的跳转*/
    router.navigate(['/home/erick']);
    return false;
  }
};
```

### app.routes.ts

```ts
import {Routes} from '@angular/router';
import {CartComponent} from './cart/cart.component';
import {basicGuard} from './basic.guard';

/*定义好两个路由*/
export const routes: Routes = [
  {
    /*可添加多个guard*/
    path: 'cart', component: CartComponent, canActivate: [basicGuard]
  }
];
```

## 路径传参

### 1. Path Parameters

- 路由参数，会在URL中进行显示
- 刷新页面，参数不会丢失

```bash
# 跳转后的路由
http://localhost:4200/hyperlink/erick/Beijing
```

#### app.route.ts

```ts
import {Routes} from '@angular/router';
import {DashboardComponent} from './dashboard/dashboard.component';
import {HyberlinkComponent} from './hyberlink/hyberlink.component';

/*定义好两个路由*/
export const routes: Routes = [
  /*src路由*/
  {
    path: 'dashboard', component: DashboardComponent
  },

  /*dest路由*/
  {
    path: 'hyperlink/:name/:address', component: HyberlinkComponent
  }
];
```

#### SRC

```ts
import {Component} from '@angular/core';
import {ActivatedRoute, Router} from '@angular/router';

@Component({
  selector: 'app-dashboard',
  imports: [],
  templateUrl: './dashboard.component.html',
  standalone: true,
  styleUrl: './dashboard.component.css'
})
export class DashboardComponent {

  constructor(private router: Router) {
  }

  /*带着参数，跳转到指定路由*/
  gotoDashboard() {
    this.router.navigate(['hyperlink', 'erick', 'Beijing'],);
  }
}
```

```html
<button (click)="gotoDashboard()">hyperlink click</button>
```

#### DEST

```ts
import {Component} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-hyberlink',
  imports: [],
  templateUrl: './hyberlink.component.html',
  standalone: true,
  styleUrl: './hyberlink.component.css'
})
export class HyberlinkComponent {
  /*注入当前活动的路由*/
  constructor(private router: ActivatedRoute) {
  }

  checkParams = () => {
    /*能够拿到对应的路由参数*/
    this.router.params.subscribe(params => {
      console.log(params['name']);
      console.log(params['address']);
    })
  }
}
```

```html
<button (click)="checkParams()">check</button>
```

### 2. Query Parameters

- 路由参数，会在URL中进行显示
- 刷新页面，参数不会丢失

```bash
http://localhost:4200/hyperlink?name=lucy&address=NewYork
```

#### app.route.ts

```ts
import {Routes} from '@angular/router';
import {DashboardComponent} from './dashboard/dashboard.component';
import {HyberlinkComponent} from './hyberlink/hyberlink.component';

/*定义好两个路由*/
export const routes: Routes = [
  /*src路由*/
  {
    path: 'dashboard', component: DashboardComponent
  },

  /*dest路由*/
  {
    path: 'hyperlink', component: HyberlinkComponent
  }
];
```

#### SRC

```ts
import {Component} from '@angular/core';
import {ActivatedRoute, Router} from '@angular/router';

@Component({
  selector: 'app-dashboard',
  imports: [],
  templateUrl: './dashboard.component.html',
  standalone: true,
  styleUrl: './dashboard.component.css'
})
export class DashboardComponent {

  constructor(private router: Router) {
  }

  /*带着参数，跳转到指定路由*/
  gotoDashboard() {
    this.router.navigate(['hyperlink'], {
      /*参数必须是：queryParams*/
      queryParams: {
        name: 'lucy',
        address: 'NewYork'
      }
    });
  }
}
```

#### DEST

```ts
import {Component} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-hyberlink',
  imports: [],
  templateUrl: './hyberlink.component.html',
  standalone: true,
  styleUrl: './hyberlink.component.css'
})
export class HyberlinkComponent {
  /*注入当前活动的路由*/
  constructor(private router: ActivatedRoute) {
  }

  checkParams = () => {
    /*能够拿到对应的路由参数*/
    this.router.queryParams.subscribe(params => {
      console.log(params['name']);
      console.log(params['address']);
    })
  }
}
```

### 3. Fragement

- 路由参数，会在URL中进行显示
- 刷新页面，参数不会丢失

```bash
http://localhost:4200/hyperlink#hello
```

#### app.route.ts

```ts
import {Routes} from '@angular/router';
import {DashboardComponent} from './dashboard/dashboard.component';
import {HyberlinkComponent} from './hyberlink/hyberlink.component';

/*定义好两个路由*/
export const routes: Routes = [
  /*src路由*/
  {
    path: 'dashboard', component: DashboardComponent
  },

  /*dest路由*/
  {
    path: 'hyperlink', component: HyberlinkComponent
  }
];
```

####  SRC

```ts
import {Component} from '@angular/core';
import {ActivatedRoute, Router} from '@angular/router';

@Component({
  selector: 'app-dashboard',
  imports: [],
  templateUrl: './dashboard.component.html',
  standalone: true,
  styleUrl: './dashboard.component.css'
})
export class DashboardComponent {

  constructor(private router: Router) {
  }

  /*带着参数，跳转到指定路由*/
  gotoDashboard() {
    this.router.navigate(['hyperlink'], {fragment: 'hello'});
  }
}
```

#### DEST

```ts
import {Component} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-hyberlink',
  imports: [],
  templateUrl: './hyberlink.component.html',
  standalone: true,
  styleUrl: './hyberlink.component.css'
})
export class HyberlinkComponent {
  /*注入当前活动的路由*/
  constructor(private router: ActivatedRoute) {
  }

  checkParams = () => {
    /*能够拿到对应的路由参数*/
    this.router.fragment.subscribe(params => {
      console.log(params);
    })
  }
}
```

### 4. State



#### app.route.ts

```ts
import {Routes} from '@angular/router';
import {DashboardComponent} from './dashboard/dashboard.component';
import {HyberlinkComponent} from './hyberlink/hyberlink.component';

/*定义好两个路由*/
export const routes: Routes = [
  /*src路由*/
  {
    path: 'dashboard', component: DashboardComponent
  },

  /*dest路由*/
  {
    path: 'hyperlink', component: HyberlinkComponent
  }
];
```

#### SRC

```ts
import {Component} from '@angular/core';
import {Router} from '@angular/router';

@Component({
  selector: 'app-dashboard',
  imports: [],
  templateUrl: './dashboard.component.html',
  standalone: true,
  styleUrl: './dashboard.component.css'
})
export class DashboardComponent {

  constructor(private router: Router) {
  }

  /*带着参数，跳转到指定路由*/
  gotoDashboard() {
    this.router.navigate(['hyperlink'], {
      state: {
        name: 'erick',
        address: 'beijing',
      }
    });
  }
}
```

#### DEST

```ts
import {Component, OnInit} from '@angular/core';
import {Router} from '@angular/router';

@Component({
  selector: 'app-hyberlink',
  imports: [],
  templateUrl: './hyberlink.component.html',
  standalone: true,
  styleUrl: './hyberlink.component.css'
})
export class HyberlinkComponent implements OnInit {

  constructor(private router: Router) {
    /*这里的Router中获取的上次的Router，可以拿到数据*/
    console.log(`ngInit url=${this.router.url}  state=${this.router.getCurrentNavigation()?.extras.state}`)
  }

  /*不行*/
  ngOnInit(): void {
    console.log(`ngInit url=${this.router.url}  state=${this.router.getCurrentNavigation()?.extras.state}`)
  }

  /*不行*/
  checkParams = () => {
    console.log(`ngInit url=${this.router.url}  state=${this.router.getCurrentNavigation()?.extras.state}`)
  }
}
```



# Forms

## Reactive Form

- 响应式表单，数据的双向绑定

### 1. 基本表单

```ts
import {Component} from '@angular/core';
import {FormControl, ReactiveFormsModule} from '@angular/forms';

@Component({
  selector: 'app-cart',
  imports: [
    /*响应式表单*/
    ReactiveFormsModule
  ],
  templateUrl: './cart.component.html',
  standalone: true,
  styleUrl: './cart.component.css',
})
export class CartComponent {

   // FormControl
  name = new FormControl('');

  check = () => {
    console.log(this.name.value);
  }

  update = () => {
    this.name.setValue("Lucy");
  }
}
```

```html
姓名：<input type="text" [formControl]="name"/> <br/>

<!--1. 在input中修改，查看时候会动态调整： html ==> component -->
<button (click)="check()">查看姓名</button><br/>

<!--2. 修改component，html中动态修改： component ==> html -->
<button (click)="update()">修改姓名</button>
```

### 2. 表单组

- 对应的component，可以有两种写法

```ts
import {Component} from '@angular/core';
import {FormControl, FormGroup, ReactiveFormsModule} from '@angular/forms';

@Component({
  selector: 'app-cart',
  imports: [
    /*响应式表单*/
    ReactiveFormsModule
  ],
  templateUrl: './cart.component.html',
  standalone: true,
  styleUrl: './cart.component.css',
})
export class CartComponent {

  condition = new FormGroup(
    {
      name: new FormControl(''),
      address: new FormControl('')
    }
  )

  /* check */
  check = () => {
    console.log(this.condition.value.name);
    console.log(this.condition.value.address);
  }

  update = () => {
    this.condition.setValue({
      name: 'lucy',
      address: 'beijing',
    })
  }
}
```

```ts
import {Component, inject} from '@angular/core';
import {FormBuilder, ReactiveFormsModule} from '@angular/forms';

@Component({
  selector: 'app-cart',
  imports: [
    /*响应式表单*/
    ReactiveFormsModule
  ],
  templateUrl: './cart.component.html',
  standalone: true,
  styleUrl: './cart.component.css',
})
export class CartComponent {

  /*FormBuilder注入*/
  private formBuilder= inject(FormBuilder);

  condition = this.formBuilder.group({
    name: [''],
    address: [''],
  })

  /* check */
  check = () => {
    console.log(this.condition.value.name);
    console.log(this.condition.value.address);
  }

  update = () => {
    this.condition.setValue({
      name: 'lucy',
      address: 'beijing',
    })
  }
}
```

```html
<form [formGroup]="condition">
  <!--指定名字-->
  姓名：<input type="text" formControlName="name"/> <br/>
  地址：<input type="text" formControlName="address"/> <br/>
</form>

<!--1. 在input中修改，查看时会动态调整： html ==> component -->
<button (click)="check()">查看</button><br/>

<!--2. 修改component，html中动态修改： component ==> html -->
<button (click)="update()">修改</button>
```

##  Template-Driven Form

# 测试

## Service

### 1. 打桩-本类

- 执行当前类的方法调用时候，跳过某个方法的执行

```ts
import {Injectable} from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class ErickService {

  constructor() {
  }

  /*single method*/
  getName = (name: string): string => {
    return name.toUpperCase();
  }

  work = (name: string): string => {
    let res = this.getName(name);
    return res + '123';
  }
}
```

```ts
import {ErickService} from "../app/service/erick.service";
import {TestBed} from "@angular/core/testing";

describe('erick service', () => {
  // 被测试类
  let service: ErickService;

  beforeEach(() => {
    TestBed.configureTestingModule({});

    // 实例化被测试类
    service = TestBed.inject(ErickService);
  })

  /*测试1*/
  it('getName', () => {
    // 方法调用
    let result = service.getName('erick');
    // 验证
    expect(result).toEqual('ERICK');
  })

  /*测试2:需要跳过getName的方法*/
  it('work',()=>{
    /*打桩：方法调用时，执行getName的方法时，跳过*/
    // 无返回值的，用 stub()
    spyOn(service, 'getName').and.returnValue('MOCK');

    let res = service.work('erick');
    expect(res).toEqual('MOCK123');
  })
})
```

### 2. 打桩-依赖类

- 可以mock对应的方法和属性

#### 依赖类

```ts
import {Injectable} from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class AwsService {

  /*属性*/
  region: string = 'beijing';
  price: string = '899';

  constructor() {
  }

  /*方法*/
  getSns = (region: string): string => {
    return region.toUpperCase();
  }
}
```

#### 测试类

```ts
import {Injectable} from '@angular/core';
import {AwsService} from "./aws.service";

@Injectable({
  providedIn: 'root'
})
export class ErickService {

  constructor(public awsService: AwsService) {
  }

  /*测试，想跳过被依赖的服务*/
  getTargetInfra = (infra: string) => {
    let name = 'erick';
    return name + this.awsService.getSns(infra);
  }

  /*想跳过被依赖的服务的属性*/
  getInfo = () => {
    return this.awsService.price + this.awsService.region;
  }
}
```

#### UT

```ts
import {ErickService} from "../app/service/erick.service";
import {TestBed} from "@angular/core/testing";
import {AwsService} from "../app/service/aws.service";

describe('erick service', () => {
  // 被测试类
  let service: ErickService;

  // 依赖类的监控引用： 在后续方法中可以进行打桩
  let awsServiceSpy: jasmine.SpyObj<AwsService>;

  beforeEach(() => {
    /*创建一个被依赖的类的Mock:
    参数1: 类名
    参数2: 方法名
    参数3: 属性名: 可以mock具体的值*/
    const spyAwsService = jasmine.createSpyObj<AwsService>('AwsService', ['getSns'], {
      price: '123',
      region: 'usd',
    });

    TestBed.configureTestingModule({
      providers: [
        /*注入被mock的类的时候，useValue用上面创建的spyAwsService*/
        [{provide: AwsService, useValue: spyAwsService}]
      ]
    });

    // 向测试引擎注入  被测试类
    service = TestBed.inject(ErickService);
    // 向测试引擎注入  依赖类
    awsServiceSpy = TestBed.inject(AwsService) as jasmine.SpyObj<AwsService>;
  })
  
  /*测试:需要跳过getSns的方法*/
  it('getTargetInfra', () => {
    // 监控指定方法的方式
    awsServiceSpy.getSns.and.returnValue('1')

    let res = service.getTargetInfra('SNS');
    expect(res).toEqual('erick1');
  })


  /*测试:需要跳过g属性*/
  it('getInfo', () => {
    let info = service.getInfo();
    expect(info).toEqual('123usd');
  })
})
```

### 3. 非Root的Service

- 上面的Service，都是root级别的，如果是非Root的
- 被测试类就用useClass，依赖类就用useValue

```ts
  beforeEach(() => {

    const spyAwsService:AwsService = jasmine.createSpyObj('AwsService',['getSns']);

    TestBed.configureTestingModule({
      providers:[
        /*注入被测试类: useClass, 就用实际的类, 默认值就是useClass*/
        {provide: ErickService,useClass: ErickService},
        /*注入被mock的类的时候，useValue用上面创建的spyAwsService*/
        {provide: AwsService, useValue: spyAwsService}
      ]
    });

    service = TestBed.inject(ErickService);
    awsServiceSpy = TestBed.inject(AwsService) as jasmine.SpyObj<AwsService>;
  })
```

