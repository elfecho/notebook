### 使用 vue 插件

vite 2.0 提供了对 vue 项目的良好支持，但其本身并不和 vue 进行较强耦合，因此通过插件的形式来支持对 vue 技术栈的项目进行构建。vite 2.0 官网目前（2021.2.28）推荐的 vue 插件会和 [vue3 的 SFC](https://vitejs.dev/plugins/#vitejs-plugin-vue) 一起使用更好。因此这里使用了一个专门用来支持 vue2 的插件 [vite-plugin-vue2](https://www.npmjs.com/package/vite-plugin-vue2)，支持 JSX，同时目前最新版本也是[支持 vite2 的](https://github.com/underfin/vite-plugin-vue2/pull/13)。

使用上也很简单：

```diff
import { defineConfig } from 'vite';
+ import { createVuePlugin } from 'vite-plugin-vue2';

export default defineConfig({
  plugins: [
+   createVuePlugin(),
  ],
});
```

