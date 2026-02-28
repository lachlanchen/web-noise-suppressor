[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red/web-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)
[![Stars](https://img.shields.io/github/stars/sapphi-red/web-noise-suppressor?style=flat-square)](https://github.com/sapphi-red/web-noise-suppressor/stargazers)

Web Audio API 用のノイズ抑制ノードです。ブラウザ側でのリアルタイムなマイク音声のクリーンアップ向けです。

[![🎧 Try Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| フォーカス | このREADMEで扱う内容 |
| --- | --- |
| 実行時 | ブラウザ向けノイズ抑制のための Web Audio Worklet パイプライン |
| 主な成果物 | `NoiseGateWorkletNode`、`RnnoiseWorkletNode`、`SpeexWorkletNode` |
| 使い方の流れ | WASMを読み込む → プロセッサを登録する → ノードを生成する → グラフへ接続する |

このパッケージは3つのノイズ抑制ノードを提供します。

| ノード | 説明 | バックエンド |
| --- | --- | --- |
| `NoiseGateWorkletNode` | シンプルなノイズゲート実装 | ネイティブ実装 |
| `RnnoiseWorkletNode` | RNNoise ベースの抑制 | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex preprocess 抑制 | [xiph/speexdsp](https://github.com/xiph/speexdsp) の `preprocess` を [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) 経由で利用 |

> [!IMPORTANT]
> このパッケージは動作に `AudioWorklet` を必要とします。

## 概要

`@sapphi-red/web-noise-suppressor` は、ブラウザ向けの TypeScript ライブラリで、`AudioWorkletNode` のラッパーを公開し、リアルタイムで入力音声をクリーニングします。マイク向けのパイプラインを想定しており、必要に応じてブラウザの WebRTC 制約（`noiseSuppression`、`echoCancellation`）と併用できます。

基本の流れは次のとおりです。

1. WASM バイナリを読み込む (`loadSpeex` / `loadRnnoise`)
2. Worklet Processor モジュールを登録する (`audioWorklet.addModule(...)`)
3. ノードを生成する (`new SpeexWorkletNode(...)` など)
4. オーディオグラフを接続する

## 目次

- [概要](#概要)
- [機能](#機能)
- [インストール](#インストール)
- [前提条件](#前提条件)
- [使い方](#使い方)
- [設定](#設定)
- [プロジェクト構造](#プロジェクト構造)
- [例](#例)
- [開発](#開発)
- [開発ノート](#開発ノート)
- [トラブルシューティング](#トラブルシューティング)
- [❤️ Support](#-support)
- [ロードマップ](#ロードマップ)
- [コントリビュート](#コントリビュート)
- [ライセンス](#ライセンス)

## 機能

- 3つの処理方式が、同一の Web Audio 利用パターンで使用可能
- ESM + CJS のライブラリエントリポイントをエクスポート
- Worklet の JS ファイルと WASM バイナリ向けの個別エクスポートパス
- `loadRnnoise` を通じた RNNoise SIMD 検出サポート
- ライブマイク入力経路と可視化を備えたデモアプリ (`demo/`)

## インストール

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## 前提条件

- `AudioWorklet` をサポートするブラウザ/ランタイム
- マイク取得のためのセキュアコンテキスト (`https://` または `localhost`)
- worklet JS と wasm ファイルを URL として解決できるバンドラ構成
  - サンプルは Vite 前提です

## 使い方

このセクションは Vite 利用者向けに記述されています。

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

詳しくは[デモのソースコード](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts)を参照してください。

## 設定

### ノードオプション

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (`loadSpeex` から受け取る)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (`loadRnnoise` から受け取る)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, デフォルトは `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise の読み込み

`loadRnnoise` は SIMD 非対応版と SIMD 版の両方の URL を受け取り、実行時にいずれかを選択します。

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### エクスポートされるサブパス

パッケージは次をエクスポートします。

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

## プロジェクト構造

```text
.
├── src/
│   ├── noiseGate/          # ゲートロジック + worklet node/processor
│   ├── rnnoise/            # rnnoise 統合 + worklet node/processor
│   ├── speex/              # speex preprocess 統合 + worklet node/processor
│   ├── utils/              # 共通ユーティリティ
│   └── index.ts            # 公開 API エクスポート
├── demo/                   # Vite デモアプリケーション
├── patches/                # rnnoise-wasm 用の pnpm パッチ
├── i18n/                   # 翻訳アーティファクト
├── tsdown.config.ts        # ビルド設定
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

`demo/package.json` には `build:all` も用意されています。

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## 開発ノート

- RNNoise Worklet ノードは48kHz前提です (`src/rnnoise/workletNode.ts`)。
- `RnnoiseWorkletNode` と `SpeexWorkletNode` は wasm 側リソースを解放するための `destroy()` を公開します。
- ビルド成果物は `tsdown` で生成され、WASM アセットを `dist/` にコピーして出力します。
- `@shiguredo/rnnoise-wasm` には pnpm のパッチ済み依存としてローカルパッチを適用しています。

## トラブルシューティング

- `AudioWorklet is not defined`
  - ブラウザ対応状況とセキュアコンテキストを確認してください。
- `Failed to execute 'addModule'`
  - ワークレット JS のパスが実行時に有効な URL に解決されるか確認してください。
| 症状 | 確認項目 |
| --- | --- |
| `AudioWorklet is not defined` | ブラウザ対応状況とセキュアコンテキストを確認する |
| `Failed to execute 'addModule'` | ワークレット JS のパスが実行時に有効な URL に解決されることを確認する |
| wasm の読み込み失敗 (`fetch`/404/CORS) | wasm アセットのコピー/配信とバンドラ側の URL インポート方針が正しいか確認する |
| 音声的な効果が聞こえない | グラフ接続 (`source -> node -> destination`) と WebRTC 制約の有効化を両方確認する |
| RNNoise の挙動が不安定 | 48kHz サンプルレートの `AudioContext` でテスト・利用する |

## ❤️ Support

| Donate | PayPal | Stripe |
|---|---|---|
| [![Donate](https://img.shields.io/badge/Donate-LazyingArt-0EA5E9?style=for-the-badge&logo=ko-fi&logoColor=white)](https://chat.lazying.art/donate) | [![PayPal](https://img.shields.io/badge/PayPal-RongzhouChen-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen) | [![Stripe](https://img.shields.io/badge/Stripe-Donate-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## ロードマップ

- Vite以外の統合方法に関するドキュメントを拡充
- `i18n/` 配下にさらなる翻訳版 README を追加
- パフォーマンス特性やプロセッサのトレードオフをより詳しく解説

## コントリビュート

Issue と Pull Request を歓迎します。

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

PR 作成前に以下を実行してください。

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## ライセンス

MIT License. [LICENSE](./LICENSE) を参照してください。
