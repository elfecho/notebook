> webpack 默认知道如何打包 js 文件，loader 的作用就是告诉 webpack 如何打包其它不同类型的文件

## 打包图片类型的文件

### file-loader

使用 file-loader 打包一些图片文件(需要执行命令 npm i file-loader -D 安装 file-loader)

```JavaScript
// webpack.config.js
module.exports = {
  module: {
    rules: [{
      // test: /\.jpg$/,
      test: /\.(jpg|png|gif)$/, // 配置允许匹配多个文件类型
      use: {
        loader: 'file-loader',
        options: {
          name: '[name]_[hash].[ext]', // 配置打包后文件的名称(name:文件原名;hash:哈希值;ext:文件后缀;最终生成:文件原名_哈希值.文件后缀)，若不配置，文件会以哈希值命名
          outputPath: 'static/img/' // 配置打包后文件放置的路径位置
        }
      }
    }]
  }
}
```

### url-loader

与 file-loader 类似，还可以使用 url-loader 打包一些图片文件(同样需要先安装)

```JavaScript
// webpack.config.js
module.exports = {
  module: {
    rules: [{
      test: /\.(jpg|png|gif)$/,
      use: {
        loader: 'url-loader',
        options: {
          name: '[name]_[hash].[ext]',
          outputPath: 'static/img/',
          limit: 10240 // 与 file-loader 不同的是，可以配置 limit 参数(单位:字节)，当文件大于 limit 值时，会生成独立的文件，小于 limit 值时，直接打包到 js 文件里
        }
      }
    }]
  }
}
```

**注：url-loader 依赖 file-loader，使用 url-loader 同时需要安装 file-loader**

## 打包样式文件

**在 webpack 的配置里，loader 是有先后顺序的，loader 的执行顺序是从下到上，从右到左的**

### 打包 css / sass

**注：node-sass无法安装时，可采用cnpm或查看[node-sass无法安装时的解决办法](https://www.cnblogs.com/whb17bcdq/p/6439417.html)**

```JavaScript
// webpack.config.js
module.exports = {
  module: {
    rules: [{
      test: /\.css$/, // .css 结尾的文件使用 style-loader 和 css-loader 打包(需要安装 style-loader 和 css-loader)
      use: ['style-loader', 'css-loader'] // css-loader 会帮我们分析出多个 css 文件之间的关系，将多个 css 合并成一个 css；style-loader 将 css-loader 处理好的 css 挂载到页面的 head 部分
    }, {
      test: /\.scss$/, // .scss 结尾的文件使用 style-loader 和 css-loader 和 sass-loader 打包(需要安装 style-loader 和 css-loader 和 sass-loader 和 node-sass)
      use: ['style-loader', 'css-loader', 'sass-loader'] // 这里先执行 sass-loader 将 sass 代码翻译成 css 代码；然后再由 css-loader 处理；都处理好了再由 style-loader 将代码挂载到页面上
    }]
  }
}

// index.js
// 配置好之后在 js 中引入 css 即可
import './index.css'
```

### 打包时自动添加厂商前缀

这就是对postCSS一个简单的配置，引入了autoprefixer插件。让postCSS拥有添加前缀的能力，它会根据 [can i use](https://caniuse.com/) 来增加相应的css3属性前缀。

方法一：

```JavaScript
// webpack.config.js
module.exports = {
  module: {
    rules: [{
      test: /\.css/,
      use: ["style-loader", "css-loader", "postcss-loader"] // 需要执行 npm i postcss-loader -D 安装 postcss-loader
    }]
  }
}

// postcss.config.js // 在根目录下创建该文件
module.exports = {
  plugins: [
    require('autoprefixer') // 需要执行 npm i -D autoprefixer 安装 autoprefixer
  ]
}

```

方法二：

```JavaScript
// webpack.config.js
module.exports = {
  module: {
    rules: [{
      test: /\.css/,
      use: [
          "style-loader", 
          "css-loader", 
          {
            loader: 'postcss-loader',
            options: {
              plugins: [
                require("autoprefixer") /*在这里添加*/
              ]
            }
          } 
      ]
    }]
  }
}
```

**注：以上操作后，有可能会没添加前缀**

修改package.json增加如下内容

```JavaScript
"browserslist": [
    "ie>=8",
    "Firefox>=20",
    "Safari>=5",
    "Android>=4",
    "Ios>=6",
    "last 4 version"
]
```

### 给 loader 增加配置项

```JavaScript
// webpack.config.js
module.exports = {
  module: {
    rules: [{
      test: /\.css/,
      use: ['style-loader', {
        loader: 'css-loader',
        options: {
          importLoaders: 1 // 有时候会在一个样式文件里 import 另一个样式文件，这就需要配置 importLoaders 字段，是指在当前 loader 之后指定 n 个数量的 loader 来处理 import 进来的资源(这里是指在 css-loader 之后使用 sass-loader 来处理 import 进来的资源)
        }
      }, 'sass-loader']
    }]
  }
}
```

### css 打包模块化(避免全局引入)

```JavaScript
// webpack.config.js
module.exports = {
  module: {
    rules: [{
      test: /\.css$/,
      use: ['style-loader', {
        loader: 'css-loader',
        options: {
          modules: true // 开启 css 的模块化打包
        }
      }]
    }]
  }
}

// index.css(若使用 sass，增加对应 loader 即可)
.avatar {
  width: 100px;
  height: 100px;
}

// index.js
import style from './index.css'
var img = new Image()
img.src = ''
img.classList.add(style.avatar)
```

## 打包字体文件

```JavaScript
// webpack.config.js
module.exports = {
  module: {
    rules: [{
      test: /\.(eot|ttf|svg|woff)$/,
      use: ['file-loader']
    }]
  }
}
```

以上练习使用的npm包的版本如下

```json
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "private": true,
  "scripts": {
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "autoprefixer": "^9.3.1",
    "lodash": "^4.17.20",
    "css-loader": "^1.0.1",
    "file-loader": "^2.0.0",
    "node-sass": "^4.10.0",
    "postcss-loader": "^3.0.0",
    "sass-loader": "^7.1.0",
    "style-loader": "^0.23.1",
    "url-loader": "^4.1.1",
    "webpack": "^4.44.2",
    "webpack-cli": "^3.1.2"
  },
  "browserslist": [
    "ie>=8",
    "Firefox>=20",
    "Safari>=5",
    "Android>=4",
    "Ios>=6",
    "last 4 version"
  ]
}

```

