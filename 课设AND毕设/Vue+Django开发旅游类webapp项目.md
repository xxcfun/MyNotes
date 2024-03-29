## Vue+Django开发旅游类webapp项目

##### 项目简介

一个旅游类webapp项目，主要是查看旅游景点，下单订票流程的功能。使用vue+django来做前后端分离开发，可满足一般小型的课程设计，也可以完善里面的功能接口来做二次开发。

##### 技术栈

* 前端：Vue + Element-UI
* 后端：Django
* 数据库：MySQL + Redis

##### 系统功能

* 查看热门景点、精选景点、搜索查找景点
* 查看景点详情信息、景点门票信息、景点评论信息
* 查看门票详情信息、预定景点门票、支付订单、取消订单
* 查看个人所有订单信息、待付款订单、成功订单、取消订单信息
* 用户的登录注册功能、系统界面登录保护
* django后台数据管理

##### 界面预览

![trip](https://qnmlgb.top/media/editor/1_20210609181427059468.png)

##### 项目部署

vue前端项目，直接 npm install \ npm run serve 后就能打开，前提你电脑上得有node环境。

django业务接口项目，新建一个虚拟环境，然后 pip install -r requirements.txt 安装项目依赖，安装完成后配置MySQL和Redis数据库，然后 python manage runserver 运行业务接口。

打开前端首页能显示数据就说明部署成功了。

##### 具体设计思路

本系统在设计时分为用户、景点、订单、后端管理四大模块，下面每个模块做简单的诠释。

>  用户模块设计

使用了django内置的用户模块，将用户类重写，添加自定义的用户字段和用户的基本信息表。这里的登录注册模块也都引入表单的验证，保持与前端系统同步登录。在注册页面，使用redis缓存来模拟手机验证码的操作，如在真是运行环境中，这里需要调用发送短信验证码的接口，这里我们就使用redis来模拟操作。

> 景点模块设计

在首页，使用缓存来加载热门景点和精选景点列表，减少用户对数据库的压力。每一个景点下面关联景点的详细信息、评论信息、门票信息和景点的介绍，在每一个门票信息下面也都关联着门票的详细信息。在景点列表中，用户可以通过搜索框来查找想要游玩的景点。

> 订单模块设计

订单模块设计了全部订单、待支付、已完成和已取消四种状态，可供用户随机切换查看。在预定门票操作后，进入待支付状态，这里可以点击支付来完成订单的支付，点击取消可以取消对该订单的操作。在已支付、已取消的订单列表中，用户可以对该信息进行删除操作。这里的提交订单、修改删除订单的操作由同一个接口完成，三种不同的接口响应方式分别对应着增删改的操作。

> 后端管理模块

使用django自带的后台管理，稍加配置就能直接使用，添加几个快速筛选的功能以便更好维护后端数据。也引入了富文本应用，让前端的图文模块更加生动。

##### 项目仓库

前端界面代码：[https://github.com/xxcfun/trip-mobile](https://github.com/xxcfun/trip-mobile)

后端业务代码：[https://github.com/xxcfun/trip](https://github.com/xxcfun/trip)