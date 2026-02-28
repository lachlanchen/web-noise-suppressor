[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)

用于 Web Audio API 的噪声抑制节点。

[🎧 Demo](https://web-noise-suppressor.sapphi.red)

本包提供三种噪声抑制节点。

| 节点 | 说明 | 后端 |
| --- | --- | --- |
| `NoiseGateWorkletNode` | 一个简单的噪声门实现 | 原生逻辑 |
| `RnnoiseWorkletNode` | 基于 RNNoise 的抑制器 | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex preprocess 抑制器 | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> 此包依赖 `AudioWorklet` 才能工作。

## 概览

`@sapphi-red/web-noise-suppressor` 是一个面向浏览器的 TypeScript 库，提供用于实时输入清理的 `AudioWorkletNode` 封装。它专为麦克风音频链路设计，并可在需要时与浏览器 WebRTC 约束（`noiseSuppression`、`echoCancellation`）搭配使用。

核心流程如下：

1. 加载 wasm 二进制（`loadSpeex` / `loadRnnoise`）。
2. 注册 worklet 处理器模块（`audioWorklet.addModule(...)`）。
3. 构造节点（`new SpeexWorkletNode(...)` 等）。
4. 连接音频图。

## 特性

- 三种处理选项，共享一致的 Web Audio 使用模式。
- 提供 ESM + CJS 库入口导出。
- 为 worklet JS 文件和 wasm 二进制提供独立导出路径。
- `loadRnnoise` 支持 RNNoise SIMD 检测。
- 包含演示应用（`demo/`），支持实时麦克风路由与可视化。

## 安装

```shell
npm i @sapphi-red/web-noise-suppressor # yarn add @sapphi-red/web-noise-suppressor
```

## 前置条件

- 浏览器/运行时需支持 `AudioWorklet`。
- 麦克风采集需要安全上下文（`https://` 或 localhost）。
- 你需要一种 bundler 方案来为 worklet JS 和 wasm 文件提供 URL 引用。
  - 文档中的示例以 Vite 为导向。

## 用法

本节仅面向 vite 用户编写。

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url' // you can use `vite-plugin-static-copy` instead of this

const ctx = new AudioContext()

const speexWasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const stream = await navigator.mediaDevices.getUserMedia({
  audio: true
})

const source = ctx.createMediaStreamSource(stream)
const speex = new SpeexWorkletNode(ctx, {
  wasmBinary: speexWasmBinary,
  maxChannels: 2
})

source.connect(speex)
speex.connect(ctx.destination)
```

更多细节请参见 [demo source code](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts)。

## 配置

### 节点选项

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer`（来自 `loadSpeex`）
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer`（来自 `loadRnnoise`）
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number`（dB）
  - `closeThreshold?: number`（dB，默认值为 `openThreshold`）
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise 加载

`loadRnnoise` 同时接受非 SIMD 与 SIMD URL，并在运行时自动选择：

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### 导出的子路径

该包导出：

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## 示例

### Speex 示例

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### RNNoise 示例

```ts
import { RnnoiseWorkletNode, loadRnnoise } from '@sapphi-red/web-noise-suppressor'
import rnnoiseWorkletPath from '@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js?url'
import rnnoiseWasmPath from '@sapphi-red/web-noise-suppressor/rnnoise.wasm?url'
import rnnoiseSimdWasmPath from '@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm?url'

const ctx = new AudioContext({ sampleRate: 48000 })
const wasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseSimdWasmPath
})
await ctx.audioWorklet.addModule(rnnoiseWorkletPath)

const node = new RnnoiseWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### Noise Gate 示例

```ts
import { NoiseGateWorkletNode } from '@sapphi-red/web-noise-suppressor'
import noiseGateWorkletPath from '@sapphi-red/web-noise-suppressor/noiseGateWorklet.js?url'

const ctx = new AudioContext()
await ctx.audioWorklet.addModule(noiseGateWorkletPath)

const node = new NoiseGateWorkletNode(ctx, {
  openThreshold: -50,
  closeThreshold: -60,
  holdMs: 90,
  maxChannels: 2
})
```

## 项目结构

```text
.
├── src/
│   ├── noiseGate/          # gate logic + worklet node/processor
│   ├── rnnoise/            # rnnoise integration + worklet node/processor
│   ├── speex/              # speex preprocess integration + worklet node/processor
│   ├── utils/              # shared utilities
│   └── index.ts            # public API exports
├── demo/                   # Vite demo application
├── patches/                # pnpm patch for rnnoise-wasm
├── i18n/                   # translation artifacts (currently empty)
├── tsdown.config.ts        # build config
└── package.json
```

## 开发

### 环境准备

```shell
pnpm install
```

### 根目录脚本

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### Demo 脚本

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

在 `demo/package.json` 中，还提供了 `build:all`：

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## 开发说明

- RNNoise worklet 节点包含 48kHz 假设（`src/rnnoise/workletNode.ts`）。
- `RnnoiseWorkletNode` 与 `SpeexWorkletNode` 提供 `destroy()` 用于释放 wasm 侧资源。
- 构建产物由 `tsdown` 生成，并将 wasm 资源复制到 `dist/`。
- 通过 pnpm patched dependencies 对 `@shiguredo/rnnoise-wasm` 应用了本地补丁。

## 故障排查

- `AudioWorklet is not defined`
  - 请确认浏览器支持和安全上下文设置。
- `Failed to execute 'addModule'`
  - 确保 worklet JS 路径在运行时可解析为有效 URL。
- wasm 加载失败（`fetch`/404/CORS）
  - 确认 wasm 资源已被复制/托管，并且 bundler 的 URL 导入策略正确。
- 听感上没有效果
  - 检查音频图连接（`source -> node -> destination`）以及是否同时启用了 WebRTC 约束。
- RNNoise 行为看起来不稳定
  - 请使用或测试 48kHz 的音频上下文/采样率。

## 路线图

- 扩展非 Vite 集成文档。
- 在 `i18n/` 下添加 README 多语言版本。
- 更详细地记录性能特征和处理器取舍。

## 贡献

欢迎提交 issue 和 pull request：

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

在提交 PR 之前，请先运行：

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## 许可证

MIT License. See [LICENSE](./LICENSE).
