##  Django项目http改https协议

今天把博客启用ssl证书，这里也把方法分享出来，供大家做个参考

我的环境：

* 腾讯云主机
* nginx
* Django3.0项目

**1.申请免费的ssl证书**

首先在腾讯云申请个免费的ssl证书，有效期一年

购买网址：[https://buy.cloud.tencent.com/ssl](https://buy.cloud.tencent.com/ssl)；

在这个界面上证书种类选择域名型免费版(DV)，之后点”免费快速申请“

![](https://qnmlgb.top/media/editor/ssl_20210201143641838520.png)

填写资料后面选择DNS验证，记住这里的几个字段添加到你的域名解析中

![](https://qnmlgb.top/media/editor/application%20_20210201143703859651.png)

验证后下载你的证书，我这里用的是nginx，所以只用nginx里面的两个文件，将下载的这两个文件（我的是1_qnmlgb.top_bundle.crt和2_qnmlgb.top.key）上传到nginx中conf目录

![](https://qnmlgb.top/media/editor/nginx_20210201143815869049.png)

**2.centos主机配置**

进入你下载的nginx的源码目录

```python
[root@echo /]# cd usr/local/nginx-1.18.0/
```

安装pcre库和ssl

```python
[root@echo nginx-1.18.0]# yum -y install pcre
[root@echo nginx-1.18.0]# yum -y install openssl openssl-devel
```

执行下面的命令重新进行编译

```python
[root@echo nginx-1.18.0]# ./configure
[root@echo nginx-1.18.0]# ./configure --with-http_ssl_module
```

编译后执行make

```python
[root@echo nginx-1.18.0]# make
```

备份nginx运行程序

```python
[root@echo nginx-1.18.0]# cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
```

将新的nginx程序覆盖掉旧的

```python
[root@echo nginx-1.18.0]# cp objs/nginx /usr/local/nginx/sbin/nginx
```

没报任何东西说明操作成功，如果有报cannot错误，就把nginx关闭再覆盖

重新配置nginx.conf文件

```python
server {
    # 这里是你主要配置的部分  --begin--
    listen 443 ssl;    #网站运行的端口
    server_name www.qnmlgb.top qnmlgb.top;    #这里没有域名的话，就写你的服务器ip地址，使用ip来访问
    ssl_certificate /usr/local/nginx/conf/1_qnmlgb.top_bundle.crt;    #证书文件名称
    ssl_certificate_key /usr/local/nginx/conf/2_qnmlgb.top.key;   #私钥文件名称
    ssl_session_timeout 5m;
    ssl_session_cache shared:SSL:10m;
    #请按照以下协议配置
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; 
    #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
    ssl_prefer_server_ciphers on;
    charset utf-8;
    error_page 497  https://$host$request_uri;
    # 这里是你主要配置的部分  --end--
    
    #access_log  logs/host.access.log  main;
    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8999;  #端口要和uwsgi里配置的一样
        uwsgi_param UWSGI_SCRIPT DjangoBlog.wsgi;  #wsgi.py所在的目录名+.wsgi
        uwsgi_param UWSGI_CHDIR /data/wwwroot/myblog/; #项目路径
    }
}

server {
    listen 80;    #网站运行的端口
    server_name www.qnmlgb.top qnmlgb.top;    #这里没有域名的话，就写你的服务器ip地址，使用ip来访问
    charset utf-8;
    # 这里也要配置，别忘了
    return 301 https://$host$request_uri;  #访问http时跳转到https

    #access_log  logs/host.access.log  main;

    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8999;  #端口要和uwsgi里配置的一样
        uwsgi_param UWSGI_SCRIPT DjangoBlog.wsgi;  #wsgi.py所在的目录名+.wsgi
        uwsgi_param UWSGI_CHDIR /data/wwwroot/myblog/; #项目路径
    }
}
```

之后重启nginx就可以了，在网页访问https://www.qnmlgb.top就能正常访问了，如果访问http://www.qnmlgb.top会自动跳转到https://www.qnmlgb.top





***
> *如需转载，请加上本文的链接并标明出处*
>
> *[一条爱吃屎的狗：https://www.qnmlgb.top](https://www.qnmlgb.top/)*

