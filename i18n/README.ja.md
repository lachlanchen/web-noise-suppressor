[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)

Web Audio API 向けのノイズサプレッサーノードです。

[🎧 Demo](https://web-noise-suppressor.sapphi.red)

このパッケージは、3 種類のノイズ抑制ノードを提供します。

| Node | 説明 | バックエンド |
| --- | --- | --- |
| `NoiseGateWorkletNode` | シンプルなノイズゲート実装 | ネイティブロジック |
| `RnnoiseWorkletNode` | RNNoise ベースのサプレッサー | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex preprocess サプレッサー | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> このパッケージを動作させるには `AudioWorklet` が必要です。

## 概要

`@sapphi-red/web-noise-suppressor` は、リアルタイム入力のクリーンアップを行う `AudioWorkletNode` ラッパーを提供する、ブラウザ向け TypeScript ライブラリです。マイク入力パイプライン向けに設計されており、必要に応じてブラウザの WebRTC 制約（`noiseSuppression`, `echoCancellation`）と併用できます。

基本的なフローは次のとおりです。

1. wasm バイナリを読み込む（`loadSpeex` / `loadRnnoise`）。
2. worklet processor モジュールを登録する（`audioWorklet.addModule(...)`）。
3. ノードを生成する（`new SpeexWorkletNode(...)` など）。
4. オーディオグラフを接続する。

## 特長

- 共通の Web Audio 利用パターンで使える 3 つの処理オプション。
- ライブラリエントリポイントで ESM + CJS のエクスポートを提供。
- worklet JS ファイルと wasm バイナリ向けの専用エクスポートパス。
- `loadRnnoise` による RNNoise SIMD 検出サポート。
- ライブマイクルーティングと可視化を備えたデモアプリ（`demo/`）。

## インストール

```shell
npm i @sapphi-red/web-noise-suppressor # yarn add @sapphi-red/web-noise-suppressor
```

## 前提条件

- `AudioWorklet` をサポートするブラウザ/ランタイム。
- マイク取得のためのセキュアコンテキスト（`https://` または localhost）。
- worklet JS と wasm ファイルへの URL 参照を提供できるバンドラー構成。
  - ドキュメント中の例は Vite 前提です。

## 使い方

このセクションは Vite ユーザー向けです。

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

詳細は [demo source code](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) を参照してください。

## 設定

### ノードオプション

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (`loadSpeex` から取得)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (`loadRnnoise` から取得)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, 既定値は `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise の読み込み

`loadRnnoise` は non-SIMD と SIMD の両方の URL を受け取り、実行時に適切な方を選択します。

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### エクスポートされるサブパス

このパッケージは次をエクスポートします。

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## 例

### Speex の例

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### RNNoise の例

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

### Noise Gate の例

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

## プロジェクト構成

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

## 開発

### セットアップ

```shell
pnpm install
```

### ルートスクリプト

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### デモスクリプト

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

`demo/package.json` では `build:all` も利用できます。

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## 開発メモ

- RNNoise worklet node は 48kHz を前提としています（`src/rnnoise/workletNode.ts`）。
- `RnnoiseWorkletNode` と `SpeexWorkletNode` は、wasm 側リソースを終了するための `destroy()` を提供します。
- ビルド出力は `tsdown` で生成され、wasm アセットは `dist/` にコピーされます。
- `@shiguredo/rnnoise-wasm` には、pnpm の patched dependencies によるローカルパッチを適用しています。

## トラブルシューティング

- `AudioWorklet is not defined`
  - ブラウザ対応状況とセキュアコンテキストを確認してください。
- `Failed to execute 'addModule'`
  - 実行時に worklet JS パスが有効な URL に解決されることを確認してください。
- wasm load failures (`fetch`/404/CORS)
  - wasm アセットがコピー/配信されていることと、バンドラーの URL import 戦略が正しいことを確認してください。
- 音声効果が感じられない
  - グラフ接続（`source -> node -> destination`）と、WebRTC 制約も有効になっていないかを確認してください。
- RNNoise の挙動が不安定に見える
  - 48kHz の audio context/sample rate で利用またはテストしてください。

## ロードマップ

- 非 Vite 環境向けの統合ドキュメントを拡充する。
- `i18n/` 配下に翻訳 README バリアントを追加する。
- パフォーマンス特性とプロセッサー間のトレードオフをより詳細に文書化する。

## コントリビュート

Issue と Pull Request を歓迎します。

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

PR を作成する前に、次を実行してください。

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## ライセンス

MIT License. See [LICENSE](./LICENSE).
