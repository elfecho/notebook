## Http常见的状态码

### 状态码分类

- 1xx 服务器收到请求
- 2xx 请求成功，如200
- 3xx 重定向，如302
- 4xx 客户端错误，如404
- 5xx 服务端错误，如500

### 常见状态码

- 200 成功
- 301 永久重定向（配合location,浏览器自动处理）
- 302 临时重定向（配合location,浏览器自动处理）
- 304 资源未被修改
- 404 资源未找到
- 403 没有权限
- 500 服务器错误
- 504 网关超时

## 关于协议和规范

- 一种约定

- 要求大家都按照规范执行、

- 不要违反规范，例如IE浏览器

## http methods

### 传统的methods

- get 获取服务器的数据
- post 像服务器提交数据
- 简单的网页功能，就这两个操作

### 现在的methods

- get 获取数据
- post 新建数据
- patch/put 更新数据 (patch 客户端提供改变后的完整资源 / put 客户端提供改变的属性)
- delete 删除数据

### Restful API

- 一种新的API设计方法（早于推广使用）
- 传统API设计：把每个url当做一个功能
- Restful API 设计：把每个url当做一个唯一资源的标识

更多详细内容可以查看： [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

### 如何设计成一个资源？

- 尽量不用url参数
- 用 method 表示操作类型

#### 不使用url参数

- 传统API设计：/api/list?pageIndex=2
- Restful API设计： /api/list/2

#### 用method表示操作类型（传统API设计）

- post 请求 /api/create-blog
- post 请求 /api/update-blog?id=100
- get 请求 /api/get-blog?id=100

#### 用method表示操作类型（Restful API设计）

- post 请求 /api/blog
- patch请求 /api/blog/100
- get 请求 /api/blog/100

## http headers

### 常见的Request Headers

- Accept 浏览器可接受的数据格式
- Accept-Encoding 浏览器可接受的压缩算法，如gzip
- Accept-Languange 浏览器可接受的语言，如zh-CN
- Connection: keep-alive 一次TCP连接重复使用，如keep-alive
- cookie
- Host
- User-Agent (简称UA) 浏览器信息
- Content-type 发送数据的格式(除get请求方式)，如application/json

### 常见的Response Headers

- Content-type 发送数据的格式，如application/json
- Content-length 返回数据的大小，多少字节
- Content-Encoding 返回数据的压缩算法，如gzip
- Set-Cookie 

### 自定义haeder

```javascript
headers: {
	'token': 'token参数'
}
```

### 缓存相关的headers

- Cache-Control	 Expires
- Last-Modified      If-Modified-Since
- Etag                       If-None-Match

## http 缓存

### 关于缓存的介绍

#### 什么是缓存

浏览器在本地磁盘上将用户之前请求的数据存储起来，当访问者再次需要改数据的时候无需再次发送请求，直接从浏览器本地获取数据

#### 为什么需要缓存

1. 减少请求的个数           

2. 节省带宽，避免浪费不必要的网络资源           
3. 减轻服务器压力（少买几台服务器）           
4. 提高浏览器网页的加载速度，提高用户体验

### http 缓存策略 （强制缓存 + 协商缓存）

#### 强制缓存

> 初次请求，返回资源和Cache-Control，在max-age未过期的时候从本地缓存中获取资源，当缓存过期重新从服务器拉取资源

#### 协商缓存

> 初次请求，返回资源和资源标识；当再次请求，带着资源标识，返回304 或 返回资源和新的资源标识

- 服务器端缓存策略
- 服务器判断客户端资源，是否和服务端资源一样
- 一致则返回304，否则返回200 和最新的资源

资源标识：

- Last-Modified 资源的最后修改时间
- Etag 资源的唯一标识（一个字符串，类似人类的指纹）

#### 缓存分类

```
1. 强制缓存
   1. 不会向服务器发送请求，直接从本地缓存中获取数据
   2. 请求资源的的状态码为: 200 ok(from memory cache)

2. 协商缓存
   1. 向服务器发送请求，服务器会根据请求头的资源判断是否命中协商缓存
   2. 如果命中，则返回304状态码通知浏览器从缓存中读取资源

3. 强制缓存 & 协商缓存的共同点
   1. 都是从浏览器端读取资源

4. 强制缓存 VS 协商缓存的不同点
   1. 强缓存不发请求给服务器
   2. 协商缓存发请求给服务器，根据服务器返回的信息决定是否使用缓存
```

#### 缓存流程示意图

![image-20210419145807740](https://gitee.com/blog2/blog/raw/master/images/image-20210419145807740.png)

#### 缓存中的header参数

##### 1、强缓存的header参数

1. expires：

这是http1.0时的规范；它的值为一个绝对时间的GMT格式的时间字符串，如Mon, 10 Jun 2015 21:31:12 GMT，如果发送请求的时间在expires之前，那么本地缓存始终有效，否则就会发送请求到服务器来获取资源
2. cache-control：max-age=number

这是http1.1时出现的header信息，是从expires发展过来的，主要是利用该字段的max-age值来进行判断，它是一个相对值；资源第一次的请求时间和Cache-Control设定的有效期，计算出一个资源过期时间，再拿这个过期时间跟当前的请求时间比较，如果请求时间在过期时间之前，就能命中缓存，否则就不行；

cache-control其他常用的值：

- no-cache: 不使用本地缓存，需要使用协商缓存。先与服务器确认返回的响应是否被更改，如果之前的响应中存在Etag，那么请求的额时候会与服务器端进行验证，如果资源为被更改则使用缓存。
- no-store: 直接禁止游览器缓存数据，每次用户请求该资源，都会向服务器发送一个请求，每次都会下载完整的资源。
- public：可以被所有的用户缓存，包括终端用户和CDN等中间代理服务器。
- private：只能被终端用户的浏览器缓存，不允许CDN等中继缓存服务器对其缓存。

##### 2、协商缓存的header参数

`重点：协商缓存都是由服务器来确定缓存资源是否可用的，所以客户端与服务器端要通过某种标识来进行通信，从而让服务器判断请求资源是否可以缓存访问`

**Last-Modified/If-Modified-Since:二者的值都是GMT格式的时间字符串**

1. 浏览器第一次跟服务器请求一个资源，服务器在返回这个资源的同时，在respone的header加上Last-Modified的header，这个header表示这个资源在服务器上的最后修改时间
2. 浏览器再次跟服务器请求这个资源时，在request的header上加上If-Modified-Since的header，这个header的值就是上一次请求时返回的Last-Modified的值
3. 服务器再次收到资源请求时，根据浏览器传过来If-Modified-Since和资源在服务器上的最后修改时间判断资源是否有变化，如果没有变化则返回304 Not Modified，但是不会返回资源内容；如果有变化，就正常返回资源内容。当服务器返回304 Not Modified的响应时，response header中不会再添加Last-Modified的header，因为既然资源没有变化，那么Last-Modified也就不会改变，这是服务器返回304时的response header
4. 浏览器收到304的响应后，就会从缓存中加载资源
5. 如果协商缓存没有命中，浏览器直接从服务器加载资源时，Last-Modified的Header在重新加载的时候会被更新，下次请求时，If-Modified-Since会启用上次返回的Last-Modified值

![image-20210419144850552](https://gitee.com/blog2/blog/raw/master/images/image-20210419144850552.png)

**Etag/If-None-Match（是从Last-Modified/If-Modified-Since发展过来的）**

1. 这两个值是由`服务器生成的每个资源的唯一标识字符串`，只`要资源有变化就这个值就会改变`
2. 其判断过程与Last-Modified/If-Modified-Since类似

![image-20210419144922165](https://gitee.com/blog2/blog/raw/master/images/image-20210419144922165.png)

###### Last-Modified和Etag

- 会优先使用Etag
- Last-Modified 只能精确到秒级
- 如果资源被重复生成，而内容不变，则Etag更精确

#### 小结

- `利用Etag能够更加准确的控制缓存`，因为Etag是服务器自动生成或者由开发者生成的对应资源在服务器端的唯一标识符。
- Last-Modified与ETag是可以一起使用的，**服务器会优先验证ETag**，一致的情况下，才会继续比对Last-Modified，最后才决定是否返回304。

#### 强缓存如何重新加载新的资源

通过更新页面中引用的资源路径，让浏览器主动放弃加载缓存去加载新的资源
示例：https://www.baidu.com/s?t=7aec0h3KB3Ba8lAbuyPg0AC0eDa59IvtDSmtMQBc6eW
好处：
每次文件改变后query的值就会发生修改，当query值不同的时候也就是页面引用的资源路径不同。此时浏览器会主动加载新的资源。

[http协商缓存VS强缓存](https://www.cnblogs.com/wonyun/p/5524617.html)

### 刷新操作方式，对缓存的影响

#### 三种刷新操作

- 正常操作：地址栏输入url，跳转链接，前进后退等
- 手动刷新：F5，点击刷新按钮，点击菜单刷新
- 强制刷新：ctrl + F5

#### 不同刷新操作，不同的缓存策略

- 正常操作：强制缓存有效，协商缓存有效
- 手动刷新：强制缓存失效，协商缓存有效
- 强制刷新：强制缓存失效，协商缓存失效

