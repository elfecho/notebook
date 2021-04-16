# 简介

> Koa 是一个新的 web 框架，由 Express 幕后的原班人马打造， 致力于成为 web 应用和 API 开发领域中的一个更小、更富有表现力、更健壮的基石。

[官方中文文档](https://koa.bootcss.com/)

# koa的中间件

中间件通过next()方法向下传递

```javascript
//简单示例
const koa = require('koa')
const app = new koa()
app.use((ctx,next)=>{
    //上下文
    console.log(next)
    console.log('hello,7月')
    next()
})
app.use((ctx,next)=>{
    console.log(next)
    console.log('hello')
    next()
})
app.listen(3000)
console.log('[demo] start-quick is starting at port 3000')
```

# 洋葱模型

```javascript
const koa = require('koa')
const app = new koa()
app.use((ctx,next)=>{
    //上下文，洋葱模型
    console.log(1)
    next()
    console.log(2)
})
app.use((ctx,next)=>{
    console.log(3)
    next()
    console.log(4)
})
app.listen(3000)
```

访问后的输出顺序为，1，3，4，2，中间件的的传递类似于普通函数

获取另外一个中间件返回的值

```javascript
const koa = require('koa')
const app = new koa()
app.use((ctx,next)=>{
    //上下文，洋葱模型
    console.log(1)
    const a=next()
    a.then((res)=>{
        console.log(res)
    })
    console.log(2)
})
app.use((ctx,next)=>{
    console.log(3)
    next()
    console.log(4)
    return 'hello,yue'
})
app.listen(3000)
```

输出结果

```shell
1
3
4
2
hello,yue
```

但是上述的代码可能存在某些问题，比如某个中间件是耗时操作，因此，我们有必要让每个中间件都使用async和await关键字，并且通过await关键之来获取另外一个中间件的值

```javascript
const koa = require('koa')
const app = new koa()
app.use(async(ctx,next)=>{
    //上下文，洋葱模型
    console.log(1)
    const a=await next()
    console.log(a)
    // a.then((res)=>{
    //     console.log(res)
    // })
    console.log(2)
})
app.use((ctx,next)=>{
    console.log(3)
    next()
    console.log(4)
    return 'hello,yue'
})
app.listen(3000)
```

输出结果

```javascript
1
3
4
hello,yue
2
```