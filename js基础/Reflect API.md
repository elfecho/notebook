> ES6 中将 Object 的一些明显属于语言内部的方法移植到了 Reflect 对象上（当前某些方法会同时存在于 Object 和 Reflect 对象上），未来的新方法会只部署在 Reflect 对象上。
>
> Reflect 对象对某些方法的返回结果进行了修改，使其更合理。
>
> Reflect 对象使用函数的方式实现了 Object 的命令式操作。

**静态方法**

## get()

```js
Reflect.get(target, name, receiver)
```

查找并返回 target 对象的 name 属性。

```js
let exam = {
    name: "Tom",
    age: 24,
    get info(){
        return this.name + this.age;
    }
}
Reflect.get(exam, 'name'); // "Tom"
 
// 当 target 对象中存在 name 属性的 getter 方法， getter 方法的 this 会绑定 // receiver
let receiver = {
    name: "Jerry",
    age: 20
}
Reflect.get(exam, 'info', receiver); // Jerry20
 
// 当 name 为不存在于 target 对象的属性时，返回 undefined
Reflect.get(exam, 'birth'); // undefined
 
// 当 target 不是对象时，会报错
Reflect.get(1, 'name'); // TypeError
```

## set()

```js
Reflect.set(target, name, value, receiver)
```

将 target 的 name 属性设置为 value。返回值为 boolean ，true 表示修改成功，false 表示失败。当 target 为不存在的对象时，会报错。

```js
let exam = {
    name: "Tom",
    age: 24,
    set info(value){
        return this.age = value;
    }
}
exam.age; // 24
Reflect.set(exam, 'age', 25); // true
exam.age; // 25
 
// value 为空时会将 name 属性清除
Reflect.set(exam, 'age', ); // true
exam.age; // undefined
 
// 当 target 对象中存在 name 属性 setter 方法时，setter 方法中的 this 会绑定 // receiver , 所以修改的实际上是 receiver 的属性,
let receiver = {
    age: 18
}
Reflect.set(exam, 'info', 1, receiver); // true
receiver.age; // 1
 
let receiver1 = {
    name: 'oppps'
}
Reflect.set(exam, 'info', 1, receiver1);
receiver1.age; // 1
```

## has()

```javascript
Reflect.has(obj, name)
```

是 name in obj 指令的函数化，用于查找 name 属性在 obj 对象中是否存在。返回值为 boolean。如果 obj 不是对象则会报错 TypeError。

```
let exam = {
    name: "Tom",
    age: 24
}
Reflect.has(exam, 'name'); // true
```

## deleteProperty()

```javascript
Reflect.deleteProperty(obj, property)
```

是 delete obj[property] 的函数化，用于删除 obj 对象的 property 属性，返回值为 boolean。如果 obj 不是对象则会报错 TypeError。

```javascript
let exam = {
    name: "Tom",
    age: 24
}
Reflect.deleteProperty(exam , 'name'); // true
exam // {age: 24} 
// property 不存在时，也会返回 true
Reflect.deleteProperty(exam , 'name'); // true
```

## construct()

```javascript
Reflect.construct(obj, args)
```

等同于 new target(…args)。

```javascript
function exam(name){
    this.name = name;
}
Reflect.construct(exam, ['Tom']); // exam {name: "Tom"}
```

## getPrototypeOf()

```js
Reflect.getPrototypeOf(obj)
```

用于读取 obj 的 _proto_ 属性。在 obj 不是对象时不会像 Object 一样把 obj 转为对象，而是会报错。

```javascript
class Exam{}
let obj = new Exam()
Reflect.getPrototypeOf(obj) === Exam.prototype // true
```

## setPrototypeOf()

```javascript
Reflect.setPrototypeOf(obj, newProto)
```

用于设置目标对象的 prototype。

```javascript
let obj ={};
Reflect.setPrototypeOf(obj, Array.prototype); // true
```

## apply()

```javascript
Reflect.apply(func, thisArg, args)
```

等同于 Function.prototype.apply.call(func, thisArg, args) 。func 表示目标函数；thisArg 表示目标函数绑定的 this 对象；args 表示目标函数调用时传入的参数列表，可以是数组或类似数组的对象。若目标函数无法调用，会抛出 TypeError 。

```javascript
Reflect.apply(Math.max, Math, [1, 3, 5, 3, 1]); // 5
```

## defineProperty()

```javascript
Reflect.getOwnPropertyDescriptor(target, propertyKey)
```

用于得到 target 对象的 propertyKey 属性的描述对象。在 target 不是对象时，会抛出错误表示参数非法，不会将非对象转换为对象。

```javascript
var exam = {}
Reflect.defineProperty(exam, 'name', {
  value: true,
  enumerable: false,
})
Reflect.getOwnPropertyDescriptor(exam, 'name')
// { configurable: false, enumerable: false, value: true, writable:
// false}
 
 
// propertyKey 属性在 target 对象中不存在时，返回 undefined
Reflect.getOwnPropertyDescriptor(exam, 'age') // undefined
```

## isExtensible()

```javascript
Reflect.isExtensible(target)
```

用于判断 target 对象是否可扩展。返回值为 boolean 。如果 target 参数不是对象，会抛出错误。

```javascript
let exam = {}Reflect.isExtensible(exam) // true
```

## preventExtensions()

```javascript
Reflect.preventExtensions(target)
```

用于让 target 对象变为不可扩展。如果 target 参数不是对象，会抛出错误。

```javascript
let exam = {}Reflect.preventExtensions(exam) // true
```

## ownKeys()

```javascript
Reflect.ownKeys(target)
```

用于返回 target 对象的所有属性，等同于 Object.getOwnPropertyNames 与Object.getOwnPropertySymbols 之和。

```javascript
var exam = {
  name: 1,
  [Symbol.for('age')]: 4
}
Reflect.ownKeys(exam) // ["name", Symbol(age)]
```

