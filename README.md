[English](README.md) · [العربية](i18n/README.ar.md) · [Español](i18n/README.es.md) · [Français](i18n/README.fr.md) · [日本語](i18n/README.ja.md) · [한국어](i18n/README.ko.md) · [Tiếng Việt](i18n/README.vi.md) · [中文 (简体)](i18n/README.zh-Hans.md) · [中文（繁體）](i18n/README.zh-Hant.md) · [Deutsch](i18n/README.de.md) · [Русский](i18n/README.ru.md)




[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# @sapphi-red/web-noise-suppressor

[![npm version](https://img.shields.io/npm/v/@sapphi-red/web-noise-suppressor?style=flat-square&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)
[![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)](https://github.com/sapphi-red/web-noise-suppressor/actions)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF?style=flat-square&logo=webaudio&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
[![License](https://img.shields.io/badge/License-MIT-22C55E?style=flat-square)](./LICENSE)
[![Downloads](https://img.shields.io/npm/dm/@sapphi-red/web-noise-suppressor?style=flat-square&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)
[![Stars](https://img.shields.io/github/stars/sapphi-red/web-noise-suppressor?style=flat-square)](https://github.com/sapphi-red/web-noise-suppressor/stargazers)

Noise suppressor nodes for Web Audio API — built for browser-side real-time microphone cleanup.

[![🎧 Try Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Quick access | Link |
| --- | --- |
| 🎧 Live demo | [web-noise-suppressor.sapphi.red](https://web-noise-suppressor.sapphi.red) |
| 📦 npm package | [@sapphi-red/web-noise-suppressor](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor) |
| 📄 Source entry points | [src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/src/index.ts) |
| 🧪 Demo source | [demo/src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) |

| Focus | What this README covers |
| --- | --- |
| Runtime | Web Audio Worklet pipeline for browser noise suppression |
| Core output | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Usage pattern | Load WASM → register processor → instantiate node → connect graph |

This package provides three noise suppression nodes.

| Node | Description | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | A simple noise gate implementation | Native logic |
| `RnnoiseWorkletNode` | RNNoise-based suppressor | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex preprocess suppressor | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> This package requires `AudioWorklet` to work.

## Overview

`@sapphi-red/web-noise-suppressor` is a browser-focused TypeScript library that exposes `AudioWorkletNode` wrappers for real-time input cleanup. It is designed for microphone pipelines and can be used alongside browser WebRTC constraints (`noiseSuppression`, `echoCancellation`) when needed.

The core flow is:

1. Load wasm binary (`loadSpeex` / `loadRnnoise`).
2. Register worklet processor module (`audioWorklet.addModule(...)`).
3. Construct node (`new SpeexWorkletNode(...)`, etc.).
4. Connect audio graph.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Installation](#installation)
- [Prerequisites](#prerequisites)
- [Usage](#usage)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [Examples](#examples)
- [Development](#development)
- [Development Notes](#development-notes)
- [Troubleshooting](#troubleshooting)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [❤️ Support](#-support)
- [License](#license)

## Features

- Three processing options with a shared Web Audio usage pattern.
- ESM + CJS library entrypoint exports.
- Dedicated export paths for worklet JS files and wasm binaries.
- RNNoise SIMD detection support through `loadRnnoise`.
- Demo app (`demo/`) with live microphone routing and visualizer.

## Installation

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## Prerequisites

- A browser/runtime with `AudioWorklet` support.
- A secure context for microphone capture (`https://` or localhost).
- A bundler strategy that can provide URL references for worklet JS and wasm files.
  - The documented examples are Vite-oriented.

## Usage

This section is written only for vite users.

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

For more details, see [demo source code](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## Configuration

### Node options

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (from `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (from `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, defaults to `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise loading

`loadRnnoise` accepts both non-SIMD and SIMD URLs and selects one at runtime:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Exported subpaths

The package exports:

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## Examples

### Speex example

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### RNNoise example

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

### Noise Gate example

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

## Project Structure

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

## Development

### Setup

```shell
pnpm install
```

### Root scripts

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### Demo scripts

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

From `demo/package.json`, `build:all` is also available:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Development Notes

- RNNoise worklet node includes a 48kHz assumption (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` and `SpeexWorkletNode` expose `destroy()` to terminate wasm-side resources.
- Build output is generated by `tsdown` and includes copied wasm assets into `dist/`.
- A local patch is applied to `@shiguredo/rnnoise-wasm` via pnpm patched dependencies.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| `AudioWorklet is not defined` | Verify browser support and secure context. |
| `Failed to execute 'addModule'` | Ensure worklet JS path resolves to a valid URL at runtime. |
| wasm load failures (`fetch`/404/CORS) | Confirm wasm assets are copied/served and your bundler URL import strategy is correct. |
| No audible effect | Check graph wiring (`source -> node -> destination`) and whether WebRTC constraints are also enabled. |
| RNNoise behavior seems unstable | Use or test with a 48kHz audio context/sample rate. |

## Roadmap

- Expand non-Vite integration documentation.
- Add translated README variants under `i18n/`.
- Document performance characteristics and processor tradeoffs in more detail.

## Contributing

Issues and pull requests are welcome:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

Before opening a PR, run:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## License

MIT License. See [LICENSE](./LICENSE).
