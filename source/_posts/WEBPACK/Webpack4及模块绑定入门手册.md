---
title: WebPack4入门
date: 2018-07-10 10:20:05
tags: WebPack
categories: webpack
---

[Webpack4及模块绑定入门手册](https://www.sitepoint.com/beginners-guide-webpack-module-bundling/)

本文在我们的书《现代JavaScript工具和技能》中有所介绍。 让我们一起熟悉支持现代JavaScript开发的基本工具。

<!-- more -->

Webpack4 文档中说到：

>Webpack是一个module bundler。 它的主要目的是捆绑JavaScript文件以便在浏览器中使用，但它也能够转换，捆绑或打包任何资源。

Webpack已成为现代Web开发最重要的工具之一。 它主要是JavaScript的module bundler，但它可以用来转换所有前端资源，如HTML，CSS，甚至图像。 它可以让您更好地控制应用程序正在进行的HTTP请求的数量，并允许您使用其他类型的这些资源（例如，Pug，Sass和ES8）。 Webpack还允许您轻松地从npm使用包。

本文面向那些刚接触Webpack的人，将介绍初始设置和配置，模块，加载器（loaders），插件，代码拆分和热模块更换。 如果您发现视频教程很有帮助，我强烈推荐Glen Maddern的**Webpack from First Principles** 作为起点，了解Webpack的特殊之处。 它现在有点旧了，但原则仍然相同，这是一个很好的介绍。

要在家中进行操作，您需要安装Node.js. 您也可以从我们的GitHub仓库下载演示应用程序。

##  开始  ##

让我们用npm初始化一个新项目并安装webpack和webpack-cli：

	mkdir webpack-demo && cd webpack-demo
	npm init -y
	npm install --save-dev webpack webpack-cli

接下来我们将创建下面文件结构和内容：

	webpack-demo
	|- package.json
	|- webpack.config.js
	|- /src
	 |- index.js
	|- /dist
	 |- index.html

dist/index.html

	<!doctype html>
	<html>
	  <head>
	    <title>Hello Webpack</title>
	  </head>
	  <body>
	    <script src="bundle.js"></script>
	  </body>
	</html>

src/index.js

	const root = document.createElement("div")
	root.innerHTML = `<p>Hello Webpack.</p>`
	document.body.appendChild(root)

webpack.config.js

	const path = require('path')
	
	module.exports = {
	  entry: './src/index.js',
	  output: {
	    filename: 'bundle.js',
	    path: path.resolve(__dirname, 'dist')
	  }
	}

这告诉Webpack在我们的入口点**src / index.js**中编译代码并在**/dist/bundle.js**中输出一个bundle。 我们来添加一个用于运行Webpack的npm脚本。

package.json

	 {
	    ...
	    "scripts": {
	     "test": "echo \"Error: no test specified\" && exit 1",
	     "develop": "webpack --mode development --watch",
	     "build": "webpack --mode production"
	    },
	    ...
	  }

使用npm run develop命令，我们可以创建我们的第一个包！

	Asset      Size      Chunks           Chunk Names
	bundle.js  2.92 KiB  main  [emitted]  main

您现在应该可以在浏览器中加载dist / index.html并看到“Hello Webpack”。

打开**dist / bundle.js**以查看Webpack的功能。 顶部是Webpack的模块引导代码，底部是我们的模块。 你可能对此还没有很深刻的印象，但是如果你已经走到这一步，你现在可以开始使用ES模块，而Webpack将能够生成一个适用于所有浏览器的生产包。

使用Ctrl + C重新编译并运行npm run build以在生产模式下编译我们的bundle。
	
	Asset      Size       Chunks           Chunk Names
	bundle.js  647 bytes  main  [emitted]  main

请注意，捆绑包大小已从2.92 KiB降至647字节。

再看一下**dist / bundle.js**，你会看到一堆丑陋的代码。 我们的软件包已经用UglifyJS缩小了：代码将运行完全相同，但它是以尽可能小的文件大小完成的。

* - **模式开发**优化了构建速度和调试
* - **模式生产**优化了运行时的执行速度和输出文件大小。

## 模块 ##

使用ES模块，您可以将大型程序拆分为许多小型自包含程序。

创造性地，Webpack知道如何使用导入和导出语句来使用ES模块。 举个例子，让我们现在通过安装lodash-es并添加第二个模块来尝试这个：

	npm install --save-dev lodash-es

src/index.js
	
	import { groupBy } from "lodash-es"
	import people from "./people"
	
	const managerGroups = groupBy(people, "manager")
	
	const root = document.createElement("div")
	root.innerHTML = `<pre>${JSON.stringify(managerGroups, null, 2)}</pre>`
	document.body.appendChild(root)

src/people.js

	const people = [
	  {
	    manager: "Jen",
	    name: "Bob"
	  },
	  {
	    manager: "Jen",
	    name: "Sue"
	  },
	  {
	    manager: "Bob",
	    name: "Shirley"
	  }
	]
	
	export default people

运行**npm run develop**启动Webpack并刷新**index.html**。 您应该看到按管理器分组的一组人员打印到屏幕上。

注意：导入一个像***'es-lodash'***这样没有相对路径的模块，是从npm安装到 node_modules的模块。 你自己的模块总是需要一个像***'./people'***这样的相对路径，因此你可以区分它们。

请注意，在控制台中我们的捆绑包大小已增加到**1.41 MiB**！ 这值得关注，但在这种情况下，没有理由担心。 使用**npm run build**在生产模式下编译，lodash-es中的所有未使用的lodash模块都将从bundle中删除。 删除未使用的导入的过程称为tree-shaking，并且是Webpack免费获得的。

	> npm run develop
	
	Asset      Size      Chunks                  Chunk Names
	bundle.js  1.41 MiB  main  [emitted]  [big]  main


	> npm run build
	
	Asset      Size      Chunks        Chunk Names
	bundle.js  16.7 KiB  0  [emitted]  main

## 装载 ##

加载程序允许您在导入文件时运行预处理程序。 这允许您将静态资源捆绑到JavaScript之外，但让我们看看在首先加载**.js**模块时可以做些什么。

让我们通过下一代JavaScript转换器Babel运行所有**.js**文件来保持代码的现代化：
	
	npm install --save-dev "babel-loader@^8.0.0-beta" @babel/core @babel/preset-env

webpack.config.js

	const path = require('path')
	
	  module.exports = {
	    entry: './src/index.js',
	    output: {
	      filename: 'bundle.js',
	      path: path.resolve(__dirname, 'dist')
	    },
	   module: {
	     rules: [
	       {
	         test: /\.js$/,
	         exclude: /(node_modules|bower_components)/,
	         use: {
	           loader: 'babel-loader',
	         }
	       }
	     ]
	   }
	  }

.babelrc

	{
	  "presets": [
	    ["@babel/env", {
	      "modules": false
	    }]
	  ],
	  "plugins": ["syntax-dynamic-import"]
	}

此配置可防止Babel将导入和导出语句转换为ES5，并启用动态导入，这个我们将在后面的“代码拆分”一节中介绍。

我们现在可以自由使用现代语言功能，它们将被编译为在所有浏览器中运行的ES5。

## Sass ##

Loaders可以链接在一起进行一系列变换。 从我们的JavaScript导入Sass可以较好地演示它是如何工作的：
	
	npm install --save-dev style-loader css-loader sass-loader node-sass

webpack.config.js

	 module.exports = {
    ...
    module: {
      rules: [
        ...
	       {
	         test: /\.scss$/,
	         use: [{
	           loader: 'style-loader'
	         }, {
	           loader: 'css-loader'
	         }, {
	           loader: 'sass-loader'
	         }]
	       }
	      ]
	    }
	  }

这些加载器以相反的顺序处理:

* sass-loader将Sass转换为CSS。
* css-loader将CSS解析为JavaScript并解析任何依赖项。
* style-loader将我们的CSS输出到文档中的<style\>标记中。

您可以将这些视为函数调用。 一个加载器的输出作为输入提供给下一个：

	styleLoader(cssLoader(sassLoader("source")))

让我们添加一个Sass源文件，import是一个模块。

src/style.scss

	$bluegrey: #2b3a42;
	
	pre {
	  padding: 8px 16px;
	  background: $bluegrey;
	  color: #e1e6e9;
	  font-family: Menlo, Courier, monospace;
	  font-size: 13px;
	  line-height: 1.5;
	  text-shadow: 0 1px 0 rgba(23, 31, 35, 0.5);
	  border-radius: 3px;
	}

src/index.js

	import { groupBy } from 'lodash-es'
	import people from './people'
	
	import './style.scss'
	
	  ...

使用Ctrl + C和npm run develop重新启动构建。 在浏览器中刷新index.html，您应该看到一些样式。

## JS中的样式表 ##

我们刚从JavaScript中导入了一个Sass文件作为模块。

打开dist / bundle.js并搜索“pre {”。 实际上，我们的Sass已被编译为一串CSS并在我们的包中保存为模块。 当我们将这个模块导入JavaScript时，style-loader将字符串输出到嵌入的<style\>标记中。

你为什么需要做这样的事？

我不会在这里深入研究这个主题，但这里有几个理由需要考虑：

* 您可能希望包含在项目中的JavaScript组件可能依赖于其他资源才能正常运行（HTML，CSS，图像，SVG）。 如果这些都可以捆绑在一起，则导入和使用起来要容易得多。
* 消除Dead code：当代码不再导入JS组件时，也不再导入CSS。 生成的包只会包含执行某些操作的代码。
* CSS模块：CSS的全局命名空间使得很难确信对CSS的更改不会产生任何副作用。 CSS模块通过设置CSS默认本地并显示您可以在JavaScript中引用的唯一类名来解决该问题。
* 通过巧妙的方式捆绑/拆分代码来减少HTTP请求的数量。

## 图片 ##
我们将看到的最后一个加载器示例是使用文件加载器处理图像。

在标准HTML文档中，当浏览器遇到img标记或具有background-image属性的元素时，将获取图像。 使用Webpack，您可以在小图像的情况下通过将图像源作为字符串存储在JavaScript中来优化它。 通过执行此操作，您可以预加载它们，浏览器不必在以后使用单独的请求获取它们：

	npm install --save-dev file-loader

webpack.config.js
	
	module.exports = {
	    ...
	    module: {
	      rules: [
	        ...
	       {
	         test: /\.(png|svg|jpg|gif)$/,
	         use: [
	           {
	             loader: 'file-loader'
	           }
	         ]
	       }
	      ]
	    }
	  }

用以下命令下载一个测试图像：

		curl https://raw.githubusercontent.com/sitepoint-editors/webpack-demo/master/src/code.png --output src/code.png

使用Ctrl + C和npm run develop重新启动构建，您现在可以将图像作为模块导入！

src/index.js

	import { groupBy } from 'lodash-es'
	import people from './people'
	
	import './style.scss'
	import './image-example'
	
	  ...


src/image-example.js

	import codeURL from "./code.png"
	
	const img = document.createElement("img")
	img.src = codeURL
	img.style = "background: #2B3A42; padding: 20px"
	img.width = 32
	document.body.appendChild(img)

这将包括一个图像，其中src属性包含图像本身的数据URI：

	<img src="data:image/png;base64,iVBO..." style="background: #2B3A42; padding: 20px" width="32">

我们的CSS中的背景图像也由文件加载器处理。

src/style.scss

	$bluegrey: #2b3a42;
	
	  pre {
	    padding: 8px 16px;
	    background: $bluegrey;
	    background: $bluegrey url("code.png") no-repeat center center / 32px 32px;
	    color: #e1e6e9;
	    font-family: Menlo, Courier, monospace;
	    font-size: 13px;
	    line-height: 1.5;
	    text-shadow: 0 1px 0 rgba(23, 31, 35, 0.5);
	    border-radius: 3px;
	  }

在文档中查看更多Loaders示例：

* [加载字体](https://webpack.js.org/guides/asset-management/#loading-fonts)
* [加载数据](https://webpack.js.org/guides/asset-management/#loading-data)

## 依赖图 Dependency Graph##

您现在应该能够看到加载器如何帮助您在资源中构建依赖关系树。 这就是Webpack主页上的图像所展示的内容。

![](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2017/01/1484692838webpack-dependency-tree.png)

尽管JavaScript是切入点，但Webpack明白您的其他资源类型（如HTML，CSS和SVG）各自具有自己的依赖关系，这应该被视为构建过程的一部分。

## 代码拆分 ##

Webpack文档中写到：

>代码拆分是Webpack最引人注目的功能之一。 此功能允许您将代码拆分为各种捆绑包，然后可以按需或并行加载。 它可用于实现更小的捆绑并控制资源负载优先级，如果使用得当，可能会对加载时间产生重大影响。

到目前为止，我们只看到了一个入口点 - **src / index.js** - 和一个输出包 - dist / bundle.js。 当您的应用程序增长时，您需要将其拆分，以便在开始时不会下载整个代码库。 一种好的方法是使用Code Splitting和Lazy Loading来按需获取内容，因为代码路径需要它们。

我们通过添加一个“聊天”模块来证明这一点，该模块在有人与之交互时被提取和初始化。 我们将创建一个新的入口点并为其命名，我们还将使输出的文件名动态，因此每个块的不同。

webpack.config.js

	 const path = require('path')
	
	  module.exports = {
	   entry: './src/index.js',
	   entry: {
	     app: './src/app.js'
	   },
	   output: {
	     filename: 'bundle.js',
	     filename: '[name].bundle.js',
	     path: path.resolve(__dirname, 'dist')
	    },
	    ...
	  }

src/app.js

	import './app.scss'
	
	const button = document.createElement("button")
	button.textContent = 'Open chat'
	document.body.appendChild(button)
	
	button.onclick = () => {
	  import(/* webpackChunkName: "chat" */ "./chat").then(chat => {
	    chat.init()
	  })
	}

src/chat.js

	import people from "./people"
	
	export function init() {
	  const root = document.createElement("div")
	  root.innerHTML = `<p>There are ${people.length} people in the room.</p>`
	  document.body.appendChild(root)
	}

src/app.scss

	button {
	  padding: 10px;
	  background: #24b47e;
	  border: 1px solid rgba(#000, .1);
	  border-width: 1px 1px 3px;
	  border-radius: 3px;
	  font: inherit;
	  color: #fff;
	  cursor: pointer;
	  text-shadow: 0 1px 0 rgba(#000, .3), 0 1px 1px rgba(#000, .2);
	}

注意：尽管*/ * webpackChunkName * /* comment为包提供了名称，但此语法不是特定于Webpack的。 它是动态导入的建议语法，旨在直接在浏览器中支持。

我们运行npm run build并查看它生成的内容：

	Asset           Size       Chunks        Chunk Names
	chat.bundle.js  377 bytes  0  [emitted]  chat
	app.bundle.js   7.65 KiB   1  [emitted]  app

由于我们的条目包已经更改，我们还需要更新它的路径。

dist/index.html

	<!doctype html>
	  <html>
	    <head>
	      <title>Hello Webpack</title>
	    </head>
	    <body>
	    <script src="bundle.js"></script>
	    <script src="app.bundle.js"></script>
	    </body>
	  </html>

我们从dist目录启动一个服务器，看看这个实际应用：

	cd dist
	npx serve

在浏览器中打开http://localhost:5000，看看会发生什么。 最初只获取bundle.js。 单击该按钮时，将导入并初始化聊天模块。
![](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2018/03/1521906882lazy-loading.png)

只需很少的工作，我们就可以在我们的应用程序中添加动态代码拆分和延迟加载模块。 这是构建高性能Web应用程序的一个很好的起点。

## 插件 ##

当加载器对单个文件进行转换时，插件可以在更大的代码块上运行。

现在我们正在打包代码，外部模块和静态资产，我们的包将会迅速增长。 插件可以帮助我们以巧妙的方式分割代码并优化生产。

在不知情的情况下，我们实际上已经使用了许多默认的Webpack插件“模式”

发展

* 提供process.env.NODE_ENV，其值为“development”
* NamedModulesPlugin

生产

* 提供process.env.NODE_ENV，其值为“production”
* UglifyJsPlugin
* ModuleConcatenationPlugin
* NoEmitOnErrorsPlugin

## 产品 ##

在添加其他插件之前，我们首先拆分我们的配置，以便我们可以应用特定于每个环境的插件。

将webpack.config.js重命名为webpack.common.js并添加用于开发和生产的配置文件。

	- |- webpack.config.js
	+ |- webpack.common.js
	+ |- webpack.dev.js
	+ |- webpack.prod.js

我们将使用webpack-merge将我们的公共配置与特定于环境的配置相结合：

	npm install --save-dev webpack-merge

webpack.dev.js

	const merge = require('webpack-merge')
	const common = require('./webpack.common.js')
	
	module.exports = merge(common, {
	  mode: 'development'
	})

webpack.prod.js

	const merge = require('webpack-merge')
	const common = require('./webpack.common.js')
	
	module.exports = merge(common, {
	  mode: 'production'
	})

package.json

	 "scripts": {
	    "develop": "webpack --watch --mode development",
	    "build": "webpack --mode production"
	    "develop": "webpack --watch --config webpack.dev.js",
	    "build": "webpack --config webpack.prod.js"
	   },

现在我们可以将特定于开发的插件添加到webpack.dev.js和webpack.prod.js中特定于生产的插件中。

## 拆分CSS ##

在使用ExtractTextWebpackPlugin捆绑生产时，最好将CSS与JavaScript分开。

当前的.scss加载器非常适合开发，所以我们将把它们从webpack.common.js移到webpack.dev.js中，并将ExtractTextWebpackPlugin添加到webpack.prod.js中。

	npm install --save-dev extract-text-webpack-plugin@4.0.0-beta.0

webpack.common.js

	 ...
	  module.exports = {
	    ...
	    module: {
	      rules: [
	        ...
	-       {
	-         test: /\.scss$/,
	-         use: [
	-           {
	-             loader: 'style-loader'
	-           }, {
	-             loader: 'css-loader'
	-           }, {
	-             loader: 'sass-loader'
	-           }
	-         ]
	-       },
	        ...
	      ]
	    }
	  }

webpack.dev.js

	  const merge = require('webpack-merge')
	+ const ExtractTextPlugin = require('extract-text-webpack-plugin')
	  const common = require('./webpack.common.js')
	
	  module.exports = merge(common, {
	    mode: 'production',
	    module: {
	     rules: [
          {
	         test: /\.scss$/,
	         use: ExtractTextPlugin.extract({
	           fallback: 'style-loader',
	           use: ['css-loader', 'sass-loader']
	         })
	       }
	     ]
	   },
	   plugins: [
	     new ExtractTextPlugin('style.css')
	   ]
	  })

我们来比较两个构建脚本的输出：

	> npm run develop
	
	Asset           Size      Chunks           Chunk Names
	app.bundle.js   28.5 KiB  app   [emitted]  app
	chat.bundle.js  1.4 KiB   chat  [emitted]  chat

	> npm run build
	
	Asset           Size       Chunks        Chunk Names
	chat.bundle.js  375 bytes  0  [emitted]  chat
	app.bundle.js   1.82 KiB   1  [emitted]  app
	style.css       424 bytes  1  [emitted]  app

现在CSS被从JavaScript包中提取出来用于生产，我们需要从我们的HTML <link\>到它。

dist/index.html

	<!DOCTYPE html>
	  <html>
	    <head>
	      <meta charset="UTF-8">
	      <title>Code Splitting</title>
	      <link href="style.css" rel="stylesheet">
	    </head>
	    <body>
	      <script type="text/javascript" src="app.bundle.js"></script>
	    </body>
	  </html>

这允许在浏览器中并行下载CSS和JavaScript，因此加载速度比单个捆绑包更快。 它还允许在JavaScript完成下载之前显示样式。

## 生成HTML ##

每当我们的输出发生变化时，我们必须不断更新index.html以引用新的文件路径。 这正是html-webpack-plugin自动为我们创建的。

我们也可以在每次构建之前同时添加clean-webpack-plugin来清除/dist目录。

	npm install --save-dev html-webpack-plugin clean-webpack-plugin

webpack.common.js

	 const path = require('path')
	 const CleanWebpackPlugin = require('clean-webpack-plugin');
	 const HtmlWebpackPlugin = require('html-webpack-plugin');
	
	  module.exports = {
	    ...
	   plugins: [
	     new CleanWebpackPlugin(['dist']),
	     new HtmlWebpackPlugin({
	       title: 'My killer app'
	     })
	   ]
	  }

现在每次我们建立时，dist都会被清除掉。 我们现在也会看到index.html输出，以及我们的入口包的正确路径。

运行npm run develop会产生以下结果：

	<!DOCTYPE html>
	<html>
	  <head>
	    <meta charset="UTF-8">
	    <title>My killer app</title>
	  </head>
	  <body>
	    <script type="text/javascript" src="app.bundle.js"></script>
	  </body>
	</html>

而npm run build产生了这个：

	<!DOCTYPE html>
	<html>
	  <head>
	    <meta charset="UTF-8">
	    <title>My killer app</title>
	    <link href="style.css" rel="stylesheet">
	  </head>
	  <body>
	    <script type="text/javascript" src="app.bundle.js"></script>
	  </body>
	</html>

## 发展 ##

webpack-dev-server为您提供了一个简单的Web服务器，并为您提供实时重新加载，因此您无需手动刷新页面即可查看更改。

	npm install --save-dev webpack-dev-server

package.json

	{
	    ...
	    "scripts": {
	     "develop": "webpack --watch --config webpack.dev.js",
	     "develop": "webpack-dev-server --config webpack.dev.js",
	    }
	    ...
	  }

	> npm run develop
	
	 ｢wds｣: Project is running at http://localhost:8080/
	 ｢wds｣: webpack output is served from /

在浏览器中打开http://localhost:8080/并对其中一个JavaScript或CSS文件进行更改。 您应该看到它自动构建和刷新。

## HotModuleReplacement ##

HotModuleReplacement插件比Live Reloading更进一步，并在运行时交换模块而无需刷新。 正确配置后，这可以节省开发单页应用程序的大量时间。 如果页面中有很多状态，则可以对组件进行增量更改，只更换和更新已更改的模块。

webpack.dev.js

	  const webpack = require('webpack')
	  const merge = require('webpack-merge')
	  const common = require('./webpack.common.js')
	
	  module.exports = merge(common, {
	    mode: 'development',
	   devServer: {
	     hot: true
	   },
	   plugins: [
	     new webpack.HotModuleReplacementPlugin()
	   ],
	    ...
	  }

现在我们需要从代码中接受更改的模块来重新初始化事物。

src/app.js

	 if (module.hot) {
	   module.hot.accept()
	 }
	
	  ...

注意：启用热模块替换时，module.hot设置为true以进行开发，false设置为生产，因此这些将从捆绑中剥离。

重新启动构建，看看执行以下操作时会发生什么：

* 单击打开聊天
* 将新人添加到people.js模块
* 再次单击“打开聊天”

![](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2018/03/1521906879hmr.png)

这是发生了什么：

1. 单击“打开聊天”时，将获取并初始化chat.js模块
2. HMR检测perple.js何时被修改
3. index.js中的module.hot.accept（）会导致替换此条目块加载的所有模块
4. 再次单击“打开聊天”时，将使用更新模块中的代码运行chat.init（）。

## CSS替换 ##

让我们将按钮颜色更改为红色，看看会发生什么：

	src/app.scss
	
	  button {
	    ...
	   background: #24b47e;
	   background: red;
	    ...
	  }

![](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2018/03/1521906881hmr2.png)

现在我们可以看到我们的样式的即时更新，而不会丢失任何状态。这是一个大大改善的开发人员体验！感觉就像未来。

## HTTP/2 ##

使用像Webpack这样的module bundler的主要好处之一是它可以帮助您控制资源的构建方式，然后在客户端上获取，从而帮助您提高性能。多年来，连接文件以减少需要在客户端上进行的请求数量被认为是最佳实践。这仍然有效，但HTTP/2现在允许在单个请求中传递多个文件，因此连接不再是银弹。您的应用程序实际上可能会受益于单独缓存许多小文件。然后，客户端可以获取单个已更改的模块，而不必再次获取具有大部分相同内容的整个包。

Webpack的创建者Tobias Koppers撰写了一篇内容丰富的文章，解释了为什么捆绑仍然很重要，即使在HTTP/2时代也是如此。

在Webpack和HTTP / 2上阅读更多相关信息。

## 给你的话 ##

我希望你已经发现Webpack的这个介绍很有帮助，并且能够开始使用它并产生很大的效果。可能需要一些时间来围绕Webpack的配置，加载器和插件，但了解这个工具如何工作将会有所回报。

Webpack 4的文档目前正在进行中，但实际上很好地组合在一起。我强烈建议您阅读概念和指南以获取更多信息。以下是您可能感兴趣的一些其他主题：

* [源地图开发](https://webpack.js.org/guides/development/#using-source-maps)
* [生产的源地图](https://webpack.js.org/guides/production/#source-mapping)
* [缓存破坏与散列文件名](https://webpack.js.org/guides/caching/)
* [拆分供应商包](https://webpack.js.org/plugins/commons-chunk-plugin/#explicit-vendor-chunk)

Webpack 4是您选择的module bundler吗？请在下面的评论中告诉我。










