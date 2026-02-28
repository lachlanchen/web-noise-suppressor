[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red/web-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)
[![Stars](https://img.shields.io/github/stars/sapphi-red/web-noise-suppressor?style=flat-square)](https://github.com/sapphi-red/web-noise-suppressor/stargazers)

Các node khử tiếng ồn cho Web Audio API — thiết kế để làm sạch tiếng nói từ microphone theo thời gian thực trên trình duyệt.

[![🎧 Try Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Tiêu chí | Nội dung README |
| --- | --- |
| Runtime | Pipeline Web Audio Worklet để khử tiếng ồn trên trình duyệt |
| Kết quả chính | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Mẫu sử dụng | Tải WASM → đăng ký processor → tạo node → nối graph |

Gói này cung cấp ba node khử tiếng ồn.

| Node | Mô tả | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Triển khai noise gate đơn giản | Logic gốc |
| `RnnoiseWorkletNode` | Bộ khử tiếng ồn dựa trên RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) qua [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Bộ khử tiếng ồn Speex preprocess | `preprocess` của [xiph/speexdsp](https://github.com/xiph/speexdsp) qua [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Gói này yêu cầu `AudioWorklet` để hoạt động.

## Tổng quan

`@sapphi-red/web-noise-suppressor` là thư viện TypeScript hướng trình duyệt, cung cấp wrapper cho `AudioWorkletNode` để làm sạch đầu vào theo thời gian thực. Được thiết kế cho pipeline micro và có thể dùng cùng các ràng buộc WebRTC của trình duyệt (`noiseSuppression`, `echoCancellation`) khi cần.

Luồng hoạt động chính là:

1. Tải nhị phân wasm (`loadSpeex` / `loadRnnoise`).
2. Đăng ký module worklet processor (`audioWorklet.addModule(...)`).
3. Tạo node (`new SpeexWorkletNode(...)`, v.v.).
4. Kết nối đồ thị âm thanh.

## Mục lục

- [Tổng quan](#tổng-quan)
- [Tính năng](#tính-năng)
- [Cài đặt](#cài-đặt)
- [Yêu cầu](#yêu-cầu)
- [Cách dùng](#cách-dùng)
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

- Ba lựa chọn xử lý với cùng một mẫu sử dụng Web Audio.
- Điểm vào thư viện ESM + CJS.
- Đường dẫn export riêng cho file JS worklet và binary wasm.
- Hỗ trợ phát hiện SIMD RNNoise thông qua `loadRnnoise`.
- Ứng dụng demo (`demo/`) với định tuyến micro trực tiếp và visualizer.

## Cài đặt

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## Yêu cầu

- Trình duyệt/runtime hỗ trợ `AudioWorklet`.
- Môi trường an toàn cho việc ghi âm microphone (`https://` hoặc localhost).
- Chiến lược bundler có thể cung cấp tham chiếu URL cho file JS của worklet và file wasm.
  - Các ví dụ trong tài liệu được thiết kế theo hướng Vite.

## Cách dùng

Phần này chỉ viết cho người dùng Vite.

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

Xem thêm chi tiết ở [mã nguồn demo](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## Cấu hình

### Tùy chọn node

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

`loadRnnoise` chấp nhận URL non-SIMD và SIMD và chọn một trong hai lúc runtime:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Đường dẫn export con

Gói cung cấp các subpath sau:

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
  simdUrl: rnnoiseSimdWasmPath
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
│   ├── noiseGate/          # gate logic + worklet node/processor
│   ├── rnnoise/            # rnnoise integration + worklet node/processor
│   ├── speex/              # speex preprocess integration + worklet node/processor
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

Từ `demo/package.json`, có sẵn lệnh `build:all`:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Ghi chú phát triển

- Node worklet RNNoise có giả định tần số lấy mẫu 48kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` và `SpeexWorkletNode` cung cấp `destroy()` để giải phóng tài nguyên ở phía wasm.
- Đầu ra build được sinh bởi `tsdown` và bao gồm tài nguyên wasm đã sao chép vào `dist/`.
- Một patch cục bộ được áp dụng cho `@shiguredo/rnnoise-wasm` thông qua các dependency đã vá của pnpm.

## Khắc phục sự cố

- `AudioWorklet is not defined`
  - Kiểm tra trình duyệt có hỗ trợ và đang chạy trong secure context.
- `Failed to execute 'addModule'`
  - Đảm bảo đường dẫn JS của worklet được resolve thành URL hợp lệ tại runtime.
- Lỗi khi tải wasm (`fetch`/404/CORS)
  - Kiểm tra tài nguyên wasm được sao chép/phục vụ đúng và chiến lược import URL của bundler là chính xác.
- Không có hiệu ứng nghe thấy
  - Kiểm tra việc nối graph (`source -> node -> destination`) và xem các ràng buộc WebRTC có đồng thời bật không.
- Hành vi RNNoise không ổn định
  - Sử dụng/kiểm thử với audio context có sample rate 48kHz.

| Symptom | What to check |
| --- | --- |
| `AudioWorklet is not defined` | Kiểm tra hỗ trợ trình duyệt và secure context. |
| `Failed to execute 'addModule'` | Đảm bảo đường dẫn JS của worklet được resolve thành URL hợp lệ ở runtime. |
| Lỗi khi tải wasm (`fetch`/404/CORS) | Kiểm tra tài nguyên wasm đã sao chép/phục vụ và chiến lược import URL của bundler có đúng hay không. |
| Không có hiệu ứng nghe thấy | Kiểm tra việc nối graph (`source -> node -> destination`) và xem các ràng buộc WebRTC có đang bật cùng lúc không. |
| RNNoise có vẻ không ổn định | Dùng hoặc kiểm thử với audio context/sample rate 48kHz. |

## ❤️ Support

| Donate | PayPal | Stripe |
|---|---|---|
| [![Donate](https://img.shields.io/badge/Donate-LazyingArt-0EA5E9?style=for-the-badge&logo=ko-fi&logoColor=white)](https://chat.lazying.art/donate) | [![PayPal](https://img.shields.io/badge/PayPal-RongzhouChen-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen) | [![Stripe](https://img.shields.io/badge/Stripe-Donate-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## Lộ trình

- Mở rộng tài liệu tích hợp ngoài Vite.
- Thêm các biến thể README đã dịch trong `i18n/`.
- Tài liệu hóa chi tiết hơn về đặc tính hiệu năng và trade-off của từng processor.

## Đóng góp

Issue và pull request đều được hoan nghênh:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

Trước khi mở PR, vui lòng chạy:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## Giấy phép

Giấy phép MIT. Xem [LICENSE](./LICENSE).
