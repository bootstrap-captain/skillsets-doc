# 引入方式

- CSS: Cascading Style Sheets
- 组成：选择器和对应的属性声明
- 格式规范：分行写，属性和值用小写

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>css入门</title>
    <!--css要写在head中的style中去-->
    <style>
        p {
            color: red;
            font-size: 30px; /*px代表像素*/
        }
    </style>
</head>
<body>
<p>Hello Word</p>
</body>
</html>
```

| 样式表             | 作用域   | 优点             | 缺点                   | 场景            |
| ------------------ | -------- | ---------------- | ---------------------- | --------------- |
| 行内样式(内联样式) | 当前标签 | 书写简单样式     | 结构样式混合，不能复用 | 较少使用        |
| 内部样式           | 当前页面 | 结构样式部分分离 | 没彻底分离             | 较多            |
| 外部样式           | 多页面   | 结构样式完全分离 | NA                     | **Recommended** |

## 1. 行内样式

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>css入门</title>
</head>
<body>
<p style="color: red; font-size: 30px">Hello Word</p>
</body>
</html>
```

## 2. 内部样式

- style理论上可以放在html中任何位置，但是不管如何，最终浏览器会把style放在head中
- 一般会放在head中

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>css入门</title>
    <style>
        p {
            color: red;
            font-size: 30px; 
        }
    </style>
</head>
<body>
<p>Hello Word</p>
</body>
</html>
```

## 3. 外部样式

```css
p {
    color: blue;
}
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <!--引进来
       rel: relation-->
    <link rel="stylesheet" href="first.css">
</head>
<body>

<p>hello</p>
</body>
</html>
```

## 4. 引入方式的优先级

```bash
# 不同名的
- merge操作：行内 + 内部 + 外部

# 同名
- 行内 > 内部 = 外部

- 内部样式和外部样式的优先级：取决于在html中的声明顺序(后写的会覆盖前面的，指的是在style中声明顺序)

- 同一个样式表(内部或者外部)：优先级和编写顺序有关，后面的会覆盖前面的
```

# 选择器

## 1. 基础选择器

- 根据不同需求，把不同的标签选择出来
- 由单个选择器组成

### 1.1 通配符选择器

- 选取html中所有的标签，设置当前页面的所有标签的通用属性
- 一般用来样式清除

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        /*通配符*/
        * {
            color: red;
        }
    </style>
</head>
<body>
你好

<p>Hello</p>
<h1>Word</h1>
</body>
</html>
```

### 1.2 元素选择器

- 以HTML的标签作为选择器
- 将页面某一类的标签全部选择出来，比如所有的p和div标签
- 优点：快速为页面中同类型的标签设置统一样式
- 缺点：页面同类型标签，如果想区分，则做不到

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            color: blue;
        }

        p {
            color: green;
        }
    </style>
</head>
<body>

<div>男人</div>
<div>女人</div>
<div>人妖</div>

<p>apple</p>
<p>peach</p>
<p>melon</p>

</body>
</html>
```

### 1.3 类选择器

#### 单class

- 单独选择一个或者某几个标签
- 开发最常用

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        /*类选择器，对应class
           . : 告知该选择器是类选择器
               选中页面中所有的class为该类名的
         */
        .fruit {
            color: green;
            font-size: 20px;
        }

        .old-phone {
            color: blue;
            font-size: 50px;
        }
    </style>
</head>
<body>

<!--class：指定类选择器的名称-->
<ul class="fruit">
    <li>西瓜</li>
    <li>蜜桃</li>
    <li>葡萄</li>
</ul>

<ul>
    <li class="old-phone">华为</li>
    <li>小米</li>
    <li>诺基亚</li>
</ul>


</body>
</html>
```

#### 多class

- 可以为某个标签指定多个类名，从而达到更多的选择目的
- 多个不同的组件： 公共的样式和个性化的样式，可以通过多class来定义

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .first {
            color: red;
        }

        .second {
            font-size: 50px;
        }
    </style>
</head>
<body>

<p class="first second">Hello Word</p>

</body>
</html>
```

### 1.4 ID选择器

- 和类选择器比较类似
- 缺陷：id只能是页面唯一一个节点，因此一个id选择器对应的css样式，只能被使用一次

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        /*id选择器
         # ： 表明是一个id选择器*/
        #erick {
            color: red;
        }
    </style>
</head>
<body>

<p id="erick">Hello Word</p>
</body>
</html>
```

## 2. 复合选择器

- 建立在基础选择器上，对基本选择器进行组合

### 2.1 交集选择器

- 多个基础选择器，连接在一起
- 某个页面元素，同时符合两种基础选择器才能选中的

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        /*   p: 代表元素选择器
            #nav： id选择器
         目标元素：p标签且id为nav   */
        p#nav {
            color: green;
        }

    </style>
</head>
<body>

<p id="nav">nihao</p>

</body>
</html>
```

### 2.2 并集选择器

- 多种不同的基础选择器，或
- 一般竖着写

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        /*目标元素： div或者p标签*/
        div,
        p {
            color: red;
        }
    </style>
</head>
<body>

<div>nihao</div>
<p>hello</p>
</body>
</html>
```

## 3. 父子选择器

### 3.1  后代选择器

- 选择父元素里面的子元素，孙子元素

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        /*选中ul中的所有后代的li： 儿子以及孙子元素*/
        ul li {
            color: red;
        }
    </style>
</head>
<body>

<ul>
    <li>抽烟</li>
    <li>喝酒</li>
    <li>烫头</li>
    <div>
        <li>踢球</li>
    </div>
</ul>
</body>
</html>
```

### 3.2 子选择器

- 某元素的最近一级子元素，亲儿子元素

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>

        /*只会渲染子元素*/
        div > a {
          color: red;
        }
    </style>
</head>
<body>

<div>
    <a href="#">子元素</a>
    <p>
        <a href="#">孙子元素</a>
    </p>

</div>

</body>
</html>
```

### 3.3 兄弟选择器

#### 紧紧相联

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        /*定位元素：和div相连的，下面的，紧紧相连的 一个p元素
        <p>舒展的下面兄弟</p>
          */
        div + p {
            color: red;
        }
    </style>
</head>
<body>

<p>舒展的上面兄弟</p>
<div>舒展</div>
<p>舒展的下面兄弟</p>
<p>其他兄弟1</p>

</body>
</html>
```

#### 非紧紧相连

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        /*定位元素：1. 定位div
                   2. 把div的所有的下面的兄弟元素
          */
        div ~ p {
            color: red;
        }
    </style>
</head>
<body>

<p>舒展的上面兄弟</p>
<div>舒展</div>
<p>舒展的下面兄弟</p>
<p>其他兄弟1</p>

</body>
</html>
```

## 4. 伪类选择器

- 用于向某些选择器添加特殊的效果，比如给链接添加特殊效果，或选择第一个，第n个元素
- 很像类，但不是类，是元素特殊状态的一种描述
- 非常重要

### 4.1 动态伪类

#### link伪类

- 书写的时候，按照顺序来写， LVHA
- a链接在浏览器中具有默认样式，实际工作中都需要给链接单独指定样式
- 一般是指定： a:link  和 a.hover即可

| 选择器    | Desc                               | 使用场景            |
| --------- | ---------------------------------- | ------------------- |
| a:link    | 选择所有未被访问的链接             | a                   |
| a:visited | 选择所有已被访问的链接             | a                   |
| a:hover   | 选择鼠标指针位于其上的链接         | 其他元素也可 span等 |
| a:active  | 选择活动链接(鼠标按下未弹起的链接) | 其他元素也可 span等 |

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        a:link {
            color: black;
        }

        a:visited {
            color: blue;
        }

        a:hover {
            color: pink;
        }

        a:active {
            color: gray;
        }
    </style>
</head>
<body>

<a href="#">hello</a><br/>
<a href="#">hello</a><br/>
<a href="#">hello</a><br/>
<a href="#">hello</a><br/>
</body>
</html>
```

#### focus伪类

- 选取获得鼠标焦点的表单元素，只能存在于表单元素
- input:focus

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        input:focus {
            background-color: yellow;
        }
    </style>
</head>
<body>

<form action="#">
    用户名：<input type="text"><br/>
    密码：<input type="password"><br/>
</form>

</body>
</html>
```

# 三大特性

## 1. 层叠性

- 相同选择器，相同样式(**冲突的属性：就近原则，后来者居上**)
- 可以通过chrom的console来检查

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        /*相同选择器，相同样式，就近原则*/
        div {
            color: red;
        }

        div {
            color: pink;
        }
    </style>
</head>
<body>

<div>nihao</div>
</body>
</html>
```

## 2. 继承

- 子标签会继承父标签的样式，子标签可以重写

### 普通属性

- 继承text-， font-， line-， color这些元素开头的样式。都是和盒子模型没关系的属性
- 盒子属性类的，都不会继承

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        /*p会继承div的*/
        div {
            color: red;
        }

    </style>
</head>
<body>

<div>nihao
    <p>hello</p>
</div>
</body>
</html>
```

### 样式继承

```bash
# element.style
- 行内样式

# .d3  1.html
- 内部样式

# user agent stylesheet
- 浏览器默认样式

# inherited from div.d1
- 亮着的部分，就是继承过来的样式
```

![image-20240721200423418](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721200423418.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .d1 {
            font-size: 40px;
            color: red;
            width: 600px;
            height: 400px;
            background-color: gray;
        }

        .d2 {
            width: 300px;
            height: 200px;
            background-color: orange;
        }

        .d3 {
            width: 200px;
            height: 50px;
            background-color: skyblue;
        }
    </style>

</head>
<body>
<div class="d1">
    <div class="d2">
        <div class="d3" style="font-family: 'Times New Roman',serif">你好啊</div>
    </div>
</div>

</body>
</html>
```

## 3. 优先级

### 基础选择器

- 选择器相同，执行层叠性
- 选择器不同，则根据**选择器权重**执行
- a标签：浏览器默认给了一个选择器对应的样式

![image-20230703224943530](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230703224943530.png)

```css
<style>
    div {
        font-size: 20px !important;
    }
</style>
```

### 复合选择器

- 会有权重叠加

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        /*权重为： 0,0,0,1 + 0,0,0,1 = 0,0,0,2*/
        ul li {
            color: red;
        }

        /*权重为： 0,0,0,1*/
        li {
            color: pink;
        }
    </style>
</head>
<body>

<ul>
    <li>hello</li>
</ul>

</body>
</html>
```

# CSS样式

## 1. 颜色

```bash
# 英文
- blue 
- 表示的颜色少(使用较少） 

# 十六进制(HEX)/HEXA
- #0e1b6d
- 最常用
- 可以用snipaste取色
 
# rgb
-  rgb(200,0,0)      rgb(90%，5%,5%)
-  rgba(90%，5%,5%,0.5)      # 带透明的：0-1
- 可以用snipaste取色 
```

## 2. 字体

- 字体系列，大小，粗细，文字样式(斜体)

| 标签        | Desc     | Value                                                        |
| ----------- | -------- | ------------------------------------------------------------ |
| font-family | 字体样式 |                                                              |
| font-size   | 字体大小 | px                                                           |
| font-weight | 字体粗细 | normal:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;正常，相当于400<br/>bold: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;加粗，相当于700<br/>bolder: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 特粗<br/>lighter:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;细体<br/>number: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;100-900，整百， **Recmmended** |
| font-style  | 字体样式 | normal:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;正常<br/>italic: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;倾斜 |

```css
<style>
    /*分开写法*/
    p {
        /*字体如果是多个字母，则加引号
         1. 判断标准：按声明顺序，判断电脑是否安装，安装了就会选用
         2. 使用标准：从前往后匹配，匹配上就可以*/
        font-family: "Times New Roman", Times;

        /*字体大小： px是常用单位
        谷歌浏览器默认是16px
        不同浏览器默认的字号大小不同，因此尽量给显式的值，不要给默认值*/
        font-size: 50px;

        /*字体粗细：更加提倡使用数字*/
        font-weight: 700;

        /*字体样式：很少给文字加清晰，反而要给斜体标签 em和i，改为不倾斜*/
        font-style: normal;
    }

    /* 复合写法：
       1. 必须严格按照书写顺序
       2. 不需要的属性可以省略，但必须保留font-size和font-family，否则不起作用*/
    /*font:{font-style font-weight font-size/line-height font-family}*/
    h1 {
        font: normal 400 100px/200px "Times New Roman";
    }

</style>
```

### 2.1 font-size

- 设置的时候，40px指的是单个格子的高度，宽度会随着自适应
- 字体设计的原因，文字最终呈现的大小，并不一定与font-size的值一致，可能大， 也可能小
- 文字相对字体设计框，并不是垂直居中的，一般都靠小一些

![image-20240616133932500](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240616133932500.png)

## 3. 文本

| 标签            | Desc                                              | Value                                                        |
| --------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| color           | 文本颜色                                          | 三种方式都可                                                 |
| letter-spacing  | 字母之间的间距                                    | px                                                           |
| word-sapcing    | 单词之间的间距，对中文不起作用                    | px                                                           |
| text-align      | 对齐文本<br>**<u>针对盒子，在盒子内部的布局</u>** | center: 水平居中&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br/>left: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;default value<br/>right |
| text-decoration | 文本装饰                                          | none: &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Recommended**， 用来删除本来带删除线的文本<br/>underline:  &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;下划线<br/>overline： &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;上划线<br/>line-through:  &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 删除线 |
| text-indent     | 首行缩进                                          | px:   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;像素<br/>em: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;相对单位，当前元素(font-size)一个文字的大小。**Recommended** |
|                 |                                                   | px<br/>修改行间距时，文本高度font-size不会变，改变的是上间距和下间距<img src="https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230703104652164.png" alt="image-20230703104652164" style="zoom: 25%;" /> |

```css
    <style>
        p {
            /*1. 文本颜色*/
            color: rgb(52, 173, 26);
            /*2. 文本布局*/
            text-align: left;

            /*3. 上划线等*/
            text-decoration: none;

            /*4.  首行缩进*/
            text-indent: 20px;

            /*5. 行间距*/
            line-height:  26px;
        }
    </style>
```

### 3.1 行高

- line-height: px
- 修改line-height时，文本高度font-size不会变，改变的是上间距和下间距
- 如果不写line-heigh时，浏览器会根据font-size值，给一个默认的line-height值
- 受字体原因影响，文本在一行中，并不是绝对垂直居中

![image-20240616135657189](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240616135657189.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>

        /*写法一：显示根据font-size值，给一个line-height值*/
        #first {
            font-size: 40px;
            line-height: 100px;
            background-color: green;
        }

        /*写法二：normal，
               1. 浏览器根据font-size值，给line-height一个合适的值
               2. 不写line-height，就是默认normal*/
        #second {
            font-size: 40px;
            line-height: normal;
            background-color: blue;
        }

        /*写法三：倍数，用的比较多
             1. 根据font-size值，乘以对应的倍数，就是line-height
             2. 可以写数值 2， 或者 200%*/

        #third {
            font-size: 40px;
            line-height: 200%;
            background-color: gray;
        }
    </style>

</head>
<body>

<div id="first">
    舒展Hi,what are you doing?
</div>

<div id="second">
    舒展Hi,what are you doing?
</div>

<div id="third">
    舒展Hi,what are you doing?
</div>

</body>
</html>
```

### 3.2 文本垂直

#### 垂直-top-line-height

- 默认就是这种方式

#### 垂直居中~line-height

- 不太推荐的一种方式

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            /*div盒子的高度*/
            height: 400px;
            font-size: 40px;
            /*行高和盒子一样高*/
            line-height: 400px;
            background-color: green;
        }
    </style>

</head>
<body>

<div id="first">
    舒展Hi,what are you doing?
</div>

</body>
</html>
```

![image-20240616142034070](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240616142034070.png)

#### 垂直-bottom~line-height

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            /*div盒子的高度*/
            height: 400px;
            font-size: 40px;
            /*行高是盒子的两倍-fontsize-字体设计的原因
             400*2-40-10*/
            line-height: 750px;
            background-color: green;
        }
    </style>

</head>
<body>

<div id="first">
    舒展Hi,what are you doing?
</div>

</body>
</html>
```

![image-20240616142335012](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240616142335012.png)

## 4. 背景

| 标签                  | Desc             | Value                                                        |
| --------------------- | ---------------- | ------------------------------------------------------------ |
| background-color      | 背景颜色         | transparent:&nbsp;             默认透明<br/>颜色：                       red等<br/>rgb(200,100,30):        rgb色<br/>十六进制：&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#0e1b6d    **Recommended** |
| background-image      | 背景图片         | url("../image/1.jpg")：    路径或网络url                     |
| background-repeat     | 背景平铺重复方式 | no-repeat：&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不平铺<br/>repeat-x:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;沿x平铺<br/>repeat-y: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;沿y平铺<br/>repeat:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;沿x和y平铺 |
| background-position   | 背景图片方位     | 方位名次：left/center/right + top/center/bottom 两个参数顺序不影响<br/>精确单位：x,y<br/>混合单位： x,y |
| background-attachment | 背景附着         | fixed:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;固定的<br/>scroll: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;滚动的-Default |

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>

        /*分开写法*/
        .first {
            width: 1000px;
            height: 1000px;

            /*既可以选择背景颜色，也可以选择背景图片，背景图片会压住背景颜色*/
            /*背景颜色
              rgba(200, 100, 30,0.3): 背景颜色半透明，0代表全透明，1代表不透明
            */
            background-color: rgba(200, 100, 30,0.5);

            /*背景图片*/
            background-image: url("../image/1.jpg");

            /*背景图片的平铺方式*/
            background-repeat: no-repeat;

            /*背景图片方位
            参数方式一： 方位名词：
                    1.1 两个方位名词都全，前后顺序不影响 left/center/right + top/center/bottom
                    1.2 如果只写一个参数，则另外一个默认居中
            参数方式二： 精确单位：x,y: 距离左+上的距离， 顺序按照x，y;

            参数方式三： 混合单位，按照x，y
                */
            background-position: 20px 10px;

            background-attachment: scroll;
        }

        /*复合写法: 没有特定的书写顺序*/
        .second {
            width: 1000px;
            height: 1000px;
            background: red url("../image/1.jpg") no-repeat scroll 20px 10px;
        }
    </style>
</head>
<body>

<div class="first">
    第一个
</div>

<div class="second">
    第二个
</div>

</body>
</html>
```

## 5. 长度

### 5.1 像素--px

- 具体的像素

### 5.2 em

- 当前元素的font-size的倍数

```css
<style>
    /*10*font-size = 200px     默认font-size：16px
      如果没定义，则向上找父亲的font-size， font-size也可以用 em 表示*/
    div {
        width: 10em;
        height: 10em;
        font-size: 20px;
        background-color: skyblue;
    }
</style>
```

### 5.3 rem

- 相对于根元素html的font-size

```css
<style>
    div {
        width: 10rem;
        height: 10rem;
        background-color: skyblue;
    }
</style>
```

### 5.4 10%

- 相对父元素的宽和高

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .outer {
            width: 200px;
            height: 200px;
            background-color: skyblue;
        }

        /*相对父元素的宽和高的百分比*/
        .inside {
            width: 20%;
            height: 50%;
            background-color: gray;
        }
    </style>
</head>
<body>

<div class="outer">
    <div class="inside"></div>
</div>

</body>
</html>
```

### 5.2 视口宽度--vw

- viwport width
- 宽和高，都是指的是视口宽度的百分比
- 在PC端不多，在移动端多，响应式布局
- 用的比较多

```css
<style>
    .box1 {
        width: 50vw;
        height: 20vw;
        background-color: skyblue;
    }
</style>
```

### 5.3 视口高度--vh

- viewport hight

```css
<style>
    .box1 {
        width: 50vh;
        height: 50vh;
        background-color: skyblue;
    }
</style>
```

### 5.4 vw/vh为100-滚动条问题

- 当指定一个div宽为100vw，高为100vh，会出现一开始横向和纵向就有滚动条的问题
- 这是因为body的默认的margin为8px

![image-20240724210051940](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240724210051940.png)

- 为了避免这种情况，需要将body的默认的margin进行重写为0

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        body {
            margin: 0;
        }

        .outer {
            background-color: gray;
            width: 100vw;
            height: 100vh;
        }
    </style>
</head>
<body>
<div class="outer">
</div>
</body>
</html>
```



# 显示模式

- 标签元素以什么方式进行显示，比如div独占一行，一行可以放多个span

## 1. Block元素

- div是最典型的块元素
- 是一个容器及盒子，里面可以放block或者inline元素

```bash
# 特点
- 独占一行，不会与任何元素共用一行，从上到下排列
- width：默认撑满父元素
- height：默认由内容撑开

- height，width，外边距，内边距可控

# 注意
- 文字类的元素内，不能使用块级别元素， 比如p，h系列，里面不能再放其他块级元素

# 常见
- h1-h6, p, div, ul, ol, li, from   option
```

### 默认方式

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>

        body{
            background-color: aqua;
        }

        /*父元素是：body*/
        #first {
            background-color: green;
        }

        #second {
            background-color: pink;
        }

        #third {
            background-color: gray;
        }
    </style>

</head>
<body>

<div id="first">
    舒展Hi,what are you doing?
</div>
<div id="second">
    LucyHi,what are you doing?
</div>
<div id="third">
    张三Hi,what are you doing?
</div>

</body>
</html>
```

![image-20240616151242189](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240616151242189.png)

### 设置高宽

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>

        body {
            background-color: aqua;
        }

        /*父元素是：body*/
        #first {
            background-color: green;
            width: 400px;
            height: 200px;
        }

        #second {
            background-color: pink;
            width: 400px;
            height: 200px;
        }

        #third {
            background-color: gray;
            width: 400px;
            height: 200px;
        }
    </style>

</head>
<body>

<div id="first">
    舒展Hi,what are you doing?
</div>
<div id="second">
    LucyHi,what are you doing?
</div>
<div id="third">
    张三Hi,what are you doing?
</div>

</body>
</html>
```

![image-20240616151338513](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240616151338513.png)

## 2. Inline元素

- span是经典的行内元素
- Inline元素内，只能容纳文本或其他Inline元素

```bash
# 特点
- 相邻元素在一行上，一行可以显示多个
- 如果一行容纳不下，则继续在下一行从左往右排列

- width：默认就是本身内容的宽度， 直接设置无效
- height：默认由内容撑开，直接设置无效

# tip
- 链接里面不能再放链接

# 常见
- br  em   strong      a     label
```

![image-20240616151730959](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240616151730959.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>

        body {
            background-color: aqua;
        }

        /*父元素是：body*/
        #first {
            background-color: green;
            width: 400px;
            height: 200px;
        }

        #second {
            background-color: pink;
            width: 400px;
            height: 200px;
        }

        #third {
            background-color: gray;
            width: 400px;
            height: 200px;
        }
    </style>

</head>
<body>

<sapn id="first">
    舒展Hi,what are you doing?
    <sapn/>
    <sapn id="second">
        LucyHi,what are you doing?
    </sapn>
    <sapn id="third">
        张三Hi,what are you doing?
    </sapn>

</body>
</html>
```

## 3. Inline-Block元素

- 行内块元素，内联块元素
- 同时具有块元素和行内元素的特点

```bash
# Inline特点
- 不独占一行，和相邻行内块元素在一行上，之间会有空白缝隙
- 一行中不能容纳的行内元素，会在下一行继续从左向右排列
- 默认宽度：由内容撑开
- 默认高度：由内容撑开

# Block特点
- 可以通过CSS来设置宽高

#  常见
- 图片：img
- 单元格：td th
- 表单控件： input textarea    select  button
```

![image-20240616152459810](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240616152459810.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>

        body {
            background-color: aqua;
        }

        img {
            height: 500px;
        }

    </style>

</head>
<body>

<img src="../image/1.jpg">
<img src="../image/1.jpg">
<img src="../image/1.jpg">
<img src="../image/1.jpg">

</body>
</html>
```

## 4. 模式转换

- 经常需要
- 一个模式的元素需要另外一种模式的特性， 比如增加链接a的触发范围

```bash
# 行内转换块
display: block

# 块转换行内
display：inline

# 转换行内块元素
display: inline-block
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        body {
            background-color: gray;
        }

        /*行内元素转块元素*/
        a {
            color: black;
            width: 150px;
            height: 30px;
            background-color: pink;
            display: block;
        }
         
         /*block元素转inline元素*/
        div {
            width: 150px;
            height: 30px;
            background-color: yellow;
            display: inline;
        }
    </style>
</head>
<body>

<a href="#">百度一下</a>

<div>
    我是块级元素
</div>

</body>
</html>
```

![image-20240616154244320](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240616154244320.png)

# 盒子模型

- 会把所有的HTML元素都看成是一个盒子
- 调试：鼠标放在对应的盒子上，打开inspect，会直接打开该盒子的html元素的代码
- 盒子的最终大小：content + padding + border

## 1. content

- 盒子的内容区

### 1.1 基本使用

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            /*设置的宽高：是content的宽和高*/
            width: 400px;
            height: 300px;
            background-color: gray;
        }
    </style>
</head>
<body>

<div>你好啊</div>

</body>
</html>
```

![image-20240721105948023](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721105948023.png)

### 1.2 宽

- 默认是父元素的宽度，充满整个父容器

#### 不给宽度

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            height: 200px;
            /*设置的背景颜色，会填充内边距区域,边框*/
            background-color: gray;

            /*width：
            1. 充满整个父容器
            2. 父容器为body：铺满整个视口*/
        }
    </style>
</head>
<body>

<div>
    Lorem ipsum dolor sit amet, consectetur adipisicing elit. Architecto culpa eligendi fuga itaque laborum recusandae
    tenetur! Adipisci error facilis incidunt laborum neque nihil placeat reiciendis! Fuga iure maxime nihil quasi quos
    reprehenderit similique? Eum iusto minus officia repellendus voluptatem. Consequatur deserunt, dolore ex ipsam
    labore minima nemo officia rerum vero?
</div>

</body>
</html>
```

![image-20240721134553453](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721134553453.png)

```bash
# 调整浏览器的窗口大小
# 响应式布局
- 视口会从 1712--> 1284 --> ****
```

![image-20240721134657362](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721134657362.png)

![image-20240721134744585](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721134744585.png)

#### 指定宽度

```css
<style>
    div {
        height: 200px;
        /*设置的背景颜色，会填充内边距区域,边框*/
        background-color: gray;

        width: 800px;
    }
</style>
```

```bash
# 响应式布局
- 初始值为800px
- 当视口变小，小到800px以下时候，横向就会出现浏览器的滚动条
```

![image-20240721135155461](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721135155461.png)

![image-20240721135413145](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721135413145.png)

### 1.3 默认宽度

#### 初始值

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            height: 200px;
            background-color: gray;
        }
    </style>

</head>
<body>
<div>
    你好啊
</div>

</body>
</html>
```

- body为1712，div为1712

![image-20240721140933382](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721140933382.png)

#### 不给宽度

- margin是否会影响盒子大小，取决于是否给了盒子具体的宽度
- 给了宽度，并且margin+盒子宽不会大于视口，则不会影响
- border和padding不会影响盒子大小，因为本身就是盒子的一部分

```bash
# 默认宽度： 不设置width属性时，元素呈现的宽度
- 总宽度 = 父的content-自身的左右margin
- content宽度 = 父的contet-自身的左右margin-自身的左右border-自身的左右padding
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            height: 200px;
            background-color: gray;
            margin: 50px;
        }
    </style>

</head>
<body>
<div>
    你好啊
</div>

</body>
</html>
```

- 盒子大小：body宽为1712，div为1612

![image-20240721141630304](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721141630304.png)

### 1.4 高

- 默认由内容撑开
- 可以指定高度
- 一般高和min-height任选一个即可

## 2. padding

- 内边距：盒子中，content距离盒子边缘的距离
- 会撑大盒子的总的大小：content + padding

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            /*设置的宽高：是content的宽和高*/
            width: 400px;
            height: 300px;
            /*设置的背景颜色，会填充内边距区域*/
            background-color: gray;
            /*内边距*/
            padding: 10px 20px 30px 40px;
        }
    </style>
</head>
<body>

<div>你好啊</div>

</body>
</html>
```

![image-20240721110204537](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721110204537.png)

```bash
# 分开写法
 padding-left: 10px;
 padding-right
 padding-top
 padding-bottom

# 复合写法
padding: 50px                           四个都是50px
padding: 50px 20px                      上下是50，左右是20
padding: 50px 10px 20px                 上50，左右10，下20
padding: 10px 20px 30px 40px            上右下左    
```

## 3. border

- 盒子的边框
- 盒子的边框+盒子的padding，都会撑大盒子的总大小

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            /*设置的宽高：是content的宽和高*/
            width: 400px;
            height: 300px;
            /*设置的背景颜色，会填充内边距区域,边框*/
            background-color: gray;
            /*内边距*/
            padding: 10px 20px 30px 40px;
            /*边框*/
            border: 10px dashed black;
        }
    </style>
</head>
<body>

<div>你好啊</div>

</body>
</html>
```

![image-20240721142806228](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721142806228.png)

```bash
# 分别设置
border-width               边框粗细            10px
border-style               边框样式            solid dashed dotted   none
border-color               边框颜色

# 复合写法： 没有顺序
 border: 10px dashed black;
 
# 分别定义不同方向的边框
border-top: 5px solid gray;
border-left: 5px solid olivedrab;
border-right: 5px solid olivedrab;
border-bottom: 5px solid olivedrab;
```

## 4. margin

- 外边距：当前设置的盒子和其他盒子之间的距离
- 默认盒子和盒子之间，距离为0
- marign不会影响盒子大小

```bash
# 分开写
margin-left: 20px;
margin-right: 30px;
margin-bottom: 40px;
margin-top: 50px;

# 复合写法
- 和padding差不多
```

### 子元素的margin参考父元素的content计算

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .outer {
            width: 400px;
            height: 300px;
            background-color: gray;
            padding: 50px;
        }

        .inner {
            width: 100px;
            height: 100px;
            background-color: orange;
            margin: 20px;
        }
    </style>

</head>
<body>
<div class="outer">
    <div class="inner"></div>
</div>

</body>
</html>
```

![image-20240723105642044](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240723105642044.png)

### 盒子元素移动

- margin-top, margin-left, 会影响当前元素的位置
- margin-bottom，margin-right会影响后面的元素的位置

### 行内元素margin

- 对于行内元素，左右的margin可以设置，上下的margin，设置后是无效的

### margin: 0 auto -- 块级元素水平居中

```bash
# 盒子距离左边距，能有多远就多远
margin-left: auto,  距离左边能有多远就多远

# 盒子距离右边距，能有多远就多远
margin-right:auto   距离右边，能有多远就多远

# 可以左右都设置为auto，则就会居中
- 必须是一个块级元素
- 盒子中，让盒子内部的一个块级元素居中
- 盒子必须指定宽度，盒子左右的外边距设置为auto

# margin-bottom和margin-top不会有该功能
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            width: 500px;
            height: 100px;
            background-color: skyblue;
            margin-left: auto;
        }
    </style>

</head>
<body>
<div></div>

</body>
</html>
```

![image-20240721190417916](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721190417916.png)

![image-20240721190546823](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721190546823.png)

![image-20240721190752824](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721190752824.png)

### margin塌陷

#### 问题引出

- 父元素中包裹了多个块元素，第一个块元素的margin-top，最后一个块元素的margin-bottom，会存在margin塌陷
- margin塌陷：子元素没有上下移动，而是父元素上下移动了

![image-20240721192452429](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721192452429.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .outer {
            width: 500px;
            background-color: gray;
        }

        .inner1 {
            width: 100px;
            height: 50px;
            background-color: skyblue;
            margin-top: 50px;
        }

        .inner2 {
            width: 100px;
            height: 50px;
            background-color: orange;
            margin-bottom: 50px;
        }
    </style>

</head>
<body>
<div>测试上</div>
<div class="outer">
    <div class="inner1">inner1</div>
    <div class="inner2">inner2</div>
</div>
<div>测试下</div>

</body>
</html>
```

#### overflow: hidden;

- 在父容器中，添加属性overflow: hidden即可

![image-20240721192610432](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721192610432.png)

### margin合并

- 上下两个块级元素，上面的设置了margin-bottom，下面的设置了margin-top，会取二者最大值，作为距离，而不是二者相加
- 注意有这个事情就行，不一定非要去解决

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .box {
            width: 200px;
            height: 200px;
        }

        .box1 {
            background-color: skyblue;
            margin-bottom: 50px;
        }

        .box2 {
            background-color: orange;
            margin-top: 70px;
        }
    </style>

</head>
<body>

<div class="box box1">inner1</div>
<div class="box box2">inner2</div>

</body>
</html>
```

![image-20240721193352767](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721193352767.png)

## 5. 怪异盒模型--box-sizing: border-box

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .box1 {
            width: 200px;
            height: 200px;
            padding: 5px;
            border: 5px solid black;
            background-color: skyblue;
            /*默认值：标准盒子模型
            1. 盒子大小=内容+padding+border = 220*220*/
            box-sizing: content-box;

            margin-bottom: 10px;
        }

        .box2 {
            width: 200px;
            height: 200px;
            padding: 5px;
            border: 5px solid black;
            background-color: skyblue;
            /* IE盒模型
            2. 盒子总大小=200*200，content区域变成了180*180*/
            box-sizing: border-box;
        }
    </style>

</head>

<body>
<div class="box1">1</div>
<div class="box2">2</div>
</body>

</html>
```

![image-20240722163643769](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240722163643769.png)

# 样式问题

## 1. 隐藏元素

### 1.1 display: none

- display: none;  不会占位，但是在html代码中，依然会存在

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .box {
            width: 200px;
            height: 200px;
        }

        .box1 {
            background-color: skyblue;
            display: none;
        }

        .box2 {
            background-color: orange;
        }
    </style>

</head>
<body>

<div class="box box1">inner1</div>
<div class="box box2">inner2</div>

</body>
</html>
```

![image-20240721195123882](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721195123882.png)

### 1.2 visibility: hidden

- 会隐藏，但是会占位

![image-20240721195313586](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721195313586.png)

# 浮动

## 1. 文字环绕图片

### 1.1 未环绕

![image-20240728160331890](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240728160331890.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        div {
            width: 600px;
            height: 400px;
            background-color: skyblue;
        }

        img {
            width: 200px;
        }
    </style>
</head>
<body>

<div>
    <img src="1.jpg">
    <div>
        lorem100
    </div>
   
</div>
</body>
</html>
```

### 1.2 环绕--float: left

- 在图片身上开启左浮动

![image-20240728160409060](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240728160409060.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        div {
            width: 600px;
            height: 400px;
            background-color: skyblue;
        }

        img {
            width: 200px;
            float: left;
        }
    </style>
</head>
<body>

<div>
    <img src="1.jpg">
    <div>
        lorem100
    </div>

</div>
</body>
</html>
```

## 2. 文字环绕文字

### 2.1 未环绕

![image-20240728160937361](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240728160937361.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        div {
            width: 600px;
            height: 400px;
            background-color: skyblue;
        }

        /*伪类选择器：选择第一个字符*/
        .test::first-letter {
            font-size: 50px;
        }
    </style>
</head>
<body>

<div>
    <div class="test">
       lorem100 
    </div>

</div>
</body>
</html>
```

### 2.2 环绕

![image-20240728161111042](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240728161111042.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        div {
            width: 600px;
            height: 400px;
            background-color: skyblue;
        }

        /*伪类选择器：选择第一个字符*/
        .test::first-letter {
            font-size: 50px;
            float: left;
        }
    </style>
</head>
<body>

<div>
    <div class="test">
        lorem100
    </div>

</div>
</body>
</html>
```

## 3. 浮动元素特点

![image-20240727105323015](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240727105323015.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 1200px;
            height: 300px;
            background-color: gray;
            padding: 10px;
        }

        .box {
            font-size: 20px;
            padding: 10px;
        }

        .box1 {
            background-color: skyblue;
        }

        .box2 {
            background-color: orange;
        }

        .box3 {
            background-color: green;
        }
    </style>
</head>
<body>

<div class="outer">
    <div class="box box1">盒子一</div>
    <div class="box box2">盒子二</div>
    <div class="box box3">盒子三</div>
</div>
</body>
</html>
```

### 3.1 脱离文本流

- 盒子二左浮

```bash
# 让出位置，下面的会跑上来
- 本来盒子二是在第二行，盒子三在第三行
- 浮动后盒子二漂浮在第二行，盒子二上位到第二行
- 盒子二背后也是绿色的(可以设置盒子三的宽度来观察)

# 文字被甩出来
- 类似文字环绕图片案例，盒子三的文字会被浮动盒子挤到后面去

# 标准流
- 紧紧贴着第一层来排列

# 脱离文本流
- 飘起来了，相当于添加了z轴
```

![image-20240727105530203](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240727105530203.png)

```css
  .box2 {
      background-color: orange;
      /*给盒子二开启左浮*/
      float: left;
  }
```

### 3.2 宽/高

```bash
# 默认值
- 浮动元素，宽/高尽可能的小，通过内容撑开。越瘦越小，越好飘

# 可显示的设置宽高
- 设置浮动盒子的宽和高，就不会用内容撑开
```

![image-20240727110256278](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240727110256278.png)

![image-20240727110325159](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240727110325159.png)

### 3.3 和其他元素共用一行

- 共用一行，但是其实是在同一行的，z轴不一样
- 上面的元素占据自己的位置，下面的第一个元素浮动后，再下面的元素上位，浮动元素和上位元素就共用一行

### 3.4 margin完美设置

- 浮动元素，不存在margin塌陷，margin合并问题，可以完美设置
- padding类似

### 3.5 浮动起来的元素在一层

- 给box2开启浮动后，再给box3开启浮动
- box2已经浮动了，box3开启浮动后，会贴着box2，跑到第二层

![image-20240727111129127](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240727111129127.png)

## 4. 浮动影响

### 4.1 父元素 - 高度塌陷

- 父元素不给高度的情况下，高度默认是由内容撑开
- 父元素中所有的子元素都浮动后，父元素就塌陷了，就变成了一条线

![image-20240728165351959](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240728165351959.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 1200px;
            background-color: gray;
            /*父元素不给高度*/
            border: 1px solid black;
        }

        .box {
            width: 100px;
            height: 100px;
            margin: 10px;
            background-color: skyblue;
            border: 1px solid black;
        }


        /*子元素都浮动*/
        .box1 {
            float: left;
        }

        .box2 {
            float: left;
        }

        .box3 {
            float: left;
        }
    </style>
</head>
<body>

<div class="outer">
    <div class="box box1">1</div>
    <div class="box box2">2</div>
    <div class="box box3">3</div>
</div>
</body>
</html>
```

### 4.2 父元素 - 高度塌陷 - 父元素的兄弟元素顶上来

- 父元素中的所有子元素浮动后，父元素塌陷了，对应的父元素的兄弟元素就会顶上来

![image-20240728171009664](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240728171009664.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 1200px;
            background-color: gray;
            /*父元素不给高度*/
            border: 1px solid black;
        }

        .box {
            width: 100px;
            height: 100px;
            margin: 10px;
            background-color: skyblue;
            border: 1px solid black;
        }


        /*子元素都浮动*/
        .box1 {
            float: left;
        }

        .box2 {
            float: left;
        }

        .box3 {
            float: left;
        }
    </style>
</head>
<body>

<!--父元素高度塌陷-->
<div class="outer">
    <!--子元素全部浮动-->
    <div class="box box1">1</div>
    <div class="box box2">2</div>
    <div class="box box3">3</div>
</div>

<!--父元素的兄弟元素-->
<div style="background-color: orange">
    lorem100
</div>
</body>
</html>
```

### 4.3 父元素 - 宽度依然束缚浮动元素

- 父元素是有宽度的，三个盒子的总宽如果大于了父元素的宽度，就会换行
- 父元素的宽度会束缚浮动的子元素的宽度

![image-20240728165437743](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240728165437743.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 600px;
            background-color: gray;
            border: 1px solid black;
        }

        .box {
            width: 220px;
            height: 100px;
            margin: 10px;
            background-color: skyblue;
            border: 1px solid black;
        }

        .box1 {
            float: left;
        }

        .box2 {
            float: left;
        }

        .box3 {
            float: left;
        }
    </style>
</head>
<body>

<div class="outer">
    <div class="box box1">1</div>
    <div class="box box2">2</div>
    <div class="box box3">3</div>
</div>
</body>
</html>
```

### 4.4 兄弟元素 - 浮动元素的兄弟元素被遮挡

- 浮动元素的前面的兄弟元素不会受到影响
- 浮动元素的后面的兄弟元素，会占据浮动元素之前的位置，在浮动元素的下面，同时后面的兄弟元素的文字会被甩出来

![image-20240728171532553](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240728171532553.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 1200px;
            background-color: gray;
            /*父元素不给高度*/
            border: 1px solid black;
        }

        .box {
            width: 100px;
            height: 100px;
            margin: 10px;
            background-color: skyblue;
            border: 1px solid black;
        }
        
        .box1 {
            float: left;
        }

        .box2 {
            float: left;
        }

        .box3 {
            float: left;
        }
    </style>
</head>
<body>

<!--父元素高度塌陷-->
<div class="outer">
    <!--子元素全部浮动-->
    <div class="box box1">1</div>
    <div class="box box2">2</div>
    <div class="box box3">3</div>

    <!--后面的兄弟元素-->
    <div class="box">4</div>
</div>

</body>
</html>
```

## 5. 消除浮动影响

- 消除影响1：子元素都浮动后，父元素高度别塌陷
- 消除影响2:   后面的兄弟元素被遮挡的问题

### 5.1 清除浮动

```bash
# 父元素中
- 前面几个子元素都浮动
- 后面的子元素的兄弟元素，可以选择清空上面几个浮动的影响
      # 清除浮动的元素，必须本身不能浮动
      # 清除浮动的元素，必须是块元素或者行内块元素
```

- 父元素高度塌陷：会被盒子4撑开，不会塌陷
- 子元素被遮挡：不会被遮挡，单独成为一行

![image-20240728174651349](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240728174651349.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 1200px;
            background-color: gray;
            border: 1px solid black;
            /*高度会被盒子4撑开*/
        }

        .box {
            width: 100px;
            height: 100px;
            margin: 10px;
            background-color: skyblue;
            border: 1px solid black;
        }

        .box1 {
            float: left;
        }

        .box2 {
            float: left;
        }

        .box3 {
            float: left;
        }

        .box4 {
            /*left：如果上面的子元素中有左浮的，清空浮动元素对后面元素的影响
              right：同理
              both：清空left和right*/
            clear: both;
        }
    </style>
</head>
<body>

<div class="outer">
    <!--子元素全部浮动-->
    <div class="box box1">1</div>
    <div class="box box2">2</div>
    <div class="box box3">3</div>
    <div class="box box4">4</div>
</div>

</body>
</html>
```

### 5.2 清除浮动 -- 设置魔法块

- 父元素中所有元素都浮动了，但是希望父元素不会塌陷
- 再加一个块元素，不给宽和高以及内容，自动撑起来

![image-20240728180341859](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240728180341859.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 1200px;
            background-color: gray;
            border: 1px solid black;
            /*高度会被盒子4撑开*/
        }

        .box {
            width: 100px;
            height: 100px;
            margin: 10px;
            background-color: skyblue;
            border: 1px solid black;
        }

        .box1 {
            float: left;
        }

        .box2 {
            float: left;
        }

        .box3 {
            float: left;
        }

        .magic {
            clear: both;
        }
    </style>
</head>
<body>

<div class="outer">
    <!--子元素全部浮动-->
    <div class="box box1">1</div>
    <div class="box box2">2</div>
    <div class="box box3">3</div>
    <div class="magic">4</div>
</div>

</body>
</html>
```

### 5.3 优化版本

- 针对上面方案的优化
- 布局原则：父元素中的子元素，要么都float，要么都不float，否则就会出现问题

![image-20240728180627486](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240728180627486.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 1200px;
            background-color: gray;
            border: 1px solid black;
        }

        /*在outer中设置一个空的*/
        .outer::after {
            content: '';
            display: block; /*默认是inline*/
            clear: both;
        }

        .box {
            width: 100px;
            height: 100px;
            margin: 10px;
            background-color: skyblue;
            border: 1px solid black;
        }

        .box1 {
            float: left;
        }

        .box2 {
            float: left;
        }

        .box3 {
            float: left;
        }
        
    </style>
</head>
<body>

<div class="outer">
    <!--子元素全部浮动-->
    <div class="box box1">1</div>
    <div class="box box2">2</div>
    <div class="box box3">3</div>
</div>

</body>
</html>
```



# 定位

## 1. 相对定位--position: relative

```bash
# position: relative;

# left right top bottom
- 1. 相对原来盒子的位置
- 2. 相对定位：元素没有脱离原来的文本流，下面的元素不会顶上来
- 3. 可以超出父容器的大小
- 4. 左和右不能同时用，左生效     上和下不能同时用，上生效
- 5. 上下：
        - 5.1 开启相对定位的元素会盖住下面或者上面的元素
        - 5.2 两个元素都开启定位了，后来者的权重大，会盖住前面的
        
# 应用场景
- 1. 对元素的位置进行微调，比如向右下角微调。 对其他元素不会影响
- 2. 99%场景，相对定位和绝对定位配合使用

# tip
- 一般不要把相对定位和margin配合
- 一般不要把相对定位和float配合
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .outer {
            width: 500px;
            background-color: gray;
            border: 1px solid black;
            padding: 20px;
        }

        .box {
            width: 200px;
            height: 200px;
            font-size: 20px;
        }

        .box1 {
            background-color: skyblue;
        }

        .box2 {
            background-color: orange;
            /*给盒子2开启相对定位*/
            position: relative;
            /*1. 相对原来盒子的位置，距离左边100px
              2. 相对定位：元素没有脱离原来的文本流，下面的元素不会顶上来
              3. 可以超出父容器的大小
              4. 左和右不能同时用，左生效
              5. */
            left: 550px;
        }

        .box3 {
            background-color: green;
        }
    </style>

</head>

<body>
<div class="outer">
    <div class="box box1">1</div>
    <div class="box box2">2</div>
    <div class="box box3">3</div>
</div>
</body>

</html>
```

![image-20240722094849459](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240722094849459.png)

## 2. 绝对定位--position: absolute

### 2.1 参考定位

- 会盖住其他元素

```bash
# 开启绝对定位的元素
- 参考点：包含块

# 包含块
- 没有脱离文档流的元素，   父元素就是包含块
- 脱离文档流的元素，第一个开启定位的祖先元素，就是他的包含块 （比如box2）

  - box2开启了绝对定位，脱离了文档流
     - 1. box2找outer，outer没开启相对定位，继续找body，继续找html，最终就是以html作为参考定位
     - 2. box2找outer，outer开启了相对定位，则就以outer为标准

# 子绝父相
- 父元素开启相对定位，但是不给上下左右的值，就不会变化
- 子元素开启绝对定位，就会相对父元素移动
```

- 左边是父元素没有开启相对定位，右边开启了

![image-20240722101049249](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240722101049249.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .outer {
            width: 500px;
            background-color: gray;
            border: 1px solid black;
            padding: 20px;
            /*父元素开启相对定位*/
            position: relative;
        }

        .box {
            width: 200px;
            height: 200px;
            font-size: 20px;
        }

        .box1 {
            background-color: skyblue;
        }

        .box2 {
            background-color: orange;
            /*给盒子2开启绝对定位*/
            position: absolute;
            /*距离左边*/
            left: 0;
        }

        .box3 {
            background-color: green;
        }
    </style>

</head>

<body>
<div class="outer">
    <div class="box box1">1</div>
    <div class="box box2">2</div>
    <div class="box box3">3</div>
</div>
</body>

</html>
```

## 3. 固定定位-position: fixed

- 脱离了文档流，定位参考方式：直接找视口
- 牛皮癣广告的右底脚，紧紧的贴在视口上

## 4. 粘性定位

## 5. 定位层级

- 开启了定位的，一定比普通的元素层级高

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>

        .outer {
            width: 500px;
            background-color: skyblue;
            border: 1px solid black;
            padding: 20px;
            position: relative;
        }

        .box {
            width: 200px;
            height: 200px;
            font-size: 20px;
        }

        .box1 {
            background-color: gray;
        }

        .box2 {
            background-color: orange;
            /*相对定位*/
            position: relative;
            top: -150px;
            left: 50px;
        }

        .box3 {
            background-color: green;
            /*绝对定位*/
            position: absolute;
            top: 130px;
            left: 130px;
            /*控制层级*/
            z-index: 10;
        }

        .box4 {
            background-color: red;
            /*固定定位*/
            position: fixed;
            top: 200px;
            left: 200px;
        }
    </style>

</head>

<body>
<div class="outer">
    <div class="box box1">1</div>
    <div class="box box2">2</div>
    <div class="box box3">3</div>
    <div class="box box4">4</div>
</div>
</body>

</html>
```

![image-20240722134315123](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240722134315123.png)

# Flex布局

## 1. 简介

- Flexible Box, 弹性盒子，伸缩盒模型
- 盒子元素分布方式，盒子元素对齐方式，盒子元素视觉顺序
- 随着伸缩盒的出现，逐渐演变出一套新的布局方案：flex布局
- 在移动端应用比较广泛，因为传统布局不能很好的呈现在移动设备上

## 2. 伸缩容器与伸缩项目

- 伸缩容器：外面包裹的，大的伸缩容器
- 伸缩项目：伸缩容器中的东西

### 2.1 普通容器

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 600px;
            height: 300px;
            background-color: gray;
        }

        .inner {
            width: 50px;
            height: 50px;
            background-color: skyblue;
            border: 1px solid black;
            box-sizing: border-box; /*怪异盒子*/
        }
    </style>
</head>
<body>

<div class="outer">
    <div class="inner">1</div>
    <div class="inner">2</div>
    <div class="inner">3</div>
</div>
</body>
</html>
```

![image-20240721094759835](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721094759835.png)

### 2.2 伸缩容器 -- display: flex

- 只需要在outer中定义了该属性，则该盒子就变成了伸缩容器
- 里面的伸缩项目，就会沿着主轴方向排列

```css
     .outer {
            width: 600px;
            height: 300px;
            background-color: gray;
            display: flex;   <!--开启了flex布局-->
        }
```

![image-20240721095005364](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721095005364.png)

- 如果outer父元素不给高度，则会用内部的自动撑大

![image-20240721095127813](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721095127813.png)

### 2.3 伸缩容器的子元素是伸缩项目

- 对当前容器开启了Flex布局后，里面所有的子元素，都变成了flex项目，但孙子元素，不是
- 如果想要孙子元素还能是flex项目，对该孙子元素的父元素，同样开启flex布局即可

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 600px;
            background-color: gray;
            display: flex;
        }

        .inner {
            width: 50px;
            height: 50px;
            background-color: skyblue;
            border: 1px solid black;
            box-sizing: border-box;
        }
    </style>
</head>
<body>

<div class="outer">
    <div class="inner">1</div>
    <div class="inner">2</div>
    <div class="inner">
        <div>a</div>
        <div>b</div>
        <div>c</div>
    </div>
</div>
</body>
</html>
```

![image-20240721095238467](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721095238467.png)

### 2.4 伸缩容器内都是块元素

- 一旦一个容器开启了flex布局，里面所有的项目都变成了块元素
- 块元素就可以设置宽高，比如伸缩容器中非span元素

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 600px;
            height: 200px;
            background-color: gray;
            display: flex;
        }

        .inner {
            width: 50px;
            height: 50px;
            background-color: skyblue;
            border: 1px solid black;
            box-sizing: border-box;
        }
    </style>
</head>
<body>

<div class="outer">
    <!--span本来是行内元素，但是变成了flex项目，宽高依然起作用-->
    <span class="inner">1</span>
    <span class="inner">2</span>
    <span class="inner">3</span>
</div>
</body>
</html>
```

![image-20240721095517901](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721095517901.png)

- 可以通过浏览器的Computed属性，查看浏览器计算后的，该元素的显示方式是block，即块元素

![image-20240721095651110](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721095651110.png)

## 3. 主轴

- 关注的是方向，主轴和侧轴垂直
- 主轴：红色，默认水平，从左到右
- 侧轴：蓝色，默认垂直，从上到下
- 伸缩元素会在主轴上进行排列

![image-20240721095952443](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721095952443.png)

### 3.1 主轴切换--flex-direction

- 伸缩容器的主轴
- 切换后，一般侧轴跟着变化

#### row

```bash
# 默认值
# 主轴： 水平，从左到右
# 侧轴： 垂直，从上到下
flex-direction: row;
```

![image-20240721100345534](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721100345534.png)

#### row-reverse

```bash
 # 主轴：水平，从右到左 
 # 侧轴：垂直，从上到下
 flex-direction: row-reverse;
```

![image-20240721100442980](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721100442980.png)

#### column

```bash
 # 主轴：垂直，从上到下
 # 侧轴：水平，从左到右
 flex-direction: column;
```

![image-20240721100544239](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721100544239.png)

#### column-reverse

```bash
# 主轴：垂直，从下到上
# 侧轴：水平，从左到右
flex-direction: column-reverse;
```

![image-20240721100639664](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721100639664.png)

### 3.2 主轴换行--flex-wrap

- 当伸缩容器中，沿着主轴的伸缩项目很多时，换行的处理方式

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .outer {
            width: 200px;
            height: 200px;
            background-color: gray;
            display: flex;
            flex-direction: row;
            margin: 0 auto; /*让伸缩容器居中*/
        }

        .inner {
            width: 50px;
            height: 50px;
            background-color: skyblue;
            border: 1px solid black;
            box-sizing: border-box;
        }
    </style>
</head>
<body>

<div class="outer">
    <span class="inner">1</span>
    <span class="inner">2</span>
    <span class="inner">3</span>
    <span class="inner">4</span>
    <span class="inner">5</span>
</div>
</body>
</html>
```

#### nowrap

```bash
#  默认值：不换行
#  压缩：各个元素挤一下，宽度变小了
flex-wrap: nowrap
```

![image-20240721100919987](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721100919987.png)

#### wrap

```bash
# 主轴：不会压缩宽度，排列满
# 侧轴：如果侧轴还有空间，则会铺张浪费。没有的话，就不允许铺张浪费了
flex-wrap: wrap
```

![image-20240721101104591](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721101104591.png)

![image-20240721101200104](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721101200104.png)

#### wrap-reverse

```bash
 # 后来居上
 flex-wrap: wrap-reverse;
```

![image-20240721101225474](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721101225474.png)

### 3.3 主轴对齐--justify-content

- 元素在主轴的对齐方式
- 可以在子元素中设置margin，实现微调

#### flex-start

```bash
# 默认值，靠左对齐
justify-content: flex-start;
```

![image-20240721101933412](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721101933412.png)

#### flex-end

```bash
 # 靠右对齐
 justify-content: flex-end;
```

![image-20240721101954414](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721101954414.png)

#### center

```bash
# 居中对齐
justify-content: center;
```

![image-20240721102015354](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721102015354.png)

#### space-around

```bash
# 伸缩项目均匀的分布在一行中
# 项目与项目之间的距离，是项目距离边缘的二倍
 justify-content: space-around;
```

![image-20240721102044149](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721102044149.png)

#### space-between

```bash
# 伸缩项目均匀的分布在一行中
# 第一个和最后一个紧贴边缘，项目与项目之间距离是相等的
 justify-content: space-between;
```

![image-20240721102115714](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721102115714.png)

#### space-evenly

```bash
# 伸缩项目均匀的分布在一行中
# 项目与项目，项目和边缘，距离都是一样的
  justify-content: space-evenly;
```

![image-20240721102150709](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721102150709.png)

## 4. 侧轴对齐

- 在侧轴方向上的，对齐方式

### 4.1 单行对齐--align-items

```css
<style>
      .outer {
          width: 1000px;
          height: 600px;
          background-color: gray;
          margin: 0 auto; 

          display: flex;
          /*默认主轴方向*/
          flex-direction: row;
          flex-wrap: wrap;
          justify-content: flex-start;
          align-items: flex-start;
      }

      .inner1 {
          width: 200px;
          height: 200px;
          background-color: skyblue;
          border: 1px solid black;
          box-sizing: border-box;
      }

      .inner2 {
          width: 200px;
          height: 300px;
          background-color: skyblue;
          border: 1px solid black;
          box-sizing: border-box;
      }

      .inner3 {
          width: 200px;
          height: 100px;
          background-color: skyblue;
          border: 1px solid black;
          box-sizing: border-box;
      }
  </style>
```

#### flex-start

```bash
# 侧轴顶端对齐
align-items: flex-start;
```

![image-20240721102502932](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721102502932.png)

#### flex-end

```bash
# 侧轴底端对齐
 align-items: flex-end;
```

![image-20240721102523784](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721102523784.png)

#### center

```bash
 # 侧轴中间对齐
 align-items: center;
```

![image-20240721102546046](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721102546046.png)

#### stretch

```bash
# 伸缩项目不能给高度才生效
# 会自动将项目的高度拉伸到伸缩容器的高度
# 默认值：必须不能给项目高度才可以
align-items: stretch;
```

![image-20240721102623732](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721102623732.png)

### 4.2 多行对齐--align-content

- 和上面的属性二选一
- 必须开启主轴换行

#### flex-start

```bash
# 侧轴：紧紧贴合，从上到下
align-content: flex-start;
```

![image-20240721103011623](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721103011623.png)

#### flex-end

```bash
# 侧轴：紧紧贴合，从下到上
align-content: flex-end;
```

![image-20240721103036523](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721103036523.png)

#### center

```bash
# 侧轴：紧紧贴合，居中对齐
align-content: center;
```

![image-20240721103132667](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721103132667.png)

#### sapce-around

```bash
 # 伸缩项目之间距离是相等的，伸缩项目和边缘之间是1/2
 align-content: space-around;
```

![image-20240721103209999](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721103209999.png)

#### space-between

```bash
  # 紧贴边缘，伸缩项目之间距离等分
  align-content: space-between;
```

![image-20240721103258319](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721103258319.png)

#### space-evenly

```bash
# 边缘，项目之间距离等等分
align-content: space-evenly;
```

![image-20240721103345748](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721103345748.png)

## 5. 伸缩性

### 5.1 伸--flex-grow

```bash
# 伸
- 声明在伸缩项目中，该伸缩项目就参与了伸

# 伸法则
- 检查当前主轴的剩余长度
- 每个伸缩项目可以定义权重，flex-grow: 1; 通过计算权重后，进行伸
- 默认是0,不会伸缩

# 一般情况
- 各个伸缩项目的权重定义成一个，有钱了大家一起分
```

```bash
# 声明在伸缩项目中，而不是伸缩容器中
.inner1 {
            flex-grow: 1;
        }
```

![image-20240721090848699](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721090848699.png)

#### 水平划分

![image-20240723122952843](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240723122952843.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .outer {
            display: flex;
            height: 400px;
            background-color: gray;
            /*宽度默认是铺满整个视口*/
        }

        .inner {
            height: 200px;
        }

        .inner1 {
            /*默认值*/
            flex-grow: 0;
            background-color: skyblue;
            /*给定宽度，也可以通过给padding的方式来定型宽度*/
            width: 100px;
        }

        .inner2 {
            /*权重为1，其他两个是0，则就只会把当前的伸展开
            不用专门给宽度*/
            flex-grow: 1;
            background-color: green;
        }

        .inner3 {
            flex-grow: 0;
            background-color: orange;
            /*给定宽度*/
            width: 100px;
        }
    </style>
</head>
<body>
<div class="outer">
    <div class="inner inner1">1</div>
    <div class="inner inner2">2</div>
    <div class="inner inner3">3</div>
</div>
</body>
</html>
```

### 5.2 缩-flex-shrink

- 只有当父元素的宽度小于各个伸缩项目的宽度和时，才会缩
- 必须设置成换行方式为nowrap才可以缩，不然就换行了
- 默认值为1

```bash
# 父元素宽度不够了，压缩时
- 能力越大，职责越大，权重
- 尽可能保证各个伸缩项目都能显示
```

## 6. 伸缩项目

### 6.1 排序-order

```bash
# 在伸缩项目中定义
- order默认为0，按照声明顺序在主轴排序

#  order: -1;
- order值越小的，越往主轴开始方向排列
```

![image-20240721093453108](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721093453108.png)

### 6.2 单独对齐-align-self

- 某个元素，希望可以单独在侧轴对齐
- 声明在伸缩项目中

```bash
        .inner2 {
            width: 300px;
            flex-grow: 1;
            align-self: flex-end;
        }
```

![image-20240721093802660](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721093802660.png)

# 布局技巧

## 1. 子元素在父元素中：水平垂直居中

- 父子元素都是块级元素
- 父元素：水平居中
- 子元素：在父元素的水平居中，垂直居中

![image-20240721202131991](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240721202131991.png)

### 1.1 margin

- 如果有border和padding的话，要考虑一下盒子撑大问题

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .d1 {
            width: 400px;
            height: 200px;
            background-color: gray;
            /*父元素位置调整：水平居中，贴着顶端*/
            margin: 0 auto;
            /*解决margin塌陷问题*/
            overflow: hidden;
        }

        .d2 {
            width: 100px;
            height: 100px;
            background-color: skyblue;

            /*子元素位置调整：
            水平居中，贴着顶端*/
            margin-left: auto;
            margin-right: auto;
            /*具体的top和bottom，根据盒子大小来计算*/
            margin-top: 50px;
            margin-bottom: 50px;
        }
    </style>

</head>
<body>

<div class="d1">
    <div class="d2"></div>
</div>

</body>
</html>
```

### 1.2 父元素flex，子元素主轴和侧轴居中对齐

```html
<style>
    .d1 {
        width: 400px;
        height: 200px;
        background-color: gray;
        /*父元素位置调整：水平居中，贴着顶端*/
        margin: 0 auto;
        /*flex布局*/
        display: flex;
        /*主轴居中*/
        justify-content: center;
        /*侧轴居中：单行*/
        align-items: center;
    }

    .d2 {
        width: 100px;
        height: 100px;
        background-color: skyblue;

    }
</style>
```

### 1.3 父元素flex，子元素margin:auto

```html
<style>
    .d1 {
        width: 400px;
        height: 200px;
        background-color: gray;
        /*父元素位置调整：水平居中，贴着顶端*/
        margin: 0 auto;
        /*flex布局*/
        display: flex;
    }

    .d2 {
        width: 100px;
        height: 100px;
        background-color: skyblue;
        /*四个margin都是auto*/
        margin: auto;
    }
</style>
```

![image-20240720111731254](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240720111731254.png)

# 布局

## 1. 页头/内容/页脚

### 1.1 Flex布局

- 页头+内容+页脚
- 利用Flex的垂直分布，和项目的伸缩性，随着内容的增多，页脚自动向下移动

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>

        body {
            /*防止一开始页面就有滚动条*/
            margin: 0;
        }

        .outer {
            /*开启Flex布局*/
            display: flex;
            /*主轴方向为列*/
            flex-direction: column;
            /*高度为视口的100%高度*/
            height: 100vh;
        }

        .header {
            background-color: skyblue;
            /*1. 通过设置padding，高度就是padding+header的高度
              2. 不要用指定height的方式，因为主体内容的扩张，会导致header的指定形式的height变小*/
            padding: 10px;
            text-align: center;
        }

        .content {
            background-color: pink;
            /*主体区域可伸缩，
            1. 初始时，先伸，占据剩余空间
            2. 后续扩张时，会使用默认值 flex-shrink:1*/
            flex-grow: 1;
        }

        .footer {
            padding: 10px;
            background-color: gray;
            text-align: center;
        }
    </style>

</head>
<body>

<div class="outer">
    <div class="header">我是页头</div>
    <div class="content"> lorem2000
    </div>
    <div class="footer">我是页脚</div>
</div>

</body>
</html>
```



# WebStorm快捷键

## 1. 快速生成文字

- 在div中，lorem20 + tab： 快速生成20个单词

## 2. 行复制

- 鼠标放在某一行代码上，CTRL+C复制整行
