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

브라우저 측 실시간 마이크 정리를 위해 제작한 Web Audio API용 노이즈 억제 노드입니다.

[![🎧 Try Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| 빠른 바로가기 | 링크 |
| --- | --- |
| 🎧 Live demo | [web-noise-suppressor.sapphi.red](https://web-noise-suppressor.sapphi.red) |
| 📦 npm 패키지 | [@sapphi-red/web-noise-suppressor](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor) |
| 📄 진입점 소스 | [src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/src/index.ts) |
| 🧪 데모 소스 | [demo/src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) |

| 초점 | 이 README에서 다루는 항목 |
| --- | --- |
| 런타임 | 브라우저 노이즈 억제를 위한 Web Audio Worklet 파이프라인 |
| 핵심 출력물 | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| 사용 패턴 | WASM 로드 → 프로세서 등록 → 노드 인스턴스화 → 그래프 연결 |

이 패키지는 세 가지 노이즈 억제 노드를 제공합니다.

| 노드 | 설명 | 백엔드 |
| --- | --- | --- |
| `NoiseGateWorkletNode` | 간단한 노이즈 게이트 구현 | 네이티브 로직 |
| `RnnoiseWorkletNode` | RNNoise 기반 억제기 | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex preprocess 억제기 | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> 이 패키지를 사용하려면 `AudioWorklet`이 필요합니다.

## 개요

`@sapphi-red/web-noise-suppressor`는 브라우저용 TypeScript 라이브러리로, 실시간 입력 정리를 위한 `AudioWorkletNode` 래퍼를 제공합니다. 마이크 파이프라인에 맞춰 설계되었으며, 필요 시 브라우저 WebRTC 제약(`noiseSuppression`, `echoCancellation`)과 함께 사용할 수 있습니다.

기본 흐름은 다음과 같습니다.

1. WASM 바이너리 로드 (`loadSpeex` / `loadRnnoise`).
2. 워크렛 프로세서 모듈 등록 (`audioWorklet.addModule(...)`).
3. 노드 생성 (`new SpeexWorkletNode(...)` 등).
4. 오디오 그래프 연결.

## 목차

- [개요](#개요)
- [기능](#기능)
- [설치](#설치)
- [사전 요구 사항](#사전-요구-사항)
- [사용법](#사용법)
- [구성](#구성)
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

- 동일한 Web Audio 사용 패턴을 공유하는 3가지 처리 옵션.
- ESM + CJS 라이브러리 진입점 export.
- Worklet JS 파일 및 wasm 바이너리 전용 export 경로.
- `loadRnnoise`를 통한 RNNoise SIMD 감지 지원.
- 라이브 마이크 라우팅과 시각화기를 포함한 데모 앱 (`demo/`).

## 설치

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## 사전 요구 사항

- `AudioWorklet`을 지원하는 브라우저/런타임.
- 마이크 캡처를 위한 보안 컨텍스트 (`https://` 또는 localhost).
- Worklet JS 및 wasm 파일에 대한 URL 참조를 제공할 수 있는 번들러 전략.
  - 문서의 예제는 Vite를 기준으로 작성되어 있습니다.

## 사용법

이 섹션은 Vite 사용자 전용입니다.

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

## 구성

### 노드 옵션

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (`loadSpeex`에서 제공)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (`loadRnnoise`에서 제공)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, 기본값 `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise 로드

`loadRnnoise`는 SIMD 비활성 URL과 SIMD URL을 모두 받아 실행 시 하나를 선택합니다.

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### 내보낸 서브경로

패키지에서 다음 경로를 export합니다.

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
│   ├── noiseGate/          # 게이트 로직 + 워크렛 노드/프로세서
│   ├── rnnoise/            # rnnoise 통합 + 워크렛 노드/프로세서
│   ├── speex/              # speex preprocess 통합 + 워크렛 노드/프로세서
│   ├── utils/              # 공유 유틸리티
│   └── index.ts            # 공개 API exports
├── demo/                   # Vite 데모 애플리케이션
├── patches/                # rnnoise-wasm용 pnpm 패치
├── i18n/                   # 번역 산출물
├── tsdown.config.ts        # 빌드 설정
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

`demo/package.json`에는 `build:all`도 사용할 수 있습니다.

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## 개발 노트

- RNNoise 워크렛 노드는 48kHz 가정이 포함되어 있습니다 (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode`와 `SpeexWorkletNode`는 wasm 측 리소스를 종료하는 `destroy()`를 노출합니다.
- 빌드 결과물은 `tsdown`으로 생성되며 `dist/`에 wasm 자산이 복사됩니다.
- `@shiguredo/rnnoise-wasm`에는 pnpm patched dependencies를 통해 로컬 패치가 적용됩니다.

## 문제 해결

| 증상 | 확인 항목 |
| --- | --- |
| `AudioWorklet is not defined` | 브라우저 지원 여부와 보안 컨텍스트를 확인하세요. |
| `Failed to execute 'addModule'` | 런타임에서 워크렛 JS 경로가 유효한 URL로 해석되는지 확인하세요. |
| wasm 로드 실패 (`fetch`/404/CORS) | wasm 자산이 복사/서빙되고 있는지, 번들러의 URL import 전략이 올바른지 확인하세요. |
| 들리지 않는 효과 | 그래프 연결 (`source -> node -> destination`) 및 WebRTC 제약도 함께 활성화되어 있는지 확인하세요. |
| RNNoise 동작이 불안정 | 48kHz 오디오 컨텍스트/샘플 레이트로 실행해 보거나 테스트하세요. |

## 로드맵

- 비-Vite 통합 문서화 확대.
- `i18n/` 아래에 번역 README 변형 추가.
- 성능 특성 및 프로세서 트레이드오프를 더 자세히 문서화.

## 기여

이슈와 풀 리퀘스트를 환영합니다.

- 이슈: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- 저장소: <https://github.com/sapphi-red/web-noise-suppressor>

PR을 열기 전에 실행하세요.

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## 라이선스

MIT License. 자세한 내용은 [LICENSE](./LICENSE)를 참조하세요.


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
