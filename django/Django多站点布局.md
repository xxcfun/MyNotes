## Django多站点布局

现在，假设你手上只有一个云主机，你想在上面部署多个django项目，或者是前端vue项目，这样，你就得在nginx上建立多个conf项目进行单独的管理，场景现在有了，下面让我们hey,we go!

在我上一个文章中，写了用nginx+uwsgi来部署我们的django项目，有需要的朋友点这个传送门：[nginx+uwsgi部署django传送门]()

**开始配置教学**

进入nginx的conf文件夹，编辑nginx.conf

```shell
vim /usr/local/nginx/conf/nginx.conf
```

里面的内容改成下面这样

```python
worker_processes  1;

events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    include /usr/local/nginx/conf/vhost/*.conf;     #主要是这个地方，把新建的两配置文件>包含进来
    server {
        listen 80 default_server;
        server_name _;
        return 404;
    }
}
```

还是这个目录，新建一个vhost文件夹，这里存放你所有站点的配置，比如你想部署的两个站点名称分别是mywebA，mywebB。那么就在这个文件夹下新建mywebA.conf和mywebB.conf，里面的内容参考下面配置：

*1. mywebA.conf*

```python
# mywebA.conf
server {
    listen       80;    # 网站运行的端口，两个配置文件的端口不能一样
    server_name  www.mywebA.com;    #这里没有域名的话，就写你的服务器ip地址，使用ip来访问

    charset utf-8;

    #access_log  logs/host.access.log  main;

    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8999;  #端口要和uwsgi里配置的一样
        uwsgi_param UWSGI_SCRIPT mywebA.wsgi;  #wsgi.py所在的目录名+.wsgi
        uwsgi_param UWSGI_CHDIR /data/wwwroot/mywebA/; #项目路径
    }

    location /static/ {
        alias /data/wwwroot/mywebA/static/;
    }
}
```

*2. mywebB.conf*

```python
# mywebA.conf
server {
    listen       8080;    # 网站运行的端口，两个配置文件的端口不能一样
    server_name  www.mywebB.com;    #这里没有域名的话，就写你的服务器ip地址，使用ip来访问

    charset utf-8;

    #access_log  logs/host.access.log  main;

    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8998;  #端口要和uwsgi里配置的一样
        uwsgi_param UWSGI_SCRIPT mywebB.wsgi;  #wsgi.py所在的目录名+.wsgi
        uwsgi_param UWSGI_CHDIR /data/wwwroot/mywebB/; #项目路径
    }

    location /static/ {
        alias /data/wwwroot/mywebB/static/;
    }
}
```

这里有几个站点就写几个.conf文件，之后重启nginx服务即可，这样就可以用你的ip+对应的端口来访问你的多个站点了。

```shell
./nginx -s reload
```

看到这里你们会觉得，这不是和你上一篇文章写nginx配置一模一样，无非就是多了几个conf文件。确实如此，前面我已经开始分别配置了，这样，又水了一篇文章。

**补充**

这上面是django项目用uwsgi来进行部署的，当然.conf也可以写你的vue配置文件，下面上一个简单的vue.conf来作说明。

```python
server {
    listen       3000;
    server_name  localhost;
    #charset koi8-r;
    #access_log  logs/host.access.log  main;
    location / {
        root   html;
        index  index.html index.htm;
    }
    # error_page  404              /404.html;
    # redirect server error pages to the static page /50x.html
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

**常见问题**

部署完后访问网站还是原来的样子

多半是没有重启nginx服务导致的，重启服务就能解决

服务器出现500 server error错误

这里一般是uwsgi未启动的错误，哪一个站点有问题就进入相应的站点重启uwsgi服务。都有错误就直接全部杀死uwsgi服务（killall -9 uwsgi），然后在进入相应的虚拟环境执行（uwsgi --ini uwsgi.ini ）来重新开启uwsgi服务，这样一般就能解决了。

如果上面还没法解决，多半你项目代码出问题了，或者是你程序的依赖没有完全安装，检查代码，多看看日志文件，一般一些错误看日志都是能解决的。

目前我自己只遇到上面两个错误，未来还有问题我会继续补充。

写到这里收笔了，希望能给小伙伴们起到微不足道的帮助，如有错误请批评指正。



***

> *如需转载，请加上本文的链接并标明出处*
>
> *[一条爱吃屎的狗：https://www.qnmlgb.top](https://www.qnmlgb.top/)*

