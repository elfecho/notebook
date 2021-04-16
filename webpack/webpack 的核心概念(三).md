# webpack 插件

plugin 可以在 webpack 运行到某个时刻的时候帮你做一些事情(类似 vue 的生命周期函数一样)

[管理输出文档](https://www.webpackjs.com/guides/output-management/)

## html-webpack-plugin(v3.2.0)

- 时刻：在打包之后开始运行
- 作用：会在打包结束后，自动生成一个 html 文件，并将打包生成的 js 自动引入到这个 html 文件中
- 安装：npm i html-webpack-plugin -D
- 配置：[详细配置](https://github.com/jantimon/html-webpack-plugin#configuration)
- 使用：

```JavaScript
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  plugins: [new HtmlWebpackPlugin({
    template: 'index.html' // 指定生成 html 的模版文件(如果不指定，则会生成一个默认的不附带其它内容的 html 文件)
  })]
}
```

## clean-webpack-plugin(v3.0.0)

- [clean-webpack-plugin 升级3.0踩坑](http://www.imooc.com/article/289614)
- 时刻：在打包之前开始运行
- 作用：删除文件夹目录(默认删除 output 下 path 指定的目录)
- 安装：npm i clean-webpack-plugin -D
- 使用：

```JavaScript
// webpack.config.js
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  plugins: [new CleanWebpackPlugin({
    cleanOnceBeforeBuildPatterns: [path.resolve(__dirname, 'dist')] // 若不配置，默认删除 output 下 path 指定的目录
  })]
}
```

# webpack entry&output

[具体可查看文档](https://www.webpackjs.com/configuration/output/)

## 打包单个文件

```JavaScript
// webpack.config.js
const path = require('path')

module.exports = {
  // entry: './src/index.js', // 简写方式
  entry: {
    main: './src/index.js' // main 为导出的文件名
  },
  output: {
    filename: 'bundle.js', // 这里是导出文件进行重命名
    path: path.resolve(__dirname, 'dist')
  }
}
```

## 打包多个文件

```JavaScript
// webpack.config.js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: {
    index1: './src/a.js',
    index2: './src/b.js'
  },
  output: {
    publicPath: 'http://cdn.com.cn', // 会在自动生成的 html 文件中，引入文件路径的前面加上此路径
    filename: '[name].[hash].js', // name 即指 entry 中配置的需要打包文件的 key (也即 index1 和 index2, 最终会生成 index1.js 和 index2.js)
    path: path.resolve(__dirname, 'dist')
  },
  plugins: [new HtmlWebpackPlugin()]
}
```

# webpack sourceMap

- sourceMap 就是一个映射关系，它会将打包后的文件与源文件对应起来，方便查找出错代码
- [配置 devtool 控制如何生成 sourceMap](https://www.webpackjs.com/configuration/devtool/)

```JavaScript
// webpack.config.js
module.exports = {
  devtool: 'source-map'
  // devtool: 'cheap-module-eval-source-map' // 常用于开发环境
  // devtool: 'cheap-module-source-map' // 常用于生产环境
}
```

- devtool 的可能值：

| devtool                 | 解释                                                         |
| ----------------------- | ------------------------------------------------------------ |
| none                    | 不生成 sourceMap                                             |
| source-map              | 生成 .map 文件                                               |
| inline-source-map       | 不生成 .map 文件，sourceMap 会被合并到打包生成的文件的最后一行里 |
| cheap-source-map        | 只告诉出错的行，不告诉出错的列（只负责业务代码的错误，不提示loader里面的错误） |
| cheap-module-source-map | 除了业务代码里的错误，还要提示一些 loader 里面的错误         |
| eval                    | 不生成 .map 文件，使用 eval 在打包后文件里生成对应关系（打包速度最快） |

# webpack WebpackDevServer

## 使用观察模式

在 webpack 命令后面加 --watch，webpack 会监听打包的文件，只要文件发生变化，就会自动重新打包

```javascript
// package.json
{
  "scripts": {
    "watch": "webpack --watch"
  }
}
```

唯一的缺点是，为了看到修改后的实际效果，你需要刷新浏览器。如果能够自动刷新浏览器就更好了，可以尝试使用 `webpack-dev-server`，恰好可以实现我们想要的功能。

## 使用webpack-dev-server

- 安装

  ```
  npm i webpack-dev-server -D
  ```

- 配置

  更多相关配置可查看[官方文档](https://www.webpackjs.com/configuration/dev-server/)

  ```javascript
  // webpack.config.js
  module.exports = {
    devServer: {}
  }
  
  // package.json
  {
    "scripts": {
      "wdserver": "webpack-dev-server"
    }
  }
  ```

- 编译项目到内存中，并启动一个服务

  ```javascript
  // package.json
  "scripts": {
      "start": "webpack-dev-server"
  }
  
  npm run start
  ```

- devServer 有很多可配置的参数：

  ```javascript
  open: true // 启动服务的时候自动在浏览器中打开当前项目(默认 false)
  port: 8888 // 自定义启动服务的端口号(默认 8080)
  
  contentBase: './static' // 指定资源的请求路径(默认 当前路径)
  contentBase: path.join(__dirname, "static") // 推荐使用绝对路径
  ```

  例如：

  ```html
  /static 文件夹下存在一张图片 /static/img.png
  
  devServer 里配置 contentBase: './static'
  
  /index.html 中使用 <img src="img.png" />
  
  这样它就会去 /static 文件夹下去找 img.png 而不是从根目录下去找 img.png
  ```

- [更多相关webpack-dev-server说明](https://www.jianshu.com/p/e547fb9747e0)

## express & webpack-dev-middleware

借助 express 和 webpack-dev-middleware 自己手动搭建服务

- npm i express webpack-dev-middleware -D # 安装 express 和 webpack-dev-middleware 包
- 根目录下新建 server.js

```javascript
// server.js(在 node 中使用 webpack)
const express = require('express');
const webpack = require('webpack');
const webpackDevMiddleware = require('webpack-dev-middleware');
const config = require('./webpack.config.js');

const complier = webpack(config);
const app = express();

app.use(webpackDevMiddleware(complier, {}));

app.listen(3000, () => {
	console.log('server is running');
});
```

- 配置 npm 命令

```
// package.json
{
  "scripts": {
    "middleware": "node server.js"
  }
}
```

- npm run middleware# 编译项目并启动服务(成功后在浏览器输入 localhost:3000 访问项目)

- 弊端：
  - 每次打包之后，需要自己手动进行刷新
  -  不能自动打开浏览器，需要做大量的篇幅加其功能

# 模块热替换

**HotModuleReplacementPlugin 是 webpack 自带的一个插件，不需要单独安装**

## 配置

```javascript
// webpack.config.js
const webpack = require('webpack')

module.exports = {
  devServer: {
    hot: true, // 让 webpack-dev-server 开启 hot module replacement 这样的一个功能
    hotOnly: true // 即便是 hot module replacement 的功能没有生效，也不让浏览器自动刷新
  },
  plugins: [new webpack.HotModuleReplacementPlugin()]
}
```

## 在css中使用

更改样式文件，页面就不会整个重新加载，而是只更新样式

```css
// /src/index.css
div:nth-of-type(odd) {
  background-color: #09f;
}
```

```javascript
// /src/index.js
import './index.css'

var btn = document.createElement('button')
btn.innerText = 'button'
document.body.appendChild(btn)

btn.onclick = () => {
  var item = document.createElement('div')
  item.innerText = 'item'
  document.body.appendChild(item)
}
```

## 在 js 中使用

更改 number.js 文件中的代码，只会从页面上移除 id 为 number 的元素，然后重新执行一遍 number() 方法，不会对页面上的其它部分产生影响，也不会导致整个页面的重载

```javascript
// /src/counter.js
function counter() {
  var div = document.createElement('div')
  div.setAttribute('id', 'counter')
  div.innerText = 1
  div.onclick = function() {
    div.innerText = parseInt(div.innerText, 10) + 1
  }
  document.body.appendChild(div)
}
export default counter
```

```javascript
// /src/number.js
function number() {
  var div = document.createElement('div')
  div.setAttribute('id', 'number')
  div.innerText = 20
  document.body.appendChild(div)
}

export default number
```

```javascript
// /src/index.js
import counter from './counter'
import number from './number'

counter()
number()

// 相比 css 需要自己书写重载的代码，那是因为 css-loader 内部已经帮 css 写好了这部分代码
if (module.hot) {
  module.hot.accept('./number', () => {
  	// 监测到number.js代码发生变化，就会执行下面的代码 		
    document.body.removeChild(document.getElementById('number'))
    number()
  })
}
```

[更多相关使用说明](https://www.webpackjs.com/guides/hot-module-replacement/)

# 使用babel处理ES6语法

[babel官网](https://babeljs.io/)

## 使用babel

[点击去官网查看(选择webpack)](https://babeljs.io/setup#installation)

1. 安装

   ```
   npm i -D babel-loader @babel/core @babel/preset-env
   
   babel-loader: 是 webpack 和 babel 通讯的桥梁，使 webpack 和 babel 打通，babel-loader 并不会把 js 中的 ES6 语法转换成 ES5 语法
   @babel/core: 是 babel 的核心语法库，它能够让 babel 识别 js 代码里面的内容并做转化
   @babel/preset-env: 将 ES6 语法转换成 ES5 语法，它包含了所有 ES6 语法转换成 ES5 语法的翻译规则
   ```

2. 配置

   ```javascript
   module.exports = {
     module: {
       rules: [
         {
           test: /\.m?js$/,
           exclude: /node_modules/, // 不去匹配 node_modules 文件夹下的 js
           use: {
             loader: "babel-loader",
             options: {
               presets: ['@babel/preset-env']
             }
           }
         }
       ]
     }
   }
   ```

   **上面的步骤，只是做了语法上的翻译(如: let/const/箭头函数/... 都会被转换)，但一些新的变量和方法并没有被翻译(如: promise/.map()/...)，这时就要使用 @babel/polyfill 来处理**

## @babel/polyfill

[使用@babel/polyfill](https://babeljs.io/docs/en/babel-polyfill)

1. 安装

   ```
   npm i -D @babel/polyfill
   ```

2. 在入口文件index.js引入@babel/polyfill

   ```
   import '@babel/polyfill'
   ```

**像上面配置好之后，会发现打包后的文件特别大，因为一些没用到的 ES6 语法也被打包了进去，因此需要做如下操作**

- [babel 7.4版本更新后，如何引入babel-polyfill](https://blog.csdn.net/A13330069275/article/details/97623074)

- npm i -D core-js # 安装 core-js(v3.3.2)

- 删除入口文件 index.js 中的 import '@babel/polyfill'

- 配置

  ```javascript
  // webpack.config.js
  module.exports = {
    module: {
      rules: [{
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: {
          presets: [
            [
              '@babel/preset-env',
              {
                corejs: 3,
                useBuiltIns: 'usage',
                targets: { // 通过 targets 指定项目运行的环境，打包时会自动判断是否需要去解析转化代码
                  chrome: '67'
                }
              }
            ]
          ]
        }
      }]
    }
  }
  ```

  **如果写的是业务代码，可采用上面方法使用 polyfill 去打包；如果是开发组件或者库的话，可使用 plugin-transform-runtime**
  polyfill 会污染全局环境，plugin-transform-runtime 会以闭包的形式帮助组件去引入相关内容

  [@babel/plugin-transform-runtime 官方文档](https://babeljs.io/docs/en/babel-plugin-transform-runtime)

