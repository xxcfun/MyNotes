## RESTful API 接口规范指北

今天一学生给了我一份文档，看到三个post请求时，直接头大，这都1202年了，怎么还在一路post打天下。

<img src="C:\Users\honor\Desktop\新增.png" alt="新增" style="zoom:70%;" />

<img src="C:\Users\honor\Desktop\修改.png" alt="修改" style="zoom:70%;" />

<img src="C:\Users\honor\Desktop\删除.png" alt="删除" style="zoom:70%;" />

我：为了后来者能看到我们的代码，咱还是得按个规则来规范接口啊、、、

给我文档的学生：好，听你的。

##### RESTful API规范

现在很多应用都是前后端分离的样式了，为了统一管理接口，让后来者能看到能维护，也就流行起了API架构，于是在开发界有了这么一套规范--RESTful API规范。为了能写一套优雅的接口，后面还是得跟着大佬们多多学习了。

##### 1.域名

在大型项目中，api的接口要全部部署在专用的域名下面：

* http://api.project.com

当然你的api设计的比较简单，数量不多，可以放主域名下面：

* http://project.com/api/

放一块也是会了以后能统一进行管理。

##### 2.版本

在项目升级换代中，接口的版本也是要进行升级的，这里我们一般会把接口的版本放url中：

* http://api.project.com/v1/

##### 3.路径

每一个资源点的名称都是一个名词，这里我们的路径也全由名词进行设计。

* http://api.project.com/v1/classes
* http://api.project.com/v1/students
* http://api.project.com/v1/teachers

单复数最好也在url中进行表示，这样开发者一看，就知道是要访问一群学生还是单个学生了。

##### 4.HTTP请求

路径是名词，这里的请求就都是动词了。

* GET：获取数据，相当于sql中的select操作
* POST：新增数据，相当于sql中的create操作
* PUT：更新数据，相当于sql中的update操作
* PATCH：同上PUT，区别在于PUT是完整更新，而PATCH是局部更新
* DELETE：删除数据，相当于sql中的delete操作

常用请求就这五个，另外的HEAD，OPTIONS啥的不经常用，如用到可自行百度。

##### 5.筛选条件

在经常使用的查询操作中，往往需要查询的是成千上万条的数据，考虑数据过多，服务器顶着巨大压力不可能将所有的数据返回给用户，因此我们就得需要添加筛选条件，过滤返回结果，这里我们的url就得如此设计：

* ?limit=100：指定返回记录的数据
* ?page=1&count_page=10：指定返回第1页数据，以及每页的数据量
* ?sortby=add_time&order=asc：指定返回的数据按照某个属性进行排序，以及排序的顺序
* ?class_id=1：指定返回记录数据的筛选条件

在有多个筛选条件时，用&链接，后面的逻辑统统交给后端同学来判断。

##### 6.举栗子

* GET /classes：拿到所有班级信息
* GET /classes/?name="网络工程"：拿到班级名为“网络工程”的班级信息
* GET /classes/ID/students：拿到某个班级里面的所有学生信息
* GET /classes/ID/students/?name="萱萱"：拿到某个班级里面名字叫“萱萱”的学生信息
* POST /classes：新建班级信息
* PUT /classes/ID：更新某个班级信息（需要提供该班级的所有信息）
* PATCH /classes/ID：更新某个班级的信息（需要提供该班级的局部信息）
* DELETE /classes/ID：删除某个班级的信息

##### 7.状态码

常用的HTTP状态码如下：

* 200为服务器成功处理了请求，网页可以正常访问
  * 200 OK ：服务器成功返回用户请求的数据
  * 201 create ：用户新建、修改数据成功
  * 202 accepted ：服务器端已经收到请求消息，但是尚未进行处理
  * 204 无内容 ：服务器成功处理请求

* 400为用户请求出错，会妨碍服务器的处理
  * 400 bad request ：用户发送错误请求，服务器没有进行操作
  * 401 unauthorized ：用户没有权限
  * 403 forbidden ：用户有权限，但是访问请求被禁止
  * 404 not found ：用户请求的是不存在的记录（找不到对象）
  * 406 not acceptable ：用户请求格式不可得，无法使用请求的内容特性响应请求的网页
  * 410 gone ：用户请求的资源已被永久删除，无法拿到

* 500为服务器尝试处理请求时发生的内部错误，一般为服务器的本身错误，而不是请求错误
  * 500 internal server error ：服务器内部发生错误
  * 503 service unavailable ：服务当前无法处理请求

如需查看更多状态码访问[https://www.php.cn/web/web-http.html](https://www.php.cn/web/web-http.html)

##### 8.参考文档

[理解RESTful架构 | 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2011/09/restful.html)

[RESTful API 设计指南 | 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

[RESTful 架构详解 | 菜鸟教程](https://www.runoob.com/w3cnote/restful-architecture.html)

