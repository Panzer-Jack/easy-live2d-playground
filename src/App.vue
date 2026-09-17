<script setup lang="ts">
import { Config, CubismSetting, Live2DSprite, LogLevel, Priority } from 'easy-live2d'
import { Application } from 'pixi.js'
import { onMounted, onUnmounted, ref } from 'vue'

const canvasRef = ref<HTMLCanvasElement>()
const ready = ref(false)
const errorMessage = ref('')
const app = new Application()
const loading = new AbortController()
let disposed = false
let appInitialized = false

Config.MotionGroupIdle = 'Idle'
Config.CubismLoggingLevel = LogLevel.LogLevel_Warning

// 示例一：直接使用模型路径。
const live2DSprite = new Live2DSprite({
  modelPath: '/Resources/Hiyori/Hiyori.model3.json',
  draggable: true,
})

// 示例二：读取配置后使用 CubismSetting 初始化。
const live2DSprite2 = new Live2DSprite()
const sprites = [live2DSprite, live2DSprite2]

function destroyResources() {
  loading.abort()
  for (const sprite of sprites)
    sprite.destroy()
  if (appInitialized) {
    app.destroy()
    appInitialized = false
  }
}

for (const sprite of sprites) {
  sprite.onLive2D('hit', ({ hitAreaName, x, y }) => {
    console.log('hit', hitAreaName, x, y)
    void sprite.startMotion({ group: 'TapBody', no: 0, priority: Priority.Normal })
      .catch(error => console.error('动作播放失败', error))
  })
}

onMounted(async () => {
  try {
    const response = await fetch('/Resources/Hiyori/Hiyori.model3.json', { signal: loading.signal })
    if (!response.ok)
      throw new Error(`模型配置加载失败：HTTP ${response.status}`)
    const modelJSON = await response.json()
    if (disposed)
      return

    live2DSprite2.init({
      modelSetting: new CubismSetting({ prefixPath: '/Resources/Hiyori/', modelJSON }),
      draggable: true,
    })

    await app.init({
      canvas: canvasRef.value,
      backgroundAlpha: 0,
      autoDensity: true,
      resizeTo: window,
      resolution: Math.max(window.devicePixelRatio || 1, 1),
    })
    appInitialized = true
    if (disposed) {
      destroyResources()
      return
    }

    const canvas = canvasRef.value!
    live2DSprite.x = -150
    live2DSprite2.x = 150
    for (const sprite of sprites) {
      sprite.width = canvas.clientWidth
      sprite.height = canvas.clientHeight
      app.stage.addChild(sprite)
    }

    // 模型跟随 Pixi 渲染，无需传入 ticker 或额外 WebGL 配置。
    await Promise.all(sprites.map(sprite => sprite.ready))
    if (!disposed)
      ready.value = true
  } catch (error) {
    if (!disposed) {
      errorMessage.value = error instanceof Error ? error.message : String(error)
      console.error('模型初始化失败', error)
      destroyResources()
    }
  }
})

async function playVoice(sprite: Live2DSprite) {
  try {
    await sprite.playVoice({
      voicePath: '/Resources/Hiyori/sounds/test3.wav',
      immediate: true,
    })
  } catch (error) {
    console.error('语音播放失败', error)
  }
}

onUnmounted(() => {
  disposed = true
  destroyResources()
})
</script>

<template>
  <div class="backdrop" />
  <canvas id="live2d" ref="canvasRef" />
  <div class="controls">
    <p v-if="errorMessage" role="alert">
      加载失败：{{ errorMessage }}
    </p>
    <p v-else role="status">
      {{ ready ? '模型已就绪，可点击或拖动' : '正在加载模型…' }}
    </p>
    <button :disabled="!ready" @click="playVoice(live2DSprite)">
      左侧模型播放语音
    </button>
    <button :disabled="!ready" @click="playVoice(live2DSprite2)">
      右侧模型播放语音
    </button>
    <button :disabled="!ready" @click="live2DSprite.stopVoice()">
      停止左侧语音
    </button>
    <button :disabled="!ready" @click="live2DSprite2.stopVoice()">
      停止右侧语音
    </button>
  </div>
</template>

<style>
#live2d {
  position: absolute;
  top: 0;
  right: 0;
  width: 100%;
  height: 100%;
}

.backdrop {
  position: absolute;
  width: 100%;
  height: 70%;
  background-color: pink;
}

.controls {
  position: relative;
  display: inline-block;
  margin: 12px;
  padding: 12px;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.92);
  font-family: sans-serif;
}

.controls p {
  margin: 0 0 8px;
}

.controls button {
  margin: 4px;
  padding: 6px 10px;
}
</style>
