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

適用於 Web Audio API 的降噪節點，專為瀏覽器端即時麥克風清理訊號而設計。

[![🎧 Try Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| 快速存取 | 連結 |
| --- | --- |
| 🎧 線上示範 | [web-noise-suppressor.sapphi.red](https://web-noise-suppressor.sapphi.red) |
| 📦 npm 套件 | [@sapphi-red/web-noise-suppressor](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor) |
| 📄 進入點原始碼 | [src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/src/index.ts) |
| 🧪 Demo 原始碼 | [demo/src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) |

| 焦點 | 本 README 說明內容 |
| --- | --- |
| 執行環境 | 適用於瀏覽器的 Web Audio Worklet 降噪流程 |
| 核心輸出 | `NoiseGateWorkletNode`、`RnnoiseWorkletNode`、`SpeexWorkletNode` |
| 使用模式 | 載入 WASM → 註冊 processor → 實例化節點 → 串接音訊圖 |

本套件提供三種降噪節點。

| 節點 | 說明 | 後端 |
| --- | --- | --- |
| `NoiseGateWorkletNode` | 簡易降噪門實作 | 原生邏輯 |
| `RnnoiseWorkletNode` | 基於 RNNoise 的降噪器 | 透過 [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) 的 [xiph/rnnoise](https://github.com/xiph/rnnoise) |
| `SpeexWorkletNode` | Speex preprocess 降噪器 | 透過 [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) 的 [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` |

> [!IMPORTANT]
> 本套件需要 `AudioWorklet` 才能運作。

## 概覽

`@sapphi-red/web-noise-suppressor` 是一個以瀏覽器為重點的 TypeScript 函式庫，提供 `AudioWorkletNode` 的封裝，用於即時輸入音訊清理。它以麥克風管線為設計目標，並可在需要時與瀏覽器 WebRTC 限制（`noiseSuppression`、`echoCancellation`）搭配使用。

核心流程如下：

1. 載入 wasm 二進位檔（`loadSpeex` / `loadRnnoise`）。
2. 註冊 worklet processor 模組（`audioWorklet.addModule(...)`）。
3. 建立節點（`new SpeexWorkletNode(...)` 等）。
4. 串接音訊圖。

## 目錄

- [概覽](#概覽)
- [特色](#特色)
- [安裝](#安裝)
- [先決條件](#先決條件)
- [使用方式](#使用方式)
- [設定](#設定)
- [專案結構](#專案結構)
- [範例](#範例)
- [開發](#開發)
- [開發備註](#開發備註)
- [故障排除](#故障排除)
- [路線圖](#路線圖)
- [貢獻](#貢獻)
- [❤️ Support](#-support)
- [授權](#授權)

## 特色

- 三種處理選項共用一套 Web Audio 使用模式。
- 支援 ESM + CJS 的函式庫匯出入口。
- 為 worklet JS 檔與 wasm 二進位檔提供專用匯出路徑。
- `loadRnnoise` 支援 RNNoise SIMD 偵測。
- Demo 應用（`demo/`）支援即時麥克風路由與視覺化。

## 安裝

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## 先決條件

- 支援 `AudioWorklet` 的瀏覽器或執行環境。
- 可進行麥克風擷取的安全環境（`https://` 或 localhost）。
- 可為 worklet JS 與 wasm 檔案提供 URL 參考的打包策略。
  - 文件中的範例以 Vite 為導向。

## 使用方式

本節僅供 vite 使用者。

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

更多細節請參考 [demo 原始碼](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts)。

## 設定

### 節點選項

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer`（來自 `loadSpeex`）
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer`（來自 `loadRnnoise`）
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number`（dB）
  - `closeThreshold?: number`（dB，預設為 `openThreshold`）
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise 載入

`loadRnnoise` 同時接受非 SIMD 與 SIMD 的 URL，並在執行時自動選擇：

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### 匯出子路徑

本套件匯出：

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## 範例

### Speex 範例

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### RNNoise 範例

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

### Noise Gate 範例

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

## 專案結構

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

## 開發

### 環境設定

```shell
pnpm install
```

### 根目錄腳本

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### Demo 腳本

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

從 `demo/package.json`，也可使用 `build:all`：

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## 開發備註

- RNNoise worklet node 含有 48kHz 的假設（`src/rnnoise/workletNode.ts`）。
- `RnnoiseWorkletNode` 與 `SpeexWorkletNode` 提供 `destroy()` 以釋放 wasm 端資源。
- Build 輸出由 `tsdown` 產生，並將 wasm 資源複製到 `dist/`。
- 透過 pnpm patched dependencies 將本機修補套用到 `@shiguredo/rnnoise-wasm`。

## 故障排除

| 症狀 | 檢查項目 |
| --- | --- |
| `AudioWorklet is not defined` | 確認瀏覽器支援度與安全環境。 |
| `Failed to execute 'addModule'` | 確認 worklet JS 路徑在執行時可解析為有效 URL。 |
| wasm 載入失敗（`fetch`/404/CORS） | 確認 wasm 資源有正確複製／提供，且 bundler 的 URL 匯入策略正確。 |
| 沒有明顯效果 | 檢查音訊圖連線（`source -> node -> destination`）與是否同時啟用 WebRTC 限制。 |
| RNNoise 行為不穩定 | 請使用 48kHz 音訊內容與取樣率進行測試。 |

## 路線圖

- 擴充非 Vite 的整合文件。
- 在 `i18n/` 下加入更多 README 翻譯版本。
- 更完整地說明效能特性與處理器取捨。

## 貢獻

歡迎提交 Issue 與 Pull Request：

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

提交 PR 前，請先執行：

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## 授權

MIT 授權。請參閱 [LICENSE](./LICENSE)。


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
