[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red/web-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)
[![Stars](https://img.shields.io/github/stars/sapphi-red/web-noise-suppressor?style=flat-square)](https://github.com/sapphi-red/web-noise-suppressor/stargazers)

Web Audio API를 위한 노이즈 억제 노드입니다. 브라우저 기반 실시간 마이크 정리 용도로 만들어졌습니다.

[![🎧 Try Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Focus | What this README covers |
| --- | --- |
| Runtime | 브라우저 노이즈 억제를 위한 Web Audio Worklet 파이프라인 |
| Core output | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Usage pattern | WASM 로드 → 프로세서 등록 → 노드 생성 → 그래프 연결 |

이 패키지는 3개의 노이즈 억제 노드를 제공합니다.

| Node | Description | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | 간단한 노이즈 게이트 구현 | Native logic |
| `RnnoiseWorkletNode` | RNNoise 기반 억제기 | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex preprocess 억제기 | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> 이 패키지를 사용하려면 `AudioWorklet`이 필요합니다.

## 개요

`@sapphi-red/web-noise-suppressor`는 브라우저 중심 TypeScript 라이브러리로, 실시간 입력 정리를 위한 `AudioWorkletNode` 래퍼를 제공합니다. 마이크 파이프라인을 위해 설계되었으며 필요 시 브라우저 WebRTC 제약(`noiseSuppression`, `echoCancellation`)과 함께 사용할 수 있습니다.

핵심 동작 흐름은 다음과 같습니다.

1. WASM 바이너리 로드 (`loadSpeex` / `loadRnnoise`)
2. Worklet processor 모듈 등록 (`audioWorklet.addModule(...)`)
3. 노드 생성 (`new SpeexWorkletNode(...)` 등)
4. 오디오 그래프 연결

## 목차

- [개요](#개요)
- [기능](#기능)
- [설치](#설치)
- [사전 요구사항](#사전-요구사항)
- [사용법](#사용법)
- [설정](#설정)
- [프로젝트 구조](#프로젝트-구조)
- [예제](#예제)
- [개발](#개발)
- [개발 노트](#개발-노트)
- [문제 해결](#문제-해결)
- [❤️ Support](#-support)
- [로드맵](#로드맵)
- [기여](#기여)
- [라이선스](#라이선스)

## 기능

- 공통 Web Audio 사용 패턴을 따르는 3가지 처리 옵션
- ESM + CJS 라이브러리 진입점 export
- Worklet JS 파일과 wasm 바이너리를 위한 전용 export 경로
- `loadRnnoise`를 통해 RNNoise SIMD 검출 지원
- 실시간 마이크 라우팅과 시각화를 갖춘 데모 앱 (`demo/`)

## 설치

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## 사전 요구사항

- `AudioWorklet`을 지원하는 브라우저/런타임
- 마이크 캡처를 위한 보안 컨텍스트 (`https://` 또는 `localhost`)
- worklet JS 및 wasm 파일에 대해 URL 참조를 제공할 수 있는 번들러 전략
  - 문서의 예제는 Vite를 기준으로 작성됩니다.

## 사용법

이 섹션은 vite 사용자만 대상으로 작성되었습니다.

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

자세한 내용은 [데모 소스 코드](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts)를 참고하세요.

## 설정

### 노드 옵션

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (`loadSpeex`에서 가져옴)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (`loadRnnoise`에서 가져옴)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, 기본값은 `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise 로딩

`loadRnnoise`는 비 SIMD URL과 SIMD URL을 모두 받아 런타임에 하나를 선택합니다.

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Export된 서브패스

패키지는 다음 항목을 export합니다.

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## 예제

### Speex 예제

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### RNNoise 예제

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

### Noise Gate 예제

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

## 프로젝트 구조

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

## 개발

### 설정

```shell
pnpm install
```

### 루트 스크립트

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### 데모 스크립트

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

`demo/package.json`에는 `build:all`도 제공합니다.

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## 개발 노트

- RNNoise worklet node는 48kHz 가정을 사용합니다 (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` 및 `SpeexWorkletNode`는 wasm 측 리소스를 정리하기 위해 `destroy()`를 노출합니다.
- 빌드 산출물은 `tsdown`으로 생성되며 `dist/`에 복사된 wasm 자산이 포함됩니다.
- pnpm 패치된 의존성으로 `@shiguredo/rnnoise-wasm`에 로컬 패치가 적용됩니다.

## 문제 해결

- `AudioWorklet is not defined`
  - 브라우저 지원 여부와 보안 컨텍스트를 확인하세요.
- `Failed to execute 'addModule'`
  - 런타임에서 worklet JS 경로가 유효한 URL로 해석되는지 확인하세요.
| Symptom | What to check |
| --- | --- |
| `AudioWorklet is not defined` | 브라우저 지원 여부와 보안 컨텍스트를 확인하세요. |
| `Failed to execute 'addModule'` | worklet JS 경로가 런타임에서 유효한 URL로 해석되는지 확인하세요. |
| wasm 로드 실패 (`fetch`/404/CORS) | wasm 자산이 복사/서빙되는지, 번들러 URL import 전략이 올바른지 확인하세요. |
| 오디오 반응이 거의 없음 | 그래프 연결 (`source -> node -> destination`)과 WebRTC 제약이 함께 켜져 있는지 확인하세요. |
| RNNoise 동작이 불안정함 | 48kHz 오디오 컨텍스트/샘플레이트에서 테스트하거나 사용하세요. |

## ❤️ Support

| Donate | PayPal | Stripe |
|---|---|---|
| [![Donate](https://img.shields.io/badge/Donate-LazyingArt-0EA5E9?style=for-the-badge&logo=ko-fi&logoColor=white)](https://chat.lazying.art/donate) | [![PayPal](https://img.shields.io/badge/PayPal-RongzhouChen-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen) | [![Stripe](https://img.shields.io/badge/Stripe-Donate-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## 로드맵

- Vite가 아닌 환경 통합 문서 확장
- `i18n/` 아래 번역된 README 추가
- 성능 특성 및 프로세서 트레이드오프를 더 자세히 문서화

## 기여

이슈와 pull request를 환영합니다.

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

PR을 열기 전에 다음을 실행하세요.

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## 라이선스

MIT License. 자세한 내용은 [LICENSE](./LICENSE)를 확인하세요.
