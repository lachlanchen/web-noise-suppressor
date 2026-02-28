[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# @sapphi-red/web-noise-suppressor

**Tùy chọn ngôn ngữ:** Tiếng Anh (`README.md`) | thư mục i18n: `i18n/` (hiện chưa có tệp README đã dịch)

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)

Các node khử nhiễu cho Web Audio API.

[🎧 Demo](https://web-noise-suppressor.sapphi.red)

Gói này cung cấp ba node khử nhiễu.

| Node | Mô tả | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Triển khai noise gate đơn giản | Logic gốc |
| `RnnoiseWorkletNode` | Bộ khử nhiễu dựa trên RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) thông qua [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Bộ khử nhiễu Speex preprocess | `preprocess` của [xiph/speexdsp](https://github.com/xiph/speexdsp) thông qua [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Gói này yêu cầu `AudioWorklet` để hoạt động.

## Tổng quan

`@sapphi-red/web-noise-suppressor` là thư viện TypeScript tập trung cho trình duyệt, cung cấp các wrapper `AudioWorkletNode` để làm sạch âm thanh đầu vào theo thời gian thực. Thư viện được thiết kế cho pipeline microphone và có thể dùng cùng các ràng buộc WebRTC của trình duyệt (`noiseSuppression`, `echoCancellation`) khi cần.

Luồng cốt lõi là:

1. Tải binary wasm (`loadSpeex` / `loadRnnoise`).
2. Đăng ký module worklet processor (`audioWorklet.addModule(...)`).
3. Khởi tạo node (`new SpeexWorkletNode(...)`, v.v.).
4. Kết nối đồ thị âm thanh.

## Tính năng

- Ba tùy chọn xử lý với cùng một mẫu sử dụng Web Audio.
- Xuất entrypoint thư viện dạng ESM + CJS.
- Các đường dẫn export riêng cho tệp JS worklet và binary wasm.
- Hỗ trợ phát hiện SIMD của RNNoise thông qua `loadRnnoise`.
- Ứng dụng demo (`demo/`) với định tuyến microphone trực tiếp và visualizer.

## Cài đặt

```shell
npm i @sapphi-red/web-noise-suppressor # yarn add @sapphi-red/web-noise-suppressor
```

## Điều kiện tiên quyết

- Trình duyệt/runtime có hỗ trợ `AudioWorklet`.
- Môi trường bảo mật cho thu microphone (`https://` hoặc localhost).
- Cách cấu hình bundler có thể cung cấp tham chiếu URL cho tệp JS worklet và tệp wasm.
  - Các ví dụ trong tài liệu được viết theo hướng Vite.

## Cách dùng

Phần này chỉ được viết cho người dùng vite.

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

Để biết thêm chi tiết, xem [mã nguồn demo](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

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

`loadRnnoise` chấp nhận cả URL non-SIMD và SIMD, sau đó chọn một phiên bản tại runtime:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Các subpath được export

Gói xuất ra các đường dẫn sau:

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
├── i18n/                   # translation artifacts (currently empty)
├── tsdown.config.ts        # build config
└── package.json
```

## Phát triển

### Thiết lập

```shell
pnpm install
```

### Script ở thư mục gốc

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### Script cho demo

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

Từ `demo/package.json`, cũng có `build:all`:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Ghi chú phát triển

- Node worklet RNNoise có giả định 48kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` và `SpeexWorkletNode` cung cấp `destroy()` để giải phóng tài nguyên phía wasm.
- Output build được tạo bởi `tsdown` và bao gồm các tài nguyên wasm đã sao chép vào `dist/`.
- Một bản vá cục bộ được áp dụng cho `@shiguredo/rnnoise-wasm` thông qua patched dependencies của pnpm.

## Khắc phục sự cố

- `AudioWorklet is not defined`
  - Xác minh trình duyệt có hỗ trợ và đang chạy trong secure context.
- `Failed to execute 'addModule'`
  - Đảm bảo đường dẫn JS worklet được resolve thành URL hợp lệ tại runtime.
- Lỗi tải wasm (`fetch`/404/CORS)
  - Xác nhận tài nguyên wasm đã được sao chép/phục vụ và chiến lược import URL của bundler là chính xác.
- Không có hiệu ứng nghe được
  - Kiểm tra nối graph (`source -> node -> destination`) và xem các ràng buộc WebRTC có đang được bật đồng thời không.
- Hành vi RNNoise có vẻ không ổn định
  - Dùng hoặc kiểm thử với audio context/sample rate 48kHz.

## Lộ trình

- Mở rộng tài liệu tích hợp ngoài Vite.
- Thêm các biến thể README đã dịch trong `i18n/`.
- Tài liệu hóa chi tiết hơn về đặc tính hiệu năng và trade-off của từng processor.

## Đóng góp

Issue và pull request đều được hoan nghênh:

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
