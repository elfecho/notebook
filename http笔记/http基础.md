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
- Restful API 设计：把每个url当做一个唯一的资源

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

一些无需再次获取的数据（如html页面，图片，js，数据等）

一个Web资源（如html页面，图片，js，数据等）存在于Web服务器和客户端（浏览器）之间的副本。缓存会根据进来的请求保存输出内容的副本；当下一个请求来到的时候，如果是相同的URL，缓存会根据缓存机制决定是直接使用副本响应访问请求，还是向源服务器再次发送请求。比较常见的就是浏览器会缓存访问过网站的网页，当再次访问这个URL地址的时候，如果网页没有更新，就不会再次下载网页，而是直接使用本地缓存的网页。只有当网站明确标识资源已经更新，浏览器才会再次下载网页。浏览器和网站服务器是根据缓存机制进行缓存的

#### 为什么需要缓存

静态资源文件是基本不会改变的，没必要每次都从服务器中获取。也就是说，我们每次向
服务器发送请求得到的静态资源是相同的。所以我们可以把静态资源缓存再浏览器，也就
是客户端，来进行性能优化。

### http 缓存策略 （强制缓存 + 协商缓存）



### 刷新操作方式，对缓存的影响















- http常见的状态码有哪些
- http常见的header有哪些
- 什么是Restful API
- 描述一下http的缓存机制（重要）

