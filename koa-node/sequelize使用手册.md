前些年，本站整理过Sequelize相关中文文档，其时，Sequelize的版本为`v3.*`。现在Sequelize版本已更新到`v5.19.6`(本文发布之日)，Sequelize的功能和API已有较大规模的更新，所以基于`v5.*`再进行一次梳理，以了解新功能及方便日后使用。

1. [概述](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#summary)
2. [快速入门(Getting started)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#getting-started)
3. [方言(Dialects)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#dialects)
4. [数据类型(Datatypes)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#data-types)
5. [模型定义(Model definition)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#models-definition)
6. [模型使用(Model usage)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#models-usage)
7. [钩子(Hooks)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#hooks)
8. [查询(Querying)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying)
9. [实例(Instances)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#instances)
10. [关联\关系(Associations)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#associations)
11. [原始查询(Raw queries)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#raw-queries)
12. [事务(Transactions)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#transactions)
13. [作用域(Scopes)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#scopes)
14. [主从复制(Read replication)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#read-replication)
15. [迁移(Migrations)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#migrations)
16. [相关资源](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#resources)
17. [TypeScript](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#typescript)
18. [升级到V5](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#upgrade-to-v5)
19. [使用遗留表](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#legacy)

## 1. 概述

Sequelize是一个用于Postgres、MySQL、MariaDB、SQLite和Microsoft SQL Server的基于Promise的Node.js ORM框架。它具有可靠的事务支持、关系管理、预加载和延迟加载、主从复制(Read replication)等功能。

Sequelize遵循[SEMVER](https://semver.org/)（Semantic Versioning-语义化版本规范）。支持Node v6及更高版本，以使用ES6相关功能。

**Sequelize v5**发布于2019年3月13，现已包含官方的[TypeScript类型](https://sequelize.org/master/manual/typescript.html)。

本文可理解为Sequelize的**教程和指南**，在使用过程中，你可能还会需要[API参考](https://sequelize.org/master/identifiers.html)。

### 简单示例

```
const { Sequelize, Model, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

class User extends Model {}
User.init({
  username: DataTypes.STRING,
  birthday: DataTypes.DATE
}, { sequelize, modelName: 'user' });

sequelize.sync()
  .then(() => User.create({
    username: 'janedoe',
    birthday: new Date(1980, 6, 20)
  }))
  .then(jane => {
    console.log(jane.toJSON());
  });
```



## 2. 快速入门(Getting started)

在本章中，将介绍Sequelize的简单设置，以学习Sequelize的基础知识。

- [安装](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#installing)
- 建立连接
  - [设置SQLite](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#note--setting-up-sqlite)
  - [连接池（生产）](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#note--connection-pool--production-)
  - [测试连接](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#testing-the-connection)
  - [关闭](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#closing-the-connection)
- 对表建模
  - [修改默认模型选项](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#changing-the-default-model-options)
- 模型与数据库同步
  - [一次同步所有模型](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#synchronizing-all-models-at-once)
  - [生产注意事项](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#note-for-production)
- [查询](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#getting-started-querying)
- [`Promise`与`async/await`](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#promises-and-async-await)

### 安装

Sequelize可以通过[`npm`](https://www.npmjs.com/package/sequelize)(或[`yarn`](https://yarnpkg.com/package/sequelize))安装：

```
npm install --save sequelize
```

除安装`sequelize`模块外，还需要手工安装你所使用的数据库驱动模块：

```
# One of the following:
$ npm install --save pg pg-hstore # Postgres
$ npm install --save mysql2
$ npm install --save mariadb
$ npm install --save sqlite3
$ npm install --save tedious # Microsoft SQL Server
```



### 建立连接

要连接到数据库，必须创建一个Sequelize实例。这可以通过将连接参数分别传递到Sequelize构造函数，或通过传递单个连接URI来完成：

```
const Sequelize = require('sequelize');

// 选项1：分别传入参数
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: /* 'mysql' | 'mariadb' | 'postgres' | 'mssql' 之一 */
});

// 选项1：传入连接URI
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');
```

更多关于Sequelize构造函数所支持的参数，请参考：[Sequelize构造函数API](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-constructor-constructor)。



#### 注意：设置SQLite

如果你使用SQLite，则应使用如下方式：

```
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: 'path/to/database.sqlite'
});
```



#### 注意：连接池（生产）

如果你在单进程中连接到数据库，则应仅创建一个Sequelize实例。Sequelize会在初始化时建立连接池。可以通过构造函数的`options`参数（使用`options.pool`）配置此连接池，如下所示：

```
const sequelize = new Sequelize(/* ... */, {
  // ...
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
});
```

更多详细介绍，可以参考：[Sequelize构造函数API](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-constructor-constructor)。

如果你在多进程中连接数据库，那么应该为每个进程创建一个实例，但每个进程的连接池应有一个合适的大小，以确保符合最大连接总数。例如：你希望最大连接池大小为`90`，并且有三个进程，则每个进程的Sequelize实例的最大连接池大小应为`30`。



#### 测试连接

可以使用`.authenticate()`函数测试连接是否正常：

```
sequelize
  .authenticate()
  .then(() => {
    console.log('Connection has been established successfully.');
  })
  .catch(err => {
    console.error('Unable to connect to the database:', err);
  });
```



#### 关闭

Sequelize默认保持打开连接，你可以在多个查询中使用相关的连接。如果需要关闭连接，可以调用`sequelize.close()`。



### 对表建模

模型是对`Sequelize.Model`类的扩展。模型可以通过两种方式定义。首先，可以通过`Sequelize.Model.init(attributes, options)`：

```
const Model = Sequelize.Model;
class User extends Model {}
User.init({
  // attributes
  firstName: {
    type: Sequelize.STRING,
    allowNull: false
  },
  lastName: {
    type: Sequelize.STRING
    // allowNull defaults to true
  }
}, {
  sequelize,
  modelName: 'user'
  // options
});
```

或者，使用`sequelize.define`：

```
const User = sequelize.define('user', {
  // attributes
  firstName: {
    type: Sequelize.STRING,
    allowNull: false
  },
  lastName: {
    type: Sequelize.STRING
    // allowNull defaults to true
  }
}, {
  // options
});
```

在内部，`sequelize.define`会调用`Model.init`。

在上面代码中，会告诉Sequelize期望数据库中有名为`users`的表，其包含`firstName`和`lastName`字段。默认情况下，表名会自动使用复数形式（通过[inflection](https://www.npmjs.com/package/inflection)库来实现）。可以使用`freezeTableName: true`选项来禁用这一特性，可以以使用[Sequelize构造函数](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-constructor-constructor)中的`define`选项来禁用所有模型的这一形为。

默认情况下，Sequelize 还会为每个模型定义`id`（主键）、`createdAt`和`updatedAt`字段。当然，也可以更改此行为，请参考[模型定义章节](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#/models-definition)或[模型配置API](https://sequelize.org/master/manual/models-definition.html#configuration)以了解更多信息。



#### 修改默认模型选项

Sequelize构造函数有一个`define`选项，其会修改所有模型的定义模型的默认选项：

```
const sequelize = new Sequelize(connectionURI, {
  define: {
    // The `timestamps` field specify whether or not the `createdAt` and `updatedAt` fields will be created.
    // This was true by default, but now is false by default
    timestamps: false
  }
});

// Here `timestamps` will be false, so the `createdAt` and `updatedAt` fields will not be created.
class Foo extends Model {}
Foo.init({ /* ... */ }, { sequelize });

// Here `timestamps` is directly set to true, so the `createdAt` and `updatedAt` fields will be created.
class Bar extends Model {}
Bar.init({ /* ... */ }, { sequelize, timestamps: true });
```

更多关于创建模型的介绍，可以参考：[`Model.init`API](https://sequelize.org/master/class/lib/model.js~Model.html#static-method-init)或[`sequelize.define`API](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-method-define)。



### 模型与数据库同步

如果你想 Sequelize 通过定义的模型自动创建表（或修改已存在的表），可以使用`sync`方法，如下所示：

```
// 注意: `force: true` 选项会在表存在时首先删除表
User.sync({ force: true }).then(() => {
  // 现在 `users` 表会与模型定义一致
  return User.create({
    firstName: 'John',
    lastName: 'Hancock'
  });
});
```



#### 一次同步所有模型

可以使用`sequelize.sync()`方法来同步所有模型，而不是调用每个模型的`sync()`方法。



#### 生产注意事项

在生产环境中，你应该考虑使用“Migration”来替代调用`sync()`。了解更多，请参考[迁移(Migrations)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#migrations)章节。



### 查询

以下是一些简单的查询：

```
// 查询所有 users
User.findAll().then(users => {
  console.log("All users:", JSON.stringify(users, null, 4));
});

// 创建一个新 user
User.create({ firstName: "Jane", lastName: "Doe" }).then(jane => {
  console.log("Jane's auto-generated ID:", jane.id);
});

// 删除每个名为 "Jane" 的记录
User.destroy({
  where: {
    firstName: "Jane"
  }
}).then(() => {
  console.log("Done");
});

// 修改每个`lastName`为`null`的记录修改为"Doe"
User.update({ lastName: "Doe" }, {
  where: {
    lastName: null
  }
}).then(() => {
  console.log("Done");
});
```

Sequelize有很多查询选项，可以通过[查询](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying)章节了解更多。如果确实需要原始 SQL 查询，也可以进行[原始查询](https://itbilu.com/nodejs/npm/raw-queries)。



### `Promise`与`async/await`

在前面示例中，广泛使用了`.then`，也就是说Sequelize使用`Promise`。这意味着，如果你的Node版本支持，就可以通过Sequelize使用ES2017的`async/await`语法进行异步调用。

另外，所有Sequelize`Promise`实际上都是[Bluebird](http://bluebirdjs.com/)`Promise`，所以你也可以使用丰富的Bluebird API（如：`final`、`tap`、`tapCatch`、`map`、`mapSeries`等）。如果要设置任何特定于Bluebird的选项，则可以使用`Sequelize.Promise`访问Sequelize内部使用的Bluebird构造函数



## 3. 方言(Dialects)

Sequelize独立于特定的方言（数据库），这意味着你必须自己将相应的数据库连接器安装到项目中。

- [MySQL](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#dialects-mysql)
- [MariaDB](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#dialects-mariadb)
- [SQLite](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#dialects-sqlite)
- [PostgreSQL](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#dialects-postgresql)
- [MSSQL](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#dialects-mssql)

### MySQL

为了使Sequelize与MySQL配合良好，需要安装`mysql2@^1.5.2`或更高版本。然后，可以像这样使用它：

```
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mysql'
})
```

**注意：**可以通过设置`dialectOptions`参数将选项直接传递到方言库。



### MariaDB

MariaDB所使用的库为`mariadb`：

```
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mariadb',
  dialectOptions: {connectTimeout: 1000} // mariadb connector option
})
```

或使用连接字符串：

```
const sequelize = new Sequelize('mariadb://user:password@example.com:9821/database')
```



### SQLite

为了与SQLite兼容，需要`sqlite3@^4.0.0`。并像这样配置Sequelize：

```
const sequelize = new Sequelize('database', 'username', 'password', {
  // sqlite! now!
  dialect: 'sqlite',

  // the storage engine for sqlite
  // - default ':memory:'
  storage: 'path/to/database.sqlite'
})
```

或者将路径做为连接字符串传入：

```
const sequelize = new Sequelize('sqlite:/home/abs/path/dbname.db')
const sequelize = new Sequelize('sqlite:relativePath/dbname.db')
```



### PostgreSQL

对于PostgreSQL，需要两个库：`pg@^7.0.0`和`pg-hstore`。然后像下面这样配置即可：

```
const sequelize = new Sequelize('database', 'username', 'password', {
  // gimme postgres, please!
  dialect: 'postgres'
})
```

要通过Unix socket连接，请在`host`选项中指定套接字目录的路径。

socket路径必须以`/`开头：

```
const sequelize = new Sequelize('database', 'username', 'password', {
  // gimme postgres, please!
  dialect: 'postgres',
  host: '/path/to/socket_directory'
})
```



### MSSQL

MSSQL所使用的库为`tedious@^6.0.0`，然后再配置方言即可。需要注意：`tedious@^6.0.0`要求将MSSQL的特定选项嵌套在`dialectOptions`对象内的`options`对象内。

```
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mssql',
  dialectOptions: {
    options: {
      useUTC: false,
      dateFirst: 1,
    }
  }
})
```



## 4. 数据类型(Datatypes)

- 数据类型
  - [Array(ENUM)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#array-enum-)
  - Range 类型
    - [特殊案例](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#special-cases)
- [扩展数据类型](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#extending-datatypes)
- PostgreSQL
  - [范围](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#ranges)

### 数据类型

以下是一些Sequelize所支持的类型。全部及更新列表，参见：[Datatyes](https://sequelize.org/master/variable/index.html#static-variable-DataTypes)。

```
Sequelize.STRING                      // VARCHAR(255)
Sequelize.STRING(1234)                // VARCHAR(1234)
Sequelize.STRING.BINARY               // VARCHAR BINARY
Sequelize.TEXT                        // TEXT
Sequelize.TEXT('tiny')                // TINYTEXT
Sequelize.CITEXT                      // CITEXT      PostgreSQL and SQLite only.

Sequelize.INTEGER                     // INTEGER
Sequelize.BIGINT                      // BIGINT
Sequelize.BIGINT(11)                  // BIGINT(11)

Sequelize.FLOAT                       // FLOAT
Sequelize.FLOAT(11)                   // FLOAT(11)
Sequelize.FLOAT(11, 10)               // FLOAT(11,10)

Sequelize.REAL                        // REAL        PostgreSQL only.
Sequelize.REAL(11)                    // REAL(11)    PostgreSQL only.
Sequelize.REAL(11, 12)                // REAL(11,12) PostgreSQL only.

Sequelize.DOUBLE                      // DOUBLE
Sequelize.DOUBLE(11)                  // DOUBLE(11)
Sequelize.DOUBLE(11, 10)              // DOUBLE(11,10)

Sequelize.DECIMAL                     // DECIMAL
Sequelize.DECIMAL(10, 2)              // DECIMAL(10,2)

Sequelize.DATE                        // DATETIME for mysql / sqlite, TIMESTAMP WITH TIME ZONE for postgres
Sequelize.DATE(6)                     // DATETIME(6) for mysql 5.6.4+. Fractional seconds support with up to 6 digits of precision
Sequelize.DATEONLY                    // DATE without time.
Sequelize.BOOLEAN                     // TINYINT(1)

Sequelize.ENUM('value 1', 'value 2')  // An ENUM with allowed values 'value 1' and 'value 2'
Sequelize.ARRAY(Sequelize.TEXT)       // Defines an array. PostgreSQL only.
Sequelize.ARRAY(Sequelize.ENUM)       // Defines an array of ENUM. PostgreSQL only.

Sequelize.JSON                        // JSON column. PostgreSQL, SQLite and MySQL only.
Sequelize.JSONB                       // JSONB column. PostgreSQL only.

Sequelize.BLOB                        // BLOB (bytea for PostgreSQL)
Sequelize.BLOB('tiny')                // TINYBLOB (bytea for PostgreSQL. Other options are medium and long)

Sequelize.UUID                        // UUID datatype for PostgreSQL and SQLite, CHAR(36) BINARY for MySQL (use defaultValue: Sequelize.UUIDV1 or Sequelize.UUIDV4 to make sequelize generate the ids automatically)

Sequelize.CIDR                        // CIDR datatype for PostgreSQL
Sequelize.INET                        // INET datatype for PostgreSQL
Sequelize.MACADDR                     // MACADDR datatype for PostgreSQL

Sequelize.RANGE(Sequelize.INTEGER)    // Defines int4range range. PostgreSQL only.
Sequelize.RANGE(Sequelize.BIGINT)     // Defined int8range range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DATE)       // Defines tstzrange range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DATEONLY)   // Defines daterange range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DECIMAL)    // Defines numrange range. PostgreSQL only.

Sequelize.ARRAY(Sequelize.RANGE(Sequelize.DATE)) // Defines array of tstzrange ranges. PostgreSQL only.

Sequelize.GEOMETRY                    // Spatial column.  PostgreSQL (with PostGIS) or MySQL only.
Sequelize.GEOMETRY('POINT')           // Spatial column with geometry type. PostgreSQL (with PostGIS) or MySQL only.
Sequelize.GEOMETRY('POINT', 4326)     // Spatial column with geometry type and SRID.  PostgreSQL (with PostGIS) or MySQL only.
```

`BLOB`允许你插入`string`或`buffer`类型数据。当你对模型的`BLOB`列使用`find`或`findAll`方法查询时，总是返回`buffer`数据。

如果你使用PostgreSQL的`TIMESTAMP WITHOUT TIME ZONE`，并需要进行时区转换，请使用`pg`本身的转换器：

```
require('pg').types.setTypeParser(1114, stringValue => {
  return new Date(stringValue + '+0000');
  // e.g., UTC offset. Use any offset that you would like.
});
```

除了上述类型外，`integer`、`bigint`、`float`和`double`还支持`unsigned`和`zerofill`属性，可以按任意顺序组合它们（请注意，这不适用于PostgreSQL！）。

```
Sequelize.INTEGER.UNSIGNED              // INTEGER UNSIGNED
Sequelize.INTEGER(11).UNSIGNED          // INTEGER(11) UNSIGNED
Sequelize.INTEGER(11).ZEROFILL          // INTEGER(11) ZEROFILL
Sequelize.INTEGER(11).ZEROFILL.UNSIGNED // INTEGER(11) UNSIGNED ZEROFILL
Sequelize.INTEGER(11).UNSIGNED.ZEROFILL // INTEGER(11) UNSIGNED ZEROFILL
```

*以上仅示例了`integer`类型，但同样也适用于`bigint`和`float`。*

在对象表示法中的用法：

```
// for enums:
class MyModel extends Model {}
MyModel.init({
  states: {
    type: Sequelize.ENUM,
    values: ['active', 'pending', 'deleted']
  }
}, { sequelize })
```



#### `Array(ENUM)`

*仅PostgreSQL适用。*

`Array(ENUM)`类型需要特殊处理。每次Sequelize与数据库进行会话时，都必须使用`ENUM`名称对`Array`值进行类型转换。

因此，此枚举名称必须遵循这一模式`enum_<table_name>_<col_name>`。如果使用`sync`同步，则将自动生成正确的名称。



#### `Range`类型

由于范围类型有包含/排除这样额外信息的绑定，因此仅使用元组在JavaScript中表示它们并不是很简单。

当提供范围值时，可以从以下API中进行选择：

```
// defaults to '["2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00")'
// inclusive lower bound, exclusive upper bound
Timeline.create({ range: [new Date(Date.UTC(2016, 0, 1)), new Date(Date.UTC(2016, 1, 1))] });

// control inclusion
const range = [
  { value: new Date(Date.UTC(2016, 0, 1)), inclusive: false },
  { value: new Date(Date.UTC(2016, 1, 1)), inclusive: true },
];
// '("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00"]'

// composite form
const range = [
  { value: new Date(Date.UTC(2016, 0, 1)), inclusive: false },
  new Date(Date.UTC(2016, 1, 1)),
];
// '("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00")'

Timeline.create({ range });
```

但请注意，只要你返回范围内的值，就会收到：

```
// stored value: ("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00"]
range // [{ value: Date, inclusive: false }, { value: Date, inclusive: true }]
```

在使用范围类型更新实例或使用`returning: true`选项后，你需要调用`reload`。

##### 特殊案例

```
// empty range:
Timeline.create({ range: [] }); // range = 'empty'

// Unbounded range:
Timeline.create({ range: [null, null] }); // range = '[,)'
// range = '[,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [null, new Date(Date.UTC(2016, 0, 1))] });

// Infinite range:
// range = '[-infinity,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [-Infinity, new Date(Date.UTC(2016, 0, 1))] });
```



### 扩展数据类型

大多数数据类型都已包含在了[Datatyes](https://sequelize.org/master/variable/index.html#static-variable-DataTypes)中。如果不包括新的数据类型，可按参考本节说明自己编写。

Sequelize不会在数据库中创建新的数据类型。本教程说明了如何使Sequelize识别新数据类型，并假定这些新数据类型已在数据库中创建。

要扩展Sequelize数据类型，需要在创建任何实例之前进行。以下示例创建了一个虚拟`NEWTYPE`，该`NEWTYPE`会复制内置数据类型`Sequelize.INTEGER(11).ZEROFILL.UNSIGNED`。

```
// myproject/lib/sequelize-additions.js

module.exports = function sequelizeAdditions(Sequelize) {

  DataTypes = Sequelize.DataTypes

  /*
   * 创建新类型
   */
  class NEWTYPE extends DataTypes.ABSTRACT {
    // Mandatory, complete definition of the new type in the database
    toSql() {
      return 'INTEGER(11) UNSIGNED ZEROFILL'
    }

    // 可选，验证器函数
    validate(value, options) {
      return (typeof value === 'number') && (! Number.isNaN(value))
    }

    // 可选, 检查器
    _sanitize(value) {
      // Force all numbers to be positive
      if (value < 0) {
        value = 0
      }

      return Math.round(value)
    }

    // 可选，在发送到数据库前对值字符串化
    _stringify(value) {
      return value.toString()
    }

    // 可选，从数据库获取值后进行解析
    static parse(value) {
      return Number.parseInt(value)
    }
  }

  DataTypes.NEWTYPE = NEWTYPE;

  // 必须，设定键
  DataTypes.NEWTYPE.prototype.key = DataTypes.NEWTYPE.key = 'NEWTYPE'

  // 可选，禁用在字符串化后转义。不建议。
  // Warning: disables Sequelize protection against SQL injections
  // DataTypes.NEWTYPE.escape = false

  // For convenience
  // `classToInvokable` allows you to use the datatype without `new`
  Sequelize.NEWTYPE = Sequelize.Utils.classToInvokable(DataTypes.NEWTYPE)

}
```

创建新数据类型后，还需要在每个数据库方言中映射此数据类型并进行一些调整。



### PostgreSQL

假设新数据类型的名称在postgres数据库中为`pg_new_type`。该名称必须映射到`DataTypes.NEWTYPE`。此外，还需要创建特定于Postgres的子数据类型。

```
Let's say the name of the new datatype is pg_new_type in the postgres database. That name has to be mapped to DataTypes.NEWTYPE. Additionally, it is required to create a child postgres-specific datatype.

// myproject/lib/sequelize-additions.js

module.exports = function sequelizeAdditions(Sequelize) {

  DataTypes = Sequelize.DataTypes

  /*
   * Create new types
   */

  ...

  /*
   * Map new types
   */

  // Mandatory, map postgres datatype name
  DataTypes.NEWTYPE.types.postgres = ['pg_new_type']

  // Mandatory, create a postgres-specific child datatype with its own parse
  // method. The parser will be dynamically mapped to the OID of pg_new_type.
  PgTypes = DataTypes.postgres

  PgTypes.NEWTYPE = function NEWTYPE() {
    if (!(this instanceof PgTypes.NEWTYPE)) return new PgTypes.NEWTYPE();
    DataTypes.NEWTYPE.apply(this, arguments);
  }
  inherits(PgTypes.NEWTYPE, DataTypes.NEWTYPE);

  // Mandatory, create, override or reassign a postgres-specific parser
  //PgTypes.NEWTYPE.parse = value => value;
  PgTypes.NEWTYPE.parse = DataTypes.NEWTYPE.parse;

  // Optional, add or override methods of the postgres-specific datatype
  // like toSql, escape, validate, _stringify, _sanitize...

}
```



#### `Range`

在[postgres中定义](https://www.postgresql.org/docs/current/static/rangetypes.html#RANGETYPES-DEFINING)了新的范围类型后，将其添加到Sequelize变得很简单。

在本示例中，postgres范围类型的名称为`newtype_range`，而基础postgres数据类型的名称为`pg_new_type`。 `subtypes`和`castTypes`的键是Sequelize数据类型`DataTypes.NEWTYPE.key`的键（小写）。

```
// myproject/lib/sequelize-additions.js

module.exports = function sequelizeAdditions(Sequelize) {

  DataTypes = Sequelize.DataTypes

  /*
   * Create new types
   */

  ...

  /*
   * Map new types
   */

  ...

  /*
   * Add suport for ranges
   */

  // Add postgresql range, newtype comes from DataType.NEWTYPE.key in lower case
  DataTypes.RANGE.types.postgres.subtypes.newtype = 'newtype_range';
  DataTypes.RANGE.types.postgres.castTypes.newtype = 'pg_new_type';

}
```

新的范围类型可以在模型定义中以`Sequelize.RANGE(Sequelize.NEWTYPE)`或`DataTypes.RANGE(DataTypes.NEWTYPE)`的方式使用。



## 5. 模型定义(Model definition)

- [时间戳](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#timestamps)
- [可延时（`Deferrable`）](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#deferrable)
- Getters & setters
  - [定义为属性的一部分](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#defining-as-part-of-a-property)
  - [定义为模型选项的一部分](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#defining-as-part-of-the-model-options)
  - [在Getter和Setter定义中使用的辅助函数](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#helper-functions-for-use-inside-getter-and-setter-definitions)
- 验证
  - [属性验证器](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#per-attribute-validations)
  - [属性验证器与`allowNull`](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#per-attribute-validators-and--code-allownull--code-)
  - [模型范围内的验证](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#model-wide-validations)
- [配置](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#configuration)
- [导入](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#import)
- [乐观锁](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#optimistic-locking)
- [数据库同步](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#database-synchronization)
- 扩展模型
  - [索引](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#indexes)

定义模型和表之间的映射时，使用`define`方法。表中每一列必须一个数据类型，详细参考[数据类型](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#data-types)。

```
class Project extends Model {}
Project.init({
  title: Sequelize.STRING,
  description: Sequelize.TEXT
}, { sequelize, modelName: 'project' });

class Task extends Model {}
Task.init({
  title: Sequelize.STRING,
  description: Sequelize.TEXT,
  deadline: Sequelize.DATE
}, { sequelize, modelName: 'task' })
```

除了数据类型，还可以在每列上设置很多选项。

```
class Foo extends Model {}
Foo.init({
 // 实例化将自动将`flag`设置为`true`（如果未设置）
 flag: { type: Sequelize.BOOLEAN, allowNull: false, defaultValue: true },

 // 日期的默认什 => 当前时间
 myDate: { type: Sequelize.DATE, defaultValue: Sequelize.NOW },

 // 设置 `allowNull` 为 `false` 时，会将 NOT NULL 添加对应列，
 // 这意味着，如果查询为`null`将从数据库抛出错误
 // 如果你想在查询前做非空较验，请参考“验证”章节
 title: { type: Sequelize.STRING, allowNull: false },

 // 使用相同值创建两个对象时将抛出错误
 // `unique` 属性可以是布尔或字符串
 // 如果对多列使用相同的字符串，它们会形成一个复合唯一键
 uniqueOne: { type: Sequelize.STRING,  unique: 'compositeIndex' },
 uniqueTwo: { type: Sequelize.INTEGER, unique: 'compositeIndex' },

 // `unique` 属性只是创建唯一约束的简写
 someUnique: { type: Sequelize.STRING, unique: true },

 // 与在模型选项中创建索引完全相同
 { someUnique: { type: Sequelize.STRING } },
 { indexes: [ { unique: true, fields: [ 'someUnique' ] } ] },

 // 主键设置，后面会继续介绍
 identifier: { type: Sequelize.STRING, primaryKey: true },

 // `autoIncrement` 会对整型列使用 `auto_incrementing` 
 incrementMe: { type: Sequelize.INTEGER, autoIncrement: true },

 // 你可以通过 'field' 属性自定义列名
 fieldWithUnderscores: { type: Sequelize.STRING, field: 'field_with_underscores' },

 // 可以创建外键
 bar_id: {
   type: Sequelize.INTEGER,

   references: {
     // 这是对另一个模型模型的引用: Bar,

     // 这是引用的模型键的列名: 'id',

     // 这声明何时检查外键约束。仅 PostgreSQL.
     deferrable: Sequelize.Deferrable.INITIALLY_IMMEDIATE
   }
 },

 // 可以为列添加注释。仅 MySQL、PostgreSQL 和 MSSQL
 commentMe: {
   type: Sequelize.INTEGER,

   comment: 'This is a column name that has a comment'
 }
}, {
  sequelize,
  modelName: 'foo'
});
```

其中，`comment`选项也可以在表上使用，参考：[模型配置](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#configuration)。



### 时间戳

默认情况下，Sequelize 会自动为模型添加`createdAt`和`updatedAt`，以记录数据何时进入数据库，以及数据最后更新时间。

请注意，如果使用的是Sequelize迁移，则需要将`createdAt`和`updatedAt`字段添加到迁移定义中：

```
module.exports = {
  up(queryInterface, Sequelize) {
    return queryInterface.createTable('my-table', {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
      },

      // Timestamps
      createdAt: Sequelize.DATE,
      updatedAt: Sequelize.DATE,
    })
  },
  down(queryInterface, Sequelize) {
    return queryInterface.dropTable('my-table');
  },
}
```

如果不希望在模型上使用时间戳，或只需要一些时间戳，或者正在使用数据库现有的其它列，请直接进入[配置](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#configuration)以查看操作。



### 可延时（`Deferrable`）

指定外键列时，可以选择在PostgreSQL中声明`deferrable`类型。可使用以下选项：

```
// 将所有外键约束检查推迟到事务结束
Sequelize.Deferrable.INITIALLY_DEFERRED

// 立即检查外键约束
Sequelize.Deferrable.INITIALLY_IMMEDIATE

// 不使用延时检查（默认）
Sequelize.Deferrable.NOT
```

以上最后一项在PostgreSQL中默认使用，并且不允许动态更改事务中的规则。更多相关信息，请参见：[事务章节](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#transactions)。



### Getters & setters

可以通过“对象-属性”的方式定义`getter`和`setter`函数，它们既可以用于“保护”映射到数据库字段的属性，也可以用于定义“伪”属性。

Getters & setters可以通过两种方式定义(也可以混合使用)：

- 做为单一属性的一部分
- 做为模型选项的一部分

如果同时使用两种定义方式，那么将始终优先使用相关属性定义中的函数。



#### 定义为属性的一部分

```
class Employee extends Model {}
Employee.init({
  name: {
    type: Sequelize.STRING,
    allowNull: false,
    get() {
      const title = this.getDataValue('title');
      // 'this' allows you to access attributes of the instance
      return this.getDataValue('name') + ' (' + title + ')';
    },
  },
  title: {
    type: Sequelize.STRING,
    allowNull: false,
    set(val) {
      this.setDataValue('title', val.toUpperCase());
    }
  }
}, { sequelize, modelName: 'employee' });

Employee
  .create({ name: 'John Doe', title: 'senior engineer' })
  .then(employee => {
    console.log(employee.get('name')); // John Doe (SENIOR ENGINEER)
    console.log(employee.get('title')); // SENIOR ENGINEER
  })
```



#### 定义为模型选项的一部分

以下是一个做为模型选项定义`getter`和`setter`的示例。

`fullName` getter在示例中被定义为伪属性，这些属性实际上不是数据库架构(schema)的一部分。实际可以通过两种方式定义伪属性：使用模型getter，或使用有`VIRTUAL`数据类型的列。虚拟数据类型可以进行验证，而虚拟属性的获取程序则不能。

请注意，`fullName` getter函数中`this.firstname`和`this.lastname`引用将触发对相应getter函数的调用。如果不想这样，请使用`getDataValue()`方法访问原始值（请参见下文）。

```
class Foo extends Model {
  get fullName() {
    return this.firstname + ' ' + this.lastname;
  }

  set fullName(value) {
    const names = value.split(' ');
    this.setDataValue('firstname', names.slice(0, -1).join(' '));
    this.setDataValue('lastname', names.slice(-1).join(' '));
  }
}
Foo.init({
  firstname: Sequelize.STRING,
  lastname: Sequelize.STRING
}, {
  sequelize,
  modelName: 'foo'
});

// Or with `sequelize.define`
sequelize.define('Foo', {
  firstname: Sequelize.STRING,
  lastname: Sequelize.STRING
}, {
  getterMethods: {
    fullName() {
      return this.firstname + ' ' + this.lastname;
    }
  },

  setterMethods: {
    fullName(value) {
      const names = value.split(' ');

      this.setDataValue('firstname', names.slice(0, -1).join(' '));
      this.setDataValue('lastname', names.slice(-1).join(' '));
    }
  }
});
```



#### 在Getter和Setter定义中使用的辅助函数

- 获取基础属性值-始终使用：`this.getDataValue()`

```
/* a getter for 'title' property */
get() {
  return this.getDataValue('title')
}
```

- 设置基础属性值-始终使用：`this.setDataValue()`

```
/* a setter for 'title' property */
set(title) {
  this.setDataValue('title', title.toString().toLowerCase());
}
```

**注意：**一定要使用`getDataValue()`和`setDataValue()`函数（而不是直接访问基础的“数据值”属性），这样可以保护自定义getter和setter免受底层模型实现的更改。



### 验证

模型验证允许你为每个模型属性指定`format`/`content`/`inheritance`验证。

验证会在`create`、`update`、`save`时自动调用。可以手工调用`validate()`对实例进行验证。



#### 属性验证器

你可以自定义属性验证器，也可以使用由[validator.js](https://github.com/chriso/validator.js)实现的内置验证器。如下所示：

```
class ValidateMe extends Model {}
ValidateMe.init({
  bar: {
    type: Sequelize.STRING,
    validate: {
      is: ["^[a-z]+$",'i'],     // will only allow letters
      is: /^[a-z]+$/i,          // same as the previous example using real RegExp
      not: ["[a-z]",'i'],       // will not allow letters
      isEmail: true,            // checks for email format (foo@bar.com)
      isUrl: true,              // checks for url format (http://foo.com)
      isIP: true,               // checks for IPv4 (129.89.23.1) or IPv6 format
      isIPv4: true,             // checks for IPv4 (129.89.23.1)
      isIPv6: true,             // checks for IPv6 format
      isAlpha: true,            // will only allow letters
      isAlphanumeric: true,     // will only allow alphanumeric characters, so "_abc" will fail
      isNumeric: true,          // will only allow numbers
      isInt: true,              // checks for valid integers
      isFloat: true,            // checks for valid floating point numbers
      isDecimal: true,          // checks for any numbers
      isLowercase: true,        // checks for lowercase
      isUppercase: true,        // checks for uppercase
      notNull: true,            // won't allow null
      isNull: true,             // only allows null
      notEmpty: true,           // don't allow empty strings
      equals: 'specific value', // only allow a specific value
      contains: 'foo',          // force specific substrings
      notIn: [['foo', 'bar']],  // check the value is not one of these
      isIn: [['foo', 'bar']],   // check the value is one of these
      notContains: 'bar',       // don't allow specific substrings
      len: [2,10],              // only allow values with length between 2 and 10
      isUUID: 4,                // only allow uuids
      isDate: true,             // only allow date strings
      isAfter: "2011-11-05",    // only allow date strings after a specific date
      isBefore: "2011-11-05",   // only allow date strings before a specific date
      max: 23,                  // only allow values <= 23
      min: 23,                  // only allow values >= 23
      isCreditCard: true,       // check for valid credit card numbers

      // Examples of custom validators:
      isEven(value) {
        if (parseInt(value) % 2 !== 0) {
          throw new Error('Only even values are allowed!');
        }
      }
      isGreaterThanOtherField(value) {
        if (parseInt(value) <= parseInt(this.otherField)) {
          throw new Error('Bar must be greater than otherField.');
        }
      }
    }
  }
}, { sequelize });
```

请注意，需要将多个参数传递给内置验证功能时，要传递的参数必须位于数组中。 但是，如果要传递单个数组参数，如`isIn`可接受的字符串数组，则它将被解释为多个字符串参数，而不是一个数组参数。要解决此问题，请传递一个单长度的参数数组，例如，上面所示的`[['one', 'two']]`。

要使用自定义错误消息而不是[validator.js](https://github.com/chriso/validator.js)提供的错误消息，请使用对象而不是纯值或参数数组，例如，可以为不需要参数的验证器提供一条自定义消息：

```
isInt: {
  msg: "Must be an integer number of pennies"
}
```

或者将所需参数做为`args`属性传入：

```
isIn: {
  args: [['en', 'zh']],
  msg: "Must be English or Chinese"
}
```

使用自定义验证器功能时，错误消息将是抛出`Error`对象所保留的任何消息。

更多关于内置验证器的介绍，请参考：[validator.js](https://github.com/chriso/validator.js)项目。

提示：还可以为日志记录部分定义自定义函数。传递一个函数即可。其中第一个参数将是所记录的字符串。



#### 属性验证器与`allowNull`

如果将模型的特定字段设置为不允许`null`（`allowNull: false`），并且该值已设置为许`null`，则将跳过所有验证器，并抛出`ValidationError`。

另外，如果将其设置为允许`null`（`allowNull: true`）并且该值已设置为`null`，则仅将跳过内置验证器，而自定义验证器仍将执行。

这意味着，对于一个字符串字段，你对该字段验证其长度在5到10个字符之间，但也允许为`null`（因为当该值为`null`时，长度验证器将被自动跳过）：

```
class User extends Model {}
User.init({
  username: {
    type: Sequelize.STRING,
    allowNull: true,
    validate: {
      len: [5, 10]
    }
  }
}, { sequelize });
```

也可以使用自定义验证器有条件地允许`null`，因为不会跳过它：

```
class User extends Model {}
User.init({
  age: Sequelize.INTEGER,
  name: {
    type: Sequelize.STRING,
    allowNull: true,
    validate: {
      customValidator(value) {
        if (value === null && this.age !== 10) {
          throw new Error("name can't be null unless age is 10");
        }
      })
    }
  }
}, { sequelize });
```

还可以通过设置验证器中的`notNull`属性来自定义`allowNull`错误消息：

```
class User extends Model {}
User.init({
  name: {
    type: Sequelize.STRING,
    allowNull: false,
    validate: {
      notNull: {
        msg: 'Please enter your name'
      }
    }
  }
}, { sequelize });
```



#### 模型范围内的验证

还可以定义验证，在字段的验证器之后检查模型。例如，通过这一方法，可以确保同时设置或未设置`latitude`和`longitude`，并且其中之一未设置则失败。

可以通过模型对象的上下文调用模型验证器方法，如果它们抛出错误，则认为失败，否则将通过。这与自定义字段验证器相同。

所有错误消息都将与字段验证错误一起放入验证结果对象中，并以`validate`选项对象中失败的验证方法的键命名。在任何时候每种模型验证方法都只有一个错误消息，但它会在数组中显示为单个字符串错误，以最大程度地提高与字段错误的保持一致。

示例：

```
class Pub extends Model {}
Pub.init({
  name: { type: Sequelize.STRING },
  address: { type: Sequelize.STRING },
  latitude: {
    type: Sequelize.INTEGER,
    allowNull: true,
    defaultValue: null,
    validate: { min: -90, max: 90 }
  },
  longitude: {
    type: Sequelize.INTEGER,
    allowNull: true,
    defaultValue: null,
    validate: { min: -180, max: 180 }
  },
}, {
  validate: {
    bothCoordsOrNone() {
      if ((this.latitude === null) !== (this.longitude === null)) {
        throw new Error('Require either both latitude and longitude or neither')
      }
    }
  },
  sequelize,
})
```

在本例中，我们添加了`latitude`和`longitude`同时设置或同时未设置验证。如果其中一个超出指定范围，则返回`raging_bullock_arms.validate()`。

```
{
  'latitude': ['Invalid number: latitude'],
  'bothCoordsOrNone': ['Require either both latitude and longitude or neither']
}
```

也可以使用在单个属性（例如，`latitude`属性，通过检查`(value === null) !== (this.longitude === null)`）上定义的自定义验证程序来完成这种验证，但是模型验证方法更为常用。



### 配置

你还可以影响Sequelize列名的处理方式：

```
class Bar extends Model {}
Bar.init({ /* bla */ }, {
  // 模型名。本模型将以该名称存储在`sequelize.models`中。
  // 默认为类的名称，即在这种情况下为`Bar`。 这将控制自动生成的`foreignKey`的名称和关联命名
  modelName: 'bar',

  // 不要添加时间戳属性 (updatedAt, createdAt)
  timestamps: false,

  // 不实际删除数据库记录，而是设置一个新 deletedAt 属性，其值为当前日期
  // `paranoid` 仅在 `timestamps` 启用时可用
  paranoid: true,

  // 自动设置字段为蛇型命名规则
  // 不会覆盖已定义的字段选项属性
  underscored: true,

  // 禁止修改表名
  // 默认情况下，sequelize 会自动将所有传递的模型名称转换为复数形式。 如果不想这样做，请设置以下内容
  freezeTableName: true,

  // 定义表名
  tableName: 'my_very_custom_table_name',

  // 启用乐观锁定。启用后，sequelize将向模型添加版本计数属性，并在保存旧实例时引发 `OptimisticLockingError` 错误。
  // 设置为`true`或使用要启用的属性名称的字符串。
  version: true,

  // Sequelize 实例
  sequelize,
})
```

如果你想sequelize仅处理部分时间戳，或者对时间戳相关列使用不同的名称，那么可对每一列单独设置：

```
class Foo extends Model {}
Foo.init({ /* bla */ }, {
  // 不要忘了启用时间戳!
  timestamps: true,

  // 不需要 `createdAt`
  createdAt: false,

  // 需要 `updatedAt`，但列名为"updateTimestamp"
  updatedAt: 'updateTimestamp',

  // `deletedAt`列名为"destroyTime" (注意，启用`paranoid`才会生效)
  deletedAt: 'destroyTime',
  paranoid: true,

  sequelize,
})
```

还可以修改数据库引擎。如，改为默认为`MyISAM. InnoDB`：

```
class Person extends Model {}
Person.init({ /* attributes */ }, {
  engine: 'MYISAM',
  sequelize
})

// or globally
const sequelize = new Sequelize(db, user, pw, {
  define: { engine: 'MYISAM' }
})
```

最后，可以为MySQL和PG添加表注释：

```
class Person extends Model {}
Person.init({ /* attributes */ }, {
  comment: "I'm a table comment!",
  sequelize
})
```



### 导入

模型定义可以保存在独立的文件中，并通过`import`方法导入。返回的对象与导入文件的函数中定义的对象完全相同。`v1:5.0`Sequelize会将导入缓存，因此在两次或更多次调用文件的导入时都不会有问题。

```
// 在你的服务器文件中 - 如: app.js
const Project = sequelize.import(__dirname + "/path/to/models/project")

// 模型定义在：/path/to/models/project.js
// 你可能会注意到，数据类型与上面所述完全相同
module.exports = (sequelize, DataTypes) => {
  class Project extends sequelize.Model { }
  Project.init({
    name: DataTypes.STRING,
    description: DataTypes.TEXT
  }, { sequelize });
  return Project;
}
```

`import`方法还可以接受回调作为参数。

```
sequelize.import('project', (sequelize, DataTypes) => {
  class Project extends sequelize.Model {}
  Project.init({
    name: DataTypes.STRING,
    description: DataTypes.TEXT
  }, { sequelize })
  return Project;
})
```

还有个额外功能很有用，在`/path/to/models/project`似乎正确的情况下也会抛出`Error: Cannot find module`。 一些框架，例如：Meteor，重载`require`和吐出“意外”结果，例如：

```
Error: Cannot find module '/home/you/meteorApp/.meteor/local/build/programs/server/app/path/to/models/project.js'
```

这可以通过传入Meteor的`require`版本来解决。因此，虽然这可能会失败...

```
const AuthorModel = db.import('./path/to/models/project');
```

...这样会成功：

```
const AuthorModel = db.import('project', require('./path/to/models/project'));
```



### 乐观锁

Sequelize通过模型实例的`version`计数内置了对乐观锁的支持。“乐观锁”默认情况下处于禁用状态，可以通过在特定模型定义或全局模型配置中将`version`属性设置为`true`来启用。更多相关详细信息，请参见：[模型配置](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#configuration)。

乐观锁定允许并发访问模型记录以进行编辑，并防止冲突覆盖数据。它通过检查自从读取以来另一个进程是否对记录进行了更改，并在检测到冲突时抛出`OptimisticLockError`来执行此操作。



### 数据库同步

当开始一个新项目时，你还没有数据库结构，而使用Sequelize则不再需要数据库结构。只需指定模型结构，然后让库完成其余工作即可。当前支持创建和删除表：

```
// Create the tables:
Project.sync()
Task.sync()

// Force the creation!
Project.sync({force: true}) // this will drop the table first and re-create it afterwards

// drop the tables:
Project.drop()
Task.drop()

// event handling:
Project.[sync|drop]().then(() => {
  // ok ... everything is nice!
}).catch(error => {
  // oooh, did you enter wrong database credentials?
})
```

因为同步和删除所有表可能要写很多行原生SQL，现在你可以让Sequelize为帮你完成工作：

```
// Sync all models that aren't already in the database
sequelize.sync()

// Force sync all models
sequelize.sync({force: true})

// Drop all tables
sequelize.drop()

// emit handling:
sequelize.[sync|drop]().then(() => {
  // woot woot
}).catch(error => {
  // whooops
})
```

因为`.sync({ force: true })`是破坏性操作，可以通过`match`来添加额外的检查。`match`选项告诉sequelize在同步之前将正则表达式与数据库名称进行匹配-对在测试中使用`force: true`而不在生产代码中使用的情况，进行安全检查。

```
// This will run .sync() only if database name ends with '_test'
sequelize.sync({ force: true, match: /_test$/ });
```



### 扩展模型

Sequelize 的`Model`是ES6类，你可以轻松的定义实例或类级别的方法。

```
class User extends Model {
  // 添加类级别的方法
  static classLevelMethod() {
    return 'foo';
  }

  // 添加实例级别的方法
  instanceLevelMethod() {
    return 'bar';
  }
}
User.init({ firstname: Sequelize.STRING }, { sequelize });
```

当然，也可以访问实例数据并生成虚拟getter：

```
class User extends Model {
  getFullname() {
    return [this.firstname, this.lastname].join(' ');
  }
}
User.init({ firstname: Sequelize.STRING, lastname: Sequelize.STRING }, { sequelize });

// Example:
User.build({ firstname: 'foo', lastname: 'bar' }).getFullname() // 'foo bar'
```



#### 索引

Sequelize 支持通过模型定义添加索引，索引会在`Model.sync()`或`sequelize.sync`时创建：

```
class User extends Model {}
User.init({}, {
  indexes: [
    // Create a unique index on email
    {
      unique: true,
      fields: ['email']
    },

    // Creates a gin index on data with the jsonb_path_ops operator
    {
      fields: ['data'],
      using: 'gin',
      operator: 'jsonb_path_ops'
    },

    // By default index name will be [table]_[fields]
    // Creates a multi column partial index
    {
      name: 'public_by_author',
      fields: ['author', 'status'],
      where: {
        status: 'public'
      }
    },

    // A BTREE index with an ordered field
    {
      name: 'title_index',
      using: 'BTREE',
      fields: ['author', {attribute: 'title', collate: 'en_US', order: 'DESC', length: 5}]
    }
  ],
  sequelize
});
```



## 6. 模型使用(Model usage)

- [数据检索/查找器](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#data-retrieval---finders)

- - [`find` - 搜索数据库中一个特定元素](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#-code-find)
  - [`findOrCreate` - 搜索一个特定元素不存在时则新建](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#-code-findorcreate)
  - [`findAndCountAll` - 搜索数据库中多个元素，同时返回数据和总数](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#-code-findandcountall)
  - [`findAll` - 搜索数据库中多个元素](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#-code-findall)
  - [复合过滤 / OR / NOT 查询](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#complex-filtering---or---not-queries)
  - [对数据集使用`limit`、`offset`、`order`和`group`](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#manipulating-the-dataset-with-limit--offset--order-and-group)
  - [原始查询](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#finders-raw-queries)
  - [`count` - 统计数据库中元素数](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#-code-count)
  - [`max` - 获取表中特定属性的最大值](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#-code-max)
  - [`min` - 获取表中特定属性的最小值](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#-code-min)
  - [`sum` - 对特定属性的值求和](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#-code-sum)

- [预加载](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#eager-loading)

- - [在顶层`where`中预加载模型](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#top-level-where-with-eagerly-loaded-models)
  - [包含所有](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#including-everything)
  - [包含软删除的记录](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#including-soft-deleted-records)
  - [对预加载的关联排序](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#ordering-eager-loaded-associations)
  - [嵌套预加载](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#nested-eager-loading)

### 数据检索/查找器(Finder)

Finder方法用于从数据库查询数据。它们不会返回普通对象，而是返回模型实例。因为finder方法返回模型实例，所以你可以按照[实例](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#instances)文档的所述在查询结果上调用任何模型实例方法。

在本章节，将介绍一些查找器方法可以执行的操作：

#### `find` - 搜索数据库中一个特定元素

```
// 根据已知ID查询
Project.findByPk(123).then(project => {
  // project 会是 Project 的实例，而且为表中 id 为 123 的存储内容
  // 如果未定义此类条目，则将为 null
})

// 根据属性查询
Project.findOne({ where: {title: 'aProject'} }).then(project => {
  // project 会是所匹配到的第一条`title`为'aProject'的 Project || null
})


Project.findOne({
  where: {title: 'aProject'},
  attributes: ['id', ['name', 'title']]
}).then(project => {
  // project 会是所匹配到的第一条`title`为'aProject'的 Project || null
  // project.get('title') 将包含该项目的名称
})
```



#### `findOrCreate` - 搜索一个特定元素不存在时则新建

`findOrCreate`方法可用于检查数据库中是否已存在某个元素。如果已存在，则返回相应的实例。 如果该元素不存在，则将创建它。

假设我们有一个有`User`模型的空数据库，其有一个`username`和一个`job`属性。

对于创建的情况，可以在`where`选项后面添加`defaults`。

```
User
  .findOrCreate({where: {username: 'sdepold'}, defaults: {job: 'Technical Lead JavaScript'}})
  .then(([user, created]) => {
    console.log(user.get({
      plain: true
    }))
    console.log(created)

    /*
     findOrCreate 会返回一个包含所找到或创建的对象的数组，以及一个布尔值，如果创建了新对象，则该布尔值为true，否则为false:

    [ {
        username: 'sdepold',
        job: 'Technical Lead JavaScript',
        id: 1,
        createdAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET),
        updatedAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET)
      },
      true ]

 In the example above, the array spread on line 3 divides the array into its 2 parts and passes them
  as arguments to the callback function defined beginning at line 39, which treats them as "user" and
  "created" in this case. (So "user" will be the object from index 0 of the returned array and
  "created" will equal "true".)
    */
  })
```

以上代码会创建新的实例，而当我们已有一个实例时：

```
User.create({ username: 'fnord', job: 'omnomnom' })
  .then(() => User.findOrCreate({where: {username: 'fnord'}, defaults: {job: 'something else'}}))
  .then(([user, created]) => {
    console.log(user.get({
      plain: true
    }))
    console.log(created)

    /*
    In this example, findOrCreate returns an array like this:
    [ {
        username: 'fnord',
        job: 'omnomnom',
        id: 2,
        createdAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET),
        updatedAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET)
      },
      false
    ]
    The array returned by findOrCreate gets spread into its 2 parts by the array spread on line 3, and
    the parts will be passed as 2 arguments to the callback function beginning on line 69, which will
    then treat them as "user" and "created" in this case. (So "user" will be the object from index 0
    of the returned array and "created" will equal "false".)
    */
  })
```

已存在的项目不会被修改。



#### `findAndCountAll` - 搜索数据库中多个元素，同时返回数据和总数

`findAndCountAll`是一个结合了`findAll`和`count`便捷方法（请参见下文），在处理与分页相关的查询时非常有用。在这一查询中，可以用来检索有`limit`和`offset`的数据，返回结果除检索数据外还会有记录总数。

本方法查询成功后，将收到有以下两个属性的对象：

- `count` - 整数，表示由`where`子句和其它过滤条件检索到的数据总数
- `rows` - 对象数组，由`where`子句和其它过滤条件匹配到的，及`limit`和`offset`的数据范围内的数据

```
Project
  .findAndCountAll({
     where: {
        title: {
          [Op.like]: 'foo%'
        }
     },
     offset: 10,
     limit: 2
  })
  .then(result => {
    console.log(result.count);
    console.log(result.rows);
  });
```

它还支持`include`查询，仅会将标记为`required`的包含项添加到计数部分。

如果，在检索用户的同时，同时想查出其概况信息：

```
User.findAndCountAll({
  include: [
     { model: Profile, required: true }
  ],
  limit: 3
});
```

因为`Profile`设置了`required`，所以结果是内连接查询，也就只有有profile信息的用户才会被统计。如果不设置`required`，则不管有没有profile都会被统计。

另外，添加`where`条件后，会自动设置`required`：

```
User.findAndCountAll({
  include: [
     { model: Profile, where: { active: true }}
  ],
  limit: 3
});
```

在以上查询中，因为添加了`where`条件，所以会自动将`required`设置为`true`。

传给`findAndCountAll`的选项参数，与`findAll`相同（参见下节）。



#### `findAll` - 搜索数据库中多个元素

```
// find multiple entries
Project.findAll().then(projects => {
  // projects will be an array of all Project instances
})

// search for specific attributes - hash usage
Project.findAll({ where: { name: 'A Project' } }).then(projects => {
  // projects will be an array of Project instances with the specified name
})

// search within a specific range
Project.findAll({ where: { id: [1,2,3] } }).then(projects => {
  // projects will be an array of Projects having the id 1, 2 or 3
  // this is actually doing an IN query
})

Project.findAll({
  where: {
    id: {
      [Op.and]: {a: 5},           // AND (a = 5)
      [Op.or]: [{a: 5}, {a: 6}],  // (a = 5 OR a = 6)
      [Op.gt]: 6,                // id > 6
      [Op.gte]: 6,               // id >= 6
      [Op.lt]: 10,               // id < 10
      [Op.lte]: 10,              // id <= 10
      [Op.ne]: 20,               // id != 20
      [Op.between]: [6, 10],     // BETWEEN 6 AND 10
      [Op.notBetween]: [11, 15], // NOT BETWEEN 11 AND 15
      [Op.in]: [1, 2],           // IN [1, 2]
      [Op.notIn]: [1, 2],        // NOT IN [1, 2]
      [Op.like]: '%hat',         // LIKE '%hat'
      [Op.notLike]: '%hat',       // NOT LIKE '%hat'
      [Op.iLike]: '%hat',         // ILIKE '%hat' (case insensitive)  (PG only)
      [Op.notILike]: '%hat',      // NOT ILIKE '%hat'  (PG only)
      [Op.overlap]: [1, 2],       // && [1, 2] (PG array overlap operator)
      [Op.contains]: [1, 2],      // @> [1, 2] (PG array contains operator)
      [Op.contained]: [1, 2],     // <@ [1, 2] (PG array contained by operator)
      [Op.any]: [2,3]            // ANY ARRAY[2, 3]::INTEGER (PG only)
    },
    status: {
      [Op.not]: false           // status NOT FALSE
    }
  }
})
```



#### 复合过滤 / OR / NOT 查询

在具有多层嵌套`AND`、`OR`和`NOT`条件的查询中，可能会很复杂。为了做到这一点，可以使用`or`、`and`或`not`操作符：

```
Project.findOne({
  where: {
    name: 'a project',
    [Op.or]: [
      { id: [1,2,3] },
      { id: { [Op.gt]: 10 } }
    ]
  }
})

Project.findOne({
  where: {
    name: 'a project',
    id: {
      [Op.or]: [
        [1,2,3],
        { [Op.gt]: 10 }
      ]
    }
  }
})
```

上面两段代码都会生成以下查询语句：

```
SELECT *
FROM `Projects`
WHERE (
  `Projects`.`name` = 'a project'
   AND (`Projects`.`id` IN (1,2,3) OR `Projects`.`id` > 10)
)
LIMIT 1;
```

`not`示例：

```
Project.findOne({
  where: {
    name: 'a project',
    [Op.not]: [
      { id: [1,2,3] },
      { array: { [Op.contains]: [3,4,5] } }
    ]
  }
});
```

会生成：

```
SELECT *
FROM `Projects`
WHERE (
  `Projects`.`name` = 'a project'
   AND NOT (`Projects`.`id` IN (1,2,3) OR `Projects`.`array` @> ARRAY[3,4,5]::INTEGER[])
)
LIMIT 1;
```



#### 对数据集使用`limit`、`offset`、`order`和`group`

要获取更多相关数据，可以使用`limit`、`offset`、`order`和`group`：

```
// limit the results of the query
Project.findAll({ limit: 10 })

// step over the first 10 elements
Project.findAll({ offset: 10 })

// step over the first 10 elements, and take 2
Project.findAll({ offset: 10, limit: 2 })
```

分组和排序的语法相同，以下是一个分组的示例，另一个是排序。

```
Project.findAll({order: [['title', 'DESC']]})
// yields ORDER BY title DESC

Project.findAll({group: 'name'})
// yields GROUP BY name
```

请注意，在上面的两个示例中，所提供的字符串是直接插入查询中的，即列名不会被转义。当你向排序/分组提供字符串时，总是这样。如果要转义列名，即使只想按单个列排序/分组，也应提供一个参数数组。

```
something.findOne({
  order: [
    // will return `name`
    ['name'],
    // will return `username` DESC
    ['username', 'DESC'],
    // will return max(`age`)
    sequelize.fn('max', sequelize.col('age')),
    // will return max(`age`) DESC
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],
    // will return otherfunction(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],
    // will return otherfunction(awesomefunction(`col`)) DESC, This nesting is potentially infinite!
    [sequelize.fn('otherfunction', sequelize.fn('awesomefunction', sequelize.col('col'))), 'DESC']
  ]
})
```

综上所述，order/group数组的元素可以如下：

- `String` - 将被添加引号````

- `Array` - 第一个元素将被添加引号，第二个将被直接追加

- ```
  Object
  ```

  - Raw将直接添加而不引用
  - 其他所有内容都将被忽略，如果未设置`raw`，查询将失败

- `Sequelize.fn`及`Sequelize.col` - 返回函数和带引号的列名



#### 原始查询

有时，你可能希望得到一个仅需显示而无需处理的庞大数据集。对于所选择的每一行，Sequelize创建一个实例，该实例具有用于更新、删除、获取关联等功能。如果数据量较大，则可能需要一些时间。如果只需要原始数据并且不想更新任何内容，可以像下面这样直接获取原始数据。

```
// 如果你需要查询大量数据，而不想为每条数据构建DAO花费时间
// 可以传入一个 `raw` 选项，以获取原始数据：
Project.findAll({ where: { ... }, raw: true })
```



#### `count` - 统计数据库中元素数

这是一个统计数据库对象数的方法：

```
Project.count().then(c => {
  console.log("There are " + c + " projects!")
})

Project.count({ where: {'id': {[Op.gt]: 25}} }).then(c => {
  console.log("There are " + c + " projects with an id greater than 25.")
})
```



#### `max` - 获取表中特定属性的最大值

用于获取某一属性的最大值：

```
/*
  Let's assume 3 person objects with an attribute age.
  The first one is 10 years old,
  the second one is 5 years old,
  the third one is 40 years old.
*/
Project.max('age').then(max => {
  // this will return 40
})

Project.max('age', { where: { age: { [Op.lt]: 20 } } }).then(max => {
  // will be 10
})
```



#### `min` - 获取表中特定属性的最小值

用于获取某一属性的最大值：

```
/*
  Let's assume 3 person objects with an attribute age.
  The first one is 10 years old,
  the second one is 5 years old,
  the third one is 40 years old.
*/
Project.min('age').then(min => {
  // this will return 5
})

Project.min('age', { where: { age: { [Op.gt]: 5 } } }).then(min => {
  // will be 10
})
```



#### `sum` - 对特定属性的值求和

要计算表中指定列的总和，可以使用`sum`方法：

```
/*
  Let's assume 3 person objects with an attribute age.
  The first one is 10 years old,
  the second one is 5 years old,
  the third one is 40 years old.
*/
Project.sum('age').then(sum => {
  // this will return 55
})

Project.sum('age', { where: { age: { [Op.gt]: 5 } } }).then(sum => {
  // will be 50
})
```



### 预加载

当从数据库中检索数据时，你可能会希望获得与同一查询的关联项-这称为“预加载”。其背后的基本思想是，在调用`find`或`findAll`时使用`include`属性。假设有以下设置：

```
class User extends Model {}
User.init({ name: Sequelize.STRING }, { sequelize, modelName: 'user' })
class Task extends Model {}
Task.init({ name: Sequelize.STRING }, { sequelize, modelName: 'task' })
class Tool extends Model {}
Tool.init({ name: Sequelize.STRING }, { sequelize, modelName: 'tool' })

Task.belongsTo(User)
User.hasMany(Task)
User.hasMany(Tool, { as: 'Instruments' })

sequelize.sync().then(() => {
  // this is where we continue ...
})
```

现在，让我们来获取所有task及与其相关联的user：

```
Task.findAll({ include: [ User ] }).then(tasks => {
  console.log(JSON.stringify(tasks))

  /*
    [{
      "name": "A Task",
      "id": 1,
      "createdAt": "2013-03-20T20:31:40.000Z",
      "updatedAt": "2013-03-20T20:31:40.000Z",
      "userId": 1,
      "user": {
        "name": "John Doe",
        "id": 1,
        "createdAt": "2013-03-20T20:31:45.000Z",
        "updatedAt": "2013-03-20T20:31:45.000Z"
      }
    }]
  */
})
```

请注意，访问器（在结果实例中为`User`属性）是单数的，因为关联是一对一的。

接下来，通过多对多的形式来加载数据：

```
User.findAll({ include: [ Task ] }).then(users => {
  console.log(JSON.stringify(users))

  /*
    [{
      "name": "John Doe",
      "id": 1,
      "createdAt": "2013-03-20T20:31:45.000Z",
      "updatedAt": "2013-03-20T20:31:45.000Z",
      "tasks": [{
        "name": "A Task",
        "id": 1,
        "createdAt": "2013-03-20T20:31:40.000Z",
        "updatedAt": "2013-03-20T20:31:40.000Z",
        "userId": 1
      }]
    }]
  */
})
```

请注意，访问器（结果实例中的`Tasks`属性）是复数形式，因为关联是多对多的。

如果对关联使用别名（通过`as`指定），可以在关联模型时指定别名。如，以下示例中对user的`Tool`指定别名`Instruments`。为了正确处理，必须指定要加载的模型以及别名：

```
User.findAll({ include: [{ model: Tool, as: 'Instruments' }] }).then(users => {
  console.log(JSON.stringify(users))

  /*
    [{
      "name": "John Doe",
      "id": 1,
      "createdAt": "2013-03-20T20:31:45.000Z",
      "updatedAt": "2013-03-20T20:31:45.000Z",
      "Instruments": [{
        "name": "Toothpick",
        "id": 1,
        "createdAt": null,
        "updatedAt": null,
        "userId": 1
      }]
    }]
  */
})
```

还可以通过指定与关联别名匹配的字符串来按别名包含：

```
User.findAll({ include: ['Instruments'] }).then(users => {
  console.log(JSON.stringify(users))

  /*
    [{
      "name": "John Doe",
      "id": 1,
      "createdAt": "2013-03-20T20:31:45.000Z",
      "updatedAt": "2013-03-20T20:31:45.000Z",
      "Instruments": [{
        "name": "Toothpick",
        "id": 1,
        "createdAt": null,
        "updatedAt": null,
        "userId": 1
      }]
    }]
  */
})

User.findAll({ include: [{ association: 'Instruments' }] }).then(users => {
  console.log(JSON.stringify(users))

  /*
    [{
      "name": "John Doe",
      "id": 1,
      "createdAt": "2013-03-20T20:31:45.000Z",
      "updatedAt": "2013-03-20T20:31:45.000Z",
      "Instruments": [{
        "name": "Toothpick",
        "id": 1,
        "createdAt": null,
        "updatedAt": null,
        "userId": 1
      }]
    }]
  */
})
```

在预加载时，同样可以关联模型使用`where`。以下会返回`User`的，所有符合`where`条件的`Tool`模型记录：

```
User.findAll({
    include: [{
        model: Tool,
        as: 'Instruments',
        where: { name: { [Op.like]: '%ooth%' } }
    }]
}).then(users => {
    console.log(JSON.stringify(users))

    /*
      [{
        "name": "John Doe",
        "id": 1,
        "createdAt": "2013-03-20T20:31:45.000Z",
        "updatedAt": "2013-03-20T20:31:45.000Z",
        "Instruments": [{
          "name": "Toothpick",
          "id": 1,
          "createdAt": null,
          "updatedAt": null,
          "userId": 1
        }]
      }],

      [{
        "name": "John Smith",
        "id": 2,
        "createdAt": "2013-03-20T20:31:45.000Z",
        "updatedAt": "2013-03-20T20:31:45.000Z",
        "Instruments": [{
          "name": "Toothpick",
          "id": 1,
          "createdAt": null,
          "updatedAt": null,
          "userId": 1
        }]
      }],
    */
  })
```

如果预加载使用了`include.where`过滤，则`include.required`会被设置为`true`。这意味着，父模型与子模型之间是内连接的关系。



#### 在顶层`where`中预加载模型

将`where`条件从包含模型从`ON`条件移到顶层的`WHERE`，可以使用`'$nested.column$'`语法：

```
User.findAll({
    where: {
        '$Instruments.name$': { [Op.iLike]: '%ooth%' }
    },
    include: [{
        model: Tool,
        as: 'Instruments'
    }]
}).then(users => {
    console.log(JSON.stringify(users));

    /*
      [{
        "name": "John Doe",
        "id": 1,
        "createdAt": "2013-03-20T20:31:45.000Z",
        "updatedAt": "2013-03-20T20:31:45.000Z",
        "Instruments": [{
          "name": "Toothpick",
          "id": 1,
          "createdAt": null,
          "updatedAt": null,
          "userId": 1
        }]
      }],

      [{
        "name": "John Smith",
        "id": 2,
        "createdAt": "2013-03-20T20:31:45.000Z",
        "updatedAt": "2013-03-20T20:31:45.000Z",
        "Instruments": [{
          "name": "Toothpick",
          "id": 1,
          "createdAt": null,
          "updatedAt": null,
          "userId": 1
        }]
      }],
    */
```



#### 包含所有

要包含所有属性，可以将`all: true`做为单个对象传入：

```
User.findAll({ include: [{ all: true }]});
```



#### 包含软删除的记录

如果想加载软删除的记录，可以将`include.paranoid`设置为`false`：

```
User.findAll({
    include: [{
        model: Tool,
        where: { name: { [Op.like]: '%ooth%' } },
        paranoid: false // query and loads the soft deleted records
    }]
});
```



#### 对预加载的关联排序

以下是一个一对多的关系：

```
Company.findAll({ include: [ Division ], order: [ [ Division, 'name' ] ] });
Company.findAll({ include: [ Division ], order: [ [ Division, 'name', 'DESC' ] ] });
Company.findAll({
  include: [ { model: Division, as: 'Div' } ],
  order: [ [ { model: Division, as: 'Div' }, 'name' ] ]
});
Company.findAll({
  include: [ { model: Division, as: 'Div' } ],
  order: [ [ { model: Division, as: 'Div' }, 'name', 'DESC' ] ]
});
Company.findAll({
  include: [ { model: Division, include: [ Department ] } ],
  order: [ [ Division, Department, 'name' ] ]
});
```

在这个多对多的连接中，同样可以对关联表排序：

```
Company.findAll({
  include: [ { model: Division, include: [ Department ] } ],
  order: [ [ Division, DepartmentDivision, 'name' ] ]
});
```



#### 嵌套预加载

可以使用嵌套的预加载来加载相关模型的所有相关模型：

```
User.findAll({
  include: [
    {model: Tool, as: 'Instruments', include: [
      {model: Teacher, include: [ /* etc */]}
    ]}
  ]
}).then(users => {
  console.log(JSON.stringify(users))

  /*
    [{
      "name": "John Doe",
      "id": 1,
      "createdAt": "2013-03-20T20:31:45.000Z",
      "updatedAt": "2013-03-20T20:31:45.000Z",
      "Instruments": [{ // 1:M and N:M association
        "name": "Toothpick",
        "id": 1,
        "createdAt": null,
        "updatedAt": null,
        "userId": 1,
        "Teacher": { // 1:1 association
          "name": "Jimi Hendrix"
        }
      }]
    }]
  */
})
```

这会生成一个外连接。但是，相关模型上的`where`子句将创建内联接，并且仅返回具有匹配子模型的实例。要返回所有父实例，应添加`required: false`。

```
User.findAll({
  include: [{
    model: Tool,
    as: 'Instruments',
    include: [{
      model: Teacher,
      where: {
        school: "Woodstock Music School"
      },
      required: false
    }]
  }]
}).then(users => {
  /* ... */
})
```

上面的查询将返回所有User及其所有Instrument，但仅返回与`Woodstock Music School`相关的那些Teacher。

全部包括还支持嵌套加载：

```
User.findAll({ include: [{ all: true, nested: true }]});
```



## 7. 钩子(Hooks)

- [执行顺序](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#order-of-operations)
- [声明钩子](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#declaring-hooks)
- [移除钩子](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#removing-hooks)
- 全局/通用钩子
  - [默认钩子 (`Sequelize.options.define`)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#default-hooks--sequelize-options-define-)
  - [常驻钩子 (`Sequelize.addHook`)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#permanent-hooks--sequelize-addhook-)
  - [连接钩子](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#connection-hooks)
- 实例钩子
  - [模型钩子](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#model-hooks)
- [关联/关系](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#hooks-associations)
- 有关事务的注意事项
  - [内部事务](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#internal-transactions)

钩子（也称为生命周期事件）是在执行sequelize中，调用之前和之后调用的函数。例如，如果要在保存之前始终在模型上设置值，则可以添加`beforeUpdate`钩子。

注意：不能将钩子与实例一起使用，而是与模型一起使用。

所有钩子列表，请参考[Hooks文件](https://github.com/sequelize/sequelize/blob/master/lib/hooks.js#L7)。



### 执行顺序

```
(1)
  beforeBulkCreate(instances, options)
  beforeBulkDestroy(options)
  beforeBulkUpdate(options)
(2)
  beforeValidate(instance, options)
(-)
  validate
(3)
  afterValidate(instance, options)
  - or -
  validationFailed(instance, options, error)
(4)
  beforeCreate(instance, options)
  beforeDestroy(instance, options)
  beforeUpdate(instance, options)
  beforeSave(instance, options)
  beforeUpsert(values, options)
(-)
  create
  destroy
  update
(5)
  afterCreate(instance, options)
  afterDestroy(instance, options)
  afterUpdate(instance, options)
  afterSave(instance, options)
  afterUpsert(created, options)
(6)
  afterBulkCreate(instances, options)
  afterBulkDestroy(options)
  afterBulkUpdate(options)
```



### 声明钩子

钩子参数通过引用传递。这意味着你可以修改其值，这会反映在 insert/update 语句中。钩子可能包含异步操作-在这种情况下，钩子函数应该返回一个Promise。

当前有三种添加自定义钩子的方式：

```
// Method 1 via the .init() method
class User extends Model {}
User.init({
  username: DataTypes.STRING,
  mood: {
    type: DataTypes.ENUM,
    values: ['happy', 'sad', 'neutral']
  }
}, {
  hooks: {
    beforeValidate: (user, options) => {
      user.mood = 'happy';
    },
    afterValidate: (user, options) => {
      user.username = 'Toni';
    }
  },
  sequelize
});

// Method 2 via the .addHook() method
User.addHook('beforeValidate', (user, options) => {
  user.mood = 'happy';
});

User.addHook('afterValidate', 'someCustomName', (user, options) => {
  return Promise.reject(new Error("I'm afraid I can't let you do that!"));
});

// Method 3 via the direct method
User.beforeCreate((user, options) => {
  return hashPassword(user.password).then(hashedPw => {
    user.password = hashedPw;
  });
});

User.afterValidate('myHookAfter', (user, options) => {
  user.username = 'Toni';
});
```



### 移除钩子

只能删除有名称参数的钩子。

```
class Book extends Model {}
Book.init({
  title: DataTypes.STRING
}, { sequelize });

Book.addHook('afterCreate', 'notifyUsers', (book, options) => {
  // ...
});

Book.removeHook('afterCreate', 'notifyUsers');
```

可以有许多同名钩子。调用`.removeHook()`会将所有删除。



### 全局/通用钩子

全局钩子适用于所有型的挂。他们可以为所有模型定义所需的行为，这对于插件特别有用。可以用两种方式定义它们，语义略有不同：

#### 默认钩子 (`Sequelize.options.define`)

```
const sequelize = new Sequelize(..., {
    define: {
        hooks: {
            beforeCreate: () => {
              // Do stuff
            }
        }
    }
});
```

这会向所有模型添加一个默认钩子，如果模型未定义自己的`beforeCreate`钩子，则将运行该钩子：

```
class User extends Model {}
User.init({}, { sequelize });
class Project extends Model {}
Project.init({}, {
    hooks: {
        beforeCreate: () => {
            // Do other stuff
        }
    },
    sequelize
});

User.create() // Runs the global hook
Project.create() // Runs its own hook (because the global hook is overwritten)
```

#### 常驻钩子 (`Sequelize.addHook`)

```
sequelize.addHook('beforeCreate', () => {
    // Do stuff
});
```

无论模型是否指定了自己的`beforeCreate`钩子，该钩子始终会在创建之前运行。本地钩子总是在全局钩子之前运行：

```
class User extends Model {}
User.init({}, { sequelize });
class Project extends Model {}
Project.init({}, {
    hooks: {
        beforeCreate: () => {
            // Do other stuff
        }
    },
    sequelize
});

User.create() // Runs the global hook
Project.create() // Runs its own hook, followed by the global hook
```

钩子同样可以定义在`Sequelize.options`：

```
new Sequelize(..., {
    hooks: {
        beforeCreate: () => {
            // do stuff
        }
    }
});
```



#### 连接钩子

Sequelize提供了四个钩子，它们会在获得或释放数据库连接之前和之后立即执行：

```
beforeConnect(config)
afterConnect(connection, config)
beforeDisconnect(connection)
afterDisconnect(connection)
```

如果您需要异步获取数据库认证，或者需要在创建底层数据库连接后直接访问它们，这些钩子很有用。

例如，我们可以从异步的令牌存储异步获取数据库密码，并使用新的凭证对Sequelize的配置对象进行变更：

```
sequelize.beforeConnect((config) => {
    return getAuthToken()
        .then((token) => {
             config.password = token;
         });
    });
```

这些挂钩只能声明为持久化的全局挂钩，因为所有模型都会共享连接池。



### 实例钩子

当你编辑单个对象时，会触发出以下钩子：

```
beforeValidate
afterValidate or validationFailed
beforeCreate / beforeUpdate / beforeSave  / beforeDestroy
afterCreate / afterUpdate / afterSave / afterDestroy
// ...define ...
User.beforeCreate(user => {
  if (user.accessLevel > 10 && user.username !== "Boss") {
    throw new Error("You can't grant this user an access level above 10!")
  }
})
```

本示例会返回一个错误：

```
// ...define ...
User.beforeCreate(user => {
  if (user.accessLevel > 10 && user.username !== "Boss") {
    throw new Error("You can't grant this user an access level above 10!")
  }
})
```

以下示例会返回成功：

```
User.create({username: 'Boss', accessLevel: 20}).then(user => {
  console.log(user); // user object with username as Boss and accessLevel of 20
});
```



#### 模型钩子

有时，可以利用模型上的`bulkCreate`、`update`、`destroy`方法来一次编辑多个记录。当你使用其中的方法时，会触发以下内容：

```
beforeBulkCreate(instances, options)
beforeBulkUpdate(options)
beforeBulkDestroy(options)
afterBulkCreate(instances, options)
afterBulkUpdate(options)
afterBulkDestroy(options)
```

如果要为每个单独的记录发出钩子，以及批量钩子，可以在调用时传递`individualHooks: true`。

警告：如果使用单个钩子，则在调用钩子之前，所有更新或销毁的实例将被加载到内存中。Sequelize可以使用单个钩子处理的实例数受可用内存的限制。

```
Model.destroy({ where: {accessLevel: 0}, individualHooks: true});
// Will select all records that are about to be deleted and emit before- + after- Destroy on each instance

Model.update({username: 'Toni'}, { where: {accessLevel: 0}, individualHooks: true});
// Will select all records that are about to be updated and emit before- + after- Update on each instance
```

钩子方法的`options`参数，可以做为第二个参数提供给相应方法或其扩展版本。

```
Model.beforeBulkCreate((records, {fields}) => {
  // records = the first argument sent to .bulkCreate
  // fields = one of the second argument fields sent to .bulkCreate
})

Model.bulkCreate([
    {username: 'Toni'}, // part of records argument
    {username: 'Tobi'} // part of records argument
  ], {fields: ['username']} // options parameter
)

Model.beforeBulkUpdate(({attributes, where}) => {
  // where - in one of the fields of the clone of second argument sent to .update
  // attributes - is one of the fields that the clone of second argument of .update would be extended with
})

Model.update({gender: 'Male'} /*attributes argument*/, { where: {username: 'Tom'}} /*where argument*/)

Model.beforeBulkDestroy(({where, individualHooks}) => {
  // individualHooks - default of overridden value of extended clone of second argument sent to Model.destroy
  // where - in one of the fields of the clone of second argument sent to Model.destroy
})

Model.destroy({ where: {username: 'Tom'}} /*where argument*/)
```

如果在`Model.bulkCreate(...)`方法中使用`updateOnDuplicate`选项，在钩子中对`updateOnDuplicate`数组中未提供的字段所做的更改不会持久化到数据库中。如果需要的话，可以在钩子中修改`updateOnDuplicate`选项。

```
// Bulk updating existing users with updateOnDuplicate option
Users.bulkCreate([
  { id: 1, isMember: true },
  { id: 2, isMember: false }
], {
  updateOnDuplicate: ['isMember']
});

User.beforeBulkCreate((users, options) => {
  for (const user of users) {
    if (user.isMember) {
      user.memberSince = new Date();
    }
  }

  // Add memberSince to updateOnDuplicate otherwise the memberSince date wont be
  // saved to the database
  options.updateOnDuplicate.push('memberSince');
});
```



### 关联/关系

在大多数情况下，钩子在关联时对实例的作用相同，某些情况除外：

1. 使用add/set功能时，beforeUpdate/afterUpdate 钩子将运行
2. 调用 beforeDestroy/afterDestroy 钩子的唯一方法是与`onDelete: 'cascade'`和`hooks: true`选项关联。 例如：

```
class Projects extends Model {}
Projects.init({
  title: DataTypes.STRING
}, { sequelize });

class Tasks extends Model {}
Tasks.init({
  title: DataTypes.STRING
}, { sequelize });

Projects.hasMany(Tasks, { onDelete: 'cascade', hooks: true });
Tasks.belongsTo(Projects);
```

以上代码将在“Task”表上的 beforeDestroy/afterDestroy 中运行。默认情况下，Sequelize将尝试尽可能优化查询。当在delete上关联调用时，Sequelize只需执行一个：

```
DELETE FROM `table` WHERE associatedIdentifier = associatedIdentifier.primaryKey
```

但是，当添加`hooks:true`时，会明确告诉Sequelize不必担心优化，它会对关联对象执行`SELECT`并逐个销毁每个实例，以便能够使用正确的参数调用钩子。

如果关联的类型为`n:m`，则在使用`remove`调用时，你可能会对在相关模型上触发的钩子感兴趣。在内部，sequelize使用`Model.destroy`会对每个相关实例调用`bulkDestroy`而不是`before/afterDestroy`钩子。

可以通过将` {individualHooks: true}`传递给`remove`调用来简单解决，以使每个通过实例移除的对象的钩子都被调用。



### 有关事务的注意事项

请注意，Sequelize中的许多模型操作都允许在方法的`options`参数中指定事务。如果在原始调用中指定了事务，它将出现在传递给hook函数的`options`参数中。例如：

```
// Here we use the promise-style of async hooks rather than
// the callback.
User.addHook('afterCreate', (user, options) => {
  // 'transaction' will be available in options.transaction

  // This operation will be part of the same transaction as the
  // original User.create call.
  return User.update({
    mood: 'sad'
  }, {
    where: {
      id: user.id
    },
    transaction: options.transaction
  });
});


sequelize.transaction(transaction => {
  User.create({
    username: 'someguy',
    mood: 'happy',
    transaction
  });
});
```

如果在前面的代码中对`User.update`的调用中未包含事务选项，则不会发生任何更改，因为在事务提交之前，数据库中不存在我们新创建的用户。



#### 内部事务

要知到sequelize会利用“内部事务”进行某些操作，如：`Model.findOrCreate`。如果钩子函数执行依赖于数据库中对象存在的读取或写入操作，或者像上节中的示例一样修改对象的存储值，则应始终指定`{ transaction: options.transaction }`。

如果钩子已经在事务操作过程中被调用，这样可以确保所依赖读/写的是同一个事务的一部分。如果未处理该钩子，只需指定`{ transaction: null }`，即可按默认方式执行。



## 8. 查询(Querying)

- [属性](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-attributes)
- `Where`
  - [基础](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-basics)
  - 操作符
    - [范围操作符](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-range-operators)
    - [组合](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-combinations)
    - [操作符别名](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-operators-aliases)
    - [操作符安全性](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-operators-security)
  - JSON
    - [PostgreSQL](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-postgresql)
    - [MSSQL](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-mssql)
  - JSONB
    - [嵌套对象](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-nested-object)
    - [嵌套键](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-nested-key)
    - [包含](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-containment)
  - [关系/关联](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-relations-associations)
- [分页/限制](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-pagination-limiting)
- [排序](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-ordering)
- [表提示](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-table-hint)
- [索引提示](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-index-hints)

### 属性

仅查询部分属性（字段）时，可以通过`attributes`选项来指定。大多数情况下，你可以传入一个数组：

```
Model.findAll({
  attributes: ['foo', 'bar']
});
SELECT foo, bar ...
```

属性可以通过嵌套数组的方式来重命名：

```
Model.findAll({
  attributes: ['foo', ['bar', 'baz']]
});
SELECT foo, bar AS baz ...
```

可以通过`sequelize.fn`来进行聚合：

```
Model.findAll({
  attributes: [[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']]
});
SELECT COUNT(hats) AS no_hats ...
```

使用聚合函数时，必须指定一个别名，以便它能够从模型中访问它。在上面的示例中，可以通过instance.get('no_hats')获得帽子（hats）数量。

有时，如果只想添加聚合，而列出模型的所有属性可能比较烦人：

```
// This is a tiresome way of getting the number of hats...
Model.findAll({
  attributes: ['id', 'foo', 'bar', 'baz', 'quz', [sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']]
});

// This is shorter, and less error prone because it still works if you add / remove attributes
Model.findAll({
  attributes: { include: [[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']] }
});
SELECT id, foo, bar, baz, quz, COUNT(hats) AS no_hats ...
```

类似的，可以通过`exclude`排除一些不需要的属性：

```
Model.findAll({
  attributes: { exclude: ['baz'] }
});
SELECT id, foo, bar, quz ...
```



### `Where`

无论你是使用`findAll/find`查询还是进行批量`update/destroy`，都可以传递`where`对象来过滤查询。

`where`通常会从`attribute:value`对中获取一个对象，其中`value`可以是相等匹配的原语或其他运算符的键对象。

也可以通过嵌套`or`和`and`操作符的集合来生成复杂的`AND`/`OR`条件。



#### 基础查询

```
const Op = Sequelize.Op;

Post.findAll({
  where: {
    authorId: 2
  }
});
// SELECT * FROM post WHERE authorId = 2

Post.findAll({
  where: {
    authorId: 12,
    status: 'active'
  }
});
// SELECT * FROM post WHERE authorId = 12 AND status = 'active';

Post.findAll({
  where: {
    [Op.or]: [{authorId: 12}, {authorId: 13}]
  }
});
// SELECT * FROM post WHERE authorId = 12 OR authorId = 13;

Post.findAll({
  where: {
    authorId: {
      [Op.or]: [12, 13]
    }
  }
});
// SELECT * FROM post WHERE authorId = 12 OR authorId = 13;

Post.destroy({
  where: {
    status: 'inactive'
  }
});
// DELETE FROM post WHERE status = 'inactive';

Post.update({
  updatedAt: null,
}, {
  where: {
    deletedAt: {
      [Op.ne]: null
    }
  }
});
// UPDATE post SET updatedAt = null WHERE deletedAt NOT NULL;

Post.findAll({
  where: sequelize.where(sequelize.fn('char_length', sequelize.col('status')), 6)
});
// SELECT * FROM post WHERE char_length(status) = 6;
```



#### 操作符（`Operator`）

Sequelize公开了可用于创建更复杂比较的符号运算符：

```
const Op = Sequelize.Op

[Op.and]: [{a: 5}, {b: 6}] // (a = 5) AND (b = 6)
[Op.or]: [{a: 5}, {a: 6}]  // (a = 5 OR a = 6)
[Op.gt]: 6,                // > 6
[Op.gte]: 6,               // >= 6
[Op.lt]: 10,               // < 10
[Op.lte]: 10,              // <= 10
[Op.ne]: 20,               // != 20
[Op.eq]: 3,                // = 3
[Op.is]: null              // IS NULL
[Op.not]: true,            // IS NOT TRUE
[Op.between]: [6, 10],     // BETWEEN 6 AND 10
[Op.notBetween]: [11, 15], // NOT BETWEEN 11 AND 15
[Op.in]: [1, 2],           // IN [1, 2]
[Op.notIn]: [1, 2],        // NOT IN [1, 2]
[Op.like]: '%hat',         // LIKE '%hat'
[Op.notLike]: '%hat'       // NOT LIKE '%hat'
[Op.iLike]: '%hat'         // ILIKE '%hat' (case insensitive) (PG only)
[Op.notILike]: '%hat'      // NOT ILIKE '%hat'  (PG only)
[Op.startsWith]: 'hat'     // LIKE 'hat%'
[Op.endsWith]: 'hat'       // LIKE '%hat'
[Op.substring]: 'hat'      // LIKE '%hat%'
[Op.regexp]: '^[h|a|t]'    // REGEXP/~ '^[h|a|t]' (MySQL/PG only)
[Op.notRegexp]: '^[h|a|t]' // NOT REGEXP/!~ '^[h|a|t]' (MySQL/PG only)
[Op.iRegexp]: '^[h|a|t]'    // ~* '^[h|a|t]' (PG only)
[Op.notIRegexp]: '^[h|a|t]' // !~* '^[h|a|t]' (PG only)
[Op.like]: { [Op.any]: ['cat', 'hat']}
                           // LIKE ANY ARRAY['cat', 'hat'] - also works for iLike and notLike
[Op.overlap]: [1, 2]       // && [1, 2] (PG array overlap operator)
[Op.contains]: [1, 2]      // @> [1, 2] (PG array contains operator)
[Op.contained]: [1, 2]     // <@ [1, 2] (PG array contained by operator)
[Op.any]: [2,3]            // ANY ARRAY[2, 3]::INTEGER (PG only)

[Op.col]: 'user.organization_id' // = "user"."organization_id", with dialect specific column identifiers, PG in this example
[Op.gt]: { [Op.all]: literal('SELECT 1') }
                          // > ALL (SELECT 1)
```



##### 范围操作符

可以使用所有支持的运算符查询范围类型。请记住，提供的范围值也可以定义绑定的inclusion/exclusion。

```
// All the above equality and inequality operators plus the following:

[Op.contains]: 2           // @> '2'::integer (PG range contains element operator)
[Op.contains]: [1, 2]      // @> [1, 2) (PG range contains range operator)
[Op.contained]: [1, 2]     // <@ [1, 2) (PG range is contained by operator)
[Op.overlap]: [1, 2]       // && [1, 2) (PG range overlap (have points in common) operator)
[Op.adjacent]: [1, 2]      // -|- [1, 2) (PG range is adjacent to operator)
[Op.strictLeft]: [1, 2]    // << [1, 2) (PG range strictly left of operator)
[Op.strictRight]: [1, 2]   // >> [1, 2) (PG range strictly right of operator)
[Op.noExtendRight]: [1, 2] // &< [1, 2) (PG range does not extend to the right of operator)
[Op.noExtendLeft]: [1, 2]  // &> [1, 2) (PG range does not extend to the left of operator)
```



##### 组合

```
const Op = Sequelize.Op;

{
  rank: {
    [Op.or]: {
      [Op.lt]: 1000,
      [Op.eq]: null
    }
  }
}
// rank < 1000 OR rank IS NULL

{
  createdAt: {
    [Op.lt]: new Date(),
    [Op.gt]: new Date(new Date() - 24 * 60 * 60 * 1000)
  }
}
// createdAt < [timestamp] AND createdAt > [timestamp]

{
  [Op.or]: [
    {
      title: {
        [Op.like]: 'Boat%'
      }
    },
    {
      description: {
        [Op.like]: '%boat%'
      }
    }
  ]
}
// title LIKE 'Boat%' OR description LIKE '%boat%'
```



##### 操作符别名

Sequelize允许将特定的字符串设置为运算符的别名。在v5中，这会给出弃用警告。

```
const Op = Sequelize.Op;
const operatorsAliases = {
  $gt: Op.gt
}
const connection = new Sequelize(db, user, pass, { operatorsAliases })

[Op.gt]: 6 // > 6
$gt: 6 // same as using Op.gt (> 6)
```



##### 操作符安全性

默认情况下，Sequelize将使用符号运算符。使用不带任何别名的Sequelize可提高安全性。没有任何字符串别名将使注入操作符的可能性极小，但应始终正确验证和清除用户输入。

某些框架会自动将用户输入解析为js对象，如果不清理输入内容，则可能可以使用带有字符串运算符的Object进行Sequelize注入。

为了提高安全性，强烈建议在代码中使用`Sequelize.Op`中的符号运算符，如：`Op.and`/`Op.or`，而不要完全依赖任何基于字符串的运算符，如：`$and`/`$or`。可以通过设置`operatorsAliases`选项来限制应用程序所需的别名，尤其是当直接将用户输入传递给Sequelize方法时（切记要清理用户输入）。

```
const Op = Sequelize.Op;

//use sequelize without any operators aliases
const connection = new Sequelize(db, user, pass, { operatorsAliases: false });

//use sequelize with only alias for $and => Op.and
const connection2 = new Sequelize(db, user, pass, { operatorsAliases: { $and: Op.and } });
```

如果要使用默认别名，则Sequelize会警告，如果想继续使用所有默认别名（不包括旧别名）而没有警告，可以通过下面的`operatorsAliases`选项-

```
const Op = Sequelize.Op;
const operatorsAliases = {
  $eq: Op.eq,
  $ne: Op.ne,
  $gte: Op.gte,
  $gt: Op.gt,
  $lte: Op.lte,
  $lt: Op.lt,
  $not: Op.not,
  $in: Op.in,
  $notIn: Op.notIn,
  $is: Op.is,
  $like: Op.like,
  $notLike: Op.notLike,
  $iLike: Op.iLike,
  $notILike: Op.notILike,
  $regexp: Op.regexp,
  $notRegexp: Op.notRegexp,
  $iRegexp: Op.iRegexp,
  $notIRegexp: Op.notIRegexp,
  $between: Op.between,
  $notBetween: Op.notBetween,
  $overlap: Op.overlap,
  $contains: Op.contains,
  $contained: Op.contained,
  $adjacent: Op.adjacent,
  $strictLeft: Op.strictLeft,
  $strictRight: Op.strictRight,
  $noExtendRight: Op.noExtendRight,
  $noExtendLeft: Op.noExtendLeft,
  $and: Op.and,
  $or: Op.or,
  $any: Op.any,
  $all: Op.all,
  $values: Op.values,
  $col: Op.col
};

const connection = new Sequelize(db, user, pass, { operatorsAliases });
```



#### JSON

`JSON`数据类型可以支持被PostgreSQL、SQLite、MySQL 和 MariaDB 支持。



##### PostgreSQL

在PostgreSQL中，JSON数据类型将值存储为纯文本，而不是二进制表示。如果你只想存储和检索JSON表示形式，使用JSON将占用更少的磁盘空间，并需要更少的时间从其输入表示形式进行构建。但是，如果要对JSON值执行任何操作，则应首选以下所述的JSONB数据类型。



##### MSSQL

MSSQL没有JSON数据类型，但是自SQL Server 2016开始，它通过某些函数提供了对以字符串形式存储的JSON的支持。使用这些函数，就能够查询存储在字符串中的JSON，但是需要对任何返回的值 分别解析。

```
// ISJSON - to test if a string contains valid JSON
User.findAll({
  where: sequelize.where(sequelize.fn('ISJSON', sequelize.col('userDetails')), 1)
})

// JSON_VALUE - extract a scalar value from a JSON string
User.findAll({
  attributes: [[ sequelize.fn('JSON_VALUE', sequelize.col('userDetails'), '$.address.Line1'), 'address line 1']]
})

// JSON_VALUE - query a scalar value from a JSON string
User.findAll({
  where: sequelize.where(sequelize.fn('JSON_VALUE', sequelize.col('userDetails'), '$.address.Line1'), '14, Foo Street')
})

// JSON_QUERY - extract an object or array
User.findAll({
  attributes: [[ sequelize.fn('JSON_QUERY', sequelize.col('userDetails'), '$.address'), 'full address']]
```



#### JSONB

可以通过三种方式查询JSONB。

##### 嵌套对象

```
{
  meta: {
    video: {
      url: {
        [Op.ne]: null
      }
    }
  }
}
```

##### 嵌套键

```
{
  "meta.audio.length": {
    [Op.gt]: 20
  }
}
```

##### 包含

```
{
  "meta": {
    [Op.contains]: {
      site: {
        url: 'http://google.com'
      }
    }
  }
}
```



#### 关系/关联

```
// Find all projects with a least one task where task.state === project.state
Project.findAll({
    include: [{
        model: Task,
        where: { state: Sequelize.col('project.state') }
    }]
})
```



### 分页/限制

```
// Fetch 10 instances/rows
Project.findAll({ limit: 10 })

// Skip 8 instances/rows
Project.findAll({ offset: 8 })

// Skip 5 instances and fetch the 5 after that
Project.findAll({ offset: 5, limit: 5 })
```



### 排序

`order`通过数组或sequelize方法来对查询进行排序。通常，将需要使用属性、排序方向或仅排序方向的元组/数组，以确保能够正确转义。

```
Subtask.findAll({
  order: [
    // Will escape title and validate DESC against a list of valid direction parameters
    ['title', 'DESC'],

    // Will order by max(age)
    sequelize.fn('max', sequelize.col('age')),

    // Will order by max(age) DESC
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],

    // Will order by  otherfunction(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],

    // Will order an associated model's created_at using the model name as the association's name.
    [Task, 'createdAt', 'DESC'],

    // Will order through an associated model's created_at using the model names as the associations' names.
    [Task, Project, 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using the name of the association.
    ['Task', 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at using the names of the associations.
    ['Task', 'Project', 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using an association object. (preferred method)
    [Subtask.associations.Task, 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at using association objects. (preferred method)
    [Subtask.associations.Task, Task.associations.Project, 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using a simple association object.
    [{model: Task, as: 'Task'}, 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at simple association objects.
    [{model: Task, as: 'Task'}, {model: Project, as: 'Project'}, 'createdAt', 'DESC']
  ]

  // Will order by max age descending
  order: sequelize.literal('max(age) DESC')

  // Will order by max age ascending assuming ascending is the default order when direction is omitted
  order: sequelize.fn('max', sequelize.col('age'))

  // Will order by age ascending assuming ascending is the default order when direction is omitted
  order: sequelize.col('age')

  // Will order randomly based on the dialect (instead of fn('RAND') or fn('RANDOM'))
  order: sequelize.random()
})
```



### 表提示

使用mssql时，可以使用`tableHint`选择性地传递表提示。提示必须是`Sequelize.TableHints`中的值，并且仅在绝对必要时才使用。 当前每个查询仅支持单个表提示。

表提示通过指定某些选项来覆盖mssql查询优化器的默认行为。它们仅影响该子句中引用的表或视图。

```
const TableHints = Sequelize.TableHints;

Project.findAll({
  // adding the table hint NOLOCK
  tableHint: TableHints.NOLOCK
  // this will generate the SQL 'WITH (NOLOCK)'
})
```



### 索引提示

使用MySQL时，可以使用`indexHints`选择性地传递索引提示。提示类型必须是`Sequelize.IndexHints`中的值，并且这些值应引用现有索引。

索引提示将覆盖[mysql查询优化器的默认行为](https://dev.mysql.com/doc/refman/5.7/en/index-hints.html)。

```
Project.findAll({
  indexHints: [
    { type: IndexHints.USE, values: ['index_project_on_name'] }
  ],
  where: {
    id: {
      [Op.gt]: 623
    },
    name: {
      [Op.like]: 'Foo %'
    }
  }
})
```

会生成类似如下的mysql查询：

```
SELECT * FROM Project USE INDEX (index_project_on_name) WHERE name LIKE 'FOO %' AND id > 623;
```

`Sequelize.IndexHints`包括`USE`、`FORCE`和`IGNORE`。



## 9. 实例(Instances)

- [构建非持久化的实例](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#building-a-non-persistent-instance)
- [创建持久化实例](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#creating-persistent-instances)
- [更新 / 保存 / 持久化实例](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#updating---saving---persisting-an-instance)
- [销毁 / 删除 持久化实例](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#destroying---deleting-persistent-instances)
- [恢复软删除的实例](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#restoring-soft-deleted-instances)
- [批量操作 (一次性创建、更新和销毁多行)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#working-in-bulk--creating--updating-and-destroying-multiple-rows-at-once-)
- [实例值](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#values-of-an-instance)
- [重新加载实例](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#reloading-instances)
- [递增](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#incrementing)
- [递减](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#decrementing)

### 构建非持久化的实例

为了创建已定义类的实例，执行以下操作即可。如果你过去编写过Ruby，可能就会认识该语法。使用构建方法将返回未保存的对象，必须明确保存该对象。

```
const project = Project.build({
  title: 'my awesome project',
  description: 'woot woot. this will make me a rich man'
})

const task = Task.build({
  title: 'specify the project idea',
  description: 'bla',
  deadline: new Date()
})
```

构建的内置实例将自动获得默认值：

```
// first define the model
class Task extends Model {}
Task.init({
  title: Sequelize.STRING,
  rating: { type: Sequelize.TINYINT, defaultValue: 3 }
}, { sequelize, modelName: 'task' });

// now instantiate an object
const task = Task.build({title: 'very important task'})

task.title  // ==> 'very important task'
task.rating // ==> 3
```

要将其存储到数据库中，使用`save`方法并捕获事件（如果需要）：

```
project.save().then(() => {
  // my nice callback stuff
})

task.save().catch(error => {
  // mhhh, wth!
})

// you can also build, save and access the object with chaining:
Task
  .build({ title: 'foo', description: 'bar', deadline: new Date() })
  .save()
  .then(anotherTask => {
    // you can now access the currently saved task with the variable anotherTask... nice!
  })
  .catch(error => {
    // Ooops, do some error-handling
  })
```



### 创建持久化实例

使用`.build()`创建的实例需要显式调用`.save()`存储到数据库中，而`.create()`可以忽略了这一要求，并在调用后自动存储实例的数据。

```
Task.create({ title: 'foo', description: 'bar', deadline: new Date() }).then(task => {
  // you can now access the newly created task via the variable task
})
```

也可以定义可以通过create方法设置的属性。如果根据用户填写的表单创建数据库条目，这会非常方便。例如，使用该名称将允许你限制User模型，使其仅设置用户名和地址，而不设置admin标志：

```
User.create({ username: 'barfooz', isAdmin: true }, { fields: [ 'username' ] }).then(user => {
  // let's assume the default of isAdmin is false:
  console.log(user.get({
    plain: true
  })) // => { username: 'barfooz', isAdmin: false }
})
```



### 更新 / 保存 / 持久化实例

对实例值进行修改，并将修改保存到数据库中。有两种方法可以实现：

```
// way 1
task.title = 'a very different title now'
task.save().then(() => {})

// way 2
task.update({
  title: 'a very different title now'
}).then(() => {})
```

还可以通过传递一个列名数组来定义在调用`save`时应保存哪些属性。当你基于先前定义的对象设置属性时，这会很有用。例如。 你通过网络应用的形式获取对象的值。此外，它用于内部的`update`。它是这样的：

```
task.title = 'foooo'
task.description = 'baaaaaar'
task.save({fields: ['title']}).then(() => {
 // title will now be 'foooo' but description is the very same as before
})

// The equivalent call using update looks like this:
task.update({ title: 'foooo', description: 'baaaaaar'}, {fields: ['title']}).then(() => {
 // title will now be 'foooo' but description is the very same as before
})
```

当你调用`save`而不修改任何属性，该方法不会做任务操作。



### 销毁 / 删除 持久化实例

一旦创建对象并获取对其引用，就可以将其从数据库删除。删除操作通过调用`destroy`方法：

```
Task.create({ title: 'a task' }).then(task => {
  // now you see me...
  return task.destroy();
}).then(() => {
 // now i'm gone :)
})
```

如果设置`paranoid`选择为`true`，记录将不会被删除，而是将`deletedAt`列设置为当前时间戳。要强制删除，可以将`force: true`传给`destroy`方法：

```
task.destroy({ force: true })
```

当在`paranoid`模式下对对象软删除后，将不能再创建有相同主键的新对象，除非强制删除旧实例后。



### 恢复软删除的实例

将设置为`paranoid: true`的实例软删除后，如果要撤销删除，则使用`restore`方法：

```
Task.create({ title: 'a task' }).then(task => {
  // now you see me...
  return task.destroy();
}).then((task) => {
  // now i'm gone, but wait...
  return task.restore();
})
```



### 批量操作 (一次性创建、更新和销毁多行)

除了更新单个实例之外，还可以一次创建、更新和删除多个实例。这些函数为：

- `Model.bulkCreate`
- `Model.update`
- `Model.destroy`

由于正在使用多个模型，所以回调将不会返回DAO实例。BulkCreate会返回模型实例/DAO的数组。但与`create`不同，它们没有`autoIncrement`属性的结果值。`update`和`destroy`会返回受影响的行数。

首先，我们看一个`bulkCreate`：

```
User.bulkCreate([
  { username: 'barfooz', isAdmin: true },
  { username: 'foo', isAdmin: true },
  { username: 'bar', isAdmin: false }
]).then(() => { // Notice: There are no arguments here, as of right now you'll have to...
  return User.findAll();
}).then(users => {
  console.log(users) // ... in order to get the array of user objects
})
```

插入多行并返回所有列（仅Postgres）：

```
User.bulkCreate([
  { username: 'barfooz', isAdmin: true },
  { username: 'foo', isAdmin: true },
  { username: 'bar', isAdmin: false }
], { returning: true }) // will return all columns for each row inserted
.then((result) => {
  console.log(result);
});
```

插入多行并返回指定列（仅Postgres）：

```
User.bulkCreate([
  { username: 'barfooz', isAdmin: true },
  { username: 'foo', isAdmin: true },
  { username: 'bar', isAdmin: false }
], { returning: ['username'] }) // will return only the specified columns for each row inserted
.then((result) => {
  console.log(result);
});
```

一次更新多行：

```
Task.bulkCreate([
  {subject: 'programming', status: 'executing'},
  {subject: 'reading', status: 'executing'},
  {subject: 'programming', status: 'finished'}
]).then(() => {
  return Task.update(
    { status: 'inactive' }, /* set attributes' value */
    { where: { subject: 'programming' }} /* where criteria */
  );
}).then(([affectedCount, affectedRows]) => {
  // Notice that affectedRows will only be defined in dialects which support returning: true

  // affectedCount will be 2
  return Task.findAll();
}).then(tasks => {
  console.log(tasks) // the 'programming' tasks will both have a status of 'inactive'
})
```

然后删除它们：

```
Task.bulkCreate([
  {subject: 'programming', status: 'executing'},
  {subject: 'reading', status: 'executing'},
  {subject: 'programming', status: 'finished'}
]).then(() => {
  return Task.destroy({
    where: {
      subject: 'programming'
    },
    truncate: true /* this will ignore where and truncate the table instead */
  });
}).then(affectedRows => {
  // affectedRows will be 2
  return Task.findAll();
}).then(tasks => {
  console.log(tasks) // no programming, just reading :(
})
```

如果直接从user接受值，那么可能会需要限制插入的列。`bulkCreate()`接受`options`对象做为第二个参数，该对象可以有一个`fields`参数（一个数组），以使其知道要显示构建的字段：

```
User.bulkCreate([
  { username: 'foo' },
  { username: 'bar', admin: true}
], { fields: ['username'] }).then(() => {
  // nope bar, you can't be admin!
})
```

`bulkCreate`最初是一种主流/快速的插入记录方式，但有时候即使明确告诉Sequelize要筛选的列，你也希望能够一次插入多行而不牺牲模型验证。这时可能向`options`对象添加一个`validate:true`属性来实现：

```
class Tasks extends Model {}
Tasks.init({
  name: {
    type: Sequelize.STRING,
    validate: {
      notNull: { args: true, msg: 'name cannot be null' }
    }
  },
  code: {
    type: Sequelize.STRING,
    validate: {
      len: [3, 10]
    }
  }
}, { sequelize, modelName: 'tasks' })

Tasks.bulkCreate([
  {name: 'foo', code: '123'},
  {code: '1234'},
  {name: 'bar', code: '1'}
], { validate: true }).catch(errors => {
  /* console.log(errors) would look like:
  [
    { record:
    ...
    name: 'SequelizeBulkRecordError',
    message: 'Validation error',
    errors:
      { name: 'SequelizeValidationError',
        message: 'Validation error',
        errors: [Object] } },
    { record:
      ...
      name: 'SequelizeBulkRecordError',
      message: 'Validation error',
      errors:
        { name: 'SequelizeValidationError',
        message: 'Validation error',
        errors: [Object] } }
  ]
  */
})
```



### 实例值

如果日志打印一个实例，你会注意到实例还有很多其它的东西。为了隐藏此类内容，会将其简化为非常有趣的信息，可以使用`get`-属性获取值。使用`plain=true`选项时，会只返回实例的值。

```
Person.create({
  name: 'Rambow',
  firstname: 'John'
}).then(john => {
  console.log(john.get({
    plain: true
  }))
})

// result:

// { name: 'Rambow',
//   firstname: 'John',
//   id: 1,
//   createdAt: Tue, 01 May 2012 19:12:16 GMT,
//   updatedAt: Tue, 01 May 2012 19:12:16 GMT
// }
```

提示：还可以使用`JSON.stringify(instance)`将实例转换为JSON。这基本上将返回与`values`完全相同的值。



### 重新加载实例

如果需要同步实例，可以使用`reload`方法。它会将从数据库中获取当前数据，并覆盖调用了该方法的模型的属性。

```
Person.findOne({ where: { name: 'john' } }).then(person => {
  person.name = 'jane'
  console.log(person.name) // 'jane'

  person.reload().then(() => {
    console.log(person.name) // 'john'
  })
})
```



### 递增

为了增加实例的值而不会遇到并发问题，可以使用`increment`方法。

可以定义一个字段和要添加到该字段的值：

```
User.findByPk(1).then(user => {
  return user.increment('my-integer-field', {by: 2})
}).then(user => {
  // Postgres will return the updated user by default (unless disabled by setting { returning: false })
  // In other dialects, you'll want to call user.reload() to get the updated instance...
})
```

也可以对多个字段进行更新：

```
User.findByPk(1).then(user => {
  return user.increment([ 'my-integer-field', 'my-very-other-field' ], {by: 2})
}).then(/* ... */)
```

还可以定义一个包含字段和增量的对象：

```
User.findByPk(1).then(user => {
  return user.increment({
    'my-integer-field':    2,
    'my-very-other-field': 3
  })
}).then(/* ... */)
```



### 递减

为了减少实例的值而不会遇到并发问题，可以使用`decrementing`方法。

与`increment`一样，减少实例同样有三种方式：

首先，可以定义单个字段及其要减少的值：

```
User.findByPk(1).then(user => {
  return user.decrement('my-integer-field', {by: 2})
}).then(user => {
  // Postgres will return the updated user by default (unless disabled by setting { returning: false })
  // In other dialects, you'll want to call user.reload() to get the updated instance...
})
```

其次，可以定义多个字段及其要减少的值：

```
User.findByPk(1).then(user => {
  return user.decrement([ 'my-integer-field', 'my-very-other-field' ], {by: 2})
}).then(/* ... */)
```

最后，可以定义一个包含字段及其要减少值的对象：

```
User.findByPk(1).then(user => {
  return user.decrement({
    'my-integer-field':    2,
    'my-very-other-field': 3
  })
}).then(/* ... */)
```



## 10. 关联\关系(Associations)

本节描述了Sequelize中的各种关联类型，Sequelize中有四种可用的关联类型：

1. `BelongsTo`
2. `HasOne`
3. `HasMany`
4. `BelongsToMany`

- 基本概念
  - [`Source` & `Target`](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#source--amp--target)
  - 外键
    - [`underscored`选项](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#underscored-option)
    - [循环依赖& 禁用约束](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#cyclic-dependencies--amp--disabling-constraints)
    - [没有约束的情况下强制执行外键引用](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#enforcing-a-foreign-key-reference-without-constraints)
- `一对一`关联
  - `BelongsTo`
    - [外键](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#one-to-one-foreign-keys)
    - [目标键(Target key)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#one-to-one-target-keys)
  - `HasOne`
    - [源键(Source key)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#one-to-one-source-keys)
  - [`HasOne`和`BelongsTo`之间的不同](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#difference-between-hasone-and-belongsto)
- [`一对多`关联 (`hasMany`)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#one-to-many-associations--hasmany-)
- [`多对多`关联 (`BelongsToMany`)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#belongs-to-many-associations)
- [源键和目标键](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#source-and-target-keys)
- [命名策略](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#naming-strategy)
- [关联对象](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#associating-objects)
- [检查关联](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#check-associations)
- 高级概念
  - 范围/作用域
    - [`1:n`](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#1-n)
    - [`n:m`](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#n-m)
  - 创建关联
    - [`BelongsTo` / `HasMany` / `HasOne`关联](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#belongsto---hasmany---hasone-association)
    - [用别名进行`BelongsTo`关联](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#belongsto-association-with-an-alias)
    - [`hasMany` / `BelongsToMany`关联](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#hasmany---belongstomany-association)

### 基本概念

#### `Source` & `Target`

在大多数关联中，你都会看到**source**(“源”)和**targe**(“目标”)模型。假如你正在尝试在两个模型之间添加关联，如下，我们在`User`和`Project`之间添加了`hasOne`关联。

```
class User extends Model {}
User.init({
  name: Sequelize.STRING,
  email: Sequelize.STRING
}, {
  sequelize,
  modelName: 'user'
});

class Project extends Model {}
Project.init({
  name: Sequelize.STRING
}, {
  sequelize,
  modelName: 'project'
});

User.hasOne(Project);
```

在上例中，`User`模型（在其上调用函数的模型）是**source**，`Project`模型（做为参数传入的模型）是**target**。



#### 外键(Foreign Key)

在Sequelize模型之间创建关联时，将自动创建具有约束的外键引用。对于以下设置：

```
class Task extends Model {}
Task.init({ title: Sequelize.STRING }, { sequelize, modelName: 'task' });
class User extends Model {}
User.init({ username: Sequelize.STRING }, { sequelize, modelName: 'user' });

User.hasMany(Task); // Will add userId to Task model
Task.belongsTo(User); // Will also add userId to Task model
```

会生成以下SQL：

```
CREATE TABLE IF NOT EXISTS "users" (
  "id" SERIAL,
  "username" VARCHAR(255),
  "createdAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  PRIMARY KEY ("id")
);

CREATE TABLE IF NOT EXISTS "tasks" (
  "id" SERIAL,
  "title" VARCHAR(255),
  "createdAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "userId" INTEGER REFERENCES "users" ("id") ON DELETE
  SET
    NULL ON UPDATE CASCADE,
    PRIMARY KEY ("id")
);
```

`tasks`和`users`之间的关系会将`userId`外键插入到`tasks`表，并将其标记为对`users`表的引用。默认情况下，如果删除了所引用的`user`，`userId`会被设置为`NULL`；用户ID更新后，`userId`也将更新。将`onUpdate`和`onDelete`选项传递给关联调用，可以覆盖这些选项。验证选项为：`RESTRICT`、`CASCADE`、`NO ACTION`、`SET DEFAULT`、`SET NULL`。

对于`1:1`和`1:m`关联，删除时的默认选项是`SET NULL`，更新则是`CASCADE`。对于`n:m`关联，两者都默认设置为`CASCADE`。这意味着，如果你从`n:m`关联的一侧删除或更新一行，则联接表中引用该行的所有行也将被删除或更新。



##### `underscored`选项

Sequelize允许对模型设置`underscored`选项。如果为`true`，则此选项会将所有属性上的`field`选项设置为其名称的下划线版本。这也适用于关联生成的外键。

以下是一个修改`underscored`选项的示例：

```
class Task extends Model {}
Task.init({
  title: Sequelize.STRING
}, {
  underscored: true,
  sequelize,
  modelName: 'task'
});

class User extends Model {}
User.init({
  username: Sequelize.STRING
}, {
  underscored: true,
  sequelize,
  modelName: 'user'
});

// Will add userId to Task model, but field will be set to `user_id`
// This means column name will be `user_id`
User.hasMany(Task);

// Will also add userId to Task model, but field will be set to `user_id`
// This means column name will be `user_id`
Task.belongsTo(User);
```

会生成以下SQL：

```
CREATE TABLE IF NOT EXISTS "users" (
  "id" SERIAL,
  "username" VARCHAR(255),
  "created_at" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updated_at" TIMESTAMP WITH TIME ZONE NOT NULL,
  PRIMARY KEY ("id")
);

CREATE TABLE IF NOT EXISTS "tasks" (
  "id" SERIAL,
  "title" VARCHAR(255),
  "created_at" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updated_at" TIMESTAMP WITH TIME ZONE NOT NULL,
  "user_id" INTEGER REFERENCES "users" ("id") ON DELETE
  SET
    NULL ON UPDATE CASCADE,
    PRIMARY KEY ("id")
);
```

带有下划线选项的属性仍被插入到驼峰模型中，但`field`选项已设置为其下划线版本。



##### 循环依赖& 禁用约束

在表之间添加约束意味着使用`sequelize.sync`时，必须按一定顺序在数据库中创建表。如果`Task`引用了`User`，则必须先创建`users`表示，然后才能创建`tasks`表。有时这会导致循环引用，然后Sequelize找不到同步的顺序。想象一下文档和版本的情况。 一个文档可以有多个版本，为方便起见，文档引用了其当前版本。

```
class Document extends Model {}
Document.init({
  author: Sequelize.STRING
}, { sequelize, modelName: 'document' });
class Version extends Model {}
Version.init({
  timestamp: Sequelize.DATE
}, { sequelize, modelName: 'version' });

Document.hasMany(Version); // This adds documentId attribute to version
Document.belongsTo(Version, {
  as: 'Current',
  foreignKey: 'currentVersionId'
}); // This adds currentVersionId attribute to document
```

以上代码会有这个错误：`Cyclic dependency found. documents is dependent of itself. Dependency chain: documents -> versions => documents`。

要解决这一问题，可以在其中一个关系中设置`constraints: false`：

```
Document.hasMany(Version);
Document.belongsTo(Version, {
  as: 'Current',
  foreignKey: 'currentVersionId',
  constraints: false
});
```

这样，就可以同步表了：

```
CREATE TABLE IF NOT EXISTS "documents" (
  "id" SERIAL,
  "author" VARCHAR(255),
  "createdAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "currentVersionId" INTEGER,
  PRIMARY KEY ("id")
);

CREATE TABLE IF NOT EXISTS "versions" (
  "id" SERIAL,
  "timestamp" TIMESTAMP WITH TIME ZONE,
  "createdAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "documentId" INTEGER REFERENCES "documents" ("id") ON DELETE
  SET
    NULL ON UPDATE CASCADE,
    PRIMARY KEY ("id")
);
```



##### 没有约束的情况下强制执行外键引用

有时你可能想引用另一个表，而不添加任何约束或关联。在这种情况下，可以将引用属性手动添加到架构定义中，并标记它们之间的关系。

```
class Trainer extends Model {}
Trainer.init({
  firstName: Sequelize.STRING,
  lastName: Sequelize.STRING
}, { sequelize, modelName: 'trainer' });

// Series will have a trainerId = Trainer.id foreign reference key
// after we call Trainer.hasMany(series)
class Series extends Model {}
Series.init({
  title: Sequelize.STRING,
  subTitle: Sequelize.STRING,
  description: Sequelize.TEXT,
  // Set FK relationship (hasMany) with `Trainer`
  trainerId: {
    type: Sequelize.INTEGER,
    references: {
      model: Trainer,
      key: 'id'
    }
  }
}, { sequelize, modelName: 'series' });

// Video will have seriesId = Series.id foreign reference key
// after we call Series.hasOne(Video)
class Video extends Model {}
Video.init({
  title: Sequelize.STRING,
  sequence: Sequelize.INTEGER,
  description: Sequelize.TEXT,
  // set relationship (hasOne) with `Series`
  seriesId: {
    type: Sequelize.INTEGER,
    references: {
      model: Series, // Can be both a string representing the table name or a Sequelize model
      key: 'id'
    }
  }
}, { sequelize, modelName: 'video' });

Series.hasOne(Video);
Trainer.hasMany(Series);
```



### `一对一`关联

一对一关联是通过单个外键连接的两个模型之间的关联。

#### `BelongsTo`

BelongsTo关联是**源模型**上存在一对一关系的外键的关联。

一个简单的示例，就是一个`Player`是`Team`的一部分，并且外键在`player`上。

```
class Player extends Model {}
Player.init({/* attributes */}, { sequelize, modelName: 'player' });
class Team extends Model {}
Team.init({/* attributes */}, { sequelize, modelName: 'team' });

Player.belongsTo(Team); // Will add a teamId attribute to Player to hold the primary key value for Team
```

##### 外键(Foreign key)

默认情况下，将根据目标模型名称和目标主键名称来生成`belongsTo`关系的外键。

默认大小写为`camelCase`。如果源模型配置有`underscored: true`，则将使用字段`snake_case`创建foreignKey。

```
class User extends Model {}
User.init({/* attributes */}, { sequelize, modelName: 'user' })
class Company extends Model {}
Company.init({/* attributes */}, { sequelize, modelName: 'company' });

// will add companyId to user
User.belongsTo(Company);

class User extends Model {}
User.init({/* attributes */}, { underscored: true, sequelize, modelName: 'user' })
class Company extends Model {}
Company.init({
  uuid: {
    type: Sequelize.UUID,
    primaryKey: true
  }
}, { sequelize, modelName: 'company' });

// will add companyUuid to user with field company_uuid
User.belongsTo(Company);
```

如果定义了`as`，它将代替目标模型名称。

```
class User extends Model {}
User.init({/* attributes */}, { sequelize, modelName: 'user' })
class UserRole extends Model {}
UserRole.init({/* attributes */}, { sequelize, modelName: 'userRole' });

User.belongsTo(UserRole, {as: 'role'}); // Adds roleId to user rather than userRoleId
In all cases the default foreign key can be overwritten with the foreignKey option. When the foreign key option is used, Sequelize will use it as-is:

class User extends Model {}
User.init({/* attributes */}, { sequelize, modelName: 'user' })
class Company extends Model {}
Company.init({/* attributes */}, { sequelize, modelName: 'company' });

User.belongsTo(Company, {foreignKey: 'fk_company'}); // Adds fk_company to User
```

在所有情况下，都可以使用`foreignKey`选项覆盖默认外键。 使用外键选项时，Sequelize将按原样使用它：

```
class User extends Model {}
User.init({/* attributes */}, { sequelize, modelName: 'user' })
class Company extends Model {}
Company.init({/* attributes */}, { sequelize, modelName: 'company' });

User.belongsTo(Company, {foreignKey: 'fk_company'}); // Adds fk_company to User
```

##### 目标键(Target key)

目标键是目标模型上的列，其是源模型外键列所指向的列。默认情况下，`belongsTo`关系的目标键会是目标模型的主键。要使用自定义列，可以用`targetKey`选项设置。

```
class User extends Model {}
User.init({/* attributes */}, { sequelize, modelName: 'user' })
class Company extends Model {}
Company.init({/* attributes */}, { sequelize, modelName: 'company' });

User.belongsTo(Company, {foreignKey: 'fk_companyname', targetKey: 'name'}); // Adds fk_companyname to User
```



#### `HasOne`

HasOne关联是**目标模型**上存在一对一关系的外键的关联。

```
class User extends Model {}
User.init({/* ... */}, { sequelize, modelName: 'user' })
class Project extends Model {}
Project.init({/* ... */}, { sequelize, modelName: 'project' })

// One-way associations
Project.hasOne(User)

/*
  In this example hasOne will add an attribute projectId to the User model!
  Furthermore, Project.prototype will gain the methods getUser and setUser according
  to the first parameter passed to define. If you have underscore style
  enabled, the added attribute will be project_id instead of projectId.

  The foreign key will be placed on the users table.

  You can also define the foreign key, e.g. if you already have an existing
  database and want to work on it:
*/

Project.hasOne(User, { foreignKey: 'initiator_id' })

/*
  Because Sequelize will use the model's name (first parameter of define) for
  the accessor methods, it is also possible to pass a special option to hasOne:
*/

Project.hasOne(User, { as: 'Initiator' })
// Now you will get Project.getInitiator and Project.setInitiator

// Or let's define some self references
class Person extends Model {}
Person.init({ /* ... */}, { sequelize, modelName: 'person' })

Person.hasOne(Person, {as: 'Father'})
// this will add the attribute FatherId to Person

// also possible:
Person.hasOne(Person, {as: 'Father', foreignKey: 'DadId'})
// this will add the attribute DadId to Person

// In both cases you will be able to do:
Person.setFather
Person.getFather

// If you need to join a table twice you can double join the same table
Team.hasOne(Game, {as: 'HomeTeam', foreignKey : 'homeTeamId'});
Team.hasOne(Game, {as: 'AwayTeam', foreignKey : 'awayTeamId'});

Game.belongsTo(Team);
```

虽然被称为HasOne关联，对于大多数`1:1`关系，通常也需要BelongsTo关联，因为BelongsTo会在源上添加foreignKey，而hasOne会在目标上添加。



##### 源键(Source key)

源键是目标模型上的外键属性，其会指向的源模型上的属性。默认情况下，`hasOne`关系的源键将是源模型的主键属性。要使用自定义属性，可以用`sourceKey`选项设置。

```
class User extends Model {}
User.init({/* attributes */}, { sequelize, modelName: 'user' })
class Company extends Model {}
Company.init({/* attributes */}, { sequelize, modelName: 'company' });

// Adds companyName attribute to User
// Use name attribute from Company as source attribute
Company.hasOne(User, {foreignKey: 'companyName', sourceKey: 'name'});
```



#### `HasOne`和`BelongsTo`之间的不同

在Sequelize中，`1:1`关系可用用HasOne和BelongsTo设置。它们适用的情况有所不同。让我们通过一个示例来说明这种差异。

假设有**Player**和**Team**两个表，模型定义如下：

```
class Player extends Model {}
Player.init({/* attributes */}, { sequelize, modelName: 'player' })
class Team extends Model {}
Team.init({/* attributes */}, { sequelize, modelName: 'team' });
```

当我们在Sequelize中联接两个模型时，可以将它们称为**源**和**目标**模型对。可以这样表示：

**Player**做为**源**和**Team**做为**目标**：

```
Player.belongsTo(Team);
//Or
Player.hasOne(Team);
```

**Team**做为**源**和**Player**做为**目标**：

```
Team.belongsTo(Player);
//Or
Team.hasOne(Player);
```

HasOne和Belongs会将关联键插入彼此不同的模型中。HasOne会在**目标模型**中插入关联键，而BelongsTo在会**源模型**中插入关联键。

以下是一个BelongsTo和HasOne使用示例：

```
class Player extends Model {}
Player.init({/* attributes */}, { sequelize, modelName: 'player' })
class Coach extends Model {}
Coach.init({/* attributes */}, { sequelize, modelName: 'coach' })
class Team extends Model {}
Team.init({/* attributes */}, { sequelize, modelName: 'team' });
```

假设`Player`模型存在关联`Team`信息的列`teamId`。有关每个团队教练（`Coach`）的信息存储在团队模型中，作为`coachId`列。这两种情况都需要不同类型的`1:1`关系，因为每次在不同模型上都存在外键关系。

当**源**模型中存在关联的信息时，我们可以使用`belongsTo`。在这种情况下，`Player`适用于`belongsTo`，因为它有`teamId`列。

```
Player.belongsTo(Team)  // `teamId` will be added on Player / Source model
```

当**目标**模型中存在关联的信息时，我们可以使用`hasOne`。在这种情况下，`Coach`适合`hasOne`，因为`Team`模型中有`Coach`的信息存储，即`coachId`字段。

```
Coach.hasOne(Team)  // `coachId` will be added on Team / Target model
```



### `一对多`关联 (`hasMany`)

一对多关联将一个源与多个目标连接在一起，而目标又被精确地连接到一个特定的源。

```
class User extends Model {}
User.init({/* ... */}, { sequelize, modelName: 'user' })
class Project extends Model {}
Project.init({/* ... */}, { sequelize, modelName: 'project' })

// OK. Now things get more complicated (not really visible to the user :)).
// First let's define a hasMany association
Project.hasMany(User, {as: 'Workers'})
```

这会添加`projectId`属性到User。根据下划线设置，表中的列可能是`projectId`或`project_id`。Project实例将会有`getWorkers`和`setWorkers`访问器。

有时，需要关联到记录不同的列，可以通过`sourceKey`选项设置：

```
class City extends Model {}
City.init({ countryCode: Sequelize.STRING }, { sequelize, modelName: 'city' });
class Country extends Model {}
Country.init({ isoCode: Sequelize.STRING }, { sequelize, modelName: 'country' });

// Here we can connect countries and cities base on country code
Country.hasMany(City, {foreignKey: 'countryCode', sourceKey: 'isoCode'});
City.belongsTo(Country, {foreignKey: 'countryCode', targetKey: 'isoCode'});
```

到目前为止，我们处理的都是单向关联。接下来，将在下一部分中创建多对多关联。



### `多对多`关联 (`BelongsToMany`)

多对多关联用于将源与多个目标连接，同时，目标还可以与多个源建立连接。

```
Project.belongsToMany(User, {through: 'UserProject'});
User.belongsToMany(Project, {through: 'UserProject'});
```

这将创建一个有外键`projectId`和`userId`的，名为`UserProject`的新模型。属性是否为驼峰格式取决于表连接的两个模型（本例中为User和Project）。

**必须**定义`through`。Sequelize以前会尝试自动生成名称，但这并不总是最合理的设置。

`belongsToMany`会向`Project`添加`getUsers`、`setUsers`、`addUser`、`addUsers`方法，同时会向`User`添加`getProjects`、`setProjects`、`addProject`、`addProjects`方法。

有时，在关联中使用模型时，可能需要重命名模型。让我们使用别名（`as`）选项将users定义为workers，或将projects定义为tasks。还可以手工定义要使用的外键：

```
User.belongsToMany(Project, { as: 'Tasks', through: 'worker_tasks', foreignKey: 'userId' })
Project.belongsToMany(User, { as: 'Workers', through: 'worker_tasks', foreignKey: 'projectId' })
```

`foreignKey`使你可以设置通过`through`关联的`源模型`的键。`otherKey`使你可以设置通过`through`关联的`目标模型`的键。

```
User.belongsToMany(Project, { as: 'Tasks', through: 'worker_tasks', foreignKey: 'userId', otherKey: 'projectId'})
```

在belongsToMany关系中，还可以定义自引用：

```
Person.belongsToMany(Person, { as: 'Children', through: 'PersonChildren' })
// This will create the table PersonChildren which stores the ids of the objects.
```



#### 源键和目标键

如果要创建一个不使用默认主键的“属于多个”关系，需要进行一些设置。必须为属于多个对象的两端设置合适的`sourceKey`（可选的`targetKey`）。此外，还必须确保在关系上创建了适当的索引。例如：

```
const User = this.sequelize.define('User', {
  id: {
    type: DataTypes.UUID,
    allowNull: false,
    primaryKey: true,
    defaultValue: DataTypes.UUIDV4,
    field: 'user_id'
  },
  userSecondId: {
    type: DataTypes.UUID,
    allowNull: false,
    defaultValue: DataTypes.UUIDV4,
    field: 'user_second_id'
  }
}, {
  tableName: 'tbl_user',
  indexes: [
    {
      unique: true,
      fields: ['user_second_id']
    }
  ]
});

const Group = this.sequelize.define('Group', {
  id: {
    type: DataTypes.UUID,
    allowNull: false,
    primaryKey: true,
    defaultValue: DataTypes.UUIDV4,
    field: 'group_id'
  },
  groupSecondId: {
    type: DataTypes.UUID,
    allowNull: false,
    defaultValue: DataTypes.UUIDV4,
    field: 'group_second_id'
  }
}, {
  tableName: 'tbl_group',
  indexes: [
    {
      unique: true,
      fields: ['group_second_id']
    }
  ]
});

User.belongsToMany(Group, {
  through: 'usergroups',
  sourceKey: 'userSecondId'
});
Group.belongsToMany(User, {
  through: 'usergroups',
  sourceKey: 'groupSecondId'
});
```

如果要在联接表中添加其他属性，则可以在定义关联之前先在sequelize中为联接表定义一个模型，然后告诉sequelize应使用该模型进行联接，而不是创建一个新模型：

```
class User extends Model {}
User.init({}, { sequelize, modelName: 'user' })
class Project extends Model {}
Project.init({}, { sequelize, modelName: 'project' })
class UserProjects extends Model {}
UserProjects.init({
  status: DataTypes.STRING
}, { sequelize, modelName: 'userProjects' })

User.belongsToMany(Project, { through: UserProjects })
Project.belongsToMany(User, { through: UserProjects })
```

要将新project添加到user并设置其状态，需要将额外的`options.through`传递给setter，该setter包含联接表的属性。

```
user.addProject(project, { through: { status: 'started' }})
```

默认情况下，上面的代码会将`projectId`和`userId`添加到`UserProjects`表中，并删除任何以前定义的主键属性(该表将由两个表的键的组合唯一标识，也就不再需要使用其他PK列)。要在`UserProjects`模型上强制使用主键，可以手动添加。

```
class UserProjects extends Model {}
UserProjects.init({
  id: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  status: DataTypes.STRING
}, { sequelize, modelName: 'userProjects' })
```

使用“多对多”，你可以基于**through**关系进行查询并选择指定的属性。例如，将`findAll`与**through**一起使用：

```
User.findAll({
  include: [{
    model: Project,
    through: {
      attributes: ['createdAt', 'startedAt', 'finishedAt'],
      where: {completed: true}
    }
  }]
});
```

当模型中不存在主键时，Belongs-to-Many将创建一个唯一键。可以使用`uniqueKey`选项覆盖此唯一键名称。

```
Project.belongsToMany(User, { through: UserProjects, uniqueKey: 'my_custom_unique' })
```



### 命名策略

默认情况下，sequelize将使用模型名称（传递给`sequelize.define`的名称）找到在关联中使用的模型名称。例如，名为`user`的模型会将`get/set/add User`函数添加到关联模型的实例中，并在加载时添加名为`.user`的属性，而名为`User`的模型将添加相同的函数，但将名为`.User`。

正如我们看到的，你可以使用`as`在关联中对模型指定别名。在单个关联（hasOne与belongsTo）中，别名应为单数，而对于多个关联（hasMany），别名应为复数。Sequelize会使用[inflection](https://www.npmjs.org/package/inflection)库将别名转换为其单数形式。但是，这可能并不总是适用于不规则或非英语单词。在这种情况下，可以提供别名的复数形式和单数形式：

```
User.belongsToMany(Project, { as: { singular: 'task', plural: 'tasks' }})
// Notice that inflection has no problem singularizing tasks, this is just for illustrative purposes.
```

如果知道模型在关联中将始终使用相同的别名，则可以在创建模型时提供它：

```
class Project extends Model {}
Project.init(attributes, {
  name: {
    singular: 'task',
    plural: 'tasks',
  },
  sequelize,
  modelName: 'project'
})

User.belongsToMany(Project);
```

这会向user实例添加`add/set/get Tasks`函数。

请记住，使用`as`来修改关联的名称也将更改外键的名称。使用`as`时，出于安全考虑还要指定外键：

```
Invoice.belongsTo(Subscription)
Subscription.hasMany(Invoice)
```

如果不使用`as`，则会按预期方式添加`subscriptionId`。但是，如果`Invoice.belongsTo(Subscription, { as: 'TheSubscription' })`，那么将同时拥有`subscriptionId`和`theSubscriptionId`，因为sequelize不够聪明，无法确定调用的是同一关系的两边。可以用`foreignKey`来解决这个问题：

```
Invoice.belongsTo(Subscription, { as: 'TheSubscription', foreignKey: 'subscription_id' })
Subscription.hasMany(Invoice, { foreignKey: 'subscription_id' })
```



### 关联对象

因为Sequelize做了很多事情，所以必须在设置关联后调用`Sequelize.sync`, 这样做将使你具备以下条件：

```
Project.hasMany(Task)
Task.belongsTo(Project)

Project.create()...
Task.create()...
Task.create()...

// save them... and then:
project.setTasks([task1, task2]).then(() => {
  // saved!
})

// ok, now they are saved... how do I get them later on?
project.getTasks().then(associatedTasks => {
  // associatedTasks is an array of tasks
})

// You can also pass filters to the getter method.
// They are equal to the options you can pass to a usual finder method.
project.getTasks({ where: 'id > 10' }).then(tasks => {
  // tasks with an id greater than 10 :)
})

// You can also only retrieve certain fields of a associated object.
project.getTasks({attributes: ['title']}).then(tasks => {
  // retrieve tasks with the attributes "title" and "id"
})
```

要删除己创建的关联，可以只调用set方法而不使用指定的ID：

```
// remove the association with task1
project.setTasks([task2]).then(associatedTasks => {
  // you will get task2 only
})

// remove 'em all
project.setTasks([]).then(associatedTasks => {
  // you will get an empty array
})

// or remove 'em more directly
project.removeTask(task1).then(() => {
  // it's gone
})

// and add 'em again
project.addTask(task1).then(() => {
  // it's back again
})
```

也可以这样：

```
// project is associated with task1 and task2
task2.setProject(null).then(() => {
  // and it's gone
})
```

hasOne/belongsTo基本上是相同的：

```
Task.hasOne(User, {as: "Author"})
Task.setAuthor(anAuthor)
```

可以通过两种方式将关联添加到有自定义联接表的关系中（继续上一章中定义的关联）：

```
// Either by adding a property with the name of the join table model to the object, before creating the association
project.UserProjects = {
  status: 'active'
}
u.addProject(project)

// Or by providing a second options.through argument when adding the association, containing the data that should go in the join table
u.addProject(project, { through: { status: 'active' }})


// When associating multiple objects, you can combine the two options above. In this case the second argument
// will be treated as a defaults object, that will be used if no data is provided
project1.UserProjects = {
    status: 'inactive'
}

u.setProjects([project1, project2], { through: { status: 'active' }})
// The code above will record inactive for project one, and active for project two in the join table
```

在有自定义联接表的关联上获取数据时，联接表中的数据将作为DAO实例返回：

```
u.getProjects().then(projects => {
  const project = projects[0]

  if (project.UserProjects.status === 'active') {
    // .. do magic

    // since this is a real DAO instance, you can save it directly after you are done doing magic
    return project.UserProjects.save()
  }
})
```

如果只需要联接表中的某些属性，则可以用数组提供所需的属性：

```
// This will select only name from the Projects table, and only status from the UserProjects table
user.getProjects({ attributes: ['name'], joinTableAttributes: ['status']})
```



### 检查关联

还可以检查对象是否已与另一个对象关联（仅`N:M`）。处理方式如下：

```
// check if an object is one of associated ones:
Project.create({ /* */ }).then(project => {
  return User.create({ /* */ }).then(user => {
    return project.hasUser(user).then(result => {
      // result would be false
      return project.addUser(user).then(() => {
        return project.hasUser(user).then(result => {
          // result would be true
        })
      })
    })
  })
})

// check if all associated objects are as expected:
// let's assume we have already a project and two users
project.setUsers([user1, user2]).then(() => {
  return project.hasUsers([user1]);
}).then(result => {
  // result would be true
  return project.hasUsers([user1, user2]);
}).then(result => {
  // result would be true
})
```



### 高级概念

#### 范围/作用域

本节会介绍关联的范围/作用域。有关关联作用域与关联模型作用域的定义，参考：[Scopes](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#scopes)。

关联作用域允许您在关联上设置作用域（一组用于`get`和`create`的默认属性）。范围既可以放在关联的模型（关联的目标）上，也可以放在`n:m`关系的**through**表中。

##### `1:n`

假设我们有模型`Comment`、`Post`和`Image`。可以通过`commentableId`和`commentable`将评论(comment)与图片(image)或帖子(psot)相关联-我们可以Post和Image是`Commentable`：

```
class Post extends Model {}
Post.init({
  title: Sequelize.STRING,
  text: Sequelize.STRING
}, { sequelize, modelName: 'post' });

class Image extends Model {}
Image.init({
  title: Sequelize.STRING,
  link: Sequelize.STRING
}, { sequelize, modelName: 'image' });

class Comment extends Model {
  getItem(options) {
    return this[
      'get' +
        this.get('commentable')
          [0]
          .toUpperCase() +
        this.get('commentable').substr(1)
    ](options);
  }
}

Comment.init({
  title: Sequelize.STRING,
  commentable: Sequelize.STRING,
  commentableId: Sequelize.INTEGER
}, { sequelize, modelName: 'comment' });

Post.hasMany(Comment, {
  foreignKey: 'commentableId',
  constraints: false,
  scope: {
    commentable: 'post'
  }
});

Comment.belongsTo(Post, {
  foreignKey: 'commentableId',
  constraints: false,
  as: 'post'
});

Image.hasMany(Comment, {
  foreignKey: 'commentableId',
  constraints: false,
  scope: {
    commentable: 'image'
  }
});

Comment.belongsTo(Image, {
  foreignKey: 'commentableId',
  constraints: false,
  as: 'image'
});
```

`constraints: false`会禁用引用约束，因为`commentableId`列引用了多个表，所以我们无法向其添加`REFERENCES`约束。

注意，`Image -> Comment`和`Post -> Comment`关系定义了一个作用域，分别是`commentable: 'image'`和`commentable: 'post'`。使用关联函数时，将自动应用此作用域：

```
image.getComments()
// SELECT "id", "title", "commentable", "commentableId", "createdAt", "updatedAt" FROM "comments" AS
// "comment" WHERE "comment"."commentable" = 'image' AND "comment"."commentableId" = 1;

image.createComment({
  title: 'Awesome!'
})
// INSERT INTO "comments" ("id","title","commentable","commentableId","createdAt","updatedAt") VALUES
// (DEFAULT,'Awesome!','image',1,'2018-04-17 05:36:40.454 +00:00','2018-04-17 05:36:40.454 +00:00')
// RETURNING *;

image.addComment(comment);
// UPDATE "comments" SET "commentableId"=1,"commentable"='image',"updatedAt"='2018-04-17 05:38:43.948
// +00:00' WHERE "id" IN (1)
```

`Comment`上的`getItem`功能函数完成了图片-它只是将`commentable`字符串转换为对`getImage`或`getPost`的调用，从而提供了有关注释(comment)属于帖子(post)还是图像(image)的抽象。你可以将普通选项对象作为参数传递给`getItem(options)`，以指定任何条件或包含。

##### `n:m`

继续考虑多态模型，一个标签(tag)表-一个项目(item)可以有多个标签，而一个标签可以与多个项目相关。

为简洁起见，该示例仅显示了Post模型，但实际上Tag将与其他几个模型相关。

```
class ItemTag extends Model {}
ItemTag.init({
  id: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  tagId: {
    type: Sequelize.INTEGER,
    unique: 'item_tag_taggable'
  },
  taggable: {
    type: Sequelize.STRING,
    unique: 'item_tag_taggable'
  },
  taggableId: {
    type: Sequelize.INTEGER,
    unique: 'item_tag_taggable',
    references: null
  }
}, { sequelize, modelName: 'item_tag' });

class Tag extends Model {}
Tag.init({
  name: Sequelize.STRING,
  status: Sequelize.STRING
}, { sequelize, modelName: 'tag' });

Post.belongsToMany(Tag, {
  through: {
    model: ItemTag,
    unique: false,
    scope: {
      taggable: 'post'
    }
  },
  foreignKey: 'taggableId',
  constraints: false
});

Tag.belongsToMany(Post, {
  through: {
    model: ItemTag,
    unique: false
  },
  foreignKey: 'tagId',
  constraints: false
});
```

注意scope列(`taggable`)，其现在在through模型上(`ItemTag`)。

我们还可以定义一个限制性更强的关联，例如，通过应用through模型（`ItemTag`）和目标模型（`Tag`）的范围来获取帖子的所有待处理标签：

```
Post.belongsToMany(Tag, {
  through: {
    model: ItemTag,
    unique: false,
    scope: {
      taggable: 'post'
    }
  },
  scope: {
    status: 'pending'
  },
  as: 'pendingTags',
  foreignKey: 'taggableId',
  constraints: false
});

post.getPendingTags();
SELECT
  "tag"."id",
  "tag"."name",
  "tag"."status",
  "tag"."createdAt",
  "tag"."updatedAt",
  "item_tag"."id" AS "item_tag.id",
  "item_tag"."tagId" AS "item_tag.tagId",
  "item_tag"."taggable" AS "item_tag.taggable",
  "item_tag"."taggableId" AS "item_tag.taggableId",
  "item_tag"."createdAt" AS "item_tag.createdAt",
  "item_tag"."updatedAt" AS "item_tag.updatedAt"
FROM
  "tags" AS "tag"
  INNER JOIN "item_tags" AS "item_tag" ON "tag"."id" = "item_tag"."tagId"
  AND "item_tag"."taggableId" = 1
  AND "item_tag"."taggable" = 'post'
WHERE
  ("tag"."status" = 'pending');
```

`constraints: false`会禁用`taggableId`列上的引用约束。因为该列是多态的，所以我们不能说它`REFERENCES`了特定的表。



#### 创建关联

只要所有元素都是新元素，就可以一步创建带有嵌套关联的实例。

##### `BelongsTo` / `HasMany` / `HasOne`关联

参考以下模型：

```
class Product extends Model {}
Product.init({
  title: Sequelize.STRING
}, { sequelize, modelName: 'product' });
class User extends Model {}
User.init({
  firstName: Sequelize.STRING,
  lastName: Sequelize.STRING
}, { sequelize, modelName: 'user' });
class Address extends Model {}
Address.init({
  type: Sequelize.STRING,
  line1: Sequelize.STRING,
  line2: Sequelize.STRING,
  city: Sequelize.STRING,
  state: Sequelize.STRING,
  zip: Sequelize.STRING,
}, { sequelize, modelName: 'address' });

Product.User = Product.belongsTo(User);
User.Addresses = User.hasMany(Address);
// Also works for `hasOne`
```

在以下方式中，一个新的`Product`、`User`及多个`Address`可以一步创建完成：

```
return Product.create({
  title: 'Chair',
  user: {
    firstName: 'Mick',
    lastName: 'Broadstone',
    addresses: [{
      type: 'home',
      line1: '100 Main St.',
      city: 'Austin',
      state: 'TX',
      zip: '78704'
    }]
  }
}, {
  include: [{
    association: Product.User,
    include: [ User.Addresses ]
  }]
});
```

现在，我们的用户模型称为`user`(注意小写`u`)-这意味着对象中的属性也应该是`user`。如果`sequelize.define`定义的名称是`User`，则对象中的键也应该是`User`。对于`addresses`也是如此，除了它是`hasMany`关联的复数形式。



##### 用别名进行`BelongsTo`关联

可以扩展前面的示例以支持别名关联：

```
const Creator = Product.belongsTo(User, { as: 'creator' });

return Product.create({
  title: 'Chair',
  creator: {
    firstName: 'Matt',
    lastName: 'Hansen'
  }
}, {
  include: [ Creator ]
});
```



##### `hasMany` / `BelongsToMany`关联

来介绍一下将一个产品(product)与许多标签(tag)关联的功能。设置模型如下所示：

```
class Tag extends Model {}
Tag.init({
  name: Sequelize.STRING
}, { sequelize, modelName: 'tag' });

Product.hasMany(Tag);
// Also works for `belongsToMany`.
```

现在我们来创建一个产品，及多个标签：

```
Product.create({
  id: 1,
  title: 'Chair',
  tags: [
    { name: 'Alpha'},
    { name: 'Beta'}
  ]
}, {
  include: [ Tag ]
})
```

修改这个示例，以支持别名：

```
const Categories = Product.hasMany(Tag, { as: 'categories' });

Product.create({
  id: 1,
  title: 'Chair',
  categories: [
    { id: 1, name: 'Alpha' },
    { id: 2, name: 'Beta' }
  ]
}, {
  include: [{
    association: Categories,
    as: 'categories'
  }]
})
```



## 11. 原始查询(Raw queries)

- ["Dotted"(`"."`) 属性](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#-quot-dotted-quot--attributes)
- [替换](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#replacements)
- [绑定参数](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#bind-parameter)

由于在很多情况下执行原始/已经准备好的SQL查询更加容易，此时可以使用`sequelize.query`函数。

默认情况下，该函数将返回两个参数：一个结果数组和一个包含元数据的对象（受影响的行等）。请注意，由于这是原始查询，因此元数据（属性名称等）特定于方言。一些方言会在结果对象“内部”返回元数据（作为数组的属性），但是，将始终返回两个参数。而对于MSSQL和MySQL，它将是对同一对象的两个引用。

```
sequelize.query("UPDATE users SET y = 42 WHERE x = 12").then(([results, metadata]) => {
  // Results will be an empty array and metadata will contain the number of affected rows.
})
```

在不需要访问元数据的情况下，可以传递查询类型以告诉序列化如何格式化结果。例如，对于简单的select查询，可以执行以下操作：

```
sequelize.query("SELECT * FROM `users`", { type: sequelize.QueryTypes.SELECT})
  .then(users => {
    // We don't need spread here, since only the results will be returned for select queries
  })
```

所支持的查询类型，请参考：[查询类型源码](https://github.com/sequelize/sequelize/blob/master/lib/query-types.js)。

`sequelize.query`支持传入第二可选参数，其为一个模型。传入后，将返回模型的实例：

```
// Callee is the model definition. This allows you to easily map a query to a predefined model
sequelize
  .query('SELECT * FROM projects', {
    model: Projects,
    mapToModel: true // pass true here if you have any mapped fields
  })
  .then(projects => {
    // Each record will now be an instance of Project
  })
```

更多信息，请查看：[查询API](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-method-query)。参考以下示例：

```
sequelize.query('SELECT 1', {
  // A function (or false) for logging your queries
  // Will get called for every SQL query that gets sent
  // to the server.
  logging: console.log,

  // If plain is true, then sequelize will only return the first
  // record of the result set. In case of false it will return all records.
  plain: false,

  // Set this to true if you don't have a model definition for your query.
  raw: false,

  // The type of query you are executing. The query type affects how results are formatted before they are passed back.
  type: Sequelize.QueryTypes.SELECT
})

// Note the second argument being null!
// Even if we declared a callee here, the raw: true would
// supersede and return a raw object.
sequelize
  .query('SELECT * FROM projects', { raw: true })
  .then(projects => {
    console.log(projects)
  })
```



### "Dotted"(`"."`) 属性

如果表的属性名称包含点，则将嵌套结果对象。这是因为底层使用[dottie.js](https://github.com/mickhansen/dottie.js/)。见下文：

```
sequelize.query('select 1 as `foo.bar.baz`').then(rows => {
  console.log(JSON.stringify(rows))
})
[{
  "foo": {
    "bar": {
      "baz": 1
    }
  }
}]
```



### 替换(Replacements)

查询中的“替换”可以通过两种方式：使用命名参数（以`:`开头）；或者使用`?`表示的未命名参数，而在options对象中传递替换：

- 如果传递数组，则`?`将按照它们在数组中出现的顺序进行替换
- 如果传递一个对象，`:key`将被该对象的键替换。如果对象包含在查询中找不到的键，将引发异常。

```
sequelize.query('SELECT * FROM projects WHERE status = ?',
  { replacements: ['active'], type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})

sequelize.query('SELECT * FROM projects WHERE status = :status ',
  { replacements: { status: 'active' }, type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})
```

数组替换将自动处理，以下查询将搜索`status`与值数组匹配的项目。

```
sequelize.query('SELECT * FROM projects WHERE status IN(:status) ',
  { replacements: { status: ['active', 'inactive'] }, type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})
```

要使用通配符运算符`%`，需要将其附加到替换项中。以下查询会将用户名以“ ben”开头的用户进行匹配：

```
sequelize.query('SELECT * FROM users WHERE name LIKE :search_name ',
  { replacements: { search_name: 'ben%'  }, type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})
```



### 绑定参数(Bind Parameter)

“绑定参数”就像“替换”。不同点在于：“替换”会被转义，并在查询发送到数据库之前通过序列化插入到查询中；而“绑定参数”在SQL查询文本之外发送到数据库。查询可以具有绑定参数或替换参数。绑定参数通过`$1`、`$2`、...（数字）或`$key`（字母数字）的方式引用，与方言无关。

- 如果传入了数组，则`$1`绑定到数组中的第一个元素（`bind[0]`）
- 如果传入了对象，则`$key`会绑定到`object['keys']`。每个键都必须以非数字字符开头，`$1`不是有效键，即使`object['1']`存在
- 无论哪种形式，`$$`都可用于转义`$`符号

数组或对象必须包含所有绑定值，否则Sequelize会抛出异常。这也适用于数据库可能忽略的绑定参数的情况。

数据库可能会有更多限制。绑定参数不能是SQL关键字，也不能是表名或列名。带引号的文本或数据中也将被忽略。在PostgreSQL中，如果不能从上下文`$1::varchar`推断类型，可能还需要对它们进行类型转换。

```
sequelize.query('SELECT *, "text with literal $$1 and literal $$status" as t FROM projects WHERE status = $1',
  { bind: ['active'], type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})

sequelize.query('SELECT *, "text with literal $$1 and literal $$status" as t FROM projects WHERE status = $status',
  { bind: { status: 'active' }, type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})
```



## 12. 事务(Transactions)

- 托管的事务（auto-callback）
  - [抛出错误并回滚](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#throw-errors-to-rollback)
  - [自动传递事务到所有的查询中](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#automatically-pass-transactions-to-all-queries)
- 并行/部分事务
  - [不启用`CLS`](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#without-cls-enabled)
- [隔离级别](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#isolation-levels)
- [非托管的事务 (then-callback)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#unmanaged-transaction--then-callback-)
- [与其他sequelize方法一起使用](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#usage-with-other-sequelize-methods)
- [提交(commit)后置钩子](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#after-commit-hook)
- [锁](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#locks)

Sequelize支持两种事务使用方式：

1. **托管**(Managed) - 一种将基Pomise链的结果自动提交或回滚事务，并且（如果启用了CLS）将事务传递给回调中的所有调用
2. **非托管**(Unmanaged) - 不会自动提交、回滚并将事务交由用户控制



### 托管事务（auto-callback）

托管事务自动处理提交或回滚事务。通过将回调传递给`sequelize.transaction`来启动托管事务。

注意传递给`transaction`的回调返回Pormise链的方式，并且没有显示调用`t.commit()`或`t.rollback()`。如果链中所有的Promise都已成功处理（`resolve`状态），则会自动提交事务；如果有一或多个Promise被拒绝（`reject`状态），则事务将会回滚。

```
return sequelize.transaction(t => {

  // chain all your queries here. make sure you return them.
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
  // Transaction has been committed
  // result is whatever the result of the promise chain returned to the transaction callback
}).catch(err => {
  // Transaction has been rolled back
  // err is whatever rejected the promise chain returned to the transaction callback
});
```



#### 抛出错误并回滚

使用托管事务时，切勿手动提交或回滚事务。如果所有查询都成功，但是你仍想回滚事务（例如，验证失败），则应抛出错误来中断并拒绝该Promise链：

```
return sequelize.transaction(t => {
  return User.create({
    firstName: 'Abraham',
    lastName: 'Lincoln'
  }, {transaction: t}).then(user => {
    // Woops, the query was successful but we still want to roll back!
    throw new Error();
  });
});
```



#### 自动传递事务到所有的查询中

在上面示例中，事务仍是手工传递的（通过第二个参数中的`{ transaction: t }`）。要将事务自动传递给所有查询，必须安装[continuation local storage](https://github.com/othiym23/node-continuation-local-storage)（CLS）模块并在自己的代码中实例化命名空间：

```
const cls = require('continuation-local-storage');
const namespace = cls.createNamespace('my-very-own-namespace');
```

要启用CLS，必须通过使用sequelize构造函数的静态方法来告诉sequelize使用哪个命名空间：

```
const Sequelize = require('sequelize');
Sequelize.useCLS(namespace);

new Sequelize(....);
```

注意，`useCLS()`方法在构造函数上，而不在sequelize实例上。这意味着所有实例将共享相同的命名空间，并且CLS是全有或全无（不能仅对某些实例启用它）。

CLS的工作方式类似于用于回调的线程本地存储。实际上，这也使不同的回调链可以使用CLS命名空间访问局部变量。启用CLS后，当创建新事务时，sequelize将在命名空间上设置`transaction`属性。由于在回调链中设置的变量是该链的私有变量，因此可以同时存在多个并发事务：

```
sequelize.transaction((t1) => {
  namespace.get('transaction') === t1; // true
});

sequelize.transaction((t2) => {
  namespace.get('transaction') === t2; // true
});
```

在大多数情况下，你不需要直接访问`namespace.get('transaction')`，因为所有查询都会自动在命名空间上查找事务：

```
sequelize.transaction((t1) => {
  // With CLS enabled, the user will be created inside the transaction
  return User.create({ name: 'Alice' });
});
```

使用`Sequelize.useCLS()`之后，将从sequelize返回的所有promise都会被打补丁以维护CLS上下文。CLS是一个复杂的主题，参考：[cls-bluebird](https://www.npmjs.com/package/cls-bluebird)以了解更多信息，该文件用于使bluebird promise可以与CLS一起使用。



### 并行/部分事务

你可以在一系列查询中进行并发事务，也可以将某些事务排除在查询之外。使用`{transaction:}`选项可以控制查询属于哪个事务：

警告：SQLite不能同时支持多个事务。



#### 不启用`CLS`

```
sequelize.transaction((t1) => {
  return sequelize.transaction((t2) => {
    // With CLS enable, queries here will by default use t2
    // Pass in the `transaction` option to define/alter the transaction they belong to.
    return Promise.all([
        User.create({ name: 'Bob' }, { transaction: null }),
        User.create({ name: 'Mallory' }, { transaction: t1 }),
        User.create({ name: 'John' }) // this would default to t2
    ]);
  });
});
```



### 隔离级别

启动事务时可以使用的隔离级别：

```
Sequelize.Transaction.ISOLATION_LEVELS.READ_UNCOMMITTED // "READ UNCOMMITTED"
Sequelize.Transaction.ISOLATION_LEVELS.READ_COMMITTED // "READ COMMITTED"
Sequelize.Transaction.ISOLATION_LEVELS.REPEATABLE_READ  // "REPEATABLE READ"
Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE // "SERIALIZABLE"
```

默认情况下，sequelize使用数据库的隔离级别。如果要使用其他隔离级别，请传入所需的级别作为第一个参数：

```
return sequelize.transaction({
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
  }, (t) => {

  // your transactions

  });
```

可以在初始化Sequelize实例时全局设置`isolationLevel`，也可以在每个事务本地单独设置：

```
// globally
new Sequelize('db', 'user', 'pw', {
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
});

// locally
sequelize.transaction({
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
});
```

注意：如果使用MSSQL，则不会记录`SET ISOLATION LEVEL`查询，因为会将指定的`isolationLevel`直接传递给tedious。



### 非托管事务 (then-callback)

非托管事务需要你手动回滚或提交事务。如果不这样做，事务将会挂起，直到超时。要启动非托管事务，请在没有回调的情况下调用`sequelize.transaction()`（仍然可以传递选项对象），然后对返回的promise调用`then`。需要注意，`commit()`和`rollback()`返回一个promise。

```
return sequelize.transaction().then(t => {
  return User.create({
    firstName: 'Bart',
    lastName: 'Simpson'
  }, {transaction: t}).then(user => {
    return user.addSibling({
      firstName: 'Lisa',
      lastName: 'Simpson'
    }, {transaction: t});
  }).then(() => {
    return t.commit();
  }).catch((err) => {
    return t.rollback();
  });
});
```



### 与其他sequelize方法一起使用

`transaction`选项与其他大多数选项一起使用，通常是方法的第一个参数。对于带有值的方法，如`.create()`、`.update()`等，应将`transaction`传递给第二个参数中的选项。如果不确定，请参阅API文档以了解要使用的方法以确保签名正确。



### 提交(commit)后置钩子

`transaction`对象可以跟踪是否以及何时提交。

可以将`afterCommit`钩子添加到托管和非托管事务对象中：

```
sequelize.transaction(t => {
  t.afterCommit((transaction) => {
    // Your logic
  });
});

sequelize.transaction().then(t => {
  t.afterCommit((transaction) => {
    // Your logic
  });

  return t.commit();
})
```

传递给`afterCommit`的函数可以选择返回一个promise，该promise将在创建事务的promise链解析之前

如果事务回滚，则不会触发`afterCommit`钩子

`afterCommit`钩子不修改事务的返回值，这与标准钩子不同

可以将`afterCommit`钩子与模型钩子结合使用，以了解何时保存实例并在事务外部可用

```
model.afterSave((instance, options) => {
  if (options.transaction) {
    // Save done within a transaction, wait until transaction is committed to
    // notify listeners the instance has been saved
    options.transaction.afterCommit(() => /* Notify */)
    return;
  }
  // Save done outside a transaction, safe for callers to fetch the updated model
  // Notify
})
```



### 锁

`transaction`内的查询可以使用锁执行

```
return User.findAll({
  limit: 1,
  lock: true,
  transaction: t1
})
```

事务中的查询可以跳过锁定的行

```
return User.findAll({
  limit: 1,
  lock: true,
  skipLocked: true,
  transaction: t2
})
```



## 13. 作用域(Scopes)

- [定义](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#scopes-definition)
- [使用方法](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#scopes-usage)
- 合并
  - [合并`include`](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#scopes-merging-includes)
- [关联](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#scopes-associations)

作用域范围使你可以定义常用查询，以便以后使用。作用域可以包括与常规查找器相同的所有属性，`where`、`include`、`limit`等。

### 定义

作用域是在模型定义中定义的，可以是查找器对象，也可以是返回查找器对象的函数-默认作用域除外，默认作用域(scope)只能是一个对象：

```
class Project extends Model {}
Project.init({
  // Attributes
}, {
  defaultScope: {
    where: {
      active: true
    }
  },
  scopes: {
    deleted: {
      where: {
        deleted: true
      }
    },
    activeUsers: {
      include: [
        { model: User, where: { active: true }}
      ]
    },
    random () {
      return {
        where: {
          someNumber: Math.random()
        }
      }
    },
    accessLevel (value) {
      return {
        where: {
          accessLevel: {
            [Op.gte]: value
          }
        }
      }
    }
    sequelize,
    modelName: 'project'
  }
});
```

还可以在定义模型后通过调用`addScope`添加作用域。这对于具有包含（`inlude`）的作用域特别有用，其中在定义另一个模型时可能未定义包含中的模型。

始终应用默认作用域。这意味着，使用上面的模型定义，`Project.findAll()`将创建以下查询：

```
SELECT * FROM projects WHERE active = true
```

默认作用域可以通过调用`.unscoped()`、`.scope(null)`或调用另一个作用域移除：

```
Project.scope('deleted').findAll(); // Removes the default scope
SELECT * FROM projects WHERE deleted = true
```

也可以在作用域定义中包括作用域模型。这使你避免重复`include”`、`attributes`或`where`定义。使用上面的示例，并在包含的用户模型上调用`active`作用域（而不是直接在该包含对象中指定条件）：

```
activeUsers: {
  include: [
    { model: User.scope('active')}
  ]
}
```



### 使用方法

通过在模型定义上调用`.scope`并传递一个或多个作用域的名称来应用作用域。`.scope`会返回一个功能齐全的模型实例，它会有所有模型方法：`.findAll`、`.update`、`.count`、`.destroy`等。可以保存此模型实例并在以后重用：

```
const DeletedProjects = Project.scope('deleted');

DeletedProjects.findAll();
// some time passes

// let's look for deleted projects again!
DeletedProjects.findAll();
```

作用域会应用到`.find`、`.findAll`、`.update`、`.count`、`.update`、`.increment`和`.destroy`。

作用域可以通过两种方式调用。如果作用域不带任何参数，则正常调用即可；如果作用域接受参数，则传递一个对象：

```
Project.scope('random', { method: ['accessLevel', 19]}).findAll();
SELECT * FROM projects WHERE someNumber = 42 AND accessLevel >= 19
```



### 合并

可以将作用域数组传递给`.scope`或将作用域作为连续参数传递，可以同时应用多个作用域。

```
// These two are equivalent
Project.scope('deleted', 'activeUsers').findAll();
Project.scope(['deleted', 'activeUsers']).findAll();
SELECT * FROM projects
INNER JOIN users ON projects.userId = users.id
WHERE projects.deleted = true
AND users.active = true
```

果要在默认作用域基础上应用另一个作用域，可以将键`defaultScope`传递给.`.scope`：

```
Project.scope('defaultScope', 'deleted').findAll();
SELECT * FROM projects WHERE active = true AND deleted = true
```

调用多个合并作用域时，后续合并作用域中的键将覆盖先前合并作用域中的键（类似于[Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)），除了会合并的`where`和`include`之外。考虑两个作用域：

```
{
  scope1: {
    where: {
      firstName: 'bob',
      age: {
        [Op.gt]: 20
      }
    },
    limit: 2
  },
  scope2: {
    where: {
      age: {
        [Op.gt]: 30
      }
    },
    limit: 10
  }
}
```

调用`.scope('scope1', 'scope2')`会产生以下查询：

```
WHERE firstName = 'bob' AND age > 30 LIMIT 10
```

请注意，在保留`firstName`的同时，`scope2`是如何覆盖`limit`和`age`。`limit`、`offset`、`order`、`paranoid`、`lock`和`raw`字段会被覆盖，而在此处被浅合并（这意味着相同的键将被覆盖）。`include`的合并策略会在后面讨论。

请注意，多个应用作用域的`attributes`键以始终以`attribute.exclude`的方式合并。这允许合并多个作用域，并且永远不会暴露最终合并作用域中的敏感字段。

当将查找对象直接传递给作用域模型上的`findAll`（和类似的查找器）时，会使用相同的合并逻辑：

```
Project.scope('deleted').findAll({
  where: {
    firstName: 'john'
  }
})
```

在这里，`deleted`作用域会与查询器合并。如果你传入`where: { firstName: 'john', deleted: false }`，`deleted`作用域会被覆盖。



#### 合并`include`

包含（`include`）将根据所包含的模型进行递归合并。这是v5新增的功能的，接下来通过示例展示。

考虑以下四个模型：Foo，Bar，Baz和Qux，它们有很多关联，如下所示：

```
class Foo extends Model {}
class Bar extends Model {}
class Baz extends Model {}
class Qux extends Model {}
Foo.init({ name: Sequelize.STRING }, { sequelize });
Bar.init({ name: Sequelize.STRING }, { sequelize });
Baz.init({ name: Sequelize.STRING }, { sequelize });
Qux.init({ name: Sequelize.STRING }, { sequelize });
Foo.hasMany(Bar, { foreignKey: 'fooId' });
Bar.hasMany(Baz, { foreignKey: 'barId' });
Baz.hasMany(Qux, { foreignKey: 'bazId' });
```

Foo上有以下四个作用域：

```
{
  includeEverything: {
    include: {
      model: this.Bar,
      include: [{
        model: this.Baz,
        include: this.Qux
      }]
    }
  },
  limitedBars: {
    include: [{
      model: this.Bar,
      limit: 2
    }]
  },
  limitedBazs: {
    include: [{
      model: this.Bar,
      include: [{
        model: this.Baz,
        limit: 2
      }]
    }]
  },
  excludeBazName: {
    include: [{
      model: this.Bar,
      include: [{
        model: this.Baz,
        attributes: {
          exclude: ['name']
        }
      }]
    }]
  }
}
```

这4个作用域可以很容易地深度合并，如，通过调用`Foo.scope('includeEverything', 'limitedBars', 'limitedBazs', 'excludeBazName').findAll()`，这等同于调用以下代码：

```
Foo.findAll({
  include: {
    model: this.Bar,
    limit: 2,
    include: [{
      model: this.Baz,
      limit: 2,
      attributes: {
        exclude: ['name']
      },
      include: this.Qux
    }]
  }
});
```

看一下4个作用域是如何合并成一个的。作用域的包含（`include`）会根据包含的模型进行合并。如果一个作用域包含了模型A，一个作用域包含了模型B，则合并的结果会同时包含A和B。另一方面，如果两个作用域都包含相同的模型A，但有不同的选项（如，嵌套包含或有其他属性），将会被递归合并，如上所示。

上面示例的合并会以完全相同的方式工作，而不管应用于作用域的顺序如何。如果某个选项由两个不同的作用域设置，则只会顺序有所不同（上面的示例不是这种情况，因为每个作用域执行的操作都不相同）。

此合并策略也与传递给`.findAll`、`.findOne`等的选项完全相同。



### 关联

对于关联，Sequelize有两个不同但相关的作用域概念。二者差异细微而重要：

- **关联作用域**(Association scopes) - 允许你在设置和获取关联时指定默认属性（在实现多态关系时很有用）。使用`get`、`set`、`add`和`create`关联的模型函数时，仅在两个模型之间的关联上调用此作用域
- **关联模型上的作用域**(Scopes on associated models) - 允许你在获取关联时应用默认作用域和其他作用域，并可以在创建关联时传递作用域模型。这些作用域既适用于模型上的常规查找，也适用于通过关联查找。

在下例中，一个Post和一个Comment模型。Comment会与其它几个模型关联（Image、Video等），并且Comment与其它模型之间的关联是多态的，也就是说Comment除了存储`commentable_id`外键外，还会存储一个`commentable`列。

多态关联可以通过关联作用域实现：

```
this.Post.hasMany(this.Comment, {
  foreignKey: 'commentable_id',
  scope: {
    commentable: 'post'
  }
});
```

调用`post.getComments()`时，会自动添加`WHERE commentable ='post'`。同样，在帖子(Post)中添加新评论(Comment)时，`commentable`会自动设置为`'post'`。关联作用域会驻留在后台，而程序员不必担心（无法禁用）。更多多态相关示例，请参考：[关联作用域](https://itbilu.com/nodejs/npm/l#advanced-concepts-scopes)。

再考虑，该贴子会有默认作用域，该作用域仅会显示活动的贴子：`where: { active: true }`。该作用域位于关联的模型（Post）上，而不像`commentable`作用域那样位于关联上。就像调用`Post.findAll()`时应用默认作用域一样，调用`User.getPosts()`时也应用默认作用域（只会返回该用户的活动帖子）。

要禁用作用域，可以将`scope: null`传给getter：`User.getPosts({ scope: null })`。同样，如果要应用其它作用域，则将数组传入，就像传给`.scope`一样：

```
User.getPosts({ scope: ['scope1', 'scope2']});
```

如果要为关联模型上的作用域创建快捷方式，可以将作用域模型传给关联。如，获取用户所有已删除帖子的快捷方式：

```
class Post extends Model {}
Post.init(attributes, {
  defaultScope: {
    where: {
      active: true
    }
  },
  scopes: {
    deleted: {
      where: {
        deleted: true
      }
    }
  },
  sequelize,
});

User.hasMany(Post); // regular getPosts association
User.hasMany(Post.scope('deleted'), { as: 'deletedPosts' });
User.getPosts(); // WHERE active = true
User.getDeletedPosts(); // WHERE deleted = true
```



## 14. 主从复制(Read replication)

Sequelize支持主从复制/读复制，即，当要执行SELECT查询时，可以连接多个服务器。当执行主从复制时，可以指定一台或多台服务器充当读取副本，并指定一台服务器充当写入主服务器，该服务器会处理所有写入和更新并将它们传到副本（实际的复制过程**不是**由Sequelize，而是后端数据库）。

```
const sequelize = new Sequelize('database', null, null, {
  dialect: 'mysql',
  port: 3306
  replication: {
    read: [
      { host: '8.8.8.8', username: 'read-username', password: 'some-password' },
      { host: '9.9.9.9', username: 'another-username', password: null }
    ],
    write: { host: '1.1.1.1', username: 'write-username', password: 'any-password' }
  },
  pool: { // If you want to override the options used for the read/write pool you can do so here
    max: 20,
    idle: 30000
  },
})
```

如果有适用于所有副本的通用设置，则无需为每个实例提供。在上面的代码中，数据库名称和端口将传递到所有副本，用户名、密码同样也是。每个副本都会有以下选项：`host`、`port`、`username`、`password`、`database`

Sequelize使用连接池来管理与副本的连接。在内部，Sequelize将根据`pool`配置创建的两个连接池。

如果要修改这些设置，可以在实例化Sequelize时将`pool`作为选项传入，如上所示。

每个`write`和`useMaster: true`查询都将使用写入池。对于SELECT，将使用取池。并使用基本的循环调度来切换只读副本。



## 15. 迁移(Migrations)

- CLI(Command line interface-命令行界面)
  - [安装 CLI](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#installing-cli)
  - 引导
    - [配置](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#configuration)
  - [创建第一个模型 (并迁移)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#creating-first-model--and-migration-)
  - [执行迁移](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#running-migrations)
  - [撤销迁移](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#undoing-migrations)
  - [创建第一个种子](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#creating-first-seed)
  - [执行种子](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#running-seeds)
  - [撤销种子](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#undoing-seeds)
- 高级话题
  - [“迁移”的结构](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#migration-skeleton)
  - [`.sequelizerc`文件](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#the--code--sequelizerc--code--file)
  - [动态配置](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#dynamic-configuration)
  - [使用Babel](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#using-babel)
  - [使用环境变量](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#using-environment-variables)
  - [指定方言选项](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#specifying-dialect-options)
  - [用于生产](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#production-usages)
  - 存储
    - [“迁移”存储](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#migration-storage)
    - [“种子”存储](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#seed-storage)
  - [配置连接字符串](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#configuration-connection-string)
  - [传入方言特定选项](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#passing-dialect-specific-options)
  - [程序化使用](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#programmatic-use)
- [查询接口](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#query-interface)

就像使用 Git/SVN 管理代码一样，你可以使用迁移功能（Migrations）来跟踪数据库的更改。通过迁移功能，你可以将现有数据库转移到另一个状态，反之亦然。进行迁移时，状态转换会被保存到迁移文件中，这些文件描述了如何进入新状态以及如何恢复更改以恢复到旧状态。

迁移需要使用[Sequelize CLI](https://github.com/sequelize/cli)，CLI提供了对迁移和项目引导的支持。



### CLI(Command line interface-命令行界面)

#### 安装 CLI

首先，需要安装CLI：

```
$ npm install --save sequelize-cli
```



#### 引导

可以通过执行`init`命令，来创建一个空项目：

```
$ npx sequelize-cli init
```

这会生成以下目录：

- `config`：包含配置文件，这些文件会告诉CLI怎样连接数据库
- `models`：包含项目中的所有模型（[Model](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#models-definition)）
- `migrations`：包含所有迁移文件
- `seeders`：包含所有种子文件



##### 配置

继续后续操作前，我们需要使CLI能够连接到数据库，可以在`config/config.json`文件中配置数据库。该文件类似如下：

```
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "database_production",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```

现在编辑此文件并设置正确的数据库验证信息和方言。对象的键（如`development`）会在`model/index.js`上通过`process.env.NODE_ENV`来匹配（未定义时，`development`是默认值）。

**注意**：如果你配置的数据库还不存在，调用`db:create`命令即可自动创建。



#### 创建第一个模型 (并迁移)

正确配置CLI的配置文件后，就可以创建你的第一个迁移。可以通过一个命令来轻松的完成这一操作。

创建模型使用`model:generate`命令，该命令包含以下两个参数：

- `name`-模型名
- `attributes`-模型属性列表

接下来我们来创建一个名为`User`的模型：

```
$ npx sequelize-cli model:generate --name User --attributes firstName:string,lastName:string,email:string
```

以上命令会做以下两件事：

- 在`models`文件夹下创建模型文件`user`
- 在`migrations`文件夹下创建名称类似`XXXXXXXXXXXXXX-create-user.js`的迁移文件

注意：Sequelize只会使用模型文件，模型文件是对数据库中表的表示；迁移文件是对该模型的更改，或者说是CLI所要使用的表；而“迁移”可以认为对数据库修改的一次提交或日志。



#### 执行迁移

到目前为目，我们尚未在数据库中插入任何内容。刚刚我们为第一个模型`User`创建了所需的模型和迁移文件。

现在要在数据库中创建该表，这时需要执行`db:migrate`命令：

```
npx sequelize-cli db:migrate
```

上面命令会执行以下操作：

- 确保数据库中有一个名为`SequelizeMeta`的表，此表用于记录在当前数据库上运行的迁移
- 查找尚示执行的迁移文件，这一步通过`SequelizeMeta`表来实现。在本例中，将执行上一步中创建`的XXXXXXXXXXXXXX-create-user.js`迁移文件。
- 创建一个名为`Users`的表，其中包含迁移文件中指定的所有列。



#### 撤销迁移

现在我们的表已在数据库中创建并保存。通过迁移功能，只需运行命令即可恢复到旧状态。

撤销迁移可以使用`db:migrate:undo`命令，此命令将还原到最近的迁移：

```
npx sequelize-cli db:migrate:undo
```

要撤销所有迁移，可以使用`db:migrate:undo:all`命令。还可以通过`--to`选项来传递迁移文件名称，以恢复到指定的迁移。

```
$ npx sequelize-cli db:migrate:undo:all --to XXXXXXXXXXXXXX-create-posts.js
```



#### 创建第一个种子

有些情况下我们可能需要在一个表中插入一些默认数据。如：在前面的`User`表中创建一个演示用户。

要管理所有数据迁移，可以使用`seeders`。种子(Seed)文件表示数据的一些变化，可用于使用样本数据或测试数据填充数据库表。

创建种子文件可以使用`seed:generate`命令。现在我们创建一个种子文件，它会添加一个演示用户到`User`表中：

```
$ npx sequelize-cli seed:generate --name demo-user
```

这一命令会在`seeders`目录下创建一个种子文件，文件名类似`XXXXXXXXXXXXXX-demo-user.js`。它遵循与迁移文件相同的`up/down`语法。

现在编辑这个文件，以添加用户到`User`表。

```
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.bulkInsert('Users', [{
        firstName: 'John',
        lastName: 'Doe',
        email: 'demo@demo.com',
        createdAt: new Date(),
        updatedAt: new Date()
      }], {});
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.bulkDelete('Users', null, {});
  }
};
```



#### 执行种子

上一步我们创建了种子文件，但它还未提交到数据库中。这时可以通过`db:seed:all`命令实现：

```
$ npx sequelize-cli db:seed:all
```

这会执行种子文件，并向`User`表添加一个演示用户。

注意：与使用`SequelizeMeta`表的迁移不同，种子执行不会存储在任何地方。如果想对其重写，请参考[存储](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#storage)部分。



#### 撤销种子

使用种子存储后，种子的执行可以撤销。可以通过以下命令实现：

- `db:seed:undo`命令撤销最近的一次种子操作：

```
$ npx sequelize-cli db:seed:undo
```

- 还可以使用`--seed`撤销到指定的种子：

```
$ npx sequelize-cli db:seed:undo --seed name-of-seed-as-in-data
```

- 或使用`db:seed:undo:all`命令撤销所有种子：

```
$ npx sequelize-cli db:seed:undo:all
```



### 高级话题

#### “迁移”的结构

以下是一个迁移文件的结构：

```
module.exports = {
  up: (queryInterface, Sequelize) => {
    // logic for transforming into the new state
  },

  down: (queryInterface, Sequelize) => {
    // logic for reverting the changes
  }
}
```

可以用`migration:generate`生成此文件，该命令会在`migration`文件夹中创建`xxx-migration-skeleton.js`文件：

```
$ npx sequelize-cli migration:generate --name migration-skeleton
```

通过传`queryInterface`对象来修改数据库。`Sequelize`对象中存储了数据类型，如：`STRING`和`INTEGER`。`up`和`down`函数需要返回一个`Promise`。以下是一些示例：

```
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Person', {
        name: Sequelize.STRING,
        isBetaMember: {
          type: Sequelize.BOOLEAN,
          defaultValue: false,
          allowNull: false
        }
      });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Person');
  }
}
```

下面是一个迁移示例，该迁移在数据库中会执行两次更改，并使用事务来确保所有指令均成功执行或在失败的情况下回滚：

```
module.exports = {
    up: (queryInterface, Sequelize) => {
        return queryInterface.sequelize.transaction((t) => {
            return Promise.all([
                queryInterface.addColumn('Person', 'petName', {
                    type: Sequelize.STRING
                }, { transaction: t }),
                queryInterface.addColumn('Person', 'favoriteColor', {
                    type: Sequelize.STRING,
                }, { transaction: t })
            ])
        })
    },

    down: (queryInterface, Sequelize) => {
        return queryInterface.sequelize.transaction((t) => {
            return Promise.all([
                queryInterface.removeColumn('Person', 'petName', { transaction: t }),
                queryInterface.removeColumn('Person', 'favoriteColor', { transaction: t })
            ])
        })
    }
};
```

接下来是一个有外键的迁移示例，可以用`references`来指定外键：

```
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Person', {
      name: Sequelize.STRING,
      isBetaMember: {
        type: Sequelize.BOOLEAN,
        defaultValue: false,
        allowNull: false
      },
      userId: {
        type: Sequelize.INTEGER,
        references: {
          model: {
            tableName: 'users',
            schema: 'schema'
          }
          key: 'id'
        },
        allowNull: false
      },
    });
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Person');
  }
}
```

接下来是使用`async/await`的迁移示例，我们会在新列上创建唯一索引：

```
module.exports = {
  async up(queryInterface, Sequelize) {
    const transaction = await queryInterface.sequelize.transaction();
    try {
      await queryInterface.addColumn(
        'Person',
        'petName',
        {
          type: Sequelize.STRING,
        },
        { transaction }
      );
      await queryInterface.addIndex(
        'Person',
        'petName',
        {
          fields: 'petName',
          unique: true,
        },
        { transaction }
      );
      await transaction.commit();
    } catch (err) {
      await transaction.rollback();
      throw err;
    }
  },

  async down(queryInterface, Sequelize) {
    const transaction = await queryInterface.sequelize.transaction();
    try {
      await queryInterface.removeColumn('Person', 'petName', { transaction });
      await transaction.commit();
    } catch (err) {
      await transaction.rollback();
      throw err;
    }
  },
};
```



#### `.sequelizerc`文件

`.sequelizerc`文件一个特殊的配置文件，它允许你指定通常作为参数传递给CLI的各种选项。可以使用它的一些场景：

- 需要重写默认的`migrations`, `models`, `seeders`或`config`文件夹。
- 需要重命名`config.json`时。如：重命名为`database.json`或其它。

险此之外，还有很多应用场景。让我们看看如何使用这个文件进行自定义配置。

首先，在项目根目录下创建`.sequelizerc`文件：

```
$ touch .sequelizerc
```

以下是一个示例配置：

```
const path = require('path');

module.exports = {
  'config': path.resolve('config', 'database.json'),
  'models-path': path.resolve('db', 'models'),
  'seeders-path': path.resolve('db', 'seeders'),
  'migrations-path': path.resolve('db', 'migrations')
}
```

在这个配置文件中，我们告诉CLI：

- 使用`config/database.json`进行配置设置
- 使用`db/models`做为模型目录
- 使用`db/seeders`做为种子目录
- 使用`db/migrations`做为迁移目录



#### 动态配置

配置文件默认是一个名为`config.json`的JSON文件，但有时你想执行一些代码或访问环境变量，这些操作在JSON文件中是不可能实现的。

Sequelize CLI可以从`JSON`或`JS`文件中读取配置，这可以通过`.sequelizerc`文件配置。

要使用`JS`格式的文件做为配置文件，可以在`.sequelizerc`文件中配置如下：

```
const path = require('path');

module.exports = {
  'config': path.resolve('config', 'config.js')
}
```

现在，Sequelize CLI会加载`config/config.js`以获得配置选项。由于这是一个JS文件，所以可以执行任何代码并导出最终的动态配置文件。

一个`config/config.js`文件示例：

```
const fs = require('fs');

module.exports = {
  development: {
    username: 'database_dev',
    password: 'database_dev',
    database: 'database_dev',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  test: {
    username: 'database_test',
    password: null,
    database: 'database_test',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  production: {
    username: process.env.DB_USERNAME,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOSTNAME,
    dialect: 'mysql',
    dialectOptions: {
      ssl: {
        ca: fs.readFileSync(__dirname + '/mysql-ca-master.crt')
      }
    }
  }
};
```



#### 使用Babel

现在你知道了如何使用`.sequelizerc`文件。现在，让我们看看如何文件结合babel在`sequelize-cli`中使用。这会使我们可以用`ES6/ES7`语法编写迁移和种子。

首先，安装`babel-register`：

```
$ npm i --save-dev babel-register
```

现在来创建`.sequelizerc`文件，它可以包含想要对`sequelize-cli`修改的配置，除此之外，我们还希望它为我们的代码库注册babel。像这样：

```
$ touch .sequelizerc # Create rc file
```

现在这个文件中包含了`babel-register`设置：

```
require("babel-register");

const path = require('path');

module.exports = {
  'config': path.resolve('config', 'config.json'),
  'models-path': path.resolve('models'),
  'seeders-path': path.resolve('seeders'),
  'migrations-path': path.resolve('migrations')
}
```

现在，CLI将能够从迁移/种子等程序运行ES6/ES7代码。请记住，这取决于你对`.babelrc`的配置。更多相关信息，请参考：[babeljs.io](https://babeljs.io/)。



#### 使用环境变量

使用CLI，可以直接访问`config/config.js`中的环境变量。可以通过`.sequelizerc`告诉CLI使用`config/config.js`进行配置。 在上一节中对此进行了说明。

然后，可以将需要的环境变量导出。

```
module.exports = {
  development: {
    username: 'database_dev',
    password: 'database_dev',
    database: 'database_dev',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  test: {
    username: process.env.CI_DB_USERNAME,
    password: process.env.CI_DB_PASSWORD,
    database: process.env.CI_DB_NAME,
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  production: {
    username: process.env.PROD_DB_USERNAME,
    password: process.env.PROD_DB_PASSWORD,
    database: process.env.PROD_DB_NAME,
    host: process.env.PROD_DB_HOSTNAME,
    dialect: 'mysql'
  }
};
```



#### 指定方言选项

有时需要指定一个`dialectOption`，如果是常规配置，则可以将其添加到`config/config.json`中。有时想通过执行代码来获取`dialectOption`，对于这种情况，应使用动态配置文件。

```
{
    "production": {
        "dialect":"mysql",
        "dialectOptions": {
            "bigNumberStrings": true
        }
    }
}
```



#### 用于生产

在生产环境中使用CLI和迁移设置的一些技巧。

1）使用环境变量进行配置设置。通过动态配置可以更好地实现这一点。一个简单的生产环境配置类似如下：

```
const fs = require('fs');

module.exports = {
  development: {
    username: 'database_dev',
    password: 'database_dev',
    database: 'database_dev',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  test: {
    username: 'database_test',
    password: null,
    database: 'database_test',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  production: {
    username: process.env.DB_USERNAME,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOSTNAME,
    dialect: 'mysql',
    dialectOptions: {
      ssl: {
        ca: fs.readFileSync(__dirname + '/mysql-ca-master.crt')
      }
    }
  }
};
```

我们的目标是将环境变量用于各种数据库安全，而不是无意中将其加入源代码管理。



#### 存储

CLI支持三种存储方式：`sequelize`、`json`、`none`

- `sequelize` : 在sequelize数据库的表中存储迁移和种子
- `json` : 在json文件中存储迁移和种子
- `none` : 不存储任何迁移、种子

##### “迁移”存储

默认情况下，CLI会在数据库中创建一个名为`SequelizeMeta`的表，其中包含每个已执行迁移的条目。要更改此行为，可以将三个选项添加到配置文件中。使用，您可以选择。 如果选择`json`，则可以使用`migrationStoragePath`指定文件的路径，或者CLI将写入文件`sequelize-meta.json`。 如果要使用`sequelize`将信息保留在数据库中，但希望使用其他表，则可以使用`migrationStorageTableName`更改表名。

```
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql",

    // Use a different storage type. Default: sequelize
    "migrationStorage": "json",

    // Use a different file name. Default: sequelize-meta.json
    "migrationStoragePath": "sequelizeMeta.json",

    // Use a different table name. Default: SequelizeMeta
    "migrationStorageTableName": "sequelize_meta",

    // Use a different schema for the SequelizeMeta table
    "migrationStorageTableSchema": "custom_schema"
  }
}
```

注意：不建议将`none`作为迁移存储。如果决定使用，应该充分考虑未记录迁移执行或未运行的影响。



##### “种子”存储

默认情况下，CLI不存储种子的执行记录。如果需要存储，可以使用`seederStorage`选项来配置，种子存储的各设置选项与迁移存储类似。当使用`json`存储时，可以使用`seederStoragePath`来指定路径，或者CLI使用默认的`sequelize-data.json`；如果需要存储在数据库中，则可以使用`sequelize`选项，这时可以通过`seederStorageTableName`来指定表名，默认将使用`SequelizeData`。

```
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql",
    // Use a different storage. Default: none
    "seederStorage": "json",
    // Use a different file name. Default: sequelize-data.json
    "seederStoragePath": "sequelizeData.json",
    // Use a different table name. Default: SequelizeData
    "seederStorageTableName": "sequelize_data"
  }
}
```



#### 配置连接字符串

<可以使用`--config`选项来替代数据库配置文件，使用`--url`选项传入连接字符串。例如：

```
$ npx sequelize-cli db:migrate --url 'mysql://root:password@mysql_host.com/database_name'
```



#### 传入方言特定选项

```
{
    "production": {
        "dialect":"postgres",
        "dialectOptions": {
            // dialect options like SSL etc here
        }
    }
}
```



#### 程序化使用

Sequelize有一个姊妹库：[umzug](https://github.com/sequelize/umzug)，用于以程序化处理迁移任务的执行和日志记录。



### 查询接口

使用前面介绍了通过`queryInterface`对象来更改数据库架构。查看该对象支持的方法完整列表，请参考：[QueryInterface API。](https://sequelize.org/master/class/lib/query-interface.js~QueryInterface.html)



## 16. 相关资源

- 插件（Addons & Plugins）
  - [ACL (Access control list-访问控制列表)](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#acl)
  - [自动代码生成 & 脚手架](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#auto-code-generation--amp--scaffolding)
  - [自动加载器](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#autoloader)
  - [缓存](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#caching)
  - [筛选器](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#filters)
  - [Fixtures / mock data](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#fixtures---mock-data)
  - [层次结构](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#hierarchies)
  - [历史记录 / 时间线](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#historical-records---time-travel)
  - [迁移](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#migrations)
  - [Slugification](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#slugification)
  - [Tokens](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#tokens)
  - [其它](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#miscellaneous)



### 插件（Addons & Plugins）

#### ACL (Access control list-访问控制列表)

- [ssacl](https://github.com/pumpupapp/ssacl)
- [ssacl-attribute-roles](https://github.com/mickhansen/ssacl-attribute-roles)

#### 自动代码生成 & 脚手架

- [meteor modeler](https://www.datensen.com/) - 用于可视化定义Sequelize模型和关联的桌面工具。
- [sequelize-ui](https://github.com/tomjschuster/sequelize-ui) - 用于构建模型、关系等的在线工具。
- [sequelizer](https://github.com/andyforever/sequelizer) - 用于生成Sequelize模型的GUI桌面应用程序。支持Mysql、Mariadb、Postgres、Sqlite、Mssql。
- [sequelize-auto](https://github.com/sequelize/sequelize-auto) - 通过命令行生成SequelizeJS的模型是另一种选择。
- [pg-generator](http://www.pg-generator.com/builtin-templates/sequelize/) - 为PostgreSQL数据库自动生成/搭建Sequelize模型。
- [sequelizejs-decorators](https://www.npmjs.com/package/sequelizejs-decorators) - 装饰器，用于组成sequelize模型

#### 自动加载器

- [sequelize-autoload](https://github.com/boxsnake-nodejs/sequelize-autoload) - Sequelize自动装带器。受[PSR-0](https://www.php-fig.org/psr/psr-0/)和[PSR-4](https://www.php-fig.org/psr/psr-4/)启发

#### 缓存

- [sequelize-transparent-cache](https://github.com/DanielHreben/sequelize-transparent-cache)

#### 筛选器

- [sequelize-transforms](https://www.npmjs.com/package/sequelize-transforms) - 添加可配置的属性转换

#### Fixtures / mock data

- [Fixer](https://github.com/olalonde/fixer)
- [Sequelize-fixtures](https://github.com/domasx2/sequelize-fixtures)
- [Sequelize-fixture](https://github.com/xudejian/sequelize-fixture)

#### 层次结构

- [sequelize-hierarchy](https://www.npmjs.com/package/sequelize-hierarchy) - Sequelize的嵌套层次结构。

#### 历史记录 / 时间线

- [sequelize-temporal](https://github.com/bonaval/sequelize-temporal) - 时间表（又名历史记录）

#### 迁移

- [umzug](https://github.com/sequelize/umzug)

#### Slugification

- [sequelize-slugify](https://www.npmjs.com/package/sequelize-slugify) - 为seqeulize模型添加slug

#### Tokens

- [sequelize-tokenify](https://github.com/pipll/sequelize-tokenify) - 为seqeulize模型添加唯一的Token

#### 其它

- [sequelize-deep-update](https://www.npmjs.com/package/sequelize-deep-update) - 使用新属性更新sequelize实例及其包含的关联实例。
- [sequelize-noupdate-attributes](https://www.npmjs.com/package/sequelize-noupdate-attributes) - 不向模型添加更新/只读属性支持
- [sequelize-joi](https://www.npmjs.com/package/sequelize-joi) - 允许在Sequelize中为JSONB模型属性指定[Joi](https://github.com/hapijs/joi)验证模式。



## 17. TypeScript

- [安装](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#typescript-installation)
- [使用](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#typescript-usage)
- [使用`sequelize.define`](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#typescript-usage-of-sequelize-define)

从v5开始，Sequelize提供了TypeScript定义。请需注意，仅支持`TS >= 3.1`。

由于Sequelize严重依赖于运行时属性分配，因此TypeScript在开箱即用时不会很有用。 为了使模型可用，需要大量的手动类型声明。

### 安装

为了避免非TS用户的臃肿安装，必须手动安装以下程序包：

- `@types/node` (这是必须的)
- `@types/validator`
- `@types/bluebird`



### 使用

最小的TypeScript项目示例：

```
import { Sequelize, Model, DataTypes, BuildOptions } from 'sequelize';
import { HasManyGetAssociationsMixin, HasManyAddAssociationMixin, HasManyHasAssociationMixin, Association, HasManyCountAssociationsMixin, HasManyCreateAssociationMixin } from 'sequelize';

class User extends Model {
  public id!: number; // Note that the `null assertion` `!` is required in strict mode.
  public name!: string;
  public preferredName!: string | null; // for nullable fields

  // timestamps!
  public readonly createdAt!: Date;
  public readonly updatedAt!: Date;

  // Since TS cannot determine model association at compile time
  // we have to declare them here purely virtually
  // these will not exist until `Model.init` was called.

  public getProjects!: HasManyGetAssociationsMixin; // Note the null assertions!
  public addProject!: HasManyAddAssociationMixin;
  public hasProject!: HasManyHasAssociationMixin;
  public countProjects!: HasManyCountAssociationsMixin;
  public createProject!: HasManyCreateAssociationMixin;

  // You can also pre-declare possible inclusions, these will only be populated if you
  // actively include a relation.
  public readonly projects?: Project[]; // Note this is optional since it's only populated when explicitly requested in code

  public static associations: {
    projects: Association;
  };
}

const sequelize = new Sequelize('mysql://root:asd123@localhost:3306/mydb');

class Project extends Model {
  public id!: number;
  public ownerId!: number;
  public name!: string;

  public readonly createdAt!: Date;
  public readonly updatedAt!: Date;
}

class Address extends Model {
  public userId!: number;
  public address!: string;

  public readonly createdAt!: Date;
  public readonly updatedAt!: Date;
}

Project.init({
  id: {
    type: DataTypes.INTEGER.UNSIGNED, // you can omit the `new` but this is discouraged
    autoIncrement: true,
    primaryKey: true,
  },
  ownerId: {
    type: DataTypes.INTEGER.UNSIGNED,
    allowNull: false,
  },
  name: {
    type: new DataTypes.STRING(128),
    allowNull: false,
  }
}, {
  sequelize,
  tableName: 'projects',
});

User.init({
  id: {
    type: DataTypes.INTEGER.UNSIGNED,
    autoIncrement: true,
    primaryKey: true,
  },
  name: {
    type: new DataTypes.STRING(128),
    allowNull: false,
  },
  preferredName: {
    type: new DataTypes.STRING(128),
    allowNull: true
  }
}, {
  tableName: 'users',
  sequelize: sequelize, // this bit is important
});

Address.init({
  userId: {
    type: DataTypes.INTEGER.UNSIGNED,
  },
  address: {
    type: new DataTypes.STRING(128),
    allowNull: false,
  }
}, {
  tableName: 'address',
  sequelize: sequelize, // this bit is important
});

// Here we associate which actually populates out pre-declared `association` static and other methods.
User.hasMany(Project, {
  sourceKey: 'id',
  foreignKey: 'ownerId',
  as: 'projects' // this determines the name in `associations`!
});

Address.belongsTo(User, {targetKey: 'id'});
User.hasOne(Address,{sourceKey: 'id'});

async function stuff() {
  // Please note that when using async/await you lose the `bluebird` promise context
  // and you fall back to native
  const newUser = await User.create({
    name: 'Johnny',
    preferredName: 'John',
  });
  console.log(newUser.id, newUser.name, newUser.preferredName);

  const project = await newUser.createProject({
    name: 'first!',
  });

  const ourUser = await User.findByPk(1, {
    include: [User.associations.projects],
    rejectOnEmpty: true, // Specifying true here removes `null` from the return type!
  });
  console.log(ourUser.projects![0].name); // Note the `!` null assertion since TS can't know if we included
                                          // the model or not
}
```



### 使用`sequelize.define`

当我们使`用sequelize.define`方法定义模型时，TypeScript不知道如何生成类定义。因此，我们需要做一些手工工作，声明一个接口和一个类型，并最终将`.define`的结果转换为静态类型。

```
// We need to declare an interface for our model that is basically what our class would be
interface MyModel extends Model {
  readonly id: number;
}

// Need to declare the static model so `findOne` etc. use correct types.
type MyModelStatic = typeof Model & {
  new (values?: object, options?: BuildOptions): MyModel;
}

// TS can't derive a proper class definition from a `.define` call, therefor we need to cast here.
const MyDefineModel = <MyModelStatic>sequelize.define('MyDefineModel', {
  id: {
    primaryKey: true,
    type: DataTypes.INTEGER.UNSIGNED,
  }
});

function stuffTwo() {
  MyDefineModel.findByPk(1, {
    rejectOnEmpty: true,
  })
  .then(myModel => {
    console.log(myModel.id);
  });
}
```



## 18. 升级到V5

- 重大变化
  - [支持Node 6及更高版本](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#support-for-node-6-and-up)
  - [安全操作符](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#upgrade-to-v5-secure-operators)
  - [Typescript 支持](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#upgrade-to-v5-typescript-support)
  - [连接池](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#upgrade-to-v5-pooling)
  - [模型](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#upgrade-to-v5-model)
  - [数据类型](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#upgrade-to-v5-datatypes)
  - [钩子](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#upgrade-to-v5-hooks)
  - [Sequelize](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#upgrade-to-v5-sequelize)
  - [查询接口](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#upgrade-to-v5-query-interface)
  - [其它](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#upgrade-to-v5-others)
  - 方言专用
    - [MSSQL](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#dialect-specific-mssql)
    - [MySQL](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#dialect-specific-mysql)
    - [MariaDB](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#dialect-specific-#mariadb)
  - [包](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#dialect-specific-packages)
- [修改日志](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#changelog)

Sequelize `v5`是`v4`之后的下一个主要版本



### 重大变化

#### 支持Node 6及更高版本

Sequelize v5仅支持Node 6及以上 [#9015](https://github.com/sequelize/sequelize/issues/9015)



#### 安全操作符

在v4中，开始收到弃用警告`String based operators are now deprecated`。还引入了操作符的概念，这些运算符是防止哈希注入攻击的符号。

[operators-security](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#querying-operators-security)

**在v5中**

- 运算符默认启用
- 仍然可以通过在`operatorAliases`中传递一个运算符映射来使用字符串运算符，但这会有弃用警告
- `Op.$raw`已删除



#### Typescript 支持

现在Sequelize正式发布[#10287](https://github.com/sequelize/sequelize/pull/10287) 。可以考虑从可能不同步的外部类型迁移。



#### 连接池

在v5中，Sequelize现在使用`sequelize-pool`，它是`generic-pool@2.5`的分支。你不再需要调用`sequelize.close`来关闭连接池，这有助于使用lambda表达式。[#8468](https://github.com/sequelize/sequelize/issues/8468)



#### 模型

**验证器**

现在，当属性的值为`null`且`allowNull`为`true`时，每个属性定义的自定义验证器（与模型选项中定义的自定义验证器相对）将运行（以前它们没有运行，并且验证立即成功）。为避免升级时出现问题，请检查每个属性定义的所有自定义验证器，如果`allowNull`为`true`，应确保当值为`null`时所有这些验证器行为正确。请参阅[#9143](https://github.com/sequelize/sequelize/issues/9143)。

**属性**

`Model.attributes`己被移移，应该使用`Model.rawAttributes`。[#5320](https://github.com/sequelize/sequelize/issues/5320)

**`Paranoid`模式**

在v5中，如果设置了`deletedAt`，则记录将被视为已删除。`paranoid`选项将仅使用`deletedAt`作为标志。[#8496](https://github.com/sequelize/sequelize/issues/8496)

**Model.bulkCreate**

原可以接受布尔值和数组的`updateOnDuplicate`选项，现在仅接受非空数组。[#9288](https://github.com/sequelize/sequelize/issues/9288)

**`Underscored`模式**

`Model.options.underscored`的实现已更改。可以在[这里](https://github.com/sequelize/sequelize/issues/6423#issuecomment-379472035)查看详细介绍。

主要概述

1. `underscoredAll`和`underscored`合并为一个`underscored`选项
2. 现在默认所有属性都使用驼峰命名法生成。`underscored`选项设置为`true`时，属性的`field`选项将设置为属性名称的下划线版本。
3. `underscored`会控制所有属性，包括时间戳、版本和外键，但不会影响已经指定`field`选项的属性。[#9304](https://github.com/sequelize/sequelize/pull/9304)

**移除的别名**

很多基于模型的别名已被删除[#9372](https://github.com/sequelize/sequelize/issues/9372)

| v5中移除的              | 官方替代        |
| :---------------------- | :-------------- |
| insertOrUpdate          | upsert          |
| find                    | findOne         |
| findAndCount            | findAndCountAll |
| findOrInitialize        | findOrBuild     |
| updateAttributes        | update          |
| findById, findByPrimary | findByPk        |
| all                     | findAll         |
| hook                    | addHook         |



#### 数据类型

**Range-范围**

现在仅支持新的标准格式：`[{ value: 1, inclusive: true }, { value: 20, inclusive: false }]` [#9364](https://github.com/sequelize/sequelize/pull/9364)

**不区分大小写的文本**

为Postgres和SQLite添加`CITEXT`类型支持

**移除**

`NONE`被移除，使用`VIRTUAL`代替



#### 钩子

**移除的别名**

已经移除的钩子别名[#9372](https://github.com/sequelize/sequelize/issues/9372)

| v5中移除的               | 官方替代                  |
| :----------------------- | :------------------------ |
| [after,before]BulkDelete | [after,before]BulkDestroy |
| [after,before]Delete     | [after,before]Destroy     |
| beforeConnection         | beforeConnect             |



#### Sequelize

**移除的别名**

很多常量、对象和类的原型引用已删除[#9372](https://github.com/sequelize/sequelize/issues/9372)

| v5中移除的                      | 官方替代              |
| :------------------------------ | :-------------------- |
| Sequelize.prototype.Utils       | Sequelize.Utils       |
| Sequelize.prototype.Promise     | Sequelize.Promise     |
| Sequelize.prototype.TableHints  | Sequelize.TableHints  |
| Sequelize.prototype.Op          | Sequelize.Op          |
| Sequelize.prototype.Transaction | Sequelize.Transaction |
| Sequelize.prototype.Model       | Sequelize.Model       |
| Sequelize.prototype.Deferrable  | Sequelize.Deferrable  |
| Sequelize.prototype.Error       | Sequelize.Error       |
| Sequelize.prototype[error]      | Sequelize[error]      |

```
import Sequelize from 'sequelize';
const sequelize = new Sequelize('postgres://user:password@127.0.0.1:mydb');

/**
 * In v4 you can do this
 */
console.log(sequelize.Op === Sequelize.Op) // logs `true`
console.log(sequelize.UniqueConstraintError === Sequelize.UniqueConstraintError) // logs `true`

Model.findAll({
  where: {
    [sequelize.Op.and]: [ // Using sequelize.Op or Sequelize.Op interchangeably
      {
        name: "Abc"
      },
      {
        age: {
          [Sequelize.Op.gte]: 18
        }
      }
    ]
  }
}).catch(sequelize.ConnectionError, () => {
  console.error('Something wrong with connection?');
});

/**
 * In v5 aliases has been removed from Sequelize prototype
 * You should use Sequelize directly to access Op, Errors etc
 */

Model.findAll({
  where: {
    [Sequelize.Op.and]: [ // Don't use sequelize.Op, use Sequelize.Op instead
      {
        name: "Abc"
      },
      {
        age: {
          [Sequelize.Op.gte]: 18
        }
      }
    ]
  }
}).catch(Sequelize.ConnectionError, () => {
  console.error('Something wrong with connection?');
});
```



#### 查询接口

- `changeColumn`不再生成带有`_idx`后缀的约束。现在，Sequelize没有为约束指定任何名称，默认为数据库引擎命名。这将同步`sync`、`createTable`和`changeColumn`的行为。

- ```
  addIndex
  ```

  别名选项别名已删除，使用以下替代

  - `indexName` => `name`
  - `indicesType` => `type`
  - `indexType`/`method` => `using`



#### 其它

- 现在，Sequelize对所有`INSERT/UPDATE`操作（`UPSERT`除外）使用参数化查询。它们提供了针对SQL注入攻击的更好保护。
- `ValidationErrorItem`现在在`original`属性而不是`__raw`属性中保留对原始错误的引用。
- [retry-as-promised](https://github.com/mickhansen/retry-as-promised)升级至`3.1.0`，其使用[any-promise](https://github.com/kevinbeaty/any-promise)。该模块重复所有`sequelize.query`操作。你可以将`any-promise`配置在Node 4或6上使用的`bluebird`。
- Sequelize将在`where`选项中抛出所有未定义的键，在过去的版本中`undefined`会转换为`null`。



#### 方言专用

##### MSSQL

- Sequelize现工作于`tedious >= 6.0.0`。 旧的`dialectOptions`必须更新以匹配其新格式。请参考[tedious文档](http://tediousjs.github.io/tedious/api-connection.html#function_newConnection)

以下是一个`dialectOptions`示例：

```
dialectOptions: {
  authentication: {
    domain: 'my-domain'
  },
  options: {
    requestTimeout: 60000,
    cryptoCredentialsDetails: {
      ciphers: "RC4-MD5"
    }
  }
}
```



##### MySQL

需要`mysql2 >= 1.5.2`



##### MariaDB

`dialect: 'mariadb'`现为`mariadb`包的支持



#### 包

- 移除：`terraformer-wkt-parser` [#9545](https://github.com/sequelize/sequelize/pull/9545)
- 移除：`generic-pool`
- 移除：`sequelize-pool`



### 修改日志

请参阅官方：[Changelog](https://sequelize.org/master/manual/upgrade-to-v5.html#changelog)



## 19. 使用遗留表

- [表](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#legacy-tables)
- [字段](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#legacy-fields)
- [主键](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#legacy-primary-keys)
- [外键](https://itbilu.com/nodejs/npm/sequelize-docs-v5.html#legacy-foreign-keys)

如果说Sequelize开箱即用，似乎有点夸张，它很容易处理旧表并通过定义（否则会自动生成）表和字段名称来验证你的应用。

### 表

```
class User extends Model {}
User.init({
  // ...
}, {
  modelName: 'user',
  tableName: 'users',
  sequelize,
});
```



### 字段

```
class MyModel extends Model {}
MyModel.init({
  userId: {
    type: Sequelize.INTEGER,
    field: 'user_id'
  }
}, { sequelize });
```



### 主键

默认情况下，Sequelize将假定你的表有`id`主键属性。

也可以自定义主键：

```
class Collection extends Model {}
Collection.init({
  uid: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    autoIncrement: true // Automatically gets converted to SERIAL for postgres
  }
}, { sequelize });

class Collection extends Model {}
Collection.init({
  uuid: {
    type: Sequelize.UUID,
    primaryKey: true
  }
}, { sequelize });
```

如果你的模型根本没有主键，可以使用`Model.removeAttribute('id');`



### 外键

```
// 1:1
Organization.belongsTo(User, { foreignKey: 'owner_id' });
User.hasOne(Organization, { foreignKey: 'owner_id' });

// 1:M
Project.hasMany(Task, { foreignKey: 'tasks_pk' });
Task.belongsTo(Project, { foreignKey: 'tasks_pk' });

// N:M
User.belongsToMany(Role, { through: 'user_has_roles', foreignKey: 'user_role_user_id' });
Role.belongsToMany(User, { through: 'user_has_roles', foreignKey: 'roles_identifier' });
```