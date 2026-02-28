[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)

Web Audio API용 노이즈 억제 노드입니다.

[🎧 데모](https://web-noise-suppressor.sapphi.red)

이 패키지는 세 가지 노이즈 억제 노드를 제공합니다.

| 노드 | 설명 | 백엔드 |
| --- | --- | --- |
| `NoiseGateWorkletNode` | 단순한 노이즈 게이트 구현 | 네이티브 로직 |
| `RnnoiseWorkletNode` | RNNoise 기반 억제기 | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex preprocess 억제기 | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> 이 패키지가 동작하려면 `AudioWorklet`이 필요합니다.

## 개요

`@sapphi-red/web-noise-suppressor`는 브라우저 중심 TypeScript 라이브러리로, 실시간 입력 정리를 위한 `AudioWorkletNode` 래퍼를 제공합니다. 마이크 파이프라인용으로 설계되었으며, 필요 시 브라우저 WebRTC 제약(`noiseSuppression`, `echoCancellation`)과 함께 사용할 수 있습니다.

핵심 흐름은 다음과 같습니다.

1. wasm 바이너리 로드 (`loadSpeex` / `loadRnnoise`).
2. worklet processor 모듈 등록 (`audioWorklet.addModule(...)`).
3. 노드 생성 (`new SpeexWorkletNode(...)` 등).
4. 오디오 그래프 연결.

## 기능

- 공통 Web Audio 사용 패턴을 따르는 세 가지 처리 옵션.
- ESM + CJS 라이브러리 엔트리포인트 export.
- worklet JS 파일 및 wasm 바이너리를 위한 전용 export 경로.
- `loadRnnoise`를 통한 RNNoise SIMD 감지 지원.
- 실시간 마이크 라우팅 및 시각화 기능이 있는 데모 앱 (`demo/`).

## 설치

```shell
npm i @sapphi-red/web-noise-suppressor # yarn add @sapphi-red/web-noise-suppressor
```

## 사전 요구사항

- `AudioWorklet`을 지원하는 브라우저/런타임.
- 마이크 캡처를 위한 보안 컨텍스트 (`https://` 또는 localhost).
- worklet JS 및 wasm 파일에 대한 URL 참조를 제공할 수 있는 번들러 전략.
  - 문서의 예시는 Vite 기준입니다.

## 사용법

이 섹션은 Vite 사용자만을 대상으로 작성되었습니다.

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

`loadRnnoise`는 non-SIMD URL과 SIMD URL을 모두 받아 런타임에 하나를 선택합니다.

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Export된 서브패스

패키지는 다음을 export합니다.

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

`demo/package.json`에는 `build:all`도 제공됩니다.

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## 개발 노트

- RNNoise worklet node는 48kHz를 가정합니다 (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode`와 `SpeexWorkletNode`는 wasm 측 리소스 종료를 위한 `destroy()`를 제공합니다.
- 빌드 출력은 `tsdown`으로 생성되며, wasm 에셋이 `dist/`에 복사되어 포함됩니다.
- pnpm patched dependencies를 통해 `@shiguredo/rnnoise-wasm`에 로컬 패치가 적용됩니다.

## 문제 해결

- `AudioWorklet is not defined`
  - 브라우저 지원 여부와 보안 컨텍스트를 확인하세요.
- `Failed to execute 'addModule'`
  - 런타임에서 worklet JS 경로가 유효한 URL로 해석되는지 확인하세요.
- wasm 로드 실패 (`fetch`/404/CORS)
  - wasm 에셋이 복사/서빙되는지, 번들러 URL import 전략이 올바른지 확인하세요.
- 소리 변화가 들리지 않음
  - 그래프 연결(`source -> node -> destination`)과 WebRTC 제약 활성화 여부를 확인하세요.
- RNNoise 동작이 불안정해 보임
  - 48kHz 오디오 컨텍스트/샘플레이트에서 사용하거나 테스트하세요.

## 로드맵

- 비-Vite 통합 문서를 확장.
- `i18n/` 아래에 번역 README 변형 추가.
- 성능 특성과 processor 간 트레이드오프를 더 자세히 문서화.

## 기여

이슈와 풀 리퀘스트를 환영합니다.

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

MIT License. 자세한 내용은 [LICENSE](./LICENSE)를 참고하세요.
