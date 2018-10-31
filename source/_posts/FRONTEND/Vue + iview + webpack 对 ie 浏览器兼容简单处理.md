---

title: Vue + iview + webpack ie 浏览器兼容简单处理
date: 2018-10-31 14:36:05
tags: Front end
categories: frontend

---

# Vue + iview + webpack ie 浏览器兼容简单处理

<!-- more -->

## 环境介绍：

vue: ^2.5.2

iview: ^3.1.0

Webpack: ^3.8.1

## 前情提要：

*   ie 浏览器不支持 ES6 Promise 的语法。

*   ie8 及以下对 html5 标签不兼容（可以通过引入html5shiv包解决，本文不处理IE11的更低版本，故不提及此法）。

*   ie9 以下 对 CSS3 的不兼容，各种不兼容的细节比较多，这里不说明。

*   ie10 及以下浏览器中不支持 dataset 方法（经学习实践发现ie11也是不支持的），而我在项目中使用了 iview, iview 的一些组件用到了这个方法。

*   ie 浏览器是非 webkit 内核，不支持 display: -webkit-box; 和相关样式；

*   Polyfill 是 shim 的一种，但他的 API 是遵循标准的。polyfill 的做法通常是：先检查浏览器是否支持某个标准 API,如果不支持,就使用旧的技术对浏览器做兼容处理,这样就可以在旧的浏览器上使用新的标准 API。比如,旧浏览器不支持 ES6 的 Array.prototype.find 方法,我们想要在项目中使用 Array.prototype.find,只要 polyfill 就行了,而不是封装一个新的方法。

## 处理过程

### 1、安装 polyfill 组件，使浏览器兼容 es6 的写法

*   在终端输入命令

>npm install --save babel-polyfill

*   main.js 头部引入 babel-polyfill， 注意这个放最前面：

> import 'babel-polyfill'

或者在项目的 webpack.base.conf.js 中配置：

```
entry: {

    app:['babel-polyfill','./src/main.js']

  },
```

另外，引入的一些模块需要单独引入到 babel 的配置中，不然不起作用（具体为啥我没深究），网上查到用到 echarts-v3 的需要配置，经测试我用到 iview 也是要配置的, 下面代码的 include 中就是我配置的项，这个主要是按需配置的，具体看项目里的情况：

```
module: {

    rules: [

      ...(config.dev.useEslint ? [createLintingRule()] : []),

      {

        test: /\.vue$/,

        loader: 'vue-loader',

        options: vueLoaderConfig

      },

      {

        test: /\.js$/,

        loader: 'babel-loader',

        include: [

          resolve('src'), 

          resolve('test'), 

          resolve('static'),

          resolve('node_modules/webpack-dev-server/client'),

          // resolve('node_modules/vue-echarts'),

          resolve('node_modules/iview/src'),

          // resolve('node_modules/resize-detector')

        ]

      },

}
```

### 2、兼容 dataset

我在引入了 babel-polyfill 后还是报错，信息如下图：

![ie11 报错信息](https://upload-images.jianshu.io/upload_images/3402395-fd834432578e7220.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


搜了半天关于 SCRIPT1003 和 SCRIPT5007 的错误, 发现没有直接的解决办法，就主要是说缺少项目中包含的某个模块的某种方法的引入。于是我就从我主要用到的 iview 入手去查找，发现了有相关的内容。有说到 iview 兼容 IE 需要写一个 dataset 方法才能正常加载。

dataset方法只要能够加载全局使用即可。我是写了一个脚本嵌入 index.html 文件中。代码如下：
```
<script>
  // dataset 方法兼容 IE 浏览器。ie10及以下不支持dataset
  if (window.HTMLElement) {
    if (Object.getOwnPropertyNames(HTMLElement.prototype).indexOf('dataset') === -1) {
      Object.defineProperty(HTMLElement.prototype, 'dataset', {
        get: function () {
          var attributes = this.attributes // 获取节点的所有属性
          var name = []
          var value = [] // 定义两个数组保存属性名和属性值
          var obj = {} // 定义一个空对象
          for (var i = 0; i < attributes.length; i++) { // 遍历节点的所有属性
            if (attributes[i].nodeName.slice(0, 5) === 'data-') { // 如果属性名的前面5个字符符合"data-"
              // 取出属性名的"data-"的后面的字符串放入name数组中
              name.push(attributes[i].nodeName.slice(5));
              // 取出对应的属性值放入value数组中
              value.push(attributes[i].nodeValue);
            }
          }
          for (var j = 0; j < name.length; j++) { // 遍历name和value数组
            obj[name[j]] = value[j]; // 将属性名和属性值保存到obj中
          }
          return obj // 返回对象
        }
      })
    }
  }
</script>
```
搞到这里，我的项目就已经可以在 IE 里出现了，也不打算继续支持更低的IE版本，坑太深，果断弃。但是样式还是有问题。这个搞起来也是很麻烦。点了点项目里出现的样式问题，未发现很复杂的，主要一个就是 flex 布局出现混乱，经过调整已经好了。还有就是 -webkit-box 不支持，之前显示数据使用以下方式解决多行溢出省略号显示问题失效了：

```
overflow: hidden;

display: -webkit-box; /* Chrome 4+, Safari 3.1, iOS Safari 3.2+ */

-webkit-box-orient: vertical;

-webkit-line-clamp: 2;

word-break: break-all;
```

纠结了一下，不想用js 的方式写，也不想用 伪标签（高度不好定，易出现文字被覆盖的情况），也不想特意让后台修改返回的数据，所以决定用比较low的相对保险的截取字符的方式展示。

## 总结

第一次处理这个问题，很多东西不明白，描述也不大清楚，处理的不全面，还望多交流指正！