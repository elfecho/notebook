[JsonSchema官方文档](https://links.jianshu.com/go?to=http%3A%2F%2Fjson-schema.org%2F)
[入门文档](https://links.jianshu.com/go?to=http%3A%2F%2Fjson-schema.org%2Flearn%2Fgetting-started-step-by-step.html)
[入门文档](https://links.jianshu.com/go?to=https%3A%2F%2Fjson-schema.org%2Funderstanding-json-schema%2Findex.html)
[生成Schema工具](https://links.jianshu.com/go?to=https%3A%2F%2Fjsonschema.net%2F)

## 使用Json的好处（什么是Schema）：

- 描述现有的数据格式
- 提供清晰的人工和机器可读文档
- 完整的数据结构，有利于自动化测试
- 完整的数据结构，有利于验证客户端提交数据的质量

## 什么是JSON Schema

- JSON Schema本身就是一种数据结构，可以清晰的描述JSON数据的结构。是一种描述JSON数据的JSON数据。

## 使用JSON Schema的好处

- JSON Schema 非常适用于基于JSON的HTTP的API。
- JSON Schema从Java的基本数据类型中对JSON结构进行校验，所以对JSON结构的校验可以理解为对每种不同数据类型的相应校验。
- 接口测试中可以快速的定位到自己数据格式的正确性。

## JSON模式示例

```json
{
    "$schema":"http://json-schema.org/draft-04/schema#",
    "title":"book info",
    "description":"some information about book",
    "type":"object",
    "properties":{
        "id":{
            "description":"The unique identifier for a book",
            "type":"integer",
            "minimum":1
        },
        "name":{
            "type":"string",
            "pattern":"^#([0-9a-fA-F]{6}$",
            "maxLength":6,
            "minLength":6
        },
        "price":{
            "type":"number",
            "multipleOf":0.5,
            "maximum":12.5,
            "exclusiveMaximum":true,
            "minimum":2.5,
            "exclusiveMinimum":true
        },
        "tags":{
            "type":"array",
            "items":[
                {
                    "type":"string",
                    "minLength":5
                },
                {
                    "type":"number",
                    "minimum":10
                }
            ],
            "additionalItems":{
                "type":"string",
                "minLength":2
            },
            "minItems":1,
            "maxItems":5,
            "uniqueItems":true
        }
    },
    "minProperties":1,
    "maxProperties":5,
    "required":[
        "id",
        "name",
        "price"
    ]
}
```



## JOSN模式中常用的关键字（加粗字体为常用字段）

更多详情请查看 [json-schema 文档](https://ajv.js.org/json-schema.html)

| 关键字           | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| $schema          | The $schema 关键字状态，这种模式被写入草案V4规范。           |
| title            | 将使用此架构提供一个标题，title一般用来进行简单的描述，可以省略 |
| description      | 架构的一点描述，description一般是进行详细的描述信息，可以省略 |
| **type**         | 用于约束校验的JSON元素的数据类型，是JSON数据类型关键字定义的第一个约束条件：它必须是一个JSON对象 |
| **properties**   | 定义属性：定义各个键和它们的值类型，最小和最大值中要使用JSON文件 |
| **required**     | 必需属性，这个关键字是数组，数组中的元素必须是字符串         |
| **minimum**      | 这是约束的值，并代表可接受的最小值                           |
| exclusiveMinimum | 如果“exclusiveMinimum”的存在，并且具有布尔值true的实例是有效的，如果它是严格的最低限度的值 |
| **maximum**      | 这是约束的值被提上表示可接受的最大值                         |
| exclusiveMaximum | 如果“exclusiveMaximum”的存在，并且具有布尔值true的实例是有效的，如果它是严格的值小于“最大”。 |
| multipleOf       | 数值实例有效反对“multipleOf”分工的实例此关键字的值，如果结果是一个整数。 |
| **maxLength**    | 字符串实例的长度被定义为字符的最大数目                       |
| **minLength**    | 字符串实例的长度被定义为字符的最小数目                       |
| **pattern**      | String实例被认为是有效的，如果正则表达式匹配成功实例         |

### 声明JSON模式

由于JSON Schema本身就是JSON，所以当一些东西是JSON Schema或者只是JSON的任意一块时，并不总是很容易分辨。该 ![schema关键字用于声明某些内容是JSON Schema，](https://math.jianshu.com/math?formula=schema%E5%85%B3%E9%94%AE%E5%AD%97%E7%94%A8%E4%BA%8E%E5%A3%B0%E6%98%8E%E6%9F%90%E4%BA%9B%E5%86%85%E5%AE%B9%E6%98%AFJSON%20Schema%EF%BC%8C)schema设置schema所使用的参照标准。包含它通常是一种很好的做法，尽管不是必需的。

```json
{ "$schema": "http://json-schema.org/schema#" }
```

### 声明唯一标识符

最佳做法是将$id属性包含为每个模式的唯一标识符。现在，只需将其设置为您控制的域中的URL，例如：

```json
{ "$id": "http://yourdomain.com/schemas/myschema.json"}
```

### type常见取值

Type其实就是JSON数据的基本数据类型，一般是有6种，加上null一共有7种：

- object
- array
- integer
- number 
- null
- boolean 
- string

#### type：Object

示例：

```json
{
    "type":"object",
    "properties":{
        "id":{
            "description":"The unique identifier for a book",
            "type":"integer",
            "minimum":1
        },
        "price":{
            "type":"number",
            "minimum":0,
            "exclusiveMinimum":true
        }
    },
    "patternProperties":{
        "^a":{
            "type":"number"
        },
        "^b":{
            "type":"string"
        }
    },
    "additionalProperties":{
        "type":"number"
    },
    "minProperties":1,
    "maxProperties":5,
    "required":[
        "id",
        "name",
        "price"
    ]
}
```

object类型有三个关键字:type（限定类型）,properties(定义object的各个字段),required（限定必需字段）

| 关键字               | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| type                 | 类型                                                         |
| properties           | 定义属性                                                     |
| required             | 必须属性                                                     |
| maxProperties        | 最大属性个数                                                 |
| minProperties        | 最小属性个数                                                 |
| additionalProperties | 如果待校验JSON对象中存在，既没有在properties中被定义，又没有在patternProperties中被定义，那么这些一级key必须通过additionalProperties的校验。true or false or object [参考](https://links.jianshu.com/go?to=https%3A%2F%2Fjson-schema.org%2Funderstanding-json-schema%2Freference%2Fobject.html) |

##### minProperties、maxProperties说明（用的不是很多）

这两个关键字的值都是非负整数。待校验的JSON对象中一级key的个数限制，minProperties指定了待校验的JSON对象可以接受的最少一级key的个数，maxProperties指定了待校验JSON对象可以接受的最多一级key的个数

##### patternProperties

正则表达式匹配json出现的属性，该JSON对象的每一个一级key都是一个正则表达式，用来匹配value值。

只有待校验JSON对象中的一级key，通过与之匹配的patternProperties中的一级正则表达式，对应的JSON Schema的校验，才算通过校验。例如，如果patternProperties对应的值如下：

在待校验JSON对象中，所有以S开头的key的value都必须是number，所有以I开头的一级key的value都必须是string。



```json
{
    "patternProperties": {
        "^S_": {
            "type": "number"
        },
        "^I": {
            "type": "string"
        }
    }
}
```

------

#### type：array

示例：

```bash
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "Product",
    "description": "A product from Acme's catalog",
    {
    "type":"array",
    "items":[
        {
            "type":"string",
            "minLength":5
        },
        {
            "type":"number",
            "minimum":10
        }
    ],
    "additionalItems":{
        "type":"string",
        "minLength":2
    },
    "minItems":1,
    "maxItems":5,
    "uniqueItems":true
}
```

array有三个单独的属性:items,minItems,uniqueItems:

| 关键字               | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| items                | array 每个元素的类型                                         |
| minItems             | 约束属性，数组最小的元素个数                                 |
| maxItems             | 约束属性，数组最大的元素个数                                 |
| uniqueItems          | 约束属性，每个元素都不相同                                   |
| additionalProperties | 约束items的类型，不建议使用 [示例](https://links.jianshu.com/go?to=https%3A%2F%2Fjson-schema.org%2Funderstanding-json-schema%2Freference%2Farray.html) |
| Dependencies         | 属性依赖 [用法](https://links.jianshu.com/go?to=https%3A%2F%2Fjson-schema.org%2Funderstanding-json-schema%2Freference%2Fobject.html) |
| patternProperties    | [用法](https://links.jianshu.com/go?to=https%3A%2F%2Fjson-schema.org%2Funderstanding-json-schema%2Freference%2Fobject.html) |

##### items

**该关键字的值是一个有效的JSON Schema或者一组有效的JSON Schema。**



```json
{
   "type": "array",
   "items": {
     "type": "string",
     "minLength": 5 
   }
}
```

上面的JSON Schema的意思是，待校验JSON数组的元素都是string类型，且最小可接受长度是5。那么下面这个JSON数组明显是符合要求的，具体内容如下：
 `["myhome", "green"]`
 下面这个JSON数组明显不符合要求
 `["home", "green"]`

**当该关键字的值是一组有效的JSON Schema时，只有待校验JSON数组的所有元素通过items的值中对应位置上的JSON Schema的校验，那么，整个待校验JSON数组才算通过校验。**

这里需要注意的是：**如果items定义的有效的JSON Schema的数量和待校验JSON数组中元素的数量不一致，那么就要采用“取小原则”**。
 即，如果items定义了3个JSON Schema，但是待校验JSON数组只有2个元素，这时，只要待校验JSON数组的前两个元素能够分别通过items中的前两个JSON Schema的校验，那么，我们认为待校验JSON数组通过了校验。而，如果待校验JSON数组有4个元素，这时，只要待校验JSON数组的前三个元素能够通过items中对应的JSON Schema的校验，我们就认为待校验JSON数组通过了校验。
 例如，如果items的值如下：



```json
{
    "type":"array",
    "items":[
        {
            "type":"string",
            "minLength":5
        },
        {
            "type":"number",
            "minimum":10
        },
        {
            "type":"string"
        }
    ]
}
```

上面的JSON Schema指出了待校验JSON数组应该满足的条件，数组的第一个元素是string类型，且最小可接受长度为5，数组的第二个元素是number类型，最小可接受的值为10，数组的第三个元素是string类型。那么下面这两个JSON数组明显是符合要求的，具体内容如下：
 `["green", 10, "good"]`
 `["helloworld", 11]`
 下面这两个JSON数组却是不符合要求的，具体内容如下：
 `["green", 9, "good"]`//9小于minimum
 `["good", 12]`//good小于minLength

##### additionalItems

该关键字的值是一个有效的JSON Schema。

需要注意的是，**该关键字只有在items关键字的值为一组有效的JSON Schema的时候，才可以使用，用于规定超出items中JSON Schema总数量之外的待校验JSON数组中的剩余的元素应该满足的校验逻辑**。只有这些剩余的所有元素都满足additionalItems的要求时，待校验JSON数组才算通过校验。

**可以这么理解，当items的值为一组有效的JOSN Schema的时候，一般可以和additionalItems关键字组合使用，items用于规定对应位置上应该满足的校验逻辑，而additionalItems用于规定超出items校验范围的所有剩余元素应该满足的条件。如果二者同时存在，那么只有待校验JSON数组同时通过二者的校验，才算真正地通过校验。**

另外，需要注意的是，**如果items只是一个有效的JSON Schema，那么就不能使用additionalItems**，原因也很简单，因为items为一个有效的JSON Schema的时候，其规定了待校验JSON数组所有元素应该满足的校验逻辑。additionalItems已经没有用武之地了。

强调一下，省略该关键字和该关键字的值为空JSON Schema，具有相同效果。



```json
{
    "type": "array",
    "items": [
        {
            "type": "string",
            "minLength": 5
        },
        {
            "type": "number",
            "minimum": 10
        }
    ],
    "additionalItems": {
        "type": "string",
        "minLength": 2
    }
}
```

上面的JSON Schema的意思是，待校验JSON数组第一个元素是string类型，且可接受的最短长度为5个字符，第二个元素是number类型，且可接受的最小值为10，剩余的其他元素是string类型，且可接受的最短长度为2。
 那么，下面三个JSON数组是能够通过校验的，具体内容如下：
 `["green", 10, "good"]`
 `["green", 11]`
 `["green", 10, "good", "ok"]`
 下面JSON数组是无法通过校验的，具体内容如下：
 `["green", 10, "a"]`
 `["green", 10, "ok", 2]`

##### minItems、maxItems

这两个关键字的值都是非负整数。

指定了待校验JSON数组中元素的个数限制，minItems指定了待校验JSON数组可以接受的最少元素个数，而maxItems指定了待校验JSON数组可以接受的最多元素个数。

另外，需要注意的是，省略minItems关键字和该关键字的值为0，具有相同效果。而，如果省略maxItems关键字则表示对元素的最大个数没有限制。
 例如，如果限制一个JSON数组的元素的最大个数为5，最小个数为1，则JSON Schema如下：
 `"minItems": 1,"maxItems": 5`

##### uniqueItems

该关键字的值是一个布尔值，即boolean（true、false）。

当该关键字的值为true时，只有待校验JSON数组中的所有元素都具有唯一性时，才能通过校验。当该关键字的值为false时，任何待校验JSON数组都能通过校验。

另外，需要注意的是，省略该关键字和该关键字的值为false时，具有相同的效果。例如：
 `"uniqueItems": true`

##### contains

注意：该关键字，官方说明中支持，但是，有可能你使用的平台或者第三方工具不支持。所以，使用需谨慎。

该关键字的值是一个有效的JSON Schema。

只有待校验JSON数组中至少有一个元素能够通过该关键字指定的JSON Schema的校验，整个数组才算通过校验。

另外，需要注意的是，省略该关键字和该关键字的值为空JSON Schema具有相同效果。

#### type：string

示例

```json
{
    "$schema":"http://json-schema.org/draft-04/schema#",
    "title":"Product",
    "description":"A product from Acme's catalog",
    "type":"object",
    "properties":{
        "ip":{
            "type":"string",
            "pattern":"w+([-+.]w+)*@w+([-.]w+)*.w+([-.]w+)*"
        },
        "host":{
            "type":"phoneNumber",
            "pattern":"((d{3,4})|d{3,4}-)?d{7,8}(-d{3})*"
        },
        "email":{
            "type":"string",
            "format":"email"
        }
    },
    "required":[
        "ip",
        "host"
    ]
}
```

| 关键字    | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| maxLength | 定义字符串的最大长度，>=0                                    |
| minLength | 定义字符串的最小长度，>=0                                    |
| pattern   | 用正则表达式约束字符串，只有待校验JSON元素符合该关键字指定的正则表达式，才算通过校验 |
| format    | 字符串的格式                                                 |

format该关键字的值只能是以下取值：date-time（时间格式）、email（邮件格式）、hostname（网站地址格式）、ipv4、ipv6、uri、uri-reference、uri-template、json-pointer。
 如果待校验的JSON元素正好是一个邮箱地址，那么，我们就可以使用format关键字进行校验，而不必通过pattern关键字指定复杂的正则表达式进行校验。

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

#### type：integer  or  number

示例

```json
{
    "$schema":"http://json-schema.org/draft-04/schema#",
    "title":"Product",
    "description":"A product from Acme's catalog",
    "type":"object",
    "properties":{
        "price":{
            "type":"number",
            "multipleOf":0.5,
            "maximum":12.5,
            "exclusiveMaximum":true,
            "minimum":2.5,
            "exclusiveMinimum":true
        }
    },
    "required":[
        "price"
    ]
}
```

integer和number的区别，integer相当于Java中的int类型，而number相当于Java中的int或float类型。
 number 关键字可以描述任意长度，任意小数点的数字。

| 关键字           | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| minimum          | 最小值                                                       |
| exclusiveMinimum | 如果存在 "exclusiveMinimum" 并且具有布尔值 true，如果它严格意义上大于 "minimum" 的值则实例有效。 |
| maximum          | 约束属性，最大值                                             |
| exclusiveMaximum | 如果存在 "exclusiveMinimum" 并且具有布尔值 true，如果它严格意义上小于 "maximum" 的值则实例有效。 |
| multipleOf       | 是某数的倍数，必须大于0的整数                                |

##### multipleOf

该关键字的值是一个大于0的number，即可以是大于0的int，也可以是大于0的float。
 只有待校验的值能够被该关键字的值整除，才算通过校验。
 如果含有该关键字的JSON Schema如下
 `{ "type": "integer", "multipleOf": 2 }`
 2、4、6都是可以通过校验的，但是，3、5、7都是无法通过校验的，当然了，2.0、4.0也是无法通过校验的，但是，并不是因为multipleOf关键字，而是因为type关键字。
 如果含有multipleOf关键字的JSON Schema如下
 `{ "type": "number", "multipleOf": 2.0 }`
 2、2.0、4、4.0都是可以通过校验的，但是，3、3.0、3、3.0都是无法通过校验的。
 另外，需要注意的是，省略该关键字则不对待校验数值进行该项校验。

##### maximum

该关键字的值是一个number，即可以是int，也可以是float。
 该关键字规定了待校验元素可以通过校验的最大值。
 省略该关键字，即表示对待校验元素的最大值没有要求

##### exclusiveMaximum

该关键字的值是一个boolean。
 该关键字通常和maximum一起使用，当该关键字的值为true时，表示待校验元素必须小于maximum指定的值；当该关键字的值为false时，表示待校验元素可以小于或者等于maximum指定的值。
 需要注意的是，省略该关键字和该关键字的值为false，具有相同效果。例如：
 `{ "type": "number", "maximum": 12.3, "exclusiveMaximum": true }`

##### minimum、exclusiveMinimum

minimum、exclusiveMinimum关键字的用法和含义与maximum、exclusiveMaximum相似。唯一的区别在于，一个约束了待校验元素的最小值，一个约束了待校验元素的最大值。

------

#### type：boolean

对应着true或者false

```json
{
    "type":"object",
    "properties":{
        "number":{
            "type":"boolean"
        }
    }
}
```

### 进阶

#### type：enum

该关键字的值是一个数组，该数组至少要有一个元素，且数组内的每一个元素都是唯一的。

如果待校验的JSON元素和数组中的某一个元素相同，则通过校验。否则，无法通过校验。

注意，该数组中的元素值可以是任何值，包括null。省略该关键字则表示无须对待校验元素进行该项校验。



```json
{
    "type":"object",
    "properties":{
        "street_type":{
            "type":"string",
            "enum":[
                "Street",
                "Avenue",
                "Boulevard"
            ]
        }
    }
}
```



```json
{
    "type":"object",
    "properties":{
        "street_type":[
            "Street",
            "Avenue",
            "Boulevard"
        ]
    }
}
```

#### type：const

该关键字的值可以是任何值，包括null。
 如果待校验的JSON元素的值和该关键字指定的值相同，则通过校验。否则，无法通过校验。
 省略该关键字则表示无须对待校验元素进行该项校验。
 注意，该关键字部分第三方工具，并不支持

#### 关键字：$ref

用来引用其他的schema
 示例：

```json
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "Product set",
    "type": "array",
    "items": {
        "title": "Product",
        "type": "object",
        "properties": {
            "id": {
                "description": "The unique identifier for a product",
                "type": "number"
            },
            "name": {
                "type": "string"
            },
            "price": {
                "type": "number",
                "minimum": 0,
                "exclusiveMinimum": true
            },
            "tags": {
                "type": "array",
                "items": {
                    "type": "string"
                },
                "minItems": 1,
                "uniqueItems": true
            },
            "dimensions": {
                "type": "object",
                "properties": {
                    "length": {"type": "number"},
                    "width": {"type": "number"},
                    "height": {"type": "number"}
                },
                "required": ["length", "width", "height"]
            },
            "warehouseLocation": {
                "description": "Coordinates of the warehouse with the product",
                "$ref": "http://json-schema.org/geo"
            }
        },
        "required": ["id", "name", "price"]
    }
}
```

#### 关键字definitions

当一个schema写的很大的时候，可能需要创建内部结构体，再使用$ref进行引用，示列如下：



```json
{
    "type": "array",
    "items": { "$ref": "#/definitions/positiveInteger" },
    "definitions": {
        "positiveInteger": {
            "type": "integer",
            "minimum": 0,
            "exclusiveMinimum": true
        }
    }
}
```

#### 关键字：allOf

该关键字的值是一个非空数组，数组里面的每个元素都必须是一个有效的JSON Schema。
 只有待校验JSON元素通过数组中所有的JSON Schema校验，才算真正通过校验。
 意思是展示全部属性，建议用requires替代，**不建议使用**，示例如下



```json
{
    "definitions":{
        "address":{
            "type":"object",
            "properties":{
                "street_address":{
                    "type":"string"
                },
                "city":{
                    "type":"string"
                },
                "state":{
                    "type":"string"
                }
            },
            "required":[
                "street_address",
                "city",
                "state"
            ]
        }
    },
    "allOf":[
        {
            "$ref":"#/definitions/address"
        },
        {
            "properties":{
                "type":{
                    "enum":[
                        "residential",
                        "business"
                    ]
                }
            }
        }
    ]
}
```

#### 关键字：anyOf

该关键字的值是一个非空数组，数组里面的每个元素都必须是一个有效的JSON Schema。
 如果待校验JSON元素能够通过数组中的任何一个JSON Schema校验，就算通过校验。
 意思是展示任意属性，建议用requires替代和minProperties替代，示例如下：



```json
 {
    "anyOf":[
        {
            "type":"string"
        },
        {
            "type":"number"
        }
    ]
}
```

#### 关键字：oneOf

该关键字的值是一个非空数组，数组里面的每个元素都必须是一个有效的JSON Schema。
 如果待校验JSON元素能且只能通过数组中的某一个JSON Schema校验，才算真正通过校验。不能通过任何一个校验和能通过两个及以上的校验，都不算真正通过校验。
 其中之一



```json
{
    "oneOf":[
        {
            "type":"number",
            "multipleOf":5
        },
        {
            "type":"number",
            "multipleOf":3
        }
    ]
}
```

#### 关键字：not

该关键字的值是一个JSON Schema。
 只有待校验JSON元素不能通过该关键字指定的JSON Schema校验的时候，待校验元素才算通过校验。
 非 * 类型



```json
{
    "not":{
        "type":"string"
    }
}
```

#### 关键字：default

需要特别注意的是，type关键字的值可以是一个string，也可以是一个数组。
 如果type的值是一个string，则其值只能是以下几种：null、boolean、object、array、number、string、integer。
 如果type的值是一个数组，则数组中的元素都必须是string，且其取值依旧被限定为以上几种。只要带校验JSON元素是其中的一种，则通过校验。





































