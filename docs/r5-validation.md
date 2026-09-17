# R5 升级验证记录

验证日期：2026-09-17。

## 升级内容

- 固定已发布的 `easy-live2d@1.0.0-uat.0`，不再使用指向旧正式版的 `latest`。
- Pixi 固定为 `8.17.1`，与本轮库验证使用的版本保持一致。
- Core 使用官方 Cubism 5 SDK for Web R5，运行时版本 `06.00.0001`，入口加载带缓存版本的 min.js。
- Core 的 JS、min.js 和类型声明与已校验 SHA-256 的官方 R5 SDK 文件逐字节一致；保留官方许可。删除不在可再分发清单中的旧 source map。
- 保留两种模型初始化方式，移除无效 ticker、重复事件绑定、不存在的表情和音频请求；语音改为按钮触发。
- 补齐 ready 错误处理，以及双模型、配置请求和 Pixi Application 的卸载清理。

## 构建与静态检查

使用 Node.js 22.22.2、pnpm 10.7.1：

- `pnpm install --frozen-lockfile --offline`：通过。
- `pnpm build`：类型检查及 Vite 生产构建通过。
- `pnpm test:unit --run`：已有 1 项测试通过，该模板测试不代表模型回归覆盖。
- `pnpm exec eslint src/App.vue index.html`：通过。
- 项目自身修改的 `git diff --check`：通过；官方 Core changelog 保留原文件中的尾随空格，不为格式检查修改上游原件。

构建仍提示主 chunk 超过 500 kB；这不阻塞构建。安装提示跳过 esbuild 的安装脚本，本机实际构建可用。

## 浏览器冒烟检查

使用本机 Google Chrome、WebGL 2 / SwiftShader，以一次性 Playwright 脚本检查以下 7 个场景：

1. 已发布 npm 包与 R5 Core 配套运行，两个 Hiyori 模型均 ready，模型尺寸可读取，GL 错误为 0。
2. 实际点击身体触发动作，实际指针拖动改变模型位置。
3. 两个按钮各加载一次真实 WAV；停止左侧语音后，左侧音频实例为 0、右侧仍为 1。
4. 调整画布尺寸后无 GL 错误；就绪后卸载销毁两个模型、清空音频实例、释放 renderer 并停止 ticker。
5. 在生产预览中模拟配置 HTTP 503，页面显示错误且语音按钮禁用。
6. 在生产预览中延迟配置响应并提前卸载，没有未捕获异常或重新挂载的画布。
7. 生产预览加载两个模型并通过按钮获取音频，无未捕获异常。

开发与生产画面截图均已目视检查。

## 范围与已知限制

- 未验证 Safari、Firefox、移动真机或真实 GPU 性能；未在 StackBlitz WebContainer 内复跑。
- 现有 Vue Devtools 7.7.2 在本机 Node 25 下加载配置时会因 `localStorage.getItem` 报错；验证使用 Node 22。没有为此次 SDK 升级扩大开发工具的升级范围。
- 开发模式下，在模型配置请求期间提前卸载 Vue 根应用，旧版 Vue Devtools 自身的 `getActiveInspectors` 会报错；生产构建的同一场景通过。本轮没有将该开发工具异常计入模型逻辑通过结论。
