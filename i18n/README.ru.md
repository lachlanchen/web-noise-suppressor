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

Блоки шумоподавления для Web Audio API, созданные для очистки микрофонного сигнала в реальном времени прямо в браузере.

[![🎧 Try Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Быстрый доступ | Ссылка |
| --- | --- |
| 🎧 Live demo | [web-noise-suppressor.sapphi.red](https://web-noise-suppressor.sapphi.red) |
| 📦 npm-пакет | [@sapphi-red/web-noise-suppressor](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor) |
| 📄 Точки входа | [src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/src/index.ts) |
| 🧪 Исходный код демо | [demo/src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) |

| Фокус | Что описывает этот README |
| --- | --- |
| Runtime | Конвейер Web Audio Worklet для подавления шума в браузере |
| Основной результат | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Схема использования | Загрузить WASM → зарегистрировать процессор → создать узел → подключить граф |

Эта библиотека предоставляет три узла шумоподавления.

| Узел | Описание | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Простая реализация шумового затвора | Нативная логика |
| `RnnoiseWorkletNode` | Подавление шума на основе RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) через [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Подавление с помощью Speex preprocess | `preprocess` из [xiph/speexdsp](https://github.com/xiph/speexdsp) via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Для работы этого пакета требуется `AudioWorklet`.

## Обзор

`@sapphi-red/web-noise-suppressor` — это библиотека TypeScript для браузера, которая предоставляет обёртки `AudioWorkletNode` для очистки входящего аудио в реальном времени. Она ориентирована на микрофонные пайплайны и при необходимости может использоваться вместе с ограничениями WebRTC браузера (`noiseSuppression`, `echoCancellation`).

Базовый цикл такой:

1. Загрузить бинарник wasm (`loadSpeex` / `loadRnnoise`).
2. Зарегистрировать модуль worklet-процессора (`audioWorklet.addModule(...)`).
3. Создать узел (`new SpeexWorkletNode(...)` и т.д.).
4. Подключить граф аудио.

## Содержание

- [Обзор](#обзор)
- [Возможности](#возможности)
- [Установка](#установка)
- [Требования](#требования)
- [Использование](#использование)
- [Конфигурация](#конфигурация)
- [Структура проекта](#структура-проекта)
- [Примеры](#примеры)
- [Разработка](#разработка)
- [Заметки по разработке](#заметки-по-разработке)
- [Устранение неполадок](#устранение-неполадок)
- [План развития](#план-развития)
- [Вклад](#вклад)
- [❤️ Support](#-support)
- [Лицензия](#лицензия)

## Возможности

- Три варианта обработки с единым паттерном использования Web Audio.
- Экспорт основных входных точек ESM и CJS.
- Отдельные точки экспорта для JS-файлов worklet и бинарных файлов wasm.
- Поддержка определения SIMD в `loadRnnoise`.
- Демо-приложение (`demo/`) с живым маршрутом микрофонного сигнала и визуализатором.

## Установка

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## Требования

- Браузер или runtime с поддержкой `AudioWorklet`.
- Защищённый контекст для захвата микрофона (`https://` или localhost).
- Стратегия сборки, которая может подставлять URL-ссылки для JS-файлов worklet и бинарных файлов wasm.
  - Примеры в документации ориентированы на Vite.

## Использование

Этот раздел написан только для пользователей Vite.

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url' // вы можете использовать `vite-plugin-static-copy` вместо этого

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

Подробности см. в [исходном коде демо](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## Конфигурация

### Опции узлов

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (из `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (из `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, по умолчанию `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### Загрузка RNNoise

`loadRnnoise` принимает URL без SIMD и с SIMD и выбирает один из них во время выполнения:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Экспортируемые подмодули

Пакет экспортирует:

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## Примеры

### Пример Speex

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### Пример RNNoise

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

### Пример Noise Gate

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

## Структура проекта

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

## Разработка

### Настройка

```shell
pnpm install
```

### Корневые скрипты

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### Скрипты демо

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

В `demo/package.json` также доступен `build:all`:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Заметки по разработке

- Ворклет-узел RNNoise использует предположение 48kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` и `SpeexWorkletNode` предоставляют `destroy()` для освобождения wasm-ресурсов.
- Результат сборки создаётся с помощью `tsdown` и включает копирование wasm-ресурсов в `dist/`.
- Локальный патч применяется к `@shiguredo/rnnoise-wasm` через зависимостей pnpm с patch.

## Устранение неполадок

| Симптом | Что проверить |
| --- | --- |
| `AudioWorklet is not defined` | Проверьте поддержку `AudioWorklet` в браузере и защищённый контекст. |
| `Failed to execute 'addModule'` | Убедитесь, что путь к worklet JS корректно разрешается в действительный URL во время выполнения. |
| Ошибки загрузки wasm (`fetch`/404/CORS) | Убедитесь, что wasm-ресурсы скопированы/доступны и стратегия импорта URL в вашем бандлере настроена правильно. |
| Нет слышимого эффекта | Проверьте граф (`source -> node -> destination`) и включены ли ограничения WebRTC. |
| Поведение RNNoise нестабильно | Используйте или протестируйте аудио контекст с частотой дискретизации 48kHz. |

## Путевой план

- Расширить документацию интеграции вне Vite.
- Добавить README на разных языках в `i18n/`.
- Подробнее описать характеристики производительности и компромиссы процессоров.

## Вклад

Issues и pull request приветствуются:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Репозиторий: <https://github.com/sapphi-red/web-noise-suppressor>

Перед открытием PR запустите:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## Лицензия

MIT License. См. [LICENSE](./LICENSE).


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
