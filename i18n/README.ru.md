[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)

Ноды шумоподавления для Web Audio API.

[🎧 Demo](https://web-noise-suppressor.sapphi.red)

Этот пакет предоставляет три ноды шумоподавления.

| Node | Description | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Простая реализация noise gate | Встроенная логика |
| `RnnoiseWorkletNode` | Подавление шума на основе RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) через [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Подавление через Speex preprocess | `preprocess` из [xiph/speexdsp](https://github.com/xiph/speexdsp) через [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Для работы этого пакета требуется `AudioWorklet`.

## Overview

`@sapphi-red/web-noise-suppressor` — библиотека TypeScript, ориентированная на браузер, которая предоставляет обёртки `AudioWorkletNode` для очистки входящего аудио в реальном времени. Она рассчитана на микрофонные пайплайны и при необходимости может использоваться вместе с WebRTC-ограничениями браузера (`noiseSuppression`, `echoCancellation`).

Базовый процесс выглядит так:

1. Загрузить wasm-бинарник (`loadSpeex` / `loadRnnoise`).
2. Зарегистрировать модуль процессора worklet (`audioWorklet.addModule(...)`).
3. Создать ноду (`new SpeexWorkletNode(...)` и т. д.).
4. Подключить аудиограф.

## Features

- Три варианта обработки с единым паттерном использования Web Audio.
- Экспорт входных точек библиотеки в форматах ESM + CJS.
- Отдельные пути экспорта для worklet JS-файлов и wasm-бинарников.
- Поддержка определения RNNoise SIMD через `loadRnnoise`.
- Демо-приложение (`demo/`) с маршрутизацией микрофона в реальном времени и визуализатором.

## Install

```shell
npm i @sapphi-red/web-noise-suppressor # yarn add @sapphi-red/web-noise-suppressor
```

## Prerequisites

- Браузер/среда выполнения с поддержкой `AudioWorklet`.
- Безопасный контекст для захвата микрофона (`https://` или localhost).
- Стратегия сборщика, позволяющая предоставлять URL-ссылки на worklet JS и wasm-файлы.
  - Примеры в документации ориентированы на Vite.

## Usage

Этот раздел написан только для пользователей Vite.

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url' // можно использовать `vite-plugin-static-copy` вместо этого

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

Подробнее см. [исходный код демо](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## Configuration

### Node options

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (из `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (из `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (дБ)
  - `closeThreshold?: number` (дБ, по умолчанию `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise loading

`loadRnnoise` принимает URL как для non-SIMD, так и для SIMD, и выбирает подходящий вариант во время выполнения:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Exported subpaths

Пакет экспортирует:

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
│   ├── noiseGate/          # логика gate + worklet node/processor
│   ├── rnnoise/            # интеграция rnnoise + worklet node/processor
│   ├── speex/              # интеграция speex preprocess + worklet node/processor
│   ├── utils/              # общие утилиты
│   └── index.ts            # экспорты публичного API
├── demo/                   # демо-приложение Vite
├── patches/                # pnpm-патч для rnnoise-wasm
├── i18n/                   # артефакты перевода
├── tsdown.config.ts        # конфиг сборки
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

Из `demo/package.json` также доступен `build:all`:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Development Notes

- Нода RNNoise worklet включает допущение о 48kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` и `SpeexWorkletNode` предоставляют `destroy()` для завершения ресурсов на стороне wasm.
- Выходные артефакты сборки генерируются `tsdown` и включают скопированные wasm-ассеты в `dist/`.
- Локальный патч применяется к `@shiguredo/rnnoise-wasm` через patched dependencies в pnpm.

## Troubleshooting

- `AudioWorklet is not defined`
  - Проверьте поддержку браузером и безопасный контекст.
- `Failed to execute 'addModule'`
  - Убедитесь, что путь к worklet JS во время выполнения разрешается в валидный URL.
- сбои загрузки wasm (`fetch`/404/CORS)
  - Подтвердите, что wasm-ассеты копируются/отдаются, и что стратегия URL-импорта в вашем сборщике настроена корректно.
- Нет слышимого эффекта
  - Проверьте подключение графа (`source -> node -> destination`) и включены ли одновременно WebRTC-ограничения.
- Поведение RNNoise кажется нестабильным
  - Используйте или протестируйте с аудиоконтекстом/частотой дискретизации 48kHz.

## Roadmap

- Расширить документацию по интеграции, не связанной с Vite.
- Добавить переведённые варианты README в `i18n/`.
- Подробнее описать характеристики производительности и компромиссы между процессорами.

## Contributing

Issues и pull request приветствуются:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

Перед открытием PR выполните:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## License

MIT License. См. [LICENSE](./LICENSE).
