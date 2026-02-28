[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# @sapphi-red/web-noise-suppressor


[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)

適用於 Web Audio API 的噪音抑制節點。

[🎧 Demo](https://web-noise-suppressor.sapphi.red)

此套件提供三種噪音抑制節點。

| Node | 說明 | 後端 |
| --- | --- | --- |
| `NoiseGateWorkletNode` | 簡易噪音閘實作 | 原生邏輯 |
| `RnnoiseWorkletNode` | 基於 RNNoise 的抑制器 | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex preprocess 抑制器 | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> 此套件需要 `AudioWorklet` 才能運作。

## 概覽

`@sapphi-red/web-noise-suppressor` 是一個以瀏覽器為主的 TypeScript 函式庫，提供 `AudioWorkletNode` 封裝以進行即時輸入清理。它專為麥克風音訊流程設計，必要時也可搭配瀏覽器 WebRTC 限制（`noiseSuppression`、`echoCancellation`）一同使用。

核心流程如下：

1. 載入 wasm 二進位檔（`loadSpeex` / `loadRnnoise`）。
2. 註冊 worklet 處理器模組（`audioWorklet.addModule(...)`）。
3. 建立節點（`new SpeexWorkletNode(...)` 等）。
4. 連接音訊圖。

## 功能

- 三種處理選項，採用一致的 Web Audio 使用模式。
- 提供 ESM + CJS 函式庫入口匯出。
- 為 worklet JS 檔案與 wasm 二進位檔提供專用匯出路徑。
- `loadRnnoise` 支援 RNNoise SIMD 偵測。
- 內含示範應用（`demo/`），具備即時麥克風路由與視覺化。

## 安裝

```shell
npm i @sapphi-red/web-noise-suppressor # yarn add @sapphi-red/web-noise-suppressor
```

## 前置需求

- 支援 `AudioWorklet` 的瀏覽器/執行環境。
- 麥克風擷取需在安全內容中進行（`https://` 或 localhost）。
- 需要一種 bundler 策略，可為 worklet JS 與 wasm 檔案提供 URL 參照。
  - 文件中的範例以 Vite 為導向。

## 使用方式

本節僅為 vite 使用者撰寫。

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
  - `closeThreshold?: number`（dB，預設為 `openThreshold`）
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise 載入

`loadRnnoise` 同時接受非 SIMD 與 SIMD URL，並在執行時選擇其一：

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### 匯出的子路徑

此套件匯出：

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

### 設定

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

在 `demo/package.json` 中，也可使用 `build:all`：

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## 開發備註

- RNNoise worklet node 包含 48kHz 假設（`src/rnnoise/workletNode.ts`）。
- `RnnoiseWorkletNode` 與 `SpeexWorkletNode` 提供 `destroy()` 以終止 wasm 端資源。
- 建置輸出由 `tsdown` 產生，並會將 wasm 資產複製到 `dist/`。
- 透過 pnpm patched dependencies，會對 `@shiguredo/rnnoise-wasm` 套用本機修補。

## 疑難排解

- `AudioWorklet is not defined`
  - 請確認瀏覽器支援與安全內容設定。
- `Failed to execute 'addModule'`
  - 請確認 worklet JS 路徑在執行時能解析為有效 URL。
- wasm 載入失敗（`fetch`/404/CORS）
  - 請確認 wasm 資產已複製/提供，且 bundler 的 URL 匯入策略正確。
- 沒有可聽見的效果
  - 檢查音訊圖連線（`source -> node -> destination`）以及是否同時啟用 WebRTC 限制。
- RNNoise 行為看起來不穩定
  - 請使用或測試 48kHz 音訊內容/取樣率。

## 路線圖

- 擴充非 Vite 整合文件。
- 在 `i18n/` 下新增翻譯版 README。
- 更詳細地記錄效能特性與處理器取捨。

## 貢獻

歡迎提交 issue 與 pull request：

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

在開 PR 前，請先執行：

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## 授權

MIT License. See [LICENSE](./LICENSE).
