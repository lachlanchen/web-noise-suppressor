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

Web Audio API 向けのノイズ抑制ノードです。ブラウザー側で、リアルタイムのマイク入力ノイズ除去を行うために作られています。

[![🎧 デモを試す](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| クイックアクセス | リンク |
| --- | --- |
| 🎧 Live demo | [web-noise-suppressor.sapphi.red](https://web-noise-suppressor.sapphi.red) |
| 📦 npm パッケージ | [@sapphi-red/web-noise-suppressor](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor) |
| 📄 エントリポイント | [src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/src/index.ts) |
| 🧪 デモのソース | [demo/src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) |

| 焦点 | この README で扱う内容 |
| --- | --- |
| 実行環境 | ブラウザー上でのノイズ抑制 Web Audio Worklet パイプライン |
| コア出力 | `NoiseGateWorkletNode`、`RnnoiseWorkletNode`、`SpeexWorkletNode` |
| 利用パターン | WASM を読み込む → プロセッサを登録する → ノードをインスタンス化する → グラフへ接続する |

このパッケージは 3 種類のノイズ抑制ノードを提供します。

| ノード | 説明 | バックエンド |
| --- | --- | --- |
| `NoiseGateWorkletNode` | シンプルなノイズゲート実装 | ネイティブロジック |
| `RnnoiseWorkletNode` | RNNoise ベースの抑制 | [xiph/rnnoise](https://github.com/xiph/rnnoise) を [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) 経由で利用 |
| `SpeexWorkletNode` | Speex preprocess 抑制 | [xiph/speexdsp](https://github.com/xiph/speexdsp) の `preprocess` を [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) 経由で利用 |

> [!IMPORTANT]
> このパッケージを使用するには `AudioWorklet` が必要です。

## 概要

`@sapphi-red/web-noise-suppressor` は、ブラウザー向けの TypeScript ライブラリで、リアルタイム入力クリーンアップ用の `AudioWorkletNode` ラッパーを公開しています。マイク入力パイプライン向けに設計されており、必要に応じてブラウザーの WebRTC 制約（`noiseSuppression`、`echoCancellation`）と併用できます。

基本的な処理フロー:

1. wasm バイナリを読み込む (`loadSpeex` / `loadRnnoise`)。
2. Worklet プロセッサーモジュールを登録する (`audioWorklet.addModule(...)`)。
3. ノードを構築する (`new SpeexWorkletNode(...)` など)。
4. オーディオグラフに接続する。

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

- 共通の Web Audio 利用パターンを使う 3 つの処理オプション。
- ESM + CJS のライブラリエントリポイントエクスポート。
- Worklet JS ファイルと wasm バイナリ向けの専用サブパスをエクスポート。
- `loadRnnoise` による RNNoise の SIMD 検知をサポート。
- デモアプリ (`demo/`) には、ライブでのマイクルーティングとビジュアライザーが搭載されています。

## インストール

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## 前提条件

- `AudioWorklet` をサポートするブラウザー/ランタイム。
- マイク取得のためのセキュアコンテキスト（`https://` または `localhost`）。
- Worklet JS と wasm ファイルを URL として提供できるバンドラ戦略。
  - 記載例は Vite 向けです。

## 使い方

このセクションは Vite 利用者向けのみです。

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

詳細は [デモのソースコード](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts)を参照してください。

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
  - `closeThreshold?: number` (dB, defaults to `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise 読み込み

`loadRnnoise` は SIMD 非対応/対応 URL の両方を受け取り、実行時にどちらかを選択します:

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

## プロジェクト構造

```text
.
├── src/
│   ├── noiseGate/          # ゲートロジック + worklet node/processor
│   ├── rnnoise/            # rnnoise 統合 + worklet node/processor
│   ├── speex/              # speex preprocess 統合 + worklet node/processor
│   ├── utils/              # 共通ユーティリティ
│   └── index.ts            # パブリック API エクスポート
├── demo/                   # Vite デモアプリケーション
├── patches/                # rnnoise-wasm 向け pnpm patch
├── i18n/                   # 翻訳成果物
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

`demo/package.json` では `build:all` も利用できます。

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## 開発ノート

- RNNoise Worklet Node は 48kHz を前提とします (`src/rnnoise/workletNode.ts`)。
- `RnnoiseWorkletNode` と `SpeexWorkletNode` は `destroy()` を公開し、WASM 側リソースを終了できます。
- ビルド成果物は `tsdown` で生成され、wasm アセットが `dist/` にコピーされます。
- `@shiguredo/rnnoise-wasm` には pnpm の patched dependencies を通してローカルパッチを適用します。

## トラブルシューティング

| 症状 | 確認項目 |
| --- | --- |
| `AudioWorklet is not defined` | ブラウザーの対応状況とセキュアコンテキストを確認してください。 |
| `Failed to execute 'addModule'` | 実行時に Worklet JS のパスが有効な URL へ解決されているか確認してください。 |
| wasm 読み込み失敗 (`fetch`/404/CORS) | wasm アセットがコピーされ配信されているか、バンドラの URL インポート戦略が正しいか確認してください。 |
| 効果が聞こえない | グラフ接続（`source -> node -> destination`）と WebRTC の制約が有効化されているか確認してください。 |
| RNNoise の挙動が不安定 | 48kHz のオーディオコンテキストまたはサンプリングレートでテストしてください。 |

## ロードマップ

- Vite 以外の統合ドキュメントを拡張する。
- `i18n/` に翻訳済み README バリエーションを追加する。
- パフォーマンス特性とプロセッサのトレードオフをより詳しく記述する。

## コントリビュート

Issue と Pull Request を歓迎します。

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- リポジトリ: <https://github.com/sapphi-red/web-noise-suppressor>

PR を作成する前に、以下を実行してください。

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## ライセンス

MIT License. 詳細は [LICENSE](./LICENSE) を参照してください。


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
