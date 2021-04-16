[ajv官方文档](https://ajv.js.org/)

## 使用ajv校验json schema

- 安装

  ```bash
  yarn add ajv -S
  ```

- 使用

  ```javascript
  // or ESM/TypeScript import
  import Ajv from "ajv"
  // Node.js require:
  const Ajv = require("ajv").default
  
  const schema = {
      type: 'string',
      minLength: 5
  }
  
  const ajv = new Ajv() // options can be passed, e.g. {allErrors: true}
  const validate = ajv.compile(schema)
  const valid = validate('elfecho')
  if (!valid) console.log(validate.errors)
  ```

## 自定义format

> Tips: 新版使用format时需要引入ajv-formats的库`yarn add ajv-formats -S`

```javascript
const ajv = new Ajv() 
require("ajv-formats")(ajv)
```

##### **自定义formart**

```
ajv.addFormat('test', (data) => {
  console.log(data, '--------------')
  return data === 'haha'
})
```

这个是存在ajv内部的，在其他的json-schema是不能共用的

## 自定义关键字

[详细文档](https://ajv.js.org/guide/user-keywords.html)

定义关键字的优点是：

- 允许创建无法使用预定义关键字表示的验证方案
- 简化您的架构
- 帮助将验证逻辑的大部分带入架构
- 使您的模式更具表现力，更少冗长且更接近您的应用程序域
- 实施可修改数据的数据处理器（`modifying`必须在关键字定义中使用选项）和/或在验证数据时产生副作用

```javascript
// schema
const schema = {
  type: 'object',
  properties: {
    name: {
      type: 'string',
      test: false // 自定义的关键字
    },
  }
}

const valid = validate({
  name: 'hahaha'
})
```

### 验证功能 validation function

```javascript
ajv.addKeyword({
  keyword: 'test',
  validate(schema, data) {
    console.log(schema, data) // false 'hahaha'
    // schema：上面定义的schema关键字test的value值， data: validate下输入的value值
    if (schema === true) return true
    else return data.length === 6
  }
})
```

### 编译功能 compilation function

```javascript
ajv.addKeyword({
  keyword: 'test',
  compile(sch, parentSchema) {
    console.log(sch, parentSchema) // false { type: 'string', test: false }
    // sch: 上面定义的schema关键字test的value值 parentSchema: 上面定义关键字的对象的值
    return () => true
  },
  schemaType: 'boolean' // 限制关键字test的value值的类型
})
```

### 宏功能 macro function

```javascript
ajv.addKeyword({
  keyword: 'test',
  macro() {
    return {
      minLength: 10 // 为引用自定义关键字的schema添加minLength条件
    }
  },
})
```

## 设置错误提示的语言

[github ajv-i18n](https://github.com/ajv-validator/ajv-i18n)

安装

```
yarn add ajv-i18n
```

```diff
const Ajv = require('ajv').default
+ var localize = require('ajv-i18n')

const schema = {
  type: 'object',
  properties: {
    name: {
      type: 'string',
    },
}

const validate = ajv.compile(schema)
const valid = validate({
  name: 'hahaha'
})
if (!valid) {
+  localize.zh(validate.errors)
  console.log(validate.errors)
}
```

## 自定义错误信息

[github ajv-errors](https://github.com/ajv-validator/ajv-errors) 

安装

```
yarn add ajv-errors
```

```javascript
const Ajv = require("ajv").default
const ajv = new Ajv({allErrors: true})
// Ajv option allErrors is required
require("ajv-errors")(ajv /*, {singleError: true} */)
```

```diff
const schema = {
  type: 'object',
  properties: {
    name: {
      type: 'string',
      minLength: 10,
+      errorMessage: {
+        type: '必须是字符串',
+        minLength: '长度不能小于10个字符'
+      },
    }
  }
}
+ const ajv = new Ajv({ allErrors: true, jsonPointers: true }) 
+ require("ajv-errors")(ajv)
```

