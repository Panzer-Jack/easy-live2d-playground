<script setup lang="ts">
import { Config, CubismSetting, Live2DSprite, LogLevel } from 'easy-live2d'
import { Application, Ticker } from 'pixi.js'
import { onMounted, onUnmounted, ref } from 'vue'

const canvasRef = ref<HTMLCanvasElement>()
const app = new Application()

// 设置 Config 默认配置
Config.MotionGroupIdle = 'Idle' // 设置默认的空闲动作组
// Config.MouseFollow = false // 禁用鼠标跟随
Config.CubismLoggingLevel = LogLevel.LogLevel_Off // 设置日志级别

// 创建Live2D精灵 并初始化
const live2DSprite = new Live2DSprite()
live2DSprite.init({
  modelPath: '/Resources/Hiyori/Hiyori.model3.json',
  ticker: Ticker.shared,
})

// 监听点击事件
live2DSprite.onLive2D('hit', ({ hitAreaName, x, y }) => {
  console.log('hit', hitAreaName, x, y)
})

// 你也可以直接这样初始化
// const live2DSprite = new Live2DSprite({
//   modelPath: '/Resources/Huusya/Huusya.model3.json',
//   ticker: Ticker.shared
// })

onMounted(async () => {
  // 你同时又可以直接这样初始化
  // const model2Json = await (await fetch('/Resources/Hiyori/Hiyori.model3.json')).json()
  // const modelSetting = new CubismSetting({
  //   prefixPath: '/Resources/Hiyori/',
  //   modelJSON: model2Json,
  // })
  // live2DSprite.init({
  //   modelSetting,
  //   ticker: Ticker.shared,
  // })
  await app.init({
    view: canvasRef.value,
    backgroundAlpha: 0, // 如果需要透明，可以设置alpha为0
  })
  if (canvasRef.value) {
    // live2DSprite.x = -300
    // live2DSprite.y = -300
    live2DSprite.width = canvasRef.value.clientWidth * window.devicePixelRatio
    live2DSprite.height = canvasRef.value.clientHeight * window.devicePixelRatio
    app.stage.addChild(live2DSprite)

    live2DSprite.setExpression({
      expressionId: 'normal',
    })

    // 播放声音
    live2DSprite.playVoice({
      // 当前音嘴同步 仅支持wav格式
      voicePath: '/Resources/Hiyori/sounds/test3.wav',
    })

    // 停止声音
    // live2DSprite.stopVoice()

    setTimeout(() => {
      // 播放声音
      live2DSprite.playVoice({
        voicePath: '/Resources/Hiyori/sounds/test.wav',
        immediate: true // 是否立即播放: 默认为true，会把当前正在播放的声音停止并立即播放新的声音
      })
    }, 10000)

    // live2DSprite.startMotion({
    //   group: 'test',
    //   no: 0,
    //   priority: 3,
    // })
  }
})

onUnmounted(() => {
  // 释放实例
  live2DSprite.destroy()
})
</script>

<template>
  <div class="test" />
  <canvas
    id="live2d"
    ref="canvasRef"
  />
</template>

<style>
#live2d {
  position: absolute;
  top: 0%;
  right: 0%;
  width: 100%;
  height: 100%;
}

.test {
  display: inline-block;
  position: absolute;
  width: 100%;
  height: 70%;
  background-color: pink;
}
</style>
