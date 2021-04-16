## 背景及原理

typeScript、Flow、Eslint、StyleLint都已经有自动化测试的功能了

当前端编写几个方法时，都是要到测试环境才会进行，那我们前端能不能进行白盒测试呢？答案是可以的

例如我们我们现在编写一个math.js

```javascript
function add(a, b) {
  return a + b;
}
function minus(a, b) {
  return a - b;
}
```

我们可以在其根目录再新建一个math.test.js进行对math的方法进行测试

```javascript
function expect(result) {
  return {
    toBe: function(actual) {
      if (result !== actual) {
        throw new Error('预期值和实际值不匹配')
      }
    }
  }
}

function test(desc, fn) {
  try {
    fn();
    console.log(`${desc} 通过测试`)
  } catch (e) {
    console.log(`${desc} 没有通过测试 ${e}`)
  }
}

test('测试加法 3 + 7', () => {
  expect(add(7, 3)).toBe(10);
})

test('测试减法 3 - 3', () => {
  expect(minus(3, 3)).toBe(0);
})

```

## 优势

![image-20210310134818196](https://gitee.com/blog2/blog/raw/master/images/image-20210310134818196.png)

## 安装

```bash
npm install jest
```

改造以上例子

math.js

```javascript
function add(a, b) {
  return a + b;
}
function minus(a, b) {
  return a - b;
}
function multi(a, b) {
  return a * b;
}
module.exports = {
    add,
    minus,
    multi
}
```

math.test.js

```javascript
const math = require('./math.js');
const { add, minus, multi } = math;

test('测试加法 3 + 7', () => {
  expect(add(7, 3)).toBe(10);
})

test('测试减法 3 - 3', () => {
  expect(minus(3, 3)).toBe(0);
})
```

将package.json中的

```javascript
"scripts": {
    "test": "jest"
},
```

运行 `npm run test`就可以查看出模块是否测试通过

## 配置

使用一下命令，可以进行暴露jest默认的配置

```bash
npx jest --init
```

运行完后根目录会生成jest.config.js

- 如何查看测试覆盖率，可以执行 

  ```bash
  npx jest --coverage
  ```

  此时会根目录生成一个coverage目录

### 使用 Babel

要使用 [Babel](https://links.jianshu.com/go?to=https%3A%2F%2Fbabeljs.io%2F)，请通过 `npm`的依赖项：

```javascript
 npm i @babel/core@7.4.5 @babel/preset-env@7.4.5 -D
```

在项目的根目录下创建 `.babelrc` ，通过配置 Babel 使其能够兼容当前的 Node 版本。

```javascript
{
  "presets": [
    ["@babel/preset-env", {
      "targets": {
        "node": "current"
      }
    }]
  ]
}
```

## 匹配器

### 普通匹配器

toBe()  用于`Object.is`测试完全相等

```javascript
test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
});
```

`toEqual` 递归检查对象或数组的每个字段

```javascript
test('object assignment', () => {
  const data = {one: 1};
  expect(data).toEqual({one: 1});
});
```

### 真假有关的匹配器

- `toBeNull` 仅匹配 `null`
- `toBeUndefined` 仅匹配 `undefined`
- `toBeDefined` 与...相反 `toBeUndefined`
- `toBeTruthy`匹配`if`语句视为真的任何内容
- `toBeFalsy`匹配`if`语句视为假的任何内容
- `not` 匹配器  取相反的值 !

```javascript
test('null', () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).toBeFalsy();
});
```

### Number 匹配器

- toBeGreaterThan 是否比匹配的值大
- toBeLessThan 是否比匹配的值小
- toBeGreaterThanOrEqual 是否大于或等于匹配值
- toBeLessThanOrEqual 是否小于或等于匹配值
- toBeCloseTo 判断浮点数相加时，js引起的浮点数溢出时，需使用

```javascript
test('two plus two', () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3);
  expect(value).toBeGreaterThanOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessThanOrEqual(4.5);

  // toBe and toEqual are equivalent for numbers
  expect(value).toBe(4);
  expect(value).toEqual(4);
});
```

对于浮点相等，请使用`toBeCloseTo`代替`toEqual`，因为您不希望测试依赖于微小的舍入误差。

```js
test('adding floating point numbers', () => {
  const value = 0.1 + 0.2;
  //expect(value).toBe(0.3);           This won't work because of rounding error
  expect(value).toBeCloseTo(0.3); // This works.
});
```

### String匹配器

您可以使用`toMatch`以下命令针对正则表达式检查字符串：

```javascript
test('toMatch', () => {
  const str = 'hello world';
  expect(str).toMatch(/world/);
})
```

### Array和Set匹配器

可以使用以下命令检查数组或可迭代项是否包含特定项目`toContain`：

```javascript
test('toContain', () => {
  const arr = ['hello', 'world'];
  expect(arr).toContain('hello');
  expect(new Set(arr)).toContain('hello');
})
```

### 异常匹配器

如果要测试特定函数在调用时是否引发错误，请使用`toThrow`

```javascript
const throwNewErrorFunc = function() {
  throw new Error('this is a new error.')
}
test('toThrow', () => {
  expect(throwNewErrorFunc).toThrow();
  expect(throwNewErrorFunc).toThrow(Error);

  // 如果你想判断错误的提示信息是否匹配
  expect(throwNewErrorFunc).toThrow('this is a new error.');
  expect(throwNewErrorFunc).toThrow(/error/);
})
```

## 命令行工具使用

- 按a键运行所有测试。类似于 jest --watchAll
- 按f键仅运行失败的测试。
- 按o键只运行与更改文件相关的测试。类似于 jest --watch 【要在git环境下运行】
- 按p键可按文件名regex模式进行筛选。 
- 按t键可通过测试名称regex模式进行筛选。
- 按q键退出观看模式。
- 按Enter键触发测试运行。

## 异步代码的测试方法

### 回调

当程序进行异步操作时，测试代码会不等待回调函数执行完毕，立马执行返回成功，如：

```javascript
export const fetchData = (fn) => {
  axios.get('http://www.dell-lee.com/react/api/demo.json').then((res) => {
    fn(res.data)
  })
}
```

测试用例方法：

```javascript
import { fetchData } from './fetchData';
// 回调类型异步函数的测试
test('fetchData 结果返回为 { success: true }', (done) => {
  fetchData((data) => {
    expect(data).toEqual({
      success: true
    })
    done()
  })
})

```

这里test的第二个参数需要加上done参数，等待done执行完毕，这样可以保证测试用例执行完毕

### Promise

```javascript
export const fetchData2 = (fn) => {
  return axios.get('http://www.dell-lee.com/react/api/demo1.json')
}
```

测试用例：

请确保返回断言，可以使用then方法，如果省略此`return`语句，则测试将在`fetchData`解析解决方案返回的承诺之前完成，然后then（）有机会执行回调

```javascript
test('fetchData2 结果返回为 { success: true }', () => {
  return fetchData2().then((res) => {
    expect(res.data).toEqual({
      success: true
    })
  })
})
```

返回是被拒绝的，使用`catch`方法。确保添加`expect.assertions`以验证是否调用了一定数量的expect。否则，兑现`promise`就不会使测试失败。

```javascript
test('fetchData2 结果返回为 404', () => {
  expect.assertions(1); // expect 至少执行一次，才能通过测试 【使用catch时需使用】
  return fetchData2().catch(e => {
    expect(e.toString().indexOf('404') > -1).toBe(true)
  })
})
```

### .resolves / .rejects

您也可以在`.resolves`Expect语句中使用匹配器，Jest将等待该承诺解决。如果承诺被拒绝，则测试将自动失败。

```javascript
test('fetchData2 resolves', () => {
  return expect(fetchData2()).resolves.toMatchObject({ // toMatchObject匹配器是包含有以下内容就为测试通过
    data: {
      success: true
    }
  })
})
```

请确保返回断言-如果忽略此`return`语句，则测试将在`fetchData`解析返回的promise之前完成，然后then（）有机会执行回调

返回是被拒绝的-请使用`.rejects`匹配器。它与`.resolves`匹配器类似。如果promise得以兑现，则测试将自动失败。

```javascript
test('fetchData2 rejects', () => {
  return expect(fetchData2()).rejects.toThrow()
})
```

### Async/Await

要编写异步测试，请`async`在传递给的函数前面使用关键字`test`。例如，`fetchData`可以用以下方法测试相同的场景：

```javascript
test('fetchData2 async await', async () => {
  const response = await fetchData2();
  expect(response.data).toEqual({
    success: true
  })
})

test('fetchData2 async await', async () => {
  expect.assertions(1);
  try {
    await fetchData2();
  } catch (e) {
    expect(e.toString()).toEqual('Error: Request failed with status code 404');
  }
})
```

将`async`和`await`与`.resolves`或结合使用`.rejects`

```javascript
test('fetchData2 resolves', async () => {
  await expect(fetchData2()).resolves.toMatchObject({
    data: {
      success: true
    }
  })
})

test('fetchData2 rejects', async () => {
  await expect(fetchData2()).rejects.toThrow()
})
```

## 钩子函数

更多的钩子函数参考[官网介绍](https://jestjs.io/docs/en/api#beforeallfn-timeout)

| 钩子函数名 | 描述                           |
| ---------- | ------------------------------ |
| beforeAll  | 全局的测试用例执行前           |
| afterAll   | 全局的测试用例执行完毕         |
| beforeEach | 每次执行测试用例前             |
| afterEach  | 每次执行测试用例后             |
| describe   | 将测试用例进行分组             |
| test.only  | 对单个测试用例进行调试（常用） |

```javascript
import Counter from './Counter';
describe('Counter 的测试代码', () => {
  let counter = null
  beforeAll(() => {
    console.log('beforeAll 全局执行初始化')
  })
  beforeEach(() => {
    counter = new Counter()
    console.log('beforeEach')
  })
  afterEach(() => {
    console.log('afterEach')
  })
  afterAll(() => {
    console.log('afterAll 所有测试用例执行完毕')
  })
  describe('测试增加相关的代码', () => {
    test('测试 Counter 中的 addOne 方法', () => {
      console.log('测试 Counter 中的 addOne 方法')
      counter.addOne();
      expect(counter.number).toBe(1);
    })
    test('测试 Counter 中的 addTwo 方法', () => {
      console.log('测试 Counter 中的 addTwo 方法')
      counter.addTwo();
      expect(counter.number).toBe(2);
    })
  })
  describe('测试减少相关的代码', () => {
    test('测试 Counter 中的 minusOne 方法', () => {
      console.log('测试 Counter 中的 minusOne 方法')
      counter.minusOne();
      expect(counter.number).toBe(-1);
    })
    test('测试 Counter 中的 minusTwo 方法', () => {
      console.log('测试 Counter 中的 minusTwo 方法')
      counter.minusTwo();
      expect(counter.number).toBe(-2);
    })
  })
})
```

那么，就很容易看出 beforeEach 这些钩子，是属于describe 的（作用域），我们可以在子的 describe 中也增加 钩子，且它的生效范围为它下的所有的测试用例。【由外到内的触发机制】

## Mock

Mock函数允许您通过擦除函数的实际实现，捕获对该函数的调用（以及在这些调用中传递的参数），捕获用实例化的构造函数的实例`new`以及允许对它们进行测试时配置来测试代码之间的链接。返回值。

作用：

- 捕获函数的调用和返回结果以及this的调用顺序
- 可以自由的设置返回结果 （mockReturnValue、mockReturnValueOnce）
- 改变函数的内部实现

### 使用Mock功能

```javascript
export const runCallback = (callback) => {
  callback('abc');
}
```

测试用例：

```javascript
test.only('测试 runCallback', () => {
  const func = jest.fn(); // mock 函数 捕获函数的调用和返回结果以及this的调用顺序
  // func.mockReturnValueOnce('hello').mockReturnValueOnce('world').mockReturnValueOnce('!')
  func.mockReturnValue('hello') // 自由的设置返回结果
  runCallback(func);
  runCallback(func);
  runCallback(func);
  // expect(func).toBeCalled();
  expect(func.mock.calls.length).toBe(3)
  expect(func.mock.calls[0]).toEqual(['abc'])
  expect(func.mock).toBeCalledWith('abc') // 每次匹配的都是 'abc'
  console.log(func.mock); 
  // { calls: [ [ 'abc' ], [ 'abc' ], [ 'abc' ] ], // 测试函数的回调值
  // instances: [ undefined, undefined, undefined ], // 是否是实例化的函数
  // invocationCallOrder: [ 1, 2, 3 ], // 调用顺序
  // results: // mock的返回值
  //  [ { type: 'return', value: 'hello' },
  //    { type: 'return', value: 'hello' },
  //    { type: 'return', value: 'hello' } ] }
})
```

### .mock property

```javascript
export const createObject = (classItem) => {
  new classItem()
}
```

```javascript
test.only('测试 createObject', () => {
  const func = jest.fn();
  createObject(func);
  console.log(func.mock);
  // { calls: [ [] ],
  //   instances: [ mockConstructor {} ],
  //   invocationCallOrder: [ 1 ],
  //   results: [ { type: 'return', value: undefined } ] }
})
```

检测实例化返回的测试，可以使用以下进行检测

```javascript
// 函数被实例化了一次
expect(someMockFunction.mock.instances.length).toBe(1);
// 此函数的第一个实例化返回的对象
//具有“name”属性，其值设置为“test”
expect(func.mock.instances[0].name).toEqual('test');
```

### mock 模块

假设我们有一个从API提取用户的类。该类使用[axios](https://github.com/axios/axios)调用API，然后返回`data`包含所有用户的属性：

```javascript
export const getData = () => {
  return axios.get('/api/user.json').then(res => res.data)
}
```

现在，为了测试该方法而无需实际访问API（从而创建缓慢而脆弱的测试），我们可以使用该`jest.mock(...)`函数自动模拟axios模块。

一旦对模块进行了模拟，我们就可以提供一个`mockResolvedValue`for `.get`，以返回我们要针对测试进行断言的数据。实际上，我们说的是我们要`axios.get('/users.json')`返回假响应。

```javascript
import axios from 'axios';
jest.mock('axios'); // jest对axios进行模拟，这样不会真正发送axios请求

test.only('测试 getData', async () => {
  // 改变函数的内部实现
  // axios.get.mockResolvedValue({data: 'hello'});
  axios.get.mockResolvedValueOnce({data: 'hello'}); // 模拟返回数据
  await getData().then(data => {
    expect(data).toBe('hello');
  })
})
```

## mock 实现

在某些情况下，超越指定返回值的功能并完全替换模拟功能的实现是有用的。这可以通过模拟函数`jest.fn`或`mockImplementationOnce`方法来完成。

```javascript
test.only('测试 runCallback', () => {
  const func = jest.fn()
  // func.mockImplementation(() => { return 'hello' }) 相当于 jest.fn(() => {return 'hello'})
  // 多个函数调用产生不同的结果时用 mockImplementationOnce
  func.mockImplementationOnce(() => {
    return 'hello'
  })
  func.mockImplementationOnce(() => {
    return 'world'
  })
  runCallback(func);
  runCallback(func);
  console.log(func.mock);
  expect(func.mock.results[0].value).toBe('hello');
  expect(func.mock.results[1].value).toBe('world');
})
```

对于通常具有链式方法（因此总是需要返回`this`）的情况，我们提供了一个含糖的API，以`.mockReturnThis()`函数的形式简化了该过程，该函数也位于所有模拟中：

```javascript
const myObj = {
  myMethod: jest.fn().mockReturnThis(),
};
// is the same as

const otherObj = {
  myMethod: jest.fn(function () {
    return this;
  }),
};
```

Mock的更多方法请查看[官网](https://www.jestjs.cn/docs/mock-function-api)