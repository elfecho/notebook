# 获取参数

- 获取请求体需要安装koa-bodyparser， # npm i koa-bodyparser -S

- 在app.js的中间件加入

  ```javascript
  const parser = require('koa-bodyparser')
  app.use(parser())
  ```

```javascript
const Router = require('koa-router')
const router = new Router()
router.post('/v1/:id/classic/latest', (ctx, next) => {
    //获取四种参数
    //1,获取路径上的参数
    const path=ctx.params
    //2,获取路径后面?的参数
    const query=ctx.request.query
    //3,获取头部文件
    const headers=ctx.request.header
    //4,获取请求体
    const body=ctx.request.body
})
module.exports = router
```

# 已知错误与未知错

异常捕获在异步代码中失效的问题

```javascript
function func1() {
  try {
    func2()
  } catch (error) {
    console.log('error')
  }
}

function func2() {
  setTimeout(function() {
    throw new Error('error')
  }, 1000)
}

func1()
```

解决办法为使用promise进行封装

```javascript
async function func1() {
  try {
    await func2()
  } catch (error) {
    console.log('捕捉到异常')
  }
}

function func2() {
  return new Promise((resolve, reject) => {
    setTimeout(function() {
      reject('error')
    }, 1000)
  })
  
}

func1()
```

