##  使用JWT Token进行用户认证

##### 基于Token的身份验证流程

* 客户端使用用户名跟密码请求登录
* 服务端收到请求开始验证用户名与密码
* 验证成功后，服务端生成一个Token并把这个Token发送给客户端
* 客户端收到Token以后可以把它存储起来，可以存放在Cookie里或者Local Storage里
* 客户端再次向服务端请求资源的时候携带服务端生成的Token发送给服务器
* 服务器收到请求，然后去验证客户端请求里面携带的Token，如果验证成功，就向客户端返回请求的数据，否则就拒绝请求

##### Token的组成形式

**Header 头部**

每个Token里面都有一个头部数据Header，里面包含了使用的算法并告诉我们这个Token是否加密，如果是未加密的Token，这个属性可以设置成None。

**Payload 数据**

Payload里面是Token要包含的一些数据，内容可以自行定义，也可以参考标准字段（简写：全程）iss：Issuer、sub：Subject、exp：Expiration time、iat：Issued at。

**Signature 签名**

将Header和Playload使用Base64编码生成一下再加入签名字符用加密算法加密一遍，得到唯一的签名，用来防止其他人来篡改Token中的信息。

# TODO



