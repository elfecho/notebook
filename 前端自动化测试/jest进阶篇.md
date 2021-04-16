## Snapshot 快照测试

> 每当您要确保配置文件或者UI不会意外更改时，快照测试都是非常有用的工具。

```javascript
export const generateConfig = () => {
  return {
    server: 'http://localhost',
    port: 8080,
    domain: 'localhost'
  }
}
```

```javascript
test("测试 generateConfig 函数", () => {
  expect(generateConfig()).toMatchSnapshot();
});
```

第一次运行此测试时，Jest创建一个快照文件，如下所示

```javascript
exports[`测试 generateConfig 函数 1`] = `
Object {
  "domain": "localhost",
  "port": 8080,
  "server": "http://localhost"
}
`;
```

### 更新快照

如果出现更新文件时，会提示

```
›按f键只运行失败的测试。
›按o键仅运行与更改文件相关的测试。
›按p按文件名regex模式过滤。
›按t按测试名称regex模式过滤。
›按u更新失败的快照。 		// 单个用例改变时，需要更新按u
›按i以交互方式更新失败的快照。 // 多个用例同时改变时，选择更新按i
›按q键退出观看模式。
›按回车键触发试运行。
```

### 动态匹配器

如果配置文件出现时间戳或者动态数字时，对于这些情况，Jest允许为任何属性提供非对称匹配器。在写入或测试快照之前，将检查以下匹配器，然后将其保存到快照文件中，而不是接收到的值：

```javascript
it('will check the matchers and pass', () => {
  const user = {
    createdAt: new Date(),
    id: Math.floor(Math.random() * 20)
  };

  expect(user).toMatchSnapshot({
    createdAt: expect.any(Date),
    id: expect.any(Number),
  });
});

// Snapshot
exports[`will check the matchers and pass 1`] = `
Object {
  "createdAt": Any<Date>,
  "id": Any<Number>,
}
`;
```

### 内联快照

内联快照的行为与外部快照（`.snap`文件）相同，只是快照值会自动写回到源代码中。这意味着您可以享受自动生成的快照的好处，而不必切换到外部文件来确保已写入正确的值。

使用内联快照前，需安装`prettier`

```bash
npm i prettier@1.18.2 -S
```

```diff
test("测试 config", () => {
  let config = {
    server: 'http://localhost',
    port: 8080,
    domain: 'localhost'
  }
  expect(config).toMatchInlineSnapshot(
+  `
+    Object {
+      "domain": "localhost",
+      "port": 8080,
+      "server": "http://localhost",
+    }
+  `
  );
});
```

里面配置是由`prettier`动态生成出来的

## mock 异步请求

我们不想在测试中进入网络，所以我们将`request.js`在`__mocks__`文件夹中为我们的模块创建一个手动模拟（该文件夹区分大小写，`__MOCKS__`将无法使用）。它可能看起来像这样：

```javascript
export const fetchData = () => {
  return new Promise((resolve, reject) => {
    resolve("(function(){return '123'})()")
  })
}
```

在我们真正的函数里面

```javascript
import axios from 'axios';

export const fetchData = () => {
  return axios.get('/user.json').then(res => res.data);
}
export const getNumber = () => {
  return 123
}
```

测试用例：

```javascript
jest.mock('./demo'); // 模拟指向到__mocks__目录下的demo.js
// jest.unmock('./demo'); // 取消mock指向

import { fetchData } from './demo';
const { getNumber } = jest.requireActual('./demo');  // 引用真正的函数

test('fetchData 测试', () => {
  return fetchData().then(data => {
    expect(eval(data)).toEqual('123');
  })
});

test('getNumber 测试', () => {
  expect(getNumber()).toEqual(123);
});
```

除了上面`jest.mock`手动模拟外，jest里面配置的`jest.config.js`,将其`automock`设置为true即可

更多[jest方法](https://www.jestjs.cn/docs/jest-object)、[mock方法](https://www.jestjs.cn/docs/mock-function-api)请查看官网

## mock 定时器

定时器功能（即`setTimeout`，`setInterval`，`clearTimeout`，`clearInterval`）小于理想的测试环境，因为它们依赖于实时流逝。jest可以用允许您控制时间流逝的功能交换计时器

```javascript
export default (callback) => {
  setTimeout(() => {
    callback()
    setTimeout(() => {
      callback()
    }, 3000)
  }, 3000)
}
```

如果您计划在所有测试中不使用假计时器，则需要手动清除，否则假计时器将在测试之间泄漏：

```javascript
import timer from './timer';
beforeEach(() => { // 放在beforeEach()的钩子函数里面进行重置内部计数器，防止计时器的事件累加
  jest.useFakeTimers(); // 通过调用启用假计时器jest.useFakeTimers()
})

afterEach(() => {
  jest.useRealTimers(); // 将计时器恢复为正常状态jest.useRealTimers()
});

afterAll(() => {
  jest.clearAllTimers(); // 删除所有待处理的计时器
})

test('timer 测试', () => {
  const fn = jest.fn();
  timer(fn);
  // jest.runAllTimers(); // 运行所有的定时器
  // jest.runOnlyPendingTimers(); // 只执行外层的一个定时器
  jest.advanceTimersByTime(3000); // 让时间快进3000毫秒
  expect(fn).toHaveBeenCalledTimes(1);
  jest.advanceTimersByTime(3000);
  expect(fn).toHaveBeenCalledTimes(2);
})
```

## mock ES6 Class

Jest可用于模拟导入到要测试的文件中的ES6类。

ES6类是带有一些语法糖的构造函数。因此，ES6类的任何模拟都必须是一个函数或实际的ES6类（再次是另一个函数）。因此，您可以使用[模拟函数](https://www.jestjs.cn/docs/mock-functions)模拟它们

新建一个util.js 工具类函数

```javascript
class Util {
  a() {
    // ... 复杂代码
  }
  b() {
    // ... 复杂代码
  }
}
export default Util;
```

```javascript
// demo.js
import Util from './util';
export default () => {
  const util = new Util();
  util.a();
  util.b();
}
```

### 自动模拟

现在要对demo.js做测试，但我们编写测试用例的时候又不想真实调用util的工具方法，这时我们需要调用mock模拟函数，将ES6类替换为模拟构造函数

```javascript
jest.mock('./util');
// Util,Util.a,Util.b jest.fn
import Util from './util';
import demoFunction from './demo';

test('测试 demo', () => {
  demoFunction()
  // console.log(Util.mock);
  expect(Util).toHaveBeenCalled();
  expect(Util.mock.instances[0].a).toHaveBeenCalled();
  expect(Util.mock.instances[0].b).toHaveBeenCalled();
})
```

### 手动模拟

通过将模拟实现保存在文件夹中来创建[手动模拟](https://www.jestjs.cn/docs/manual-mocks)`__mocks__`。使得调用的方法会使用下面改写的方法

```javascript
const Util = jest.fn(() => {
  console.log('constructor')
});
Util.prototype.a = jest.fn(() => {
  console.log('a')
});
Util.prototype.b = jest.fn(() => {
  console.log('b')
});
export default Util;
```

## mock DOM

通常认为难以测试的另一类功能是直接操作DOM的代码。让我们看看如何测试下面的jQuery代码片段

```javascript
import $ from 'jquery';
const addDivToBody = () => {
  $('body').append('<div />');
}
export default addDivToBody;
```

```javascript
import addDivToBody from './demo';
import $ from 'jquery';
test('addDivToBody 测试', () => {
  addDivToBody();
  addDivToBody();
  expect($('body').find('div').length).toBe(2);
})
```

被测试的函数在body元素上添加div元素，因此我们需要为测试正确设置DOM。Jest附带了一个`jsdom`可模拟DOM环境的工具，就像在浏览器中一样。这意味着我们调用的每个DOM API的观察方式都可以与浏览器中观察到的方式相同！