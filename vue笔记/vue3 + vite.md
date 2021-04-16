## 目前搭建 Vue 脚手架的三种方式

目前 vue3.0 的版本已经发布了  此次除了可以通过 vue-cli 和 webpack 搭建脚手架外 官方还提供了一种新的脚手架搭建工具 vite

## 什么是 Vite

Vite 是 Vue 作者开发的一款意图取代 webpack 的工具 

其实现原理是利用 ES6 的 import 会发送请求去加载文件的特性，拦截这些请求，做一些预编译，省去 webpack 冗长的打包时间

## Vite 的安装和运行

### 安装第一版

安装vite1.0.0版本：

```
$ npm install -g create-vite-app
```

使用 vite 创建 Vue3 项目：

```
$ create-vite-app projectName
```

根据提示运行项目：

```
$ cd projectName
$ npm install
$ npm run dev
```

### 安装vite高版本

```
# npm 6.x
npm init @vitejs/app my-vue-app --template vue

# npm 7+, extra double-dash is needed:
npm init @vitejs/app my-vue-app -- --template vue

# yarn
yarn create @vitejs/app my-vue-app --template vue
```

支持的模板预设包括：

- `vanilla`
- `vue`
- `vue-ts`
- `react`
- `react-ts`
- `preact`
- `preact-ts`
- `lit-element`
- `lit-element-ts`

>
> 这里提个醒：使用vite2时，node版本需要使用13.0.0以上的

## vite配置

插件的方式 使用vue/react
 vite.config.js

```css
import path from 'path';
export default {
  alias: {
    '@': path.resolve(__dirname, './src')//设置别名
  },
  plugins: [vue()]
}
```

defineConfig可规范类型

```jsx
import { defineConfig } from 'vite'

export default defineConfig({
 // ...
})
```

函数式配置

```dart
export default ({ command, mode }) => {
  if (command === 'serve') {
    return {
      // serve specific config
    }
  } else {
    return {
      // build specific config
    }
  }
}
```

### 配置请求代理

vite.config.ts

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  base:"./",//打包路径
  resolve: {
    alias:{
      '@': path.resolve(__dirname, './src')//设置别名
    }
  },
  server: {
    port:4000,//启动端口
    open: true,
    proxy: {
      // 选项写法
      '/api': 'http://123.56.85.24:5000'//代理网址
    },
    cors:true
  }
})
```

### setup scripte

- 导入后直接使用，就是免注册

  ```vue
  <template>
  <ShaBe>
  </template>
  <script setup>
      import ShaBe from 'comps/Shabe.vue'
  </script>
  ```

- defineProps defineEmit 子组件中定义属性和事件，

  ```jsx
  //属性定义即用
  defineProps({
      msg: String,
  });
  //方法返回派发方法函数
  const emit = defineEmit(["myClick"]);
  const handleClick = () => {
      emit("myClick", "root");
  };
  ```

- useContext()方法返回上下文

  ```vue
  //子组件中
  const ctx = useContext();
  //抛出事件
  ctx.expose({
    someMethed() {
      console.log("抛出的事件");
    },
  
  //父组件中
  <template>
   //组件实例
    <HelloWorld @myClick="handleMyClick" msg="Hello Vue 3 + Vite"  ref="hw"/>
     
  </template>
  
  <script setup>
  import HelloWorld from 'comps/HelloWorld.vue'
  //ref类型
  import { useContext ,ref} from 'vue'
  const hw=ref(null)
  console.log(useContext(),333)
  const handleMyClick=(msg)=>{
    console.log(msg)
  //hw的value属性上可获取ctx.epose的方法
    hw.value.someMethed()
  }
  </script>
  ```

  

















