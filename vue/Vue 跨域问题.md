## Vue 跨域问题

今天一个小老弟在那问我，哥给我看看代码，接口都测试通过了，怎么前端拿不到后端的数据，控制台老是报这个错误“Access-Control-Allow-Origin”。这就是跨域的问题。

**出现跨域问题的原因**

由于浏览器的同源策略导致，是浏览器对 javascript 施加的安全机制。通俗的讲就是两个不在同一个域名端口上的项目，在想获取资源时，一方对另一方施加访问的限制。

这里拿aaa.com域名来举例，aaa.com对应的外网IP地址为199.6.7.2

|  URL  | 是否存在跨域 |
|  :---  | ----  |
| http://www.aaa.com/a.js <br> http://www.aaa.com/b.js | 同一域名 不存在跨域 |
| http://www.aaa.com/aaa/a.js <br> http://www.aaa.com/bbb/b.js | 同一域名不同文件夹 不存在跨域 |
| http://www.aaa.com:8080/a.js <br> http://www.aaa.com:8000/b.js | 同一域名不同接口 存在跨域 |
| http://www.aaa.com/a.js <br> https://www.aaa.com/b.js | 同一域名不同协议 存在跨域 |
| http://www.aaa.com/a.js <br> http://199.6.7.2/b.js | 域名和对应域名的IP地址 存在跨域 |
| http://www.aaa.com/a.js <br> http://son.aaa.com/b.js | 同一域名不同子域 存在跨域 |
| http://www.aaa.com/a.js <br> http://www.son.aaa.com/b.js | 同一域名不同二级域名 存在跨域 |
| http://www.aaa.com/a.js <br> http://www.bbb.com/b.js | 不同域名 存在跨域 |

**解决跨域**

***1. Vue解决***

这是我们的前端项目地址：http://127.0.0.1:8080/

这是我们的后端接口地址：http://127.0.0.1:8000/

（1）在vue项目的根目录下，新建vue.config.js文件

（2）写入以下代码

```vue
module.exports = {
  // vue.js里解决跨域问题
  devServer: {
    proxy: {
      '/api': {
        target: 'http://127.0.0.1:8000/', // 目标地址
        changeOrigin: true,
        pathRewrite: {
          '^/api': '' // 需要重写的地址
        }
      }
    }
  }
}
```

说明：这里当前端访问到 /api 这个路径的时候，会将url代理到 http://127.0.0.1:8000/ 这个目标地址上，并将 /api 这个路径重写。

举个例子：这是我们后端提供的接口 http://127.0.0.1:8000/system/slider/list ，如果前端的axios直接访问这个接口的话，由于前端的端口是8080，后端的端口号是8000，会引起不同域名不同端口的跨域问题。于是，我们引入上面的代码段，让前端访问的接口改为 http://127.0.0.1:8080/api/system/slider/list 。这样前端访问的接口 http://127.0.0.1:8080/api 这一段url会代理到 http://127.0.0.1:8000 上，这样就满足了同源策略，因此解决跨域问题，接口得以正常访问。

***2. Nginx解决***

使用nginx作为代理服务器，这样用户只需要在80端口上进行前后端的交互，这样就能避免不同源的问题，因此来实现跨域。

（1）打开nginx.conf文件

（2）修改nginx配置为下面代码

```shell
server {
    listen 80;
        server_name 127.0.0.1;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }

    location ~ /api {
        proxy_pass http://127.0.0.1:8000;
    }
}
```

说明：Nginx监听80端口，将 http://127.0.0.1 的所有请求服务转发到127.0.0.1端口为8080；将 http://127.0.0.1/api 请求转发到 http://127.0.0.1:8000 上，nginx实现跨域的原理，实际上就是把web项目和后端接口项目放到同一个域中，这样满足同源的策略，然后根据请求地址去请求不同的服务器。

举个例子：当用户访问 http://127.0.0.1 这个地址时，会被nginx转发到 http://127.0.0.1:8080 这个地址上，这样就能访问到我们的前端项目；当用户想访问 http://127.0.0.1:8000/system/slider/list 这个接口时，就必须把接口url换个形式，用 http://127.0.0.1/api/system/slider/list 这个url来访问接口，让其接口请求转发到 http://127.0.0.1:8000/system/slider/list 这个接口url上。

**如有不当之处，欢迎指正，谢谢！！！**

***

> *如需转载，请加上本文的链接并标明出处*
>
> *[一条爱吃屎的狗：https://www.qnmlgb.top](https://www.qnmlgb.top/)*