> 源于目前 vite 搭建的项目没有vue-router、vuex、单元测试以及测试环境，自己搭建一个开箱即用的集成项目

## 集成项目必备的基建

- ts
- vuex
- vue-router
- e2e
  - cypress
- test unit
  - jest + vtu
- eslint + prettier
- verify git commit message
- CI
- alias
- server

## 步骤

### 配置vite

#### 安装

```bash
yarn create @vitejs/app my-vue-app --template vue-ts
```

#### 配置

 vite.config.js

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  base:"./", // 打包路径
  resolve: {
    alias:{
      '@': path.resolve(__dirname, './src') // 设置别名
    }
  },
  server: {
    port:4000,// 启动端口
    open: true,
    proxy: {
      // 选项写法
      '/api': 'http://123.56.85.24:5000'// 代理网址
    },
    cors:true
  }
})
```

### 配置路由

```bash
yarn add vue-router@4 --save
```

> 在src下新建router目录，新建index.ts文件

```javascript
import { createRouter, createWebHashHistory, RouteRecordRaw } from "vue-router";
const routes: Array<RouteRecordRaw> = [
  {
    path: "/",
    name: "Home",
    meta: {
      title: "首页",
      keepAlive: true
    },
    component: () => import("../views/Home/index.vue"),
  },
  {
    path: "/about",
    name: "About",
    meta: {
      title: "关于",
      keepAlive: true
    },
    component: () => import("../views/About/index.vue"),
  },
];
const router = createRouter({
  history: createWebHashHistory(),
  routes
});
export default router;
```

在main.ts挂载路由

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from "./router"
createApp(App)
.use(router)
.mount('#app')
```

### 配置数据中心vuex（4.x）

#### 安装

```bash
yarn add vuex@next --save
```

#### 配置

在src下创建store目录，并在store下创建index.ts

```javascript
import { createStore } from "vuex";
export default createStore({
  state: {
    listData:{1:10},
    num:10
  },
  mutations: {
    setData(state,value){
        state.listData=value
    },
    addNum(state){
      state.num=state.num+10
    }
  },
  actions: {
    setData(context,value){
      context.commit('setData',value)
    },
  },
  modules: {}
});
```

#### 挂载

在main.ts挂载数据中心

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from "./router";
import store from "./store";
createApp(App)
.use(router)
.use(store)
.mount('#app')
```

### 对 sass的支持

> 不需要引入 sass 与webpack 相关的依赖包和loader

```bash
yarn add -D sass
```

### 配置cssnext处理浏览器兼容差异问题

#### 安装cssnext

```csharp
yarn add postcss-cssnext -S
```

#### 配置postcss-cssnext

```javascript
module.exports = {
  plugins: {
    "postcss-cssnext": {
      browsers: [
        "Android >= 4.0",
        "iOS >= 7",
        "Chrome > 31",
        "ff > 31",
        "ie >= 8",
        "last 10 versions"
      ]
    }
  }
};
```

### 移动端适配

采用的移动端适配方式是将px转rem，大家第一时间想到的就是使用postcss-pxtorem

#### 安装

```shell
yarn add postcss-pxtorem -D
```

#### 配置

之前已经配置过postcss-cssnext了，而postcss-pxtorem同样也只是postcss的一个插件，所以可以公用一个配置文件

```javascript
module.exports = {
  plugins: {
    "postcss-cssnext": {
      browsers: [
        "Android >= 4.0",
        "iOS >= 7",
        "Chrome > 31",
        "ff > 31",
        "ie >= 8",
        "last 10 versions"
      ]
    },
    'postcss-pxtorem': {
      rootValue: 37.5,
      propList: ['*'],
      exclude: /node_modules/i
    }
  }
};
```

> 这里不再需要配置autoprefixer，因为postcss-cssnext中已经集成了autoprefixer

### 安装eslint&prettier

安装eslint、prettier

```bash
yarn add -D eslint eslint-plugin-prettier
```

对vue进行代码美化

```bash
yarn add -D eslint-plugin-vue @vue/eslint-config-prettier @vue/eslint-config-typescript
```

对ts进行代码美化

```bash
yarn add -D @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

#### 配置 prettier

在根目录创建.prettierrc.json

```json
{
  "semi": false, // 结尾不加分号
  "tabWidth": 2, //指定每个缩进级别的空格数
  "arrowParens": "avoid",
  "singleQuote": true, // 使用单引号
  "trailingComma": "all" //将>多行JSX元素放在最后一行的末尾
}
```

我们使用尤雨溪的配置，句尾不带分号 + 单引号。

- 尤雨溪配置：[vue-next/.prettierrc](https://github.com/vuejs/vue-next/blob/master/.prettierrc)
- 更多配置：[官方配置文档](https://prettier.io/docs/en/options.html)

### 集成基于jest&vtu的单元测试

安装单元测试模块 jest

```bash
yarn add jest --dev
```

安装 jest 自动代码提示

```bash
yarn add @types/jest --dev
```

运行jest脚本时，es6的语法是跑不通的，需要使用babel

#### 生成基础配置文件

Jest 将根据你的项目提出一系列问题，并且将创建一个基础配置文件。文件中的每一项都配有简短的说明：

```bash
jest --init
```

此时会生成一个文件`jest.config.js`,并添加如下

```javascript
module.exports = {
  transform: {
    '^.+\\.jsx?$': 'babel-jest', // Adding this line solved the issue
  }
}
```

#### 使用 Babel

要使用 [Babel](https://babeljs.io/)，请通过 `yarn` 来安装所需的依赖项：

```bash
yarn add --dev babel-jest @babel/core @babel/preset-env
```

在项目的根目录下创建 `babel.config.js` ，通过配置 Babel 使其能够兼容当前的 Node 版本。

```javascript
// babel.config.js
module.exports = {
  presets: [['@babel/preset-env', {targets: {node: 'current'}}]],
};
```

#### 使用vue

要使用 Vue，请通过 `yarn` 来安装所需的依赖项：

```bash
yarn add vue-jest@next --dev
```

在`jest.config.js`添加

```javascript
transform: {
	//  用 `vue-jest` 处理 `*.vue` 文件
    '^.+\\.vue$': 'vue-jest',
}
```

还得需要安装Vue Test Utils 【Vue.js 官方的单元测试实用工具库】

```
yarn add @vue/test-utils@next --dev
```

#### 使用typescript

要使用 typescript，请通过 `yarn` 来安装所需的依赖项：

```bash
yarn add ts-jest --dev
```

使用 `yarn` 安装 `@babel/preset-typescript` ：

```bash
yarn add --dev @babel/preset-typescript
```

然后将 `@babel/preset-typescript` 添加到 `babel.config.js` 中的 presets 列表中。

```diff
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', {targets: {node: 'current'}}],
+    '@babel/preset-typescript',
  ],
};
```

还得需要修改`tsconfig.json`文件

```diff
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "jsx": "preserve",
    "sourceMap": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "lib": [
      "esnext",
      "dom"
    ],
    "types": [
      "vite/client",
+      "jest"
    ]
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.d.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
+    "tests"
  ]
}
```

### 集成基于 cypress 的 e2e 测试环境

安装

```bash
yarn add cypress --dev
```

首次使用cypress时，需要安装cypress

```bash
npx cypress install
```

安装没有完全成功，执行命令重新安装

```bash
npx cypress cache clear
npx cypress install
```

## 结语

这是本人搭建[vue3集成项目](https://gitee.com/elfeach/vite-scaffold-template)仓库，欢迎大家star一下