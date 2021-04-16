[官方文档](https://sequelize.org/v5/index.html)

[中文文档](https://www.sequelize.com.cn/)

## 安装

Sequelize 的使用可以通过 [npm](https://www.npmjs.com/package/sequelize) (或 [yarn](https://yarnpkg.com/package/sequelize)).

```shell
npm install --save sequelize
```

## 连接数据库

```javascript
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize(
  'database', // 数据库名称
  'username', // 用户名
  'password', // 密码
  {
  	host: 'localhost', // 关系数据库的主机
    port: 3306, // 端口号
    logging: true, // 每次Sequelize执行的功能都会记录一些内容
    timezone: '+08:00', // 设置时区
  	dialect: 'mysql', /* 选择数据库方言 'mysql' | 'mariadb' | 'postgres' | 'mssql' 其一 */
    define: { // 
        timestamps: true, // 是否自动为表添加创建时间，更新时间，删除时间字段
    	paranoid: true, // 是否使用sequelize自动创建ID
    	createdAt: 'created_at', // 设置别名
    	updatedAt: 'updated_at',
    	deletedAt: 'deleted_at',
    	underscored: true, // 将驼峰命名改为下划线命名
    }
  }
);
```

Sequelize构造函数接受很多参数，他们记录在[API参考](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-constructor-constructor)中

### 测试连接

你可以使用 `.authenticate()` 函数测试连接是否正常：

```javascript
try {
  await sequelize.authenticate();
  console.log('Connection has been established successfully.');
} catch (error) {
  console.error('Unable to connect to the database:', error);
}
```

### 关闭连接

默认情况下,Sequelize 将保持连接打开状态,并对所有查询使用相同的连接. 如果你需要关闭连接,请调用 `sequelize.close()`(这是异步的并返回一个 Promise).

## 模型定义

在 Sequelize 中可以用两种等效的方式定义模型：

- 调用 [`sequelize.define(modelName, attributes, options)`](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-method-define)
- 扩展 [Model](https://sequelize.org/master/class/lib/model.js~Model.html) 并调用 [`init(attributes, options)`](https://sequelize.org/master/class/lib/model.js~Model.html#static-method-init)

[更多的DataTypes数据类型](https://sequelize.org/master/variable/index.html#static-variable-DataTypes)

### 使用 sequelize.define:

```javascript
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');
const User = sequelize.define('User', {
 // 在这里定义模型属性
 firstName: {
 type: DataTypes.STRING,
 allowNull: false
 },
 lastName: {
 type: DataTypes.STRING
 // allowNull 默认为 true
 }
}, {
 // 这是其他模型参数
});
// `sequelize.define` 会返回模型
console.log(User === sequelize.models.User); // true
```

### 扩展 Model

```javascript
const { Sequelize, DataTypes, Model } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory');
class User extends Model {}
User.init({
 // 在这里定义模型属性
 firstName: {
 type: DataTypes.STRING,
 allowNull: false
 },
 lastName: {
 type: DataTypes.STRING
 // allowNull 默认为 true
 }
}, {
 // 这是其他模型参数
 sequelize, // 我们需要传递连接实例
 modelName: 'User' // 我们需要选择模型名称
});
// 定义的模型是类本身
console.log(User === sequelize.models.User); // true
```

在内部,`sequelize.define` 调用 `Model.init`,因此两种方法本质上是等效的.

## 处理数据库事务

托管事务自动处理提交或回滚事务。您可以通过向传递回调来启动托管交易`sequelize.transaction`

tips: 传递给的回调如何`transaction`返回一个Promise链，并且没有显式调用`t.commit()`也没有`t.rollback()`。如果成功解决了返回链中的所有承诺，则提交事务。如果一个或多个承诺被拒绝，则交易将回滚。

```javascript
return sequelize.transaction(t => {

  // chain all your queries here. make sure you return them.
  // 把你所有的事务都列在这里，异常进行回滚
  return User.create({
    firstName: 'Abraham',
    lastName: 'Lincoln'
  }, {transaction: t}).then(user => {
    return user.setShooter({
      firstName: 'John',
      lastName: 'Boothe'
    }, {transaction: t});
  });

}).then(result => {
  // 事务已提交
  // result是返回到事务回调的承诺链的任何结果
}).catch(err => {
  // 事务已回滚
  // err是被拒绝的承诺链返回到事务回调
});
```

