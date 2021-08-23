## Vue+Django REST framework开发超市商城类项目

##### 项目简介

一个旅游类webapp项目，主要是查看旅游景点，下单订票流程的功能。使用vue+django来做前后端分离开发，可满足一般小型的课程设计，也可以完善里面的功能接口来做二次开发。

##### 技术栈

* 前端：Vue
* 后端：Django REST framework
* xadmin后台管理系统
* 数据库：MySQL

##### 系统功能

* 分类，子分类，搜索，热搜词，购物车简要展示。
* 新品分类展示，大类的推荐商品。
* 账户密码登录(json web token)。
* 注册，手机号码注册，错误提示，倒计时功能，服务器端手机号码发送频次限制。
* 商品大类，导航栏，筛选，排序，富文本。
* 商品收藏，加入购物车，结算，移出购物车。
* 留言 支付宝支付，扫码支付。跳回商户页面。
* 订单详情，收货地址。
* 个人信息，收藏商品，我的收藏。
* 留言，上传文件，提交，删除留言。

##### 界面预览

![](D:\人生苦短，我用python\images\index.png)

##### 项目部署

vue前端项目，直接 npm install \ npm run serve 后就能打开，前提你电脑上得有node环境。

django业务接口项目，新建一个虚拟环境，然后 pip install -r requirements.txt 安装项目依赖，安装完成后配置MySQL和Redis数据库，然后 python manage runserver 运行业务接口。

打开前端首页能显示数据就说明部署成功了。

##### 知识技能

vue部分:

- API 接口
- Vue 组件 与api的交互
- vue的项目组织结构分析

Django Rest Framework 技能

- 通用view实现 rest api接口
  - apiview方式实现api
  - genericView方式实现api接口
  - Viewset和router方式实现api接口和url配置
  - Django_filter searchFilter OrderFilter 分页
  - 通用mixin
- 权限和认证
  - Authentication用户认证设置
  - 动态设置permission、authentication
  - Validators实现字段验证
- 序列化和表单验证
  - Serializer
  - ModelSerializer
  - 动态设置Serializer

##### 项目仓库

前端界面代码：[https://github.com/hhdMrLion/mxshop-pc](https://github.com/hhdMrLion/mxshop-pc)

后端业务代码：[https://github.com/hhdMrLion/mxshop-api](https://github.com/hhdMrLion/mxshop-api)

