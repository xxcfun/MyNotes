## Vue3.0手撸音乐播放器WebApp

##### 项目简介

使用QQ音乐的接口写的一个音乐webapp项目，整体体验和现有音乐播放器差不多，可以满足毕业设计。接口是现成的，使用现有接口也可以完成一些pc浏览器界面的开发。

##### 技术栈

* Vue3 全家桶
* nodejs 发送接口
* better-scroll 滚动插件
* vue3-lazy 图片懒加载插件
* good-storage 本地存储插件
* lyric-parser 歌词解析插件
* create-keyframe-animation 创建动画插件
* throttle-debounce 防抖节流插件

##### 系统功能

几乎媲美原生的音乐app

* 歌单、榜单、歌手、排行推荐
* 歌曲播放、三种播放模式、歌曲切换、歌词滚动
* 歌手歌单、详情列表、播放队列
* 歌曲搜索、排行、收藏
* 最近播放列表、喜欢列表、历史搜索等功能

##### 线上预览

[http://music.qnmlgb.top:2233/music-moyu](http://music.qnmlgb.top:2233/music-moyu)

##### 界面预览

![music](https://qnmlgb.top/media/editor/music_20211112091204485928.png)

##### 项目部署

vue前端项目，如在本地，电脑安装node后，直接 npm install \ npm run serve 后就能打开。

线上部署：

1. 申请域名服务器，并在服务器里面安装node环境和nginx服务器
2. 前端代码修改，部署根目录的子目录上
3. 添加npm指令，打包启用node服务一块启动，node运行在本地（详细可见vue.config.js和prod.server.js文件）
4. 修改base.js，启用线上接口的代理地址
5. 配置nginx.conf，将域名挂在到部署的子路径上并代理到运行的node服务
6. 浏览器输入域名即可访问，部署END

##### 技术知识

使用Vue3.0来开发，学习使用组合式api来进行开发，与2.0相比的提高了代码的可读性和逻辑的复用性。以前在使用2.0做开发的时候，组件变的越来越大的时候，代码的可读性大大的降低，各种method方法罗列，并且在相同逻辑的代码中很难进行代码的复用。

使用3.0后，将一个组件里面的静态变量，hook钩子函数，vuex，计算属性，watch监听和methods函数方法放一个setup中进行返回，代码的逻辑更加的明显，在拆分组件时也能进行整体的复用，不会存在误解的代码，代码的可读性也变得不那么的复杂。

使用good-storage 本地存储插件，也能更好的管理本地local存储，减少对接口api的调用，实现前端逻辑代码的优化，当然引用了storage也会帮助我们的vuex来进行mutations管理。

使用better-scroll滚动插件来模拟手机端的操作，以此来实现真实的手指滑动功能。lyric-parser配合scroll实现歌词的滚动功能。

在搜索页面的搜索框组件中，为了让传入的搜索关键词添加延迟响应的效果，使用防抖节流函数来进行限制。

整体项目为了减少对api的重复请求，也引用了vue3中的keep-alive的缓存功能，在歌曲播放时的歌曲准备上也应用了对歌曲的缓存。

##### 项目仓库

[https://github.com/xxcfun/vue-music](https://github.com/xxcfun/vue-music)

