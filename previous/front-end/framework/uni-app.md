# 微信小程序

- [微信小程序官方文档](https://developers.weixin.qq.com/platform/)

## 申请小程序ID

- 一个邮箱只能申请一个
- 该小程序ID会在后面代码配置中使用
- [小程序控制台](https://mp.weixin.qq.com/wxamp/home/guide?token=99859328&lang=zh_CN)

## 提交备案



# Webstorm

## Uniapp Tool

- 安装对应的插件

## 微信小程序开发者工具

[官网下载](https://developers.weixin.qq.com/miniprogram/dev/devtools/stable.html)

# 项目创建

## 1. 命令行创建

```bash
# 全局安装vue/cli
sudo npm install -g @vue/cli

# 安装TS-VUE的项目
npx degit dcloudio/uni-preset-vue#vite-ts princess-kitchen
```

## 2. manifest.json

- 填写其中的appId

```json
"mp-weixin" : {
    "appid" : "wxf8642be2dd3db102",
    "setting" : {
        "urlCheck" : false
    },
    "usingComponents" : true
},
```

## 3. 项目运行

- 在webstorm中运行启动项目

```bash
# 以dev 方式运行好微信小程序，热更新
# 会在项目名/dist/生成对应的微信小程序的代码
npm run dev:mp-weixin
```

## 4. 微信开发者工具打开

- 上面填写了微信小程序ID，因此会自动识别

![image-20250302112523333](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250302112523333.png)

## 5. preview

- 点击preview，即可通过扫码在手机端查看

![image-20250302113008845](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250302113008845.png)

# 项目上线

## 1. 打包

```bash
npm run build:mp-weixin
```

## 2. 微信小程序

### 导入build

![image-20250302120338964](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250302120338964.png)

### 上传

![image-20250302120512762](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250302120512762.png)

## 提交审核

- [审核地址](https://mp.weixin.qq.com/wxamp/wacodepage/getcodepage?token=99859328&lang=zh_CN)
- 按照指示完成其他基本的验证

![image-20250302120840402](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250302120840402.png)

## 服务器相关

- 微信小程序，访问后端
- 必须是域名+https的方式

```bash
# 端口
- https的默认端口是443，可以直接使用，从而在代码中不必显式写端口

# 建议使用443端口，除非端口被占用
```



# UI组件

## view

- 小程序的专门的组件，类似于div，并不会有任何的



## uni-ui

- [组件UI官网](https://zh.uniapp.dcloud.io/component/)

```bash
npm i @dcloudio/uni-ui
npm i sass
```

- 在page.json中配置，修改完page.json后，重启服务才能生效

```json
  "easycom": {
    "autoscan": true,
    "custom": {
      // uni-ui 规则如下配置
      "^uni-(.*)": "@dcloudio/uni-ui/lib/uni-$1/uni-$1.vue"
    }
  },
```

# 开发常见

## 骨架屏

- 对当前制作好的页面，生成一个骨架屏
- 如果页面进入慢的时候，生成对应的页面，类似于loading页面

### 1. 生成

- 在微信小程序中，生成当前页面的骨架屏，文件只会存在于微信小程序开发者工具端
- index.skeleton.wxss，为对应的模版html
- index.skeleton.wxss， 为对英的模版css

### 2. Vue文件

- 在webstorm中，创建文件，将上面的html复制，css放在style中复制

### 3. 互斥配置

- 在对应的组件中，数据加载完毕后再渲染页面，否则渲染骨架屏

## 分包/预下载

```bash
# 分包
- 将小程序的代码分割为多个部分，分别打包成多个小程序包，减少小程序的加载时间，提高用户体验

# 分包预下载
- 在进入小程序某个页面时，框架自动预下载可能需要的分包，提升进入后续分包页面时候的启动速度
```

### 目录

- 在src下面

![image-20250313123827696](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250313123827696.png)

### page.json配置

```json
  /*子包配置*/
  "subPackages": [
    /*子包一*/
    {
      /*子包的根路径*/
      "root": "pagesMember",
      /*页面路径和窗口表现*/
      "pages": [
        {
          "path": "settings/settings",
          "style": {
            "navigationBarTitleText": "设置"
          }
        }
      ]
    }
  ],
  /*分包预下载规则*/
  "preloadRule": {
    /*进入主包某个页面时*/
    "pages/user/user": {
      /*所有网络，可设置为wifi*/
      "network": "all",
      "packages": ["pagesMember"]
    }
  }
```

### 检验

- 点击其他页面，分包不会预下载
- 点击到目标页面时候，控制台会有
- 分包体积越大，效果越好

```bash
preloadSubpackages: pagesMember
preloadSubpackages: success
```

