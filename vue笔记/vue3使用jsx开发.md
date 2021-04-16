## vue3中使用jsx

### vue-cli 使用 jsx

安装

```bash
yarn add @vue/babel-plugin-jsx -D
```

在项目的**babel.config.js**中的**plugins**添加，下面是在脚手架生成文件基础下添加:

```javascript
module.exports = {
  plugins: ['@vue/babel-plugin-jsx']
}
```

### vite2.0 使用 jsx

安装

```bash
yarn add @vitejs/plugin-vue-jsx -D
```

在项目的**vite.config.ts**中的**plugins**添加

```diff
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
+ import vueJsx from '@vitejs/plugin-vue-jsx'
import path from 'path'

// https://vitejs.dev/config/
export default defineConfig({
+  plugins: [vue(), vueJsx()],
  base: './', // 打包路径
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'), // 设置别名
      components: path.resolve(__dirname, './src/components'),
    },
  },
})
```

创建tsx文件以及用法 使用ts写jsx的文件就是tsx文件，用tsx文件来代替我们平常写的vue文件，下面是两种文件的区别：

**.vue文件：**

```vue
<template>
  <div>{{bar}}</div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'
export default defineComponent({
  setup () {
    const bar = ref('hello')
    return {
      bar
    }
  }
})
</script>
```

**.tsx文件：**

```tsx
import { defineComponent, ref } from 'vue'
export default defineComponent({
 setup () {
    const bar = ref('hello')
    return () => {
      return <div>{bar.value}</div>
    }
  }
})
```


