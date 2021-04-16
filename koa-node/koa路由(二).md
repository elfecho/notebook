# 安装

```shell
# npm ..  
npm i koa-router
# yarn ..  
yarn add koa-router
```

# 使用

```javascript
const Koa = require('koa');
const Router = require('koa-router')

const app = new Koa();
const router = new Router();

router.get('/', (ctx, next) => {
    // ctx.router available
})

app
    .use(router.routes())
	.use(router.allowedMethods());

```

# 客户端兼容性

对于版本升级时，我们需要考虑api携带版本号，一般做三个版本的兼容，常用的方法有

- 路径
- 查询参数
- header

编写代码需要遵循开闭原则(修改代码关闭，扩展代码开放)

# 配置

主入口js文件

```javascript
const Koa = require('koa');
const book = require('./api/v1/book')
const classic=require('./api/v1/classic')

const app = new Koa();

app.use(book.routes())
app.use(classic.routes())

app.listen(3000, () => {
  console.log('端口已开启， localhost:3000')
});
```

book.js

```javascript
const Router=require('koa-router')
const router=new Router()

router.get('/v1/book/latest', async (ctx, next) => {
  ctx.body = {
    key: 'book'
  }
})

module.exports = router
```

classic.js

```javascript
const Router=require('koa-router')
const router=new Router()

router.get('/v1/classic/latest', async (ctx, next) => {
  ctx.body = {
    key: 'classic'
  }
})

module.exports = router
```

# requireDirector实现路由自动加载

- 安装require-director  # npm i require-director

- 使用require-diretory库简化router的注册步骤

  ```javascript
  const Koa = require('koa');
  const Router=require('koa-router')
  const requireDirectory = require('require-directory')
  
  const app = new Koa();
  
  requireDirectory(module, './app/api', {
    visit: whenLoadModule
  })
  
  function whenLoadModule(obj) {
    if (obj instanceof Router) {
      app.use(obj.routes())
    }
  }
  
  app.listen(3000);
  ```

# 初始化管理器

进一步封装，把app中初始化的代码提取出来

core/init.js

```javascript
const Router=require('koa-router')
const requireDirectory = require('require-directory')
class InitManager {
  static initCore(app) {
    // 入口方法
    InitManager.app = app
    InitManager.initLoadRouters()
  }
  static initLoadRouters() {
    requireDirectory(module, '../app/api', {
      visit: whenLoadModule
    })
    
    function whenLoadModule(obj) {
      if (obj instanceof Router) {
        InitManager.app.use(obj.routes())
      }
    }
  }
}

module.exports = InitManager
```

app.js

```javascript
const Koa=require('koa')
const InitManager=require('./core/init')

const app=new Koa()
InitManager.initCore(app)
app.listen(8000)
```

