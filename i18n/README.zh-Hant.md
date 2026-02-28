[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red/web-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)
[![Stars](https://img.shields.io/github/stars/sapphi-red/web-noise-suppressor?style=flat-square)](https://github.com/sapphi-red/web-noise-suppressor/stargazers)

適用於 Web Audio API 的降噪節點，專為瀏覽器端即時麥克風清音訊而設。

[![🎧 Try Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| 關注點 | 本 README 涵蓋內容 |
| --- | --- |
| Runtime | Web Audio Worklet 流程（瀏覽器降噪） |
| 核心輸出 | `NoiseGateWorkletNode`、`RnnoiseWorkletNode`、`SpeexWorkletNode` |
| 使用模式 | 載入 WASM → 註冊處理器 → 實例化節點 → 連接音訊圖 |

本套件提供三種降噪節點。

| 節點 | 說明 | 後端 |
| --- | --- | --- |
| `NoiseGateWorkletNode` | 簡易的雜訊門（Noise Gate）實作 | 原生邏輯 |
| `RnnoiseWorkletNode` | 基於 RNNoise 的降噪器 | [xiph/rnnoise](https://github.com/xiph/rnnoise) 透過 [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex preprocess 降噪器 | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` 透過 [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> 此套件需要 `AudioWorklet` 才能運作。

## 概覽

`@sapphi-red/web-noise-suppressor` 是一個專注於瀏覽器的 TypeScript 函式庫，提供用於即時輸入清理的 `AudioWorkletNode` 封裝。它是專為麥克風輸入流程設計，必要時可搭配瀏覽器 WebRTC 約束（`noiseSuppression`、`echoCancellation`）使用。

核心流程如下：

1. 載入 wasm 二進位檔（`loadSpeex` / `loadRnnoise`）。
2. 註冊 worklet 處理器模組（`audioWorklet.addModule(...)`）。
3. 建立節點（`new SpeexWorkletNode(...)` 等）。
4. 連接音訊圖。

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
- [❤️ Support](#-support)
- [路線圖](#路線圖)
- [貢獻](#貢獻)
- [授權](#授權)

## 特色

- 三種處理選項，採用一致的 Web Audio 使用模式。
- 提供 ESM + CJS 的函式庫入口匯出。
- 為 worklet JS 檔案與 wasm 二進位檔提供專用匯出路徑。
- `loadRnnoise` 支援 RNNoise SIMD 偵測。
- 內建示範應用（`demo/`），具有即時麥克風路由與視覺化。

## 安裝

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## 先決條件

- 瀏覽器/執行環境須支援 `AudioWorklet`。
- 麥克風擷取須使用安全環境（`https://` 或 localhost）。
- 需有可提供 worklet JS 與 wasm 檔案 URL 的 bundler 策略。
  - 文件範例以 Vite 為導向。

## 使用方式

本節僅提供給 Vite 使用者。

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

更多細節請參考 [demo source code](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts)。

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
  - `closeThreshold?: number`（dB，預設值為 `openThreshold`）
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise 載入

`loadRnnoise` 可同時接受非 SIMD 與 SIMD URL，並在執行時自動選擇：

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### 匯出的子路徑

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

從 `demo/package.json` 也可使用 `build:all`：

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## 開發備註

- RNNoise worklet node 包含 48kHz 假設（`src/rnnoise/workletNode.ts`）。
- `RnnoiseWorkletNode` 與 `SpeexWorkletNode` 提供 `destroy()` 以釋放 wasm 端資源。

## 故障排除

- `AudioWorklet is not defined`
  - 請確認瀏覽器已支援並且使用安全環境。
- `Failed to execute 'addModule'`
  - 請確認 worklet JS 路徑可在執行時解析為有效 URL。
- wasm 載入失敗（`fetch`/404/CORS）
  - 請確認 wasm 資源已正確複製/提供，且 bundler 的 URL 匯入策略正確。
- 沒有明顯降噪效果
  - 檢查音訊圖（`source -> node -> destination`）與是否同時啟用了 WebRTC 限制。
- RNNoise 行為不穩定
  - 請使用或測試 48kHz 的音訊內容與取樣率。

## ❤️ Support

| Donate | PayPal | Stripe |
|---|---|---|
| [![Donate](https://img.shields.io/badge/Donate-LazyingArt-0EA5E9?style=for-the-badge&logo=ko-fi&logoColor=white)](https://chat.lazying.art/donate) | [![PayPal](https://img.shields.io/badge/PayPal-RongzhouChen-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen) | [![Stripe](https://img.shields.io/badge/Stripe-Donate-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## 路線圖

- 擴充非 Vite 整合文件。
- 在 `i18n/` 下增加更多翻譯版 README。
- 更詳細地記錄效能特性與處理器選擇取捨。

## 貢獻

歡迎提交 issue 與 pull request：

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

## 授權

MIT
