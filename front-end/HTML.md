

# 简介

## 1. 网页

- 网页是构成网站的基本元素，包含：图片，链接，文字，声音，视频等元素。一般是.html或.htm文件，统称为HTML文件
- HTML：超文本标记语言，Hyper Text Markup Language，用来描述网页的一种语言，由标记标签构成

## 2. 浏览器

- 浏览器内核：渲染引擎：读取网页内容，整理讯息，计算网页的显示方式并显示页面
- 目前国内一般浏览器采用Wenkit/Blink内核，如360,UC,QQ,搜狗等

![image-20230629060813777](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230629060813777.png)

## 3. Web标准

- 由W3C组织和其他标准化组织，指定的一些列标准的集合。W3C(万维网联盟)是国际著名的标准化组织

### 3.1 原因

- 浏览器不同，显示页面或者排版会有所差异
- 遵循Web标准，可以使开发出来的网页更加标准，统一

```bash
- 让web的发展前景更加广阔
- 内容能被更广泛的设备访问
- 更容易被搜索引擎搜索
- 降低网站流量费用
- 使网站更易于维护
- 提高页面浏览速度
```

### 3.2 构成

- 结构：对网页元素进行整理和分类，主要是HTML
- 表现：设置网页元素的版式，颜色，大小等外观，主要是CSS
- 行为：网页模型的定义及交互，主要是Javascript



# 骨架标签

| 标签名           | 定义     | 说明                    |
| ---------------- | -------- | ----------------------- |
| \<html>\<html>   | HTML标签 | 页面中最大标签，根标签  |
| \<head>\<head>   | 文档头部 | 必须在其中设置title标签 |
| \<title>\<title> | 文档标题 | 页面的网页标题          |
| \<body>\<body>   | 文档主体 | 页面所有内容            |

```html
<!--- !DOCTYPE html: 文档类型声明，告诉浏览器使用哪种HTML版本来显示网页
  1. <!DOCTYPE html>就是指当前页面采取的是HTML5来显示网页
  2. 位于文档中的最前面的位置，处于<html>标签之前
  3. <!DOCTYPE html>不是一个HTML标签，是文档类型声明标签-->
<!DOCTYPE html>

<!--lang: 定义当前文档显示的语言
   1. en：英语， zh-CN中文
   2. 对于文档显示来说，定义为en的文档可以显示中文，定义为zh-CN的文档可以显示英文-->
<html lang="en">
<head>
    <!--charset： 当前文档的字符集
      1. 一定要写，不然会出现乱码-->
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
大家好
</body>
</html>
```

# 常用标签

## 1. h1-h6

- head的缩写，字体加粗，1-6递减
- 一个标题独占一行
- 不允许互相嵌套

```html
<h1>你好</h1>
<h6>haha</h6>
```

## 2. p

- 分段必须借助标签，不能使用空格
- 分段标签会根据浏览器大小自动识别匹配，段落间会有空隙

```html
<p>
    hello
</p>
<p>
    word
</p>
shuzhan<br/>
```

## 3. 语义标签

- 标签默认效果不重要，语义最重要，因为具体的样式可以用css来调

```bash
# 语义
- 代码可读性
- 有利于SEO(搜索引擎优化)， 
- 其他设备(屏幕阅读器，盲人阅读器)

# SEO： 百度搜索
- 查看所有的网页中的比如h1标签，然后进行保存
- 后续其他人搜索的时候，就可以推送当前网页
```

| 语义   | 标签                          | Desc           |
| ------ | ----------------------------- | -------------- |
| 加粗   | \<strong>\<strong> 或\<b>\<b> | 推荐使用strong |
| 倾斜   | \<em>\<em> 或\<i>\<->         | 推荐使用em     |
| 删除线 | \<del>\<del> 或\<s>\<s>       | 推荐使用del    |
| 下划线 | \<ins>\<ins> 或\<u>\<u>       | 推荐使用ins    |

## 4. div和span

- 盒子标签，没有语义，用来装具体的内容
- 一个div独占一行，超大盒子
- span：一行可以放多个
- 文字没有被包裹，会独占一行，比如 123
- 块级元素：独占一行
- 行内元素：不独占一行

```html
<!--123会分行显示-->
<div>独占一行</div>123
<div>独占一行</div>123
<!--一行可以放多个span-->
<span>a</span>
<span>b</span>
<span>c</span>
```

## 5. 图片标签

- 标签对应的属性的顺序：可以随便顺序
- 对应的src可以传递本项目中的图片(本地路径)，也可以保存图片对应的url(更常用)

```html
<!--src是必须写的， title：当鼠标移动到图片上，进行文字提示-->
<img src="../image/1.jpg" title="mac桌面"><br/>

<!--当图片无法显示，alt 进行文字提示-->
<img src="../image/2.jpg" alt="图片无法加载">
```

```html
<!-- width和height一般只定义一个，两个都定义，可能把图片压扁
      一般不要设置高宽，遵从图片原来的大小，强制调整后会造成图片虚化
      border：加上边框，一般使用css来对其进行修饰 -->
<img src="../image/1.jpg" title="mac桌面" width="500" border="15"><br/>
```

## 6. 超链接

```html
<!--a: 超链接格式
    1. href：必须写，
            内部链接：      本项目中的其他html网页
            外部链接：      其他项目的链接
            空链接：        如果没有明确链接目标时，可以用#来代替，对应页面可以点击，但是不会跳转
            下载链接：      如果href里面地址是一个文件或压缩包(zip，.exe等文件)，则会下载这个文件
            网页元素：      在网页中的各种网页元素，如文本，图片，表格，视频，音频等：   浏览器直接放

    target：打开链接的方式
          _blank：新页面打开
          _self:  当前页面打开，默认配置   -->
<a href="first.html" target="_self">内部链接</a><br/>
<a href="https://www.baidu.com/" target="_blank">百度一下</a><br/>
<a href="#" target="_self">空链接</a><br/>
<a href="../image/JVM.pdf.zip" target="_blank">下载文件</a><br/>
<a href="../image/1.jpg" target="_blank">网页元素</a><br/>
<a href="../image/1.jpg" target="_blank" download="图图">强制下载</a><br/><!--不再让浏览器打开，强制下载，可以指定下载文件名-->
```

## 7. 锚点链接

- 一个网页内部的，文本链接快速定位

```html
<!--href：#开头，定位到id为对应数据的文本-->

<div id="top">
    <a href="#life">个人简介</a>
    <a href="#product">主要作品</a>
</div>


<h1>1</h1>
<h1>1</h1>
<h1>1</h1>
<h1>1</h1>
<h1>1</h1>


<div id="life">
    <p>刘德华（Andy Lau），1961年9月27日出生于香港新界大埔镇泰亨村，华语影视男演员、歌手、制片人、作词人。</p>
    <p> 1981年出演电影处女作《彩云曲》。1983年主演的武侠剧《神雕侠侣》在香港取得62点的收视纪录。</p>
    <p>1985年发行首张个人专辑《只知道此刻爱你》</p>
</div>
<a href="#top">回到顶部</a>

<h1>1</h1>
<h1>1</h1>
<h1>1</h1>
<h1>1</h1>
<h1>1</h1>


<div id="product">
    <p>第一个作品</p>
    <p>第二个作品</p>
    <p>第三个作品</p>
</div>
<a href="#top">回到顶部</a>
```

## 8. 特殊字符

- 书写的时候，必须加；，来进行特殊字符处理
- 代码中，多个空格，只会解析成一个

![image-20230702113229316](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230702113229316.png)

## 9. 其他

```bash
# 换行：单标签：         不要用br来增加文本之间的间隔，应该使用css的margin
<br/>    <br>

# 分割线: 屏幕分割。    如果不想要语义，请使用css 没完成
<hr/>    <hr>
```

```html
<!--按照下面定义的原文显示： 不会删除多余的空格-->
<pre>
    love    love
          love      love
</pre>
```



# Table

## 1. 基本表格

```html
<table>
    <tr>
        <td>姓名</td>
        <td>年龄</td>
        <td>性别</td>
    </tr>
    <tr>
        <td>张三</td>
        <td>20</td>
        <td>男</td>
    </tr>
    <tr>
        <td>李四</td>
        <td>21</td>
        <td>女</td>
    </tr>
</table>
```

## 2. 表头单元格

- th也是普通的td，不过文字会加粗和居中

```html
<table>
    <tr>
        <th>姓名</th>
        <th>年龄</th>
        <th>性别</th>
    </tr>
    <tr>
        <td>张三</td>
        <td>20</td>
        <td>男</td>
    </tr>
    <tr>
        <td>李四</td>
        <td>21</td>
        <td>女</td>
    </tr>
</table>
```

## 3. 结构标签

```html
<table>
    <thead>
    <tr>
        <th>姓名</th>
        <th>年龄</th>
        <th>性别</th>
    </tr>
    </thead>

    <tbody>
    <tr>
        <td>张三</td>
        <td>20</td>
        <td>男</td>
    </tr>
    <tr>
        <td>李四</td>
        <td>21</td>
        <td>女</td>
    </tr>
    </tbody>

    <!--表脚注-->
    <tfoot>
    <td></td>
    <td></td>
    <td>summary</td>
    </tfoot>
</table>
```

## 4. 单元格合并

```bash
# 1. 确定要合并的单元格
# 2. 确定是跨行还是跨列
# 3. 删除多余的单元格
```

```html
<table border="1">
    <thead>
    <tr>
        <th>姓名</th>
        <th>年龄</th>
        <th>性别</th>
    </tr>
    </thead>

    <tbody>
    <tr>
        <td rowspan="2">张三</td>
        <td>20</td>
        <td>男</td>
    </tr>
    <tr>
        <td>21</td>
        <td>女</td>
    </tr>
    </tbody>

</table>
```

# List

## 1. 无序列表

- ul里面只能放li，不允许放其他标签
- li里面可以放任何元素

```html
<ul>
  <li>apple</li>
  <li>peach</li>
  <li>melon</li>
</ul>
```

## 2. 有序列表

```html
<ol>
    <li>apple</li>
    <li>peach</li>
    <li>melon</li>
</ol>
```

## 3.自定义列表

- 用来对术语或名次进行解释和描述，自定义列表的列表项前没有任何符号
- dl中只能包含dd和dt，可能包含多个dd和dt

```html
<dl>
    <dt>小米之家</dt>
    <dd>线下门店</dd>
    <dd>淘宝商城</dd>
    <dd>京东商城</dd>
    <dd>拼爹爹商城</dd>

    <dt>华为之家</dt>
    <dd>线下门店</dd>
    <dd>淘宝商城</dd>
    <dd>京东商城</dd>
    <dd>拼爹爹商城</dd>
</dl>
```

![image-20240615162649854](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240615162649854.png)

## 4. 注意事项

```html
<ul>
    <li>apple</li>
    <li>peach</li>
    <!--1. 列表中最好使用li，如果需要其他的，则使用嵌套的方式-->
    <li>
        <a href="first.html">点击去</a>
    </li>
</ul>

<!--2. 最好不要在ul之外去使用li-->
<!--3. ul中可以进行列表的嵌套-->
```

# Form

## 1. 基础表单

```html

<!--1. action: 表单具体提交给哪个接口来处理
    2. method: 提交方法
    3. name: 表单域的id，来进行识别
    4. target: 表单跳转的方式-->
<form action="https://www.baidu.com/s"
      method="get"
      name="firstFrom"
      target="_blank">
    <!--上面接口的api-->
    <input type="text" name="wd">
    <!--button：提交表单，点击后，会触发表单的action进行跳转，并携带input框的输入数据-->
    <button>去百度搜索</button>
</form>

<hr>

<form action="https://re.jd.com/search"
      method="get"
      name="firstFrom"
      target="_blank">

    <input type="text" name="keyword">
    <button>去京东搜索</button>
</form>
```

## 2. input标签

- 不同的type，对应不同的控件

```html
<form action="https://search.jd.com/search"
      method="get"
      name="firstFrom"
      target="_blank">

    <!--1. type: 指定是text
        name: 传递过去的属性key
        placeholder：提示值
        value: 默认值，后续输入新的值，value就会替换成新值，不写的话，初始化为''
                      一般和placeholder二选一 -->
    姓名： <input type="text" name="username" value="lisi" maxlength="10"><br>
    密码： <input type="password" name="pwd" placeholder="input your name" maxlength="6"><br>

    <!-- 2. 单选框
             2.1 只有name是同一个时，才能实现单选框
             2.2 必须加value，value是传递的实际的值,否则该值就只是 on-->
    性别：
    <input type="radio" name="gender" value="boy">男
    <input type="radio" name="gender" value="girl">女
    <input type="radio" name="gender" value="ghost">人妖
    <br>

    <!--3. 多选框:  hobby=smoke&hobby=drink
           3.1 只有name是同一个时，才能实现多选框
           3.2 必须指定value，value是传递的实际的值,否则该值就只是 on
            checked="checked": 默认勾选 -->
    爱好：
    <input type="checkbox" name="hobby" value="smoke">抽烟
    <input type="checkbox" name="hobby" value="drink">喝酒
    <input type="checkbox" name="hobby" value="dohair" checked="checked">烫头
    <br>

    <!--4. select: hometown=shanxi    (name=value), 如果没有value，则会取文字 ‘河北’
            4.1 selected：默认选中-->
    籍贯:
    <select name="hometown">
        <option value="hebei">河北</option>
        <option value="shanxi" selected>陕西</option>
        <option value="beijing">北京</option>
        <option value="guangdong">广东</option>

    </select>
    <br>

    <!--5. 文本域-->
    详细信息：<textarea name="other" cols="23" rows="3"></textarea>
    <br/>

    <!--4. 隐藏域: 用户界面不显示，但是请求路径中会包含 from=%27erick-->
    <input type="hidden" name="from" value="'erick"><br>

    <br>

    <!--button-->
    <!--1. 普通按钮：只是想触发某些js的功能-->
    <button type="button" onclick={console.log('haha')}>点我笑</button>
    <input type="button" onclick={console.log('哈哈哈哈哈哈')} value="点我疯狂笑">
    <br>

    <!--2. 提交表单-->
    <!--2.1 方式一：button: 默认就是submit-->
    <button type="submit">去京东搜索</button>
    <!--2.2 方式二：
        type=submit
        value: 按钮的文字-->
    <input type="submit" value="开始搜索">
    <br>

    <!--3. 重置表单-->
    <!--3.1 方式一：button: rest-->
    <button type="reset">重置一</button>
    <!--3.2 方式二：
        type=submit
        value: 按钮的文字-->
    <input type="reset" value="重置二">
</form>
```

## 3. 禁用表单

```html
<form action="https://search.jd.com/search"
      method="get"
      name="firstFrom"
      target="_blank">

    <!--1. 禁用表单： 该文本框就不能修改了，基本可以在很多表单控件中使用
            应用场景：修改时，给了默认值，不能修改-->
    姓名： <input disabled type="text" name="username" value="lisi" maxlength="10"><br>

    <button type="submit">去京东搜索</button>
</form>
```

## 4. label属性

- 经常一起和input标签来使用
-  用户点击文本时候，就能锁定到输入框，而不用非得点击到文本输入框或单选多选按钮，增加用户体验

```html
<form action="first.html" method="get" name="fifth">

    <!--方式一：for必须和对应input框的id对应起来-->
    <label for="usr">姓名:</label>
    <input id="usr" type="text" name="username" placeholder="请输入用户名"><br/>

    <!--方式二：-->
    <label>
        地址:
        <input type="text" name="address" placeholder="请输入地址"><br/>
    </label>


    爱好:
    <label for="swim_id">游泳</label>
    <input id="swim_id" type="checkbox" name="hobby">
    <label for="sing_id">唱歌</label>
    <input id="sing_id" type="checkbox" name="hobby">
    <label for="basketball_id">打篮球</label>
    <input id="basketball_id" type="checkbox" name="hobby">

    <br>

    其他爱好:
    <label>
        抽烟<input type="checkbox" name="other">
    </label>

    <label>
        喝酒<input type="checkbox" name="other">
    </label>

    <label>
        烫头<input type="checkbox" name="other">
    </label>

</form>
```
