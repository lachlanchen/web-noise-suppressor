[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)




[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# @sapphi-red/web-noise-suppressor

[![npm version](https://img.shields.io/npm/v/@sapphi-red/web-noise-suppressor?style=flat-square&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)
[![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)](https://github.com/sapphi-red/web-noise-suppressor/actions)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF?style=flat-square&logo=webaudio&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
[![License](https://img.shields.io/badge/License-MIT-22C55E?style=flat-square)](./LICENSE)
[![Downloads](https://img.shields.io/npm/dm/@sapphi-red/web-noise-suppressor?style=flat-square&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)
[![Stars](https://img.shields.io/github/stars/sapphi-red/web-noise-suppressor?style=flat-square)](https://github.com/sapphi-red/web-noise-suppressor/stargazers)

用于 Web Audio API 的噪声抑制节点——专为浏览器端实时麦克风降噪清理而构建。

[![🎧 试用演示](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| 快速访问 | 链接 |
| --- | --- |
| 🎧 Live demo | [web-noise-suppressor.sapphi.red](https://web-noise-suppressor.sapphi.red) |
| 📦 npm 包 | [@sapphi-red/web-noise-suppressor](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor) |
| 📄 入口文件 | [src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/src/index.ts) |
| 🧪 示例源码 | [demo/src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) |

| 关注点 | 本 README 覆盖内容 |
| --- | --- |
| 运行时 | 浏览器噪声抑制 Web Audio Worklet 管道 |
| 核心输出 | `NoiseGateWorkletNode`、`RnnoiseWorkletNode`、`SpeexWorkletNode` |
| 使用方式 | 加载 WASM → 注册处理器 → 实例化节点 → 连接音频图 |

本包提供三种噪声抑制节点。

| 节点 | 说明 | 后端 |
| --- | --- | --- |
| `NoiseGateWorkletNode` | 简单的噪声门实现 | 原生逻辑 |
| `RnnoiseWorkletNode` | 基于 RNNoise 的抑制器 | 通过 [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) 使用 [xiph/rnnoise](https://github.com/xiph/rnnoise) |
| `SpeexWorkletNode` | Speex preprocess 抑制器 | 通过 [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) 使用 [xiph/speexdsp](https://github.com/xiph/speexdsp) 的 `preprocess` |

> [!IMPORTANT]
> 本包需要 `AudioWorklet` 才能运行。

## 概述

`@sapphi-red/web-noise-suppressor` 是一个面向浏览器的 TypeScript 库，提供用于实时输入清理的 `AudioWorkletNode` 封装。该库面向麦克风处理流程设计，在需要时可与浏览器 WebRTC 限制（`noiseSuppression`、`echoCancellation`）配合使用。

核心流程如下：

1. 加载 wasm 二进制文件（`loadSpeex` / `loadRnnoise`）。
2. 注册 worklet 处理器模块（`audioWorklet.addModule(...)`）。
3. 创建节点（`new SpeexWorkletNode(...)` 等）。
4. 连接音频图。

## 目录

- [概述](#概述)
- [功能](#功能)
- [安装](#安装)
- [先决条件](#先决条件)
- [用法](#用法)
- [配置](#配置)
- [项目结构](#项目结构)
- [示例](#示例)
- [开发](#开发)
- [开发说明](#开发说明)
- [故障排查](#故障排查)
- [路线图](#路线图)
- [❤️ Support](#-support)
- [贡献](#贡献)
- [许可证](#许可证)

## 功能

- 三种处理方式，共享 Web Audio 的使用模式。
- 提供 ESM + CJS 库入口导出。
- 提供 worklet JS 文件和 wasm 二进制文件的专用导出路径。
- `loadRnnoise` 支持 RNNoise 的 SIMD 检测。
- 提供示例应用（`demo/`），包含实时麦克风路由和可视化工具。

## 安装

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## 先决条件

- 支持 `AudioWorklet` 的浏览器/运行时。
- 麦克风采集时处于安全上下文（`https://` 或 `localhost`）。
- 打包器策略需支持提供 worklet JS 与 wasm 文件的 URL 引用。
  - 文档示例以 Vite 为导向。

## 用法

本节仅为 Vite 用户编写。

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

更多细节请查看 [演示源码](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts)。

## 配置

### 节点选项

- `SpeexWorkletNode`（`SpeexProcessorOptions`）
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer`（来自 `loadSpeex`）
- `RnnoiseWorkletNode`（`RnnoiseProcessorOptions`）
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer`（来自 `loadRnnoise`）
- `NoiseGateWorkletNode`（`NoiseGateProcessorOptions`）
  - `openThreshold: number`（dB）
  - `closeThreshold?: number`（dB，默认值为 `openThreshold`）
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise 加载

`loadRnnoise` 接受非 SIMD 和 SIMD 两类 URL，并在运行时选择可用版本：

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### 导出的子路径

该包导出以下路径：

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
│   ├── noiseGate/          # 门控逻辑 + worklet node/processor
│   ├── rnnoise/            # rnnoise 集成 + worklet node/processor
│   ├── speex/              # speex preprocess 集成 + worklet node/processor
│   ├── utils/              # 共享工具
│   └── index.ts            # 公共 API 导出
├── demo/                   # Vite 演示应用
├── patches/                # rnnoise-wasm 的 pnpm patch
├── i18n/                   # 翻译产物（当前为空）
├── tsdown.config.ts        # 构建配置
└── package.json
```

## 开发

### 准备

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

### 示例脚本

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

`demo/package.json` 中还提供了 `build:all`：

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## 开发说明

- RNNoise worklet node 假定采样率为 48kHz（`src/rnnoise/workletNode.ts`）。
- `RnnoiseWorkletNode` 和 `SpeexWorkletNode` 提供 `destroy()`，用于释放 wasm 侧资源。
- 构建产物由 `tsdown` 生成，并会将 wasm 资源拷贝到 `dist/`。
- `@shiguredo/rnnoise-wasm` 通过 pnpm patched dependencies 应用本地补丁。

## 故障排查

| 症状 | 建议检查项 |
| --- | --- |
| `AudioWorklet is not defined` | 验证浏览器支持情况和是否处于安全上下文。 |
| `Failed to execute 'addModule'` | 确认运行时 worklet JS 路径是否解析为有效 URL。 |
| wasm 加载失败（`fetch`/404/CORS） | 确认 wasm 资源已复制并可访问，并检查打包器的 URL 导入策略是否正确。 |
| 没有听到效果 | 检查音频链路（`source -> node -> destination`）是否正确，以及 WebRTC 约束是否也已启用。 |
| RNNoise 表现不稳定 | 使用 48kHz 的音频上下文/采样率进行测试。 |

## 路线图

- 扩展非 Vite 集成文档。
- 在 `i18n/` 下增加更多翻译版 README。
- 更详细地补充性能特性和处理器的取舍说明。

## 贡献

欢迎提交 Issue 与 Pull Request：

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- 仓库: <https://github.com/sapphi-red/web-noise-suppressor>

在提交 PR 前，请先运行：

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## 许可证

MIT License. 详见 [LICENSE](./LICENSE)。


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
