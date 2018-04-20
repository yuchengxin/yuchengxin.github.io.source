---
title: <font color=#0099ff face="微软雅黑">RESTFul API</font>
date: 2017-06-23 08:43:37
categories: 其它
tags: [Rest,微服务]
---
RESTFul API的特点：
==============
1、基于“资源”，数据也好、服务也好，在RESTFul设计里一切都是资源。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。你可以用一个URI（统一资源定位符）指向它，每种资源对应一个特定的URI。要获取这个资源，访问它的URI就可以，因此URI就成了每一个资源的地址或独一无二的识别符。所谓"上网"，就是与互联网上一系列的"资源"互动，调用它的URI。
2、无状态。一次调用一般就会返回结果，不存在类似于“打开连接-访问数据-关闭连接”这种依赖于上一次调用的情况。
3、URL中通常不出现动词，只有名词
4、URL语义清晰、明确
5、使用HTTP的GET、POST、DELETE、PUT来表示对于资源的增删改查
6、使用JSON不使用XML

有些同学可能会说，GET、POST我也经常用啊。但是在网站里的GET和POST同RESTFul中的GET、POST是不一样的。网站里使用GET、POST的选择点在于，简单的用GET、复杂对象用POST；但在REST里，GET对应的是查询一个资源，而POST对应的是新增一个资源，意义是决然不同的。理解这一点非常重要。

我们接着来看一看RESTFul API的一些最佳实践原则：
1、使用HTTP动词表示增删改查资源， GET：查询，POST：新增，PUT：更新，DELETE：删除
2、返回结果必须使用JSON
3、HTTP状态码，在REST中都有特定的意义：200，201,202,204,400,401,403,500。比如401表示用户身份认证失败，403表示你验证身份通过了，但这个资源你不能操作。
4、如果出现错误，返回一个错误码。比如我通常是这么定义的：
5、API必须有版本的概念，v1，v2，v3
6、使用Token令牌来做用户身份的校验与权限分级，而不是Cookie。
7、url中大小写不敏感，不要出现大写字母
8、使用 - 而不是使用 _ 做URL路径中字符串连接。
9、有一份漂亮的文档（很重要）

如何设计一套合理、好用的API
=====================
一、协议：API与用户的通信协议，总是使用HTTPs协议。
二、域名：应该尽量将API部署在专用域名之下。比如：
```https
https://api.example.com
```
三、版本（Versioning）应该将API的版本号放入URL:
```https
https://api.example.com/v1/
```
四、路径（Endpoint）路径又称"终点"（endpoint），表示API的具体网址。在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数。举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样:
```https
https://api.example.com/v1/zoos
https://api.example.com/v1/animals
https://api.example.com/v1/employees
```
五、HTTP动词:对于资源的具体操作类型，由HTTP动词表示。常用的HTTP动词有下面五个（括号里是对应的SQL命令）:

|HTTP动词|
|:-------|
|GET（SELECT）：从服务器取出资源（一项或多项）。|
|POST（CREATE）：在服务器新建一个资源。|
|PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。|
|PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。|
|DELETE（DELETE）：从服务器删除资源。|

还有两个不常用的HTTP动词:

|HTTP动词|
|:-------|
|HEAD：获取资源的元数据。|
|OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。|

六、过滤信息（Filtering）如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。下面是一些常见的参数:

 - ?limit=10：指定返回记录的数量
 - ?offset=10：指定返回记录的开始位置。
 - ?page=2&per_page=100：指定第几页，以及每页的记录数。
 - ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
 - ?animal_type_id=1：指定筛选条件
 
参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，GET /zoo/ID/animals 与 GET /animals?zoo_id=ID 的含义是相同的。

七、状态码（Status Codes）服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）:

 - 200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
 - 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
 - 202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
 - 204 NO CONTENT - [DELETE]：用户删除数据成功。
 - 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
 - 401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
 - 403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
 - 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
 - 406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
 - 410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
 - 422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
 - 500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
 
[状态码的完全列表][1]

八、错误处理（Error handling）如果状态码是4xx，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可。
```json
{
    error: "Invalid API key"
}
```

九、返回结果
针对不同操作，服务器向用户返回的结果应该符合以下规范:

 - GET /collection：返回资源对象的列表（数组）
 - GET /collection/resource：返回单个资源对象
 - POST /collection：返回新生成的资源对象
 - PUT /collection/resource：返回完整的资源对象
 - PATCH /collection/resource：返回完整的资源对象
 - DELETE /collection/resource：返回一个空文档

十、Hypermedia API
RESTful API最好做到Hypermedia（超媒体），即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。比如，当用户向api.example.com的根目录发出请求，会得到这样一个文档:
```json
{"link": {
  "rel":   "collection https://www.example.com/zoos",
  "href":  "https://api.example.com/zoos",
  "title": "List of zoos",
  "type":  "application/vnd.yourformat+json"
}}
```
上面代码表示，文档中有一个link属性，用户读取这个属性就知道下一步该调用什么API了。rel表示这个API与当前网址的关系（collection关系，并给出该collection的网址），href表示API的路径，title表示API的标题，type表示返回类型。

  [1]: https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html
  