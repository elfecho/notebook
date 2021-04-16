------

title: webpack 高级概念(四) date: 2020-12-13 19:59:13 tags: [webpack]

------

# TreeShaking

- TreeShaking 是基于 ES6 的静态引用，通过扫描所有 ES6 的 export，找出被 import 的内容并添加到最终代码中，排除不使用的代码

- TreeShaking 只支持 ES Module 的静态引入方式，不支持 CommonJS 的动态引入方式

- 主要用于生产环境，需要在 package.json 中配置 "sideEffects": false (对所有的模块都进行 TreeShaking 处理)

- 如果引入的库并没有导出任何内容(如: import '@babel/polyfill')，就需要配置 "sideEffects": ["@babel/polyfill"]，让 TreeShaking 不对 @babel/polyfill 进行处理

- 如果引入样式文件(如: import './style.css')，则需配置 "sideEffects": ["*.css"]

- 若要在开发环境使用 TreeShaking ，需在 webpack.config.js 中配置

  ```
  module.exports = {
    optimization: {
      usedExports: true
    }
  }
  ```

# dev&prod模式的区分打包

## 安装

```
npm i -D webpack-merge
```

作用是将公共的 webpack 配置代码与开发 / 生产环境中的 webpack 配置代码进行合并

## 配置

- /build/webpack.common.js # 存放公共的 webpack 配置代码

  ```
  // 示例仅展示部分代码(下同)
  const path = require('path')
  const HtmlWebpackPlugin = require('html-webpack-plugin')
  const { CleanWebpackPlugin } = require('clean-webpack-plugin')
  
  module.exports = {
    entry: {
      main: './src/index.js'
    },
    plugins: [
      new HtmlWebpackPlugin({
        template: 'index.html'
      }),
      new CleanWebpackPlugin({
        cleanOnceBeforeBuildPatterns: [path.resolve(process.cwd(), 'dist')] // __dirname => process.cwd()
      })
    ],
    output: {
      filename: '[name].[hash].js',
      path: path.resolve(__dirname, '../dist') // dist => ../dist
    }
  }
  ```

- /build/webpack.dev.js # 存放开发环境的 webpack 配置代码

  ```
  const merge = require('webpack-merge')
  const commonConfig = require('./webpack.common.js')
  
  const devConfig = {
    mode: 'development'
  }
  module.exports = merge(commonConfig, devConfig)
  ```

- /build/webpack.prod.js # 存放生产环境的 webpack 配置代码

  ```
  const merge = require('webpack-merge')
  const commonConfig = require('./webpack.common.js')
  
  const prodConfig = {
    mode: 'production'
  }
  module.exports = merge(commonConfig, prodConfig)
  ```

- /package.json # 配置打包命令

  ```
  {
    "scripts": {
      "dev": "webpack-dev-server --config ./build/webpack.dev.js",
      "build": "webpack --config ./build/webpack.prod.js"
    }
  }
  ```

## 运行命令

- npm run dev # 开发模式
- npm run build # 生产模式

## 附录

- process.cwd() # 执行 node 命令的文件夹地址
- __dirname # 执行 js 的文件目录
- path.join(path1, path2, path3, ...) # 将路径片段连接起来形成新的路径
- path.resolve([from...], to) # 将一个路径或路径片段的序列解析为一个绝对路径，相当于执行 cd 操作

# Webpack 和 CodeSplitting

- CodeSplitting：代码分割，代码分割和 webpack 无关
- npm i -S lodash # 安装 lodash

## 手动代码分割(配置多个入口文件)

> 当页面业务逻辑代码发生改变时，只要加载main.js即可

```
// /src/lodash.js
import _ from 'lodash'
window._ = _
// /src/index.js
console.log(_.join(['a', 'b', 'c'])) // 输出a,b,c
console.log(_.join(['a', 'b', 'c'], '***')) // 输出a***b***c
// /build/webpack.common.conf.js
module.exports = {
  entry: {
    lodash: './src/lodash.js',
    main: './src/index.js'
  },
  output: {
    filename: '[name].js',
    path: path.resolve(__dirname, '../dist')
  }
}
```

## 自动代码分割

> webpack 中的代码分割底层使用的是 SplitChunksPlugin 这个插件

### webpack 中实现同步代码分割(需要配置optimization)

```
// /src/index.js
import _ from 'lodash'

console.log(_.join(['a', 'b', 'c'])) // 输出a,b,c
console.log(_.join(['a', 'b', 'c'], '***')) // 输出a***b***c
// /build/webpack.common.conf.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all'
    }
  }
}
```

### webpack 中实现异步代码分割(通过 import 引入等，不需要任何配置)

> 无需做任何配置，会自动进行代码分割，放置到新的文件中

```
// /src/index.js
function getComponent() {
  return import('lodash').then(({ default: _ }) => {
    var element = document.createElement('div')
    element.innerHTML = _.join(['hello', 'world'], '-')
    return element
  })
}

getComponent().then(element => {
  document.body.appendChild(element)
})
```

通过 import('lodash') 引入，分割打包后的文件名称是 [id].[hash].js，打包后文件的辨识度不高； 使用 import(/* webpackChunkName: "lodash" */ 'lodash') 来为打包后的文件起别名，提升辨识度(最终生成文件名称为：vendors~lodash.[hash].js，意思是符合 vendors 组的规则，入口是main)，详情可搜索查看 SplitChunksPlugin 的配置 这种方式被称为**魔法注释**，详情可查看[魔法注释 Magic Comments 官网地址](https://links.jianshu.com/go?to=https%3A%2F%2Fwebpack.docschina.org%2Fapi%2Fmodule-methods%2F%23magic-comments)

**注意：** 如果报错“Support for the experimental syntax 'dynamicImport' isn't currently enabled”，可安装 @babel/plugin-syntax-dynamic-import 进行解决

[@babel/plugin-syntax-dynamic-import 官网地址](https://links.jianshu.com/go?to=https%3A%2F%2Fbabeljs.io%2Fdocs%2Fen%2Fbabel-plugin-syntax-dynamic-import)

```
// npm i -D @babel/plugin-syntax-dynamic-import # 安装模块包

// /.babelrc # 配置
{
  "plugins": ["@babel/plugin-syntax-dynamic-import"]
}
```

## 附录

- 打包后输出文件的命名

```
// /build/webpack.common.conf.js
module.exports = {
  output: {
    filename: '[name].[hash].js', // 入口文件根据 filename 命名
    chunkFilename: '[name].chunk.js', // 非入口文件根据 chunkFilename 命名
    path: path.resolve(__dirname, '../dist')
  }
}
```

# SplitChunksPlugin(代码分割)

[SplitChunksPlugin官网文档](https://webpack.docschina.org/plugins/split-chunks-plugin/)

```
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async', // async：做代码分割时，只对异步代码生效；all：对同步和异步代码都生效；initial：只对同步代码生效
      minSize: 30000, // 单位：字节；当打包的库大于 minSize 时才做代码分割，小于则不做代码分割
      minChunks: 1, // 当一个模块被用了至少 minChunks 次时，才对其进行代码分割
      maxAsyncRequests: 5, // 同时加载的模块数最多是 maxAsyncRequests 个，如果超过 maxAsyncRequests 个，只对前 maxAsyncRequests 个类库进行代码分割，后面的就不做代码分割
      maxInitialRequests: 3, // 整个网站首页(入口文件)加载的时候，入口文件引入的库进行代码分割时，最多只能分割 maxInitialRequests 个js文件
      automaticNameDelimiter: '~', // 打包生成文件名称之间的连接符
      name: true, // 打包起名时，让 cacheGroups 里的名字有效
      cacheGroups: { // 同步的代码会走cacheGroups这个方法 [缓存组]
        vendors: {
          test: /[\\/]node_modules[\\/]/, // 如果是从 node_modules 里引入的模块，就打包到 vendors 组里
          priority: -10, // 指定该组的优先级，若一个类库符合多个组的规则，就打包到优先级最高的组里
          // filename: 'vendors.js',
        },
        default: {
          priority: -20,
          reuseExistingChunk: true // 如果一个模块已经被打包过了(一个模块被多个文件引用)，那么再打包的时候就会忽略这个模块，直接使用之前被打包过的那个模块
        }
      }
    }
  }
};
```

**注：SplitChunksPlugin 上面的一些配置需要配合 cacheGroups 里的配置一起使用才能生效(如 chunks 的配置)**

```
// webpack.common.conf.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          filename: 'vendors.js' // 配置 filename 之后，打包会以 filename 的值为文件名，生成的文件是 vendors.js
        },
        default: false
      }
    }
  }
}
```

# LazyLoading&Chunk(懒加载)

LazyLoading：懒加载并不是webpack里面的概念，而是ES里面的概念；什么时候执行，什么时候才会加载对应的模块

- 下面这种同步代码的写法，打包时将分割后的模块对应的 js 文件直接通过 script 标签在 html 中引入，页面开始加载的时候就会去加载这些 js，导致页面加载很慢

  ```
  // /src/index.js
  import _ from 'lodash'
  
  document.addEventListener('click', () => {
    var element = document.createElement('div')
    element.innerHTML = _.join(['hello', 'world'], '-')
    document.body.appendChild(element)
  })
  ```

- 下面这种异步代码的写法可以实现一种懒加载的行为，在点击界面的时候才会去加载需要的模块

  ```
  // /src/index.js
  function getComponent() {
    return import(/* webpackChunkName: "lodash" */ 'lodash').then(
      ({ default: _ }) => {
        var element = document.createElement('div')
        element.innerHTML = _.join(['hello', 'world'], '-')
        return element
      }
    )
  }
  
  document.addEventListener('click', () => {
    getComponent().then(element => {
      document.body.appendChild(element)
    })
  })
  ```

  使用ES7的async和await后，上面的代码改写为下面这样，效果一样

  ```
  // /src/index.js
  async function getComponent() {
    const { default: _ } = await import(/* webpackChunkName: "lodash" */'lodash')
    var element = document.createElement('div')
    element.innerHTML = _.join(['hello', 'world'], '-')
    return element
  }
  
  document.addEventListener('click', () => {
    getComponent().then(element => {
      document.body.appendChild(element)
    })
  })
  ```

## Chunk

打包后生成的每一个 js 文件，都是一个 chunk