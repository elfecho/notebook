# Mixin 混入

- 组件 data，methods 优先级高于 mixin data，methods 优先级

- 生命周期函数，先执行 mixin 里面的，再执行组件里面的

- 自定义的属性，组件内的属性优先级高于 mixin 属性的优先级

  【自定义属性可以通过this.$options获取】

  ```javascript
  // 处理自定义属性的优先级
  app.config.optionMergeStrategies.number = (mixinVal, appValue, vm) => {
     return mixinVal || appValue
  }
  ```

# 自定义指令

- 新建自定义指令

  ```javascript
  // 全局自定义指令
  app.directive('focus', {
      mounted(el) {
          el.focus()
      }
  }
  // 局部自定义指令
  const directives = {
      focus: {
         	mounted(el) {
            	el.focus()
         	}
      }
  }           
  const app = Vue.createApp({
      directives: directives,
      template: `<div><input v-focus /></div>`
  })          
  ```

- 自定义指令传参

  ```javascript
  const app = Vue.createApp({
        data() { 
            return { distance: 110 }
        },
        template: `
          <div>
            <div v-pos:bottom="distance" class="header">
              <input />
            </div>
          </div>
        `
  })
  // 全局自定义指令
  // app.directive('pos', {
  //   mounted(el, binding) {
  //     el.style[binding.arg] = `${binding.value}px`
  //   },
  //   updated(el, binding) {
  //     el.style[binding.arg] = `${binding.value}px`
  //   },
  // })
  // 简写版 (条件是初始化跟更新是一样的操作时)
  app.directive('pos', (el, binding) => {
      el.style[binding.arg] = `${binding.value}px`
  })
  ```


# Teleport——任意传送门

在 `Vue2`，如果想要实现类似的功能，需要通过第三方库 [portal-vue](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FLinusBorg%2Fportal-vue) 去实现

## 为什么我们需要 Teleport

`Teleport` 是一种能够将我们的模板移动到 `DOM` 中 `Vue app` 之外的其他位置

场景：像 `modals`,`toast` 等这样的元素，很多情况下，我们将它完全的和我们的 `Vue` 应用的 `DOM` 完全剥离，管理起来反而会方便容易很多

原因在于如果我们嵌套在 `Vue` 的某个组件内部，那么处理嵌套组件的定位、`z-index` 和样式就会变得很困难

## Teleport 的使用

```javascript
// html:
<div id="hello"></div>

// 添加如下，留意 to 属性跟上面的 id 选择器一致
const app = Vue.createApp({
      methods: {
        handleBtnClick() {
          this.show = !this.show
        }
      },
      template: `
        <div>
          <div class="area">
            <button @click="handleBtnClick">按钮</button>
            <teleport to="#hello">
              <div class="mask"></div>
            </teleport>
          </div>
        </div>
      `
})
```

# 渲染函数 render

> Vue 推荐在绝大多数情况下使用模板来创建你的 HTML。然而在一些场景中，你真的需要 JavaScript 的完全编程的能力。这时你可以用**渲染函数**，它比模板更接近编译器。

## `createElement`参数

`createElement`可以是接受多个参数：

### 第一个参数：`{String | Object | Function}`

第一个参数对于`createElement`而言是一个必须的参数，这个参数可以是字符串`string`、是一个对象`object`，也可以是一个函数`function`。

```vue
<template>
  <div class="dome">
    <div id="app">
    </div>
  </div>
</template>

<script>
  Vue.components('dom', {
    render: function (createElement) {
      return createElement('div');
    }
  });
  new Vue({
    el: '#app'
  });
</script>
```

上面的示例，给`createElement`传了一个`String`参数`'div'`，即传了一个HTML标签字符

### 第二个参数:`{Object}`

`createElement`是一个可选参数，这个参数是一个`Object`。来看一个小示例：

```javascript
Vue.components('dom', {
    render: function (createElement) {
      return createElement('div'， {
        'class': {
          container: true
        },
        style: {
          cursor: 'pointer'
        },
        domProps: {
          innHTML: 'baz'
        }
      });
    }
});
new Vue({
   el: '#app'
});
```

### 第三个参数：{String | Array}

`createElement`还有第三个参数，这个参数是可选的，可以给其传一个`String`或`Array`。比如下面这个小示例：

```javascript
Vue.components('dom', {
    render: function (createElement) {
      return createElement('div', [
        createElement('h1', '主标'),
        createElement('h2', '副标')
      ]);
    }
  });
new Vue({
  el: '#app'
});

```

## 使用JavaScript代替模板功能

在使用Vue模板的时候，我们可以在模板中灵活的使用[`v-if`](https://www.w3cplus.com/vue/v-if-vs-v-show.html)、[`v-for`](https://www.w3cplus.com/vue/v-for.html)、[`v-model`](https://www.w3cplus.com/vue/v-model.html)和 [slot](https://www.w3cplus.com/vue/vue-slot.html) 但在`render`函数中是没有提供专用的API。如果在`render`使用这些，需要使用原生的JavaScript来实现

### `v-if`和`v-for`

在`render`函数中可以使用`if/else`和`map`来实现`template`中的`v-if`和`v-for`。

```xml
<ul v-if="items.length">
    <li v-for="item in items">{{ item }}</li>
</ul>
<p v-else>No items found.</p>
```

换成`render`函数，可以这样写：

```jsx
Vue.component('item-list',{
    props: ['items'],
    render: function (createElement) {
        if (this.items.length) {
            return createElement('ul', this.items.map((item) => {
                return createElement('item')
            }))
        } else {
            return createElement('p', 'No items found.')
        }
    }
})

<div id="app">
    <item-list :items="items"></item-list>
</div>

let app = new Vue({
    el: '#app',
    data () {
        return {
            items: ['大漠', 'W3cplus', 'blog']
        }
    }
})
```

### `v-model`

`render`函数中也没有与`v-model`相应的API，如果要实现`v-model`类似的功能，同样需要使用原生JavaScript来实现。

```jsx
<div id="app">
    <el-input :name="name" @input="val => name = val"></el-input>
</div>

Vue.component('el-input', {
    render: function (createElement) {
        var self = this
        return createElement('input', {
            domProps: {
                value: self.name
            },
            on: {
                input: function (event) {
                    self.$emit('input', event.target.value)
                }
            }
        })
    },
    props: {
        name: String
    }
})

let app = new Vue({
    el: '#app',
    data () {
        return {
            name: '大漠'
        }
    }
})
```

更多详细可以查看 [Vue render函数](https://www.jianshu.com/p/7508d2a114d3)

# plugin 插件

plugin 插件, 也是把通用性的功能封装起来

```javascript
const myPlugin = {
   install(app, options) {
        // 扩展vue全局变量
        app.provide('name', 'elfecho');
        app.directive('focus', {
          mounted(el) {
            el.focus()
          }
        })
        app.mixin({
          mounted() {
            console.log('mixin') // 这里打印了两次，因为父子组件都执行这个生命周期函数
          },
        })
        // vue底层扩展全局属性
        app.config.globalProperties.$sayHello = 'hello world'
    }
}

const app = Vue.createApp({
      mounted() {
        console.log(this.$sayHello)
      },
      template: `<my-title></my-title>`
    })
// 组件使用全局变量时，需要注册inject
app.component('my-title', {
    inject: ['name'],
    template: `
		<h1>{{name}}</h1>
		<input v-focus/>
	`
})

app.use(myPlugin, {
    name: 'elf'
})
    
const vm = app.mount('#root')
```

## 应用

对数据进行校验的插件

```javascript
// 这个写法，相比前面简写了install
const validatorPlugin = (app, options) => {
  app.mixin({
    created() {
      for (let key in this.$options.rules) {
        const item = this.$options.rules[key]
        this.$watch(key, (value) => {
          const result = item.validate(value)
          if (!result) console.log(item.message)
        })
      }
    },
  })
}

const app = Vue.createApp({
  data () {
    return {
      name: 'jun',
      age: 24
    }
  },
  rules: {
    age: {
      validate: age => age > 25,
      message: 'too young, to simple'
    },
    name: {
      validate: name => name.length > 4,
      message: 'name too short'
    }
  },
  template: `name: {{name}},age: {{age}}`
})
app.use(validatorPlugin)
const vm = app.mount('#root')
```

