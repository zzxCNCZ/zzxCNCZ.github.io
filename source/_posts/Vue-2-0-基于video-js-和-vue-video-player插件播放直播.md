---
title: Vue 2.0 基于video.js 和 vue-video-player插件播放直播
date: 2019-09-05 09:30:37
categories:
- Vue 2.0
tags:
- 前端
---
##### Vue 2.0 基于video.js 和 vue-video-player插件播放直播

vue-video-player是基于video.js基础上封装的vue插件（官方文档不全很乱，但是能基本使用），以下介绍播放rtmp协议的视屏资源[各种直播协议介绍](https://savokiss.com/tech/web-live-tech-with-vue.html)

- 开始使用，安装

[Github地址](https://github.com/surmon-china/vue-video-player)

[npm地址](https://www.npmjs.com/package/vue-video-player)

```javascript
npm install vue-video-player --save
```

在项目中有三种使用方式（mount with global，mount with component，mount with ssr），根据自己的实际情况装载，我使用的是mount with component。

```javascript
import 'video.js/dist/video-js.css'
import {videoPlayer} from 'vue-video-player'
import 'videojs-flash' // 本文使用的是rtmp直播源，应用到的是flash技术，所以需要引入此插件

export default {
  components: {
    videoPlayer
  }
}
```

组件代码：

```html
 <videoPlayer v-if="videoLoad" class="vjs-custom-skin videoPlayer"
                   :playsinline="true"
                   ref="videoPlayer"
                   :options="playerOptions"
                   @play="onPlayerPlay($event)"
                   @pause="onPlayerPause($event)"
                   @ended="onPlayerEnded($event)"
                   @loadeddata="onPlayerLoadeddata($event)"
                   @waiting="onPlayerWaiting($event)"
                   @playing="onPlayerPlaying($event)"
                   @timeupdate="onPlayerTimeupdate($event)"
                   @canplay="onPlayerCanplay($event)"
                   @canplaythrough="onPlayerCanplaythrough($event)"
                   @ready="playerReadied"
                   @statechanged="playerStateChanged($event)">
      </videoPlayer>
```

option参数：

```javascript
 playerOptions: {
        width: '910', // 宽度控制
        height: '360',
        sources: [{
          type: 'rtmp/flv',  // 视屏资源类型
          src: ''  // 这边为空是为了动态加载，自己尝试可以直接写死地址，ep：'rtmp://192.168.1.1/live/resource'
        }],
        techOrder: ['flash'],
        autoplay: true,  // 自动播放
        controls: true // 视屏控制按钮是否展现
        // poster: "../../static/banner-4.png" // 封面图片
      }
```

videoPlayer的常用method，可以通过获取videoPlayer对象来调用

```javascript
computed: {
   	// 在computed中获取
    player () {
      return this.$refs.videoPlayer.player
    }
  },
    
 methods: {
   videoPlayerMethods () {
     this.player.pause();//暂停 相当于停止效果
		 this.player.load();//重新加载src
     this.player.play(); // 播放
     this.player.dispose() // 杀死当前组件
   }
 }    
```

videoPlayer的各种events见上面html部分代码。

- 进阶使用

1. 动态切换视屏源

   ```javascript
   load () {
     this.playerOptions.sources[0].src = 'rtmp://192.168.1.1/live/resource1'
     this.player.load()
   },
   ```

2. 出现问题  this.el_.vjs_getProperty（发现问题可能是在于当我这边样式让控件消失的时候flash 控件会再度加载导致出现问题），我是将videoPlayer组件放在emelentui 的dialog弹出层组件中的，当dialog组件关闭时会出现此情况。

   解决方案是通过强制重新加载dialog组件，刷新videoPlayer组件，并在dialog关闭时杀死videoPlayer组件

   ```javascript
    dialogClose () {  // dialog关闭事件
         this.videoLoad = false   // dialog刷新videoPlayer
         this.player.dispose()   // 杀死
       },
         
   init () {   // dialog 初始化
         this.visible = true  // dialog显示
         this.videoLoad = true  // dialog刷新videoPlayer
         this.playerOptions.sources[0].src = this.broadcastUrl  // 切换源
         this.$nextTick(() => {
           console.log(this.player)
         })
       },      
   ```

   

- 附加内容：

本文使用的服务端采用开源软件 [Node Media Server](https://github.com/illuspas/Node-Media-Server.git).
