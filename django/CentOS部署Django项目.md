##  CentOS部署Django项目

写一个部署文章，为以后做部署项目做个参照，这也快过年了，一般在春节前后，阿里云，华为云什么的会有一波大促销，200多块钱就能买到3年期限的单核1G的云主机，这里也可以领个优惠券来购买，*[点击可使用优惠券购买](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=0yk8mnrd)*。

系统我这边安装的是CentOS 7.9，环境为uwsgi+nginx+python3+django2.2

> 下面开始写部署的详细步骤

**一. 升级你的软件并安装软件管理包和所用的依赖**

依次在shell运行下面的命令行即可，升级安装比较花时间，请耐心等待一下

```shell
[root@echo /]# yum update -y
[root@echo /]# yum -y groupinstall "Development"
[root@echo /]# yum install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel psmisc libffi-devel
```

**二. 下载python3并安装**

有些Linux系统中，都是自带python和python3的，如果你的系统中只有python2.17而没有python3，那就安装一个，这里选用最新版python3.9.1，这里有个汇总所有python版本的地址：[https://www.python.org/ftp/python/ ](https://www.python.org/ftp/python/)。

```shell
# 我一般喜欢在/usr/local路径下放所有下载的软件包，这样直接统一进行管理
[root@echo /]# cd /usr/local/
[root@echo local]# wget https://www.python.org/ftp/python/3.9.1/Python-3.9.1.tgz
```

下载完安装包后进行解压

```shell
[root@echo local]# tar -zxvf Python-3.9.1.tgz
```

进入Python-3.9.1文件夹

```shell
[root@echo local]# cd Python-3.9.1 
```

编译安装到/usr/local/python3

```sh
[root@echo Python-3.9.1]# ./configure --prefix=/usr/local/python3
```

安装

```shell
[root@echo Python-3.9.1]# make
[root@echo Python-3.9.1]# make install
```

为了在我们终端中直接使用python3这个命令，建立软连接，后面建的几个软连接也都是为了方便使用命令

```shell
[root@echo Python-3.9.1]# ln -s /usr/local/python3/bin/python3.9 /usr/bin/python3
```

pip3也一样建立软连接

```shell
[root@echo Python-3.9.1]# ln -s /usr/local/python3/bin/pip3.9 /usr/bin/pip3
```

回到根目录下，可以用下面的命令来查看python3和pip3是否安装成功

```shell
[root@echo /]# python3
Python 3.9.1 (default, Jan 14 2021, 09:08:21) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-44)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
[root@echo /]# pip3 -V
pip 20.2.3 from /usr/local/python3/lib/python3.9/site-packages/pip (python 3.9)
```

只要和上面提示的差不多，就说明安装成功了。

恭喜你，到这你成功了一小步

**三. 安装虚拟环境**

现在python3也是自带虚拟环境的，但习惯了virtualenv，还是安装一个管理我们所有的虚拟环境

```shell
[root@echo /]# pip3 install virtualenv
```

为方便使用virtualenv这个命令，我们也建立一下软连接

```shell
[root@echo /]# ln -s /usr/local/python3/bin/virtualenv /usr/bin/virtualenv
```

**四. 创建我们存放网站的文件夹**

在根目录下，创建一个data文件夹，再在data文件夹里面创建env文件夹和wwwroot文件夹，这个地方属于我自己的个人习惯，env来管理我们所有的虚拟环境，wwwroot来存放我们部署的网站文件

```shell
[root@echo /]# mkdir -p /data/env
[root@echo /]# mkdir -p /data/wwwroot
```

**五. 在env文件里创建我们的虚拟环境**

```shell
[root@echo /]# cd /data/env
[root@echo env]# virtualenv --python=/usr/bin/python3 myweb
created virtual environment CPython3.9.1.final.0-64 in 520ms
  creator CPython3Posix(dest=/data/env/myweb, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/root/.local/share/virtualenv)
    added seed packages: pip==20.3.3, setuptools==51.0.0, wheel==0.36.2
  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator
```

看到下面提示即创建成功

启动和退出虚拟环境的命令

```shell
[root@echo env]# source myweb/bin/activate
(myweb) [root@echo env]# deactivate 
```

前面能出现虚拟环境的名称，就代表启用成功，想退出就使用 deactivate 命令

**六. 安装uwsgi**

在系统里安装一次，在虚拟环境中再安装一次

```shell
[root@echo /]# pip3 install uwsgi
(myweb) [root@echo /]# pip3 install uwsgi
```

为方便使用uwsgi这个命令，继续做软链接

```shell
[root@echo /]# ln -s /usr/local/python3/bin/uwsgi /usr/bin/uwsgi
```

**七. 安装nginx**

进入软件的安装目录，下载nginx

```shell
[root@echo /]# cd /usr/local/
[root@echo local]# wget http://nginx.org/download/nginx-1.18.0.tar.gz
```

下载完成后进行解压

```shell
[root@echo local]# tar -zxvf nginx-1.18.0.tar.gz
```

进入nginx-1.18.0并进行安装

```sh
[root@echo local]# cd nginx-1.18.0
[root@echo nginx-1.18.0]# ./configure
[root@echo nginx-1.18.0]# make
[root@echo nginx-1.18.0]# make install
```

到这里，环境基本上是搞完了，下面开始上传你的网站代码

**八. 本地项目搬运到服务器上**

1. 上传sql文件

对本地数据库进行备份，使用MySQL数据库的话，直接用navicat可视化工具导出sql文件，然后上传到服务器的MySQL数据库中。如果使用的是sqlite，那么直接随着项目文件一块上传到服务器，这里使用ftp也行，宝塔运维面版都可以，新手的话，建议使用宝塔运维面板，面板直接给你集成了ftp服务和MySQL数据库，你以后上传php项目也可以用，这个东西很强大，具体怎么使用自己百度

2. 在本地项目中导出环境依赖包

```python
pip freeze > requirements.txt
```

3. 将项目的源码进行压缩打包，上传到服务器的wwwroot文件夹中，传完后进行解压缩

4. 使用之前创建的虚拟环境，怎么使用进入参考步骤五

5. 在你的虚拟环境中安装requirements.txt的依赖

```shell
(myweb) [root@echo myweb]# pip3 install requirements.txt 
```

6. 修改一下项目文件中的设置，在settings.py里面修改

```python
# 关闭生产环境中的DEBUG
DEBUG = False 
# 设置所有ip都可以访问该网站
ALLOWED_HOSTS = ['*']
# 在最下面添加样式收集目录，防止后端样式不生效
STATIC_ROOT  = os.path.join(BASE_DIR, 'static')
```

7. 收集后端的css样式，不运行的话，进入admin后台会出现样式丢失的情况

```python
(myweb) [root@echo myweb]# python3 manage.py collectstatic
```

8. python3 manage runserver 运行系统，看能否运行

**九. 配置uwsgi**

进入你网站的根目录下，创建一个uwsgi.ini文件，内容可以参考下面的配置

```shell
(myweb) [root@echo myweb]# vim uwsgi.ini
```

```ini
[uwsgi]
#配置和nginx连接的socket连接
socket=127.0.0.1:8999
#配置项目路径，项目的所在目录
chdir=/data/wwwroot/myweb/
#配置wsgi接口模块文件路径,也就是wsgi.py这个文件所在的目录名
wsgi-file=myweb/wsgi.py
#配置启动的进程数
processes=4
#配置每个进程的线程数
threads=2
#配置启动管理主进程
master=True
#配置存放主进程的进程号文件
pidfile=uwsgi.pid
#配置dump日志记录
daemonize=uwsgi.log
```

创建完成后保存退出，用下面的命令来启用uwsgi，如果显示下面的内容，那么运行成功

```shell
(myweb) [root@echo myweb]# uwsgi --ini uwsgi.ini 
[uWSGI] getting INI configuration from uwsgi.ini
```

运行成功后，输入ls命令，可以看见根目录下多了uwsgi.pid和uwsgi.log这两个文件

uwsgi.pid里面记录了uwsgi的进行pid；uwsgi.log记录uwsgi的日志记录，有什么错误可以在里面查看

再补充两个uwsgi的相关操作：

停止uwsgi

```shell
uwsgi --stop uwsgi.pid
```

重启uwsgi

```shell
uwsgi --reload uwsgi.pid
```

注意：如果程序有改动，务必重启uwsgi

**十. 配置nginx**

进入nginx的conf文件夹，编辑nginx.conf

```shell
[root@echo /]# cd /usr/local/nginx/conf/
[root@echo conf]# vim nginx.conf
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
    include /usr/local/nginx/conf/vhost/*.conf;     #主要是这个地方，把新建的配置文件全部包含进来
    server {
        listen 80 default_server;
        server_name _;
        return 404;
    }
}
```

还是这个目录，新建一个vhost文件夹，这里存放你网站的配置，在里面新建个myweb.conf

```shell
[root@echo conf]# mkdir vhost
[root@echo conf]# cd vhost
[root@echo vhost]# vim myweb.conf
```

myweb.conf内容参考下面的配置：

```python
server {
    listen       80;    #网站运行的端口
    server_name  www.myweb.com;    #这里没有域名的话，就写你的服务器ip地址，使用ip来访问

    charset utf-8;

    #access_log  logs/host.access.log  main;

    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8999;  #端口要和uwsgi里配置的一样
        uwsgi_param UWSGI_SCRIPT myweb.wsgi;  #wsgi.py所在的目录名+.wsgi
        uwsgi_param UWSGI_CHDIR /data/wwwroot/myweb/; #项目路径
    }

    location /static/ {
        alias /data/wwwroot/myweb/static/;
    }
}
```

配置好了就启动nginx服务，进入sbin文件夹执行下面的命令

```shell
[root@echo vhost]# cd /usr/local/nginx/sbin/
[root@echo sbin]# ./nginx 
```

这样用你的域名或者ip地址来访问，就能看到你的网站了

注意：如果程序有所改动，除了uwsgi要重启外，nginx也得进行重启

conf文件有改动，更应该重启nginx，重启命令如下：

```shell
[root@echo sbin]# ./nginx -s reload
```

**到这里，项目就部署好了**

***

**补充**

项目有修改，uwsgi、nginx都要进行重启，不然修改的地方不生效

如果出现500错误，而且你还找不到错误原因，一般都是程序片段有问题会报500错误，但也有一些情况，本身代码没问题，但浏览器报服务超时500错误，这时候你就得重新启动uwsgi服务，如果重启有报错，那就杀死uwsgi进程，重新进虚拟环境，重新启动uwsgi服务

查看uwsgi进行的命令：

```shell
(myweb) [root@echo myweb]# ps -ef|grep uwsgi 
```

杀死进程的命令，杀死后重新进行初始化

```shell
(myweb) [root@echo myweb]# killall -9 uwsgi
(myweb) [root@echo myweb]# uwsgi --ini uwsgi.ini 
```

这里重启完uwsgi后也要重启下nginx

```shell
[root@echo sbin]# ./nginx -s reload
```



善于从日志文件中定位错误位置，多尝试几次，多入几个坑，就有经验了

到这里，此文章也就结束，码了两千多字，希望能帮助各位小白们搭建自己的django项目，一般照着这个教程来，不大会出错，如有错误，可以在评论区进行讨论，在这里感谢下吴秀峰大大的博客给予的帮助，学习django的同学可以关注一波 [django中文网](https://www.django.cn)



***


> *如需转载，请加上本文的链接并标明出处*
>
> *[一条爱吃屎的狗：https://www.qnmlgb.top](https://www.qnmlgb.top/)*

