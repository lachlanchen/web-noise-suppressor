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

Các node khử tiếng ồn cho Web Audio API — được xây dựng để dọn tiếng ồn micro theo thời gian thực phía trình duyệt.

[![🎧 Try Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Truy cập nhanh | Liên kết |
| --- | --- |
| 🎧 Live demo | [web-noise-suppressor.sapphi.red](https://web-noise-suppressor.sapphi.red) |
| 📦 Gói npm | [@sapphi-red/web-noise-suppressor](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor) |
| 📄 Điểm vào source | [src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/src/index.ts) |
| 🧪 Source demo | [demo/src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) |

| Trọng tâm | Nội dung README này |
| --- | --- |
| Runtime | Pipeline Web Audio Worklet để khử tiếng ồn trong trình duyệt |
| Đầu ra chính | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Mẫu sử dụng | Tải WASM → đăng ký processor → khởi tạo node → kết nối đồ thị |

Gói này cung cấp ba node khử tiếng ồn.

| Node | Mô tả | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Cài đặt bộ lọc cửa tiếng ồn đơn giản | Logic native |
| `RnnoiseWorkletNode` | Bộ khử tiếng ồn dựa trên RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) qua [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Bộ khử tiếng ồn tiền xử lý Speex | `preprocess` từ [xiph/speexdsp](https://github.com/xiph/speexdsp) qua [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Gói này yêu cầu `AudioWorklet` để hoạt động.

## Tổng quan

`@sapphi-red/web-noise-suppressor` là thư viện TypeScript hướng trình duyệt, xuất `AudioWorkletNode` cho dọn tiếng ồn đầu vào theo thời gian thực. Gói này được thiết kế cho pipeline micro và có thể dùng cùng các ràng buộc WebRTC của trình duyệt (`noiseSuppression`, `echoCancellation`) khi cần.

Dòng xử lý chính:

1. Tải binary wasm (`loadSpeex` / `loadRnnoise`).
2. Đăng ký module processor của worklet (`audioWorklet.addModule(...)`).
3. Tạo node (`new SpeexWorkletNode(...)`, ...).
4. Kết nối graph âm thanh.

## Mục lục

- [Tổng quan](#tổng-quan)
- [Tính năng](#tính-năng)
- [Cài đặt](#cài-đặt)
- [Yêu cầu](#yêu-cầu)
- [Sử dụng](#sử-dụng)
- [Cấu hình](#cấu-hình)
- [Cấu trúc dự án](#cấu-trúc-dự-án)
- [Ví dụ](#ví-dụ)
- [Phát triển](#phát-triển)
- [Ghi chú phát triển](#ghi-chú-phát-triển)
- [Khắc phục sự cố](#khắc-phục-sự-cố)
- [❤️ Support](#-support)
- [Lộ trình](#lộ-trình)
- [Đóng góp](#đóng-góp)
- [Giấy phép](#giấy-phép)

## Tính năng

- Ba tùy chọn xử lý với một mẫu sử dụng Web Audio thống nhất.
- Hỗ trợ export entrypoint cho ESM + CJS.
- Đường dẫn export riêng cho file JS worklet và binary wasm.
- Hỗ trợ phát hiện SIMD của RNNoise qua `loadRnnoise`.
- Demo app (`demo/`) có định tuyến micro trực tiếp và visualizer.

## Cài đặt

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## Yêu cầu

- Trình duyệt/runtime hỗ trợ `AudioWorklet`.
- Ngữ cảnh an toàn để ghi âm micro (`https://` hoặc localhost).
- Một chiến lược bundler có thể cung cấp URL tham chiếu cho file JS worklet và wasm.
  - Các ví dụ được mô tả theo hướng Vite.

## Sử dụng

Phần này được viết riêng cho người dùng Vite.

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

Để biết chi tiết hơn, xem [mã nguồn demo](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## Cấu hình

### Tùy chọn Node

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (từ `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (từ `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, mặc định là `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### Tải RNNoise

`loadRnnoise` chấp nhận cả URL không SIMD và SIMD rồi tự chọn ở runtime:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Exported subpaths

Gói này export:

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## Ví dụ

### Ví dụ Speex

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### Ví dụ RNNoise

```ts
import { RnnoiseWorkletNode, loadRnnoise } from '@sapphi-red/web-noise-suppressor'
import rnnoiseWorkletPath from '@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js?url'
import rnnoiseWasmPath from '@sapphi-red/web-noise-suppressor/rnnoise.wasm?url'
import rnnoiseSimdWasmPath from '@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm?url'

const ctx = new AudioContext({ sampleRate: 48000 })
const wasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
await ctx.audioWorklet.addModule(rnnoiseWorkletPath)

const node = new RnnoiseWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### Ví dụ Noise Gate

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

## Cấu trúc dự án

```text
.
├── src/
│   ├── noiseGate/          # logic gate + worklet node/processor
│   ├── rnnoise/            # tích hợp rnnoise + worklet node/processor
│   ├── speex/              # tích hợp speex preprocess + worklet node/processor
│   ├── utils/              # shared utilities
│   └── index.ts            # public API exports
├── demo/                   # Vite demo application
├── patches/                # pnpm patch for rnnoise-wasm
├── i18n/                   # translation artifacts
├── tsdown.config.ts        # build config
└── package.json
```

## Phát triển

### Thiết lập

```shell
pnpm install
```

### Script gốc

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### Script demo

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

Trong `demo/package.json`, `build:all` cũng có sẵn:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Ghi chú phát triển

- Worklet node RNNoise có giả định 48kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` và `SpeexWorkletNode` cung cấp `destroy()` để giải phóng tài nguyên bên phía wasm.
- Build output được tạo bởi `tsdown` và bao gồm asset wasm đã copy vào `dist/`.
- Patch nội bộ được áp dụng cho `@shiguredo/rnnoise-wasm` qua pnpm patched dependencies.

## Khắc phục sự cố

| Triệu chứng | Cần kiểm tra |
| --- | --- |
| `AudioWorklet is not defined` | Kiểm tra trình duyệt có hỗ trợ `AudioWorklet` và context có bảo mật. |
| `Failed to execute 'addModule'` | Đảm bảo đường dẫn JS của worklet giải quyết thành URL hợp lệ tại runtime. |
| Lỗi khi load wasm (`fetch`/404/CORS) | Xác nhận asset wasm đã được sao chép/phục vụ đúng và chiến lược `url import` của bundler bạn là đúng. |
| Không có tác động nghe được | Kiểm tra nối graph (`source -> node -> destination`) và các ràng buộc WebRTC có đang được bật. |
| Hành vi RNNoise có vẻ không ổn định | Dùng hoặc thử nghiệm với audio context/sample rate 48kHz. |

## Lộ trình

- Mở rộng tài liệu tích hợp không phải Vite.
- Thêm các phiên bản README đã dịch dưới `i18n/`.
- Mô tả chi tiết hơn đặc điểm hiệu năng và tradeoff của processor.

## Đóng góp

Issue và pull request đều được chào đón:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

Trước khi mở PR, hãy chạy:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## Giấy phép

MIT License. Xem [LICENSE](./LICENSE).


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
