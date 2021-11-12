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

##### 系统功能

几乎媲美原生的音乐app

* 主要功能包括歌单、榜单、排行推荐
* 歌手、详情列表
* 歌曲搜索、播放、排行、收藏等功能。

##### 界面预览

![music](https://qnmlgb.top/media/editor/music_20211112091204485928.png)

##### 项目部署

vue前端项目，电脑安装node后，直接 npm install \ npm run serve 后就能打开。

##### 技术知识

使用Vue3.0来开发，学习使用组合式api来进行开发，与2.0相比的提高了代码的可读性和逻辑的复用性。以前在使用2.0做开发的时候，组件变的越来越大的时候，代码的可读性大大的降低，各种method方法罗列，并且在相同逻辑的代码中很难进行代码的复用。

使用3.0后，将一个组件里面的静态变量，hook钩子函数，vuex，计算属性，watch监听和methods函数方法放一个setup中进行返回，代码的逻辑更加的明显，在拆分组件时也能进行整体的复用，不会存在误解的代码，代码的可读性也变得不那么的复杂。

使用good-storage 本地存储插件，也能更好的管理本地local存储，减少对接口api的调用，实现前端逻辑代码的优化，当然引用了storage也会帮助我们的vuex来进行mutations管理。

TODO 未完

##### 项目仓库

[https://github.com/xxcfun/vue-music](https://github.com/xxcfun/vue-music)