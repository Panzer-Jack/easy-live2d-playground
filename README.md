# Easy Live2D Playground

基于 Vue 3、TypeScript 和 Pixi.js 的 easy-live2d 双模型演示。

- [在 StackBlitz 中打开](https://stackblitz.com/~/github.com/Panzer-Jack/easy-live2d-playground)
- [easy-live2d 源码](https://github.com/Panzer-Jack/easy-live2d)
- [R5 迁移指南](https://panzer-jack.github.io/easy-live2d/guide/cubism-r5-migration)

## 版本要求

本示例固定使用 `easy-live2d@1.0.0-uat.0`、`pixi.js@8.17.1`，并配套 **Cubism 5 SDK for Web R5** 的 Core（运行时版本 `06.00.0001`）。运行环境需要 WebGL 2，不支持 WebGL 1 或 SSR。

`1.0.0-uat.0` 是已发布的预发布版本；npm 的 `latest` 标签仍可能指向旧版，因此不要把依赖改回 `latest`。Core 与库必须配套升级，旧的 `.moc3` 模型可以继续使用，但应验证实际表现。

Pixi 默认选择 WebGL 2，无需额外填写渲染选项；Live2DSprite 也无需传入 `ticker`。

## 本地运行

使用 Node.js 22 和项目指定的 pnpm 10.7.1：

```bash
corepack enable
pnpm install --frozen-lockfile
pnpm dev
```

构建和预览：

```bash
pnpm build
pnpm preview
```

运行现有单元测试：

```bash
pnpm test:unit --run
```

## 演示内容

`src/App.vue` 同时展示两种初始化方式：

1. 左侧模型通过 `modelPath` 初始化。
2. 右侧模型通过 `CubismSetting` 传入已读取的模型配置和资源目录。

两侧都使用仓库已有的 Hiyori 模型，支持拖动和点击身体播放动作。等待页面显示“模型已就绪”，再点击语音按钮播放音频和口型；两侧语音可独立播放、停止。音频由用户点击触发，避免浏览器阻止自动播放。模型没有配置表情，因此示例不调用不存在的表情。

示例通过 `sprite.ready` 处理初始化失败，并在组件卸载时取消配置请求、销毁两个模型以及 Pixi Application。没有延迟播放定时器。

## Core 文件

入口 `index.html` 在应用代码之前加载：

```html
<script src="/Core/live2dcubismcore.min.js?v=5-r.5"></script>
```

Core 来自 [官方 R5 SDK](https://www.live2d.com/en/sdk/download/web/)，与 easy-live2d 使用的官方 Framework `5-r.5` 配套。R5 shader 已随 npm 包内置，无需在此仓库额外部署 Framework 或 Shaders。

`public/Core/` 保留官方运行文件、类型声明、版本记录及许可文件。升级时同步替换 Core 并更新入口缓存版本；不分发官方可再分发清单之外的 source map。现有 `public/Resources/` 模型和音频保持原样。

## 最小用法

以下代码需要 Vite 等打包环境，以及已引入 Core 的 HTML 页面：

```ts
import { Live2DSprite } from 'easy-live2d'
import { Application } from 'pixi.js'

const canvas = document.querySelector<HTMLCanvasElement>('#live2d')!
const app = new Application()
await app.init({ canvas, backgroundAlpha: 0, resizeTo: window })

const sprite = new Live2DSprite({
  modelPath: '/Resources/Hiyori/Hiyori.model3.json',
  draggable: true,
})
sprite.width = canvas.clientWidth
sprite.height = canvas.clientHeight
app.stage.addChild(sprite)

try {
  await sprite.ready
  console.log('模型已就绪', sprite.getModelCanvasSize())
} catch (error) {
  console.error('模型初始化失败', error)
  sprite.destroy()
  app.destroy()
}

// 页面或组件卸载时释放 sprite 和 app。
```

## 验证

构建、类型检查、现有单元测试以及 7 个浏览器冒烟场景已通过。环境、覆盖范围及旧版开发工具的已知限制见 [R5 验证记录](docs/r5-validation.md)。

## 许可证

本 playground 自身代码沿用 `MPL-2.0`；easy-live2d 库的许可证以其发布包为准（当前为 MIT）。Live2D Core 与模型、音频资源遵循各自许可，详见 `public/Core/LICENSE.md` 及资源中的许可说明。
