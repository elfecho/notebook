# composition api出现原因

> Vue3中提出的一个新**概念**, 作用: 聚合代码 & 逻辑重用

vue2组件化时，可以帮助我们更好的拆分代码和约束代码，增加可读性，但是有存在一些问题

1. 代码逻辑不够聚合, 比较分散, 如果我们的组件代码量一多, 找相同的代码逻辑会显得比较困难
2. 逻辑重用时只能使用 mixin 来混入逻辑，但 mixin 一旦多处使用时，很难维护
3. vue的侵入性太强, 导致对typescript这类高阶技术的用法不够自然

**基于上面的种种问题, vue推出了composition api的概念**

composition api(组合式api)本质上就是vue抽离了一系列方法可以供我们自由引入组合使用, 这里抛出一个问题

在不改变vue模板语法的情况下, vue提供了一个新的函数书写在模板的`script`标签中, 该函数名叫做 **setup**

# setup

setup() 函数是 vue3 中，专门为组件提供的新属性。它为我们使用 vue3 的 Composition API 新特性提供了统一的入口。

## 执行时机

**setup** 函数会在 **beforeCreate** 之后、**created** 之前执行

## 接收 props 数据

在 props 中定义当前组件允许外界传递过来的参数名称：

```javascript
props: {
    p1: String
}
```

通过 setup 函数的**第一个形参**，接收 props 数据：

```javascript
setup(props) {
    console.log(props.p1)
}
```

## context

setup 函数的第二个形参是一个**上下文对象**，这个上下文对象中包含了一些有用的属性，这些属性在 vue 2.x 中需要通过 this 才能访问到，在 vue 3.x 中，它们的访问方式如下：

```javascript
const MyComponent = {
  setup(props, context) {
    console.log(context.attrs) // 它是绑定到组件中的 非 props 数据，并且是非响应式的
    console.log(context.slots) // 组件的插槽，同样也不是 响应式的
    console.log(context.parent) 
    console.log(context.root)
    console.log(context.emit) // 一个方法，相当于 vue2 中的 this.$emit 方法
    console.log(context.refs)
  }
}
```

注意：在 setup() 函数中无法访问到 this 

## 生命周期函数

可以使用导入的`onXXX`的形式注册生命周期函数，举个例子：

| 选项 API        | setup内部的钩子   |
| :-------------- | :---------------- |
| beforeCreate    | 不需要            |
| created         | 不需要            |
| beforeMount     | onBeforeMount     |
| mounted         | onMounted         |
| beforeUpdate    | onBeforeUpdate    |
| updated         | onUpdated         |
| beforeUnmount   | onBeforeUnmount   |
| unmounted       | onUnmounted       |
| errorCaptured   | onErrorCaptured   |
| renderTracked   | onRenderTracked   |
| renderTriggered | onRenderTriggered |

**onRenderTracked**

- 每次渲染后重新收集响应式依赖
- 在onMounted前触发，页面更新后也会触发

**renderTriggered**

- 每次触发页面重新渲染时自动执行

- 在onBeforeUpdate之前触发

> TIP
>
> 由于setup是随着beforeCreate和created这两个生命周期钩子运行的，因此在你无需显式地定义它们。换句话说，任何想写进这两个钩子的代码，都应当直接写在setup方法里面。

```javascript
setup() {
    onMounted(()=>{
        console.log('mounted被触发')
    })
    ///...其他类似
}
```

## methods

使用普通的函数定义方法，这样可以最大程度的增加复用性。例如：

```javascript
setup() {
    function add() {
        console.log('add被触发')
    }
    return {
        add//必须将函数return
    }
}
```

## computed

当页面中有某些数据依赖其他数据进行变动的时候，可以使用计算属性。

```js
setup(props, context) {
    const { reactive, computed } = Vue;
    const countObj = reactive({
        count: 0
    })
    const handleCount = () => {
        countObj.count += 1
    }
    // const countAddFive = computed(() => {
    //   return count.value + 5;
    // })
    // 扩展方法
    const countAddFive = computed({
        get() {
            return countObj.count + 5;
        },
        set(param) {
            countObj.count = param - 5 
        }
    })
    setTimeout(() => {
        countAddFive.value = 100
    }, 3000)
    return {
        countObj,
        countAddFive,
        handleCount
    }
}
```

这样我们就定义号了一个计算属性`countAddFive`

## 侦听器

### watch

- 具有一定的惰性 lazy
- 参数可以拿到原始和当前值
- 可以侦听多个数据的变化，用一个侦听器承载

```javascript
const nameObj = reactive({
    name: 'elf'
})
watch(
    () => nameObj.name, // 监听reactive参数的话，需要使用箭头函数
    (val, oldVal) => {
        console.log(val, oldVal)
    }
)
```

当需要监听多个参数时，需要将监听参数用数组形式

```javascript
const nameObj = reactive({
    name: 'elf',
    englishName: 'echo'
})
watch(
    [ () => nameObj.name, () => nameObj.englishName],
    ([curName, curEng], [prevName, prevEng]) => {
        console.log(curName, prevName, '-----', curEng, prevEng)
    }
)
```

**watch第三个参数用法**

```javascript
watch(nameObj, (val, oldVal) => {
    console.log(val, oldVal)
}, {
    immediate: true, // 默认只有数据改变才会监听，第一次不会执行,设置true则第一次也能执行
    deep: true // 可以深度检测到 nameObj 对象的属性值的变化
})
```

### watchEffect

- 立即执行，没有惰性 immediate
- 不需要传递你要侦听的内容，自动会感知代码依赖，不需要传递很多参数，只要传递一个回调函数
- 不能获取之前数据的值

**使用场景**

异步请求或定时器时可以使用watchEffect

```javascript
watchEffect(() => {
    console.log(nameObj.name)
    console.log(nameObj.englishName)
})
```

如果需要停用侦听器，可以使用以下方法，watch也类似

```javascript
const stop = watchEffect(() => {
    console.log(nameObj.name)
    console.log(nameObj.englishName)
    setTimeout(() => {
        stop()
    }, 5000)
})
```

## provide & inject

常用的父子组件通信方式都是父组件绑定要传递给子组件的数据，子组件通过`props`属性接收，一旦组件层级变多时，采用这种方式一级一级传递值非常麻烦，而且代码可读性不高，不便后期维护。

`vue`提供了`provide`和`inject`帮助我们解决多层次嵌套嵌套通信问题。在`provide`中指定要传递给子孙组件的数据，子孙组件通过`inject`注入祖父组件传递过来的数据。

其实，**`provide` 和 `inject `主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码中**。

**注意：子孙层的`provide`会掩盖祖父层`provide`中相同`key`的属性值**

```javascript
// 父组件
provide('name', readonly(name)) // 加readonly 防止子组件修改父组件的值，违反了单向数据流的要求
provide('handleClick', () => {
    name.value = 'echo'
})

// 子组件
const name = inject('name')
const handleClick = inject('handleClick')
```

## 获取Vue组件实例或者dom节点

```javascript
// 父组件
setup(props, context) {
    const { ref, onMounted } = Vue;
    const hello = ref(null) // dom节点对象
    const child = ref(null) // child 组件的实例
    onMounted(() => {
        console.log(hello.value)
        child.value.childClick()
    })
    return {
        hello,
        child
    }
},
template: `
	<div>
		<div ref="hello">hello world</div>
		<child ref="child" />
	</div>
`
// 子组件
setup(props) {
    function childClick() {
        console.log('this is child')
    }
    return {
        childClick
    }
},
template: `
	<div>child</div>
`
```

这个hello或组件child与下面template的ref的hello、child对应，打印结果如下：

![image-20210226142041400](https://gitee.com/blog2/blog/raw/master/images/image-20210226142041400.png)

## 返回值

`setup` 函数中**返回一个对象**，可以**在模板中直接访问该对象中的属性和方法**。

```javascript
const app = Vue.createApp({
  template: `
    <div @click="handleClick">name: {{name}}</div>
  `,
  setup(props, context) {
    console.log(this); // window
    return {
      name: 'elfecho',
      handleClick() {
        console.log(123)
      }
    }
  }
})
```

## 疑惑解答

### setup函数太长了怎么办

可以使用函数拆分

```javascript
setup(props, context) {
    // 流程调度
    const { list, addItemTolist } = listRelativeEffect();
    const { inputValue, changeInputValue } = inputRelativeEffect();
    return {
        list, addItemTolist,
        inputValue, changeInputValue
    }
}
```

return的上下文太长了，我们可以使用vue3的setup script功能，把setup这个配置也优化掉，一个功能export一次

```html
<script setup>
import listRelativeEffect from './listRelativeEffect'
import inputRelativeEffect from './inputRelativeEffect'

let { list, addItemTolist } = listRelativeEffect()
export {list, addItemTolist}

let {count,double,add} = inputRelativeEffect()
export {inputValue, changeInputValue}
</script>
```



## 总结

那么我们最后总结一下 `setup` 函数的所有特性：

- setup 函数是组合式 API 的入口函数，它在 **组件创建之前** 被调用
- 因为在 `setup` 执行时组件尚未创建，`setup` 函数中的 `this` 不是当前组件的实例
- 函数接收两个参数，props 和 context，context 可以解构为 attrs、slots、emit 函数
- 函数可以返回一个对象，对象的属性可以直接在模板中进行使用，就像之前使用 data 和 methods 一样。

# ref, reactive代替data中的变量

composition-api引入了独立的数据响应式方法reactive，它可以将传入的对象做响应式处理：

```javascript
const { reactive } = Vue;
const state = reactive({
    name: 'elfecho'
})
setTimeout(() => {
    state.name = 'elf'
}, 2000)
```

这个方式类似于我们设置的data选项，能够解决我们大部分需求。但是也有以下问题：

- 当我们直接导出这个state时，我们在模板中就要带上state这个前缀

  ```javascript
  setup() {
      const state = reactive({})
      return { state }
  }
  ```

  ```html
  <div>{{ state.name }}</div>
  ```

  为了解决这个前缀的问题，又要引入toRefs

  ```javascript
  setup() {
      const state = reactive({})
      // toRefs proxy({name: 'elfecho'}) 会转化为 {
      //    name: proxy({value: 'elfecho'})
      // }
      return { ...toRefs(state) }
  }
  ```

  ```html
  <div>{{ name }}</div>
  ```

- 单个值时用reactive()显得比较多余

  于是就有了`Ref`的概念，通过包装单值为`Ref`对象，这样就可以对其做响应式代理

  ```javascript
  setup() {
      const name = ref('name')
      return { name } 
  }
  ```

  模板中使用还可以省掉前缀，toRefs就是利用这一点将reactive()返回代理对象的每个key对应的值都转换为Ref

  ```html
  <div>{{ foo }}</div>
  ```

  但是Ref对象也有副作用：

  在JS中修改这个值要额外加上value

  ```javascript
  // proxy， 'elfecho' 变成 proxy({value: 'elfecho'})这样的一个响应式引用
  let name = ref('elfecho')
  setTimeout(() => {
      name.value = 'elf'
  }, 2000)
  return {
      name
  }
  ```

## 如何选择ref or reactive

- 如果是单值，建议ref，哪怕是个单值的对象也可以

  ```javascript
  const counterRef = ref(1)
  const usersRef = ref(['tom', 'jerry'])
  ```

  一个业务关注点有多个值，建议reactive

  ```javascript
  const mouse = reactive({
      x: 0,
      y: 0
  })
  ```

- 降低Ref负担的方法：利用unref、isRef、isProxy等工具方法，利用一些命名约定。

  ```javascript
  setup(props) {
    const foo = unref(props.foo) // foo是我们要的值
    // 等效于
    const foo = isRef(props.foo) ? props.foo.value : props.foo
  }
  ```

## 扩展

### readonly 

如果不想对对象进行变更，作为只读的参数，Vue提供了 readonly 方法

```javascript
const state = reactive({
    name: 'elfecho'
})
const copyState = readonly(state) // 这个对象不允许被修改
setTimeout(() => {
    state.name = 'elf'
    copyState.name = 'elf' // 此处对这个值进行变更，会出现警告
}, 2000)
```

### toRef

如果对一个未定义的值进行编辑时，需要使用toRef

```javascript
const data = reactive({
    name: 'elfecho'
})
const { name } = toRefs(data)
const age = toRef(data, 'age')
setTimeout(() => {
    age.value = '20'
}, 2000);
return {
    name,
    age
}
```





