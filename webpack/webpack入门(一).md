## webpack究竟是什么

1. 面向过程的程序和面向对象的程序   

   面向过程的程序相当于一大长串的代码，会造成文件越来越大，不利于维护      

   面向对象是把一个部分封装到一个类中，当某个东西出现问题时，可以方便的去寻找这个类

2. import 引入js文件      

   浏览器并不认识import，需要通过webpack来解析一下

3. 安装webpack 
	
	```
	npm init      
	npm install webpack webpack-cli 
	```
4. webpack使用    

   npx webpack index.js  (会把index.js中的import转为正常的语法让浏览器认识)

### 什么是模块打包工具

1. webpack是什么？

   webpack是模块打包工具，并不是js的翻译器，因为他只能认识一些简单的import语法，别的识别不了，模块打包的意思是把所有的引入的模块（Moudule）引入到一起，生成一个最终的js文件，模块常见的几种规范：ES Moudule ,CommonJS,CMD,AMD

2. webpack的发展历史

   最开始webpack是打包js的，随着版本的发展，现在可以打包任何常见类型的文件了

## 搭建webpack环境

1. 打包速度     

   作者说明，使用最新版本的node和最新版本的webpack的打包速度是最快的 

2. webpakc安装方式

   - 全局安装 

     ```
     npm install webpack webpack-cli -g   (不建议，考虑到升级后老版本的webpack是否可用)
     
     webpack -v(全局搜索webapck)
     ```

   - 项目安装

     ```
     npm install webpack webpack-cli -D
     
     npx webpack -v (在当前目录下的node_moudule中查找)
     ```

   - 查看某个版本号 是否存在

     ```
     npm info webapck
     
     npm install webpak@4.1.5
     ```

## 使用webpack的配置

1. 默认配置文件放在 webpack.config.js

   ```javascript
   const path = require('path')
   
   module.export = {
       entry: './src/index.js',
       output: {
           filename: 'bundle.js',
           path: path.resolve(__dirname, 'dist')
       }
   }
   ```

   配置好后，运行npx webpack (webpack 先在当前目录下寻找webpack.config.js,如果没有则报错)

   ```
   tips: 使用别名进行打包 npx webpack --config 别名
   ```

2. webpack运行方式

   ```
   webpack index.js   执行全局webpack
   
   
   npx webpack index.js 执行node_modules中的webpack
   
   配置 npm srcript
   npm run bundle    执行node_modules中的webpack
   ```

3. webpack-cli 作用

   是我们可以在命令行中运行webpacks指令

4. 更多可以查阅[getting started](https://www.webpackjs.com/guides/getting-started/)

## webpack打包后打印的信息

![20191028223120307](https://gitee.com/blog2/blog/raw/master/images/20191028223120307.png)

```
hash: 本次打包唯一的一个hash

version:webpack的版本信息

time:打包所消耗的时间

Asset:打包的文件名

size:文件的大小

chunks:文件入口的唯一id，复杂项目的打包可能有多个id

chunk Names:文件入口的名字，相当于entry中的main

entryPoint main=bundle.js   打包的入口文件以及下面列出的都打包了哪些文件
```

## 附录

[webpack官方文档](https://webpack.js.org/) [webpackv4.0官方文档](https://v4.webpack.js.org/)

[webpack中文文档](https://www.webpackjs.com/)