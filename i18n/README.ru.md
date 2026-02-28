[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red/web-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)
[![Stars](https://img.shields.io/github/stars/sapphi-red/web-noise-suppressor?style=flat-square)](https://github.com/sapphi-red/web-noise-suppressor/stargazers)

Ноды шумоподавления для Web Audio API — предназначены для очистки микрофонного сигнала в браузере в реальном времени.

[![🎧 Live Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Направление | Что описывает этот README |
| --- | --- |
| Runtime | Web Audio Worklet pipeline для подавления шума в браузере |
| Основной результат | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Паттерн использования | Загрузить WASM → зарегистрировать процессор → создать узел → подключить граф |

Эта библиотека предоставляет три ноды шумоподавления.

| Узел | Описание | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Простая реализация шумового гейта | Встроенная логика |
| `RnnoiseWorkletNode` | Подавление шума на основе RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) через [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Подавление через Speex preprocess | `preprocess` из [xiph/speexdsp](https://github.com/xiph/speexdsp) через [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Для работы этого пакета требуется `AudioWorklet`.

## Обзор

`@sapphi-red/web-noise-suppressor` — это браузерная библиотека на TypeScript, которая предоставляет обёртки `AudioWorkletNode` для очистки входного сигнала в реальном времени. Она ориентирована на конвейеры микрофона и при необходимости может использоваться вместе с ограничениями WebRTC браузера (`noiseSuppression`, `echoCancellation`).

Основной поток такой:

1. Загрузка бинарного wasm-файла (`loadSpeex` / `loadRnnoise`).
2. Регистрация модуля процессора Worklet (`audioWorklet.addModule(...)`).
3. Создание ноды (`new SpeexWorkletNode(...)` и т.д.).
4. Подключение аудиографа.

## Содержание

- [Обзор](#обзор)
- [Возможности](#возможности)
- [Установка](#установка)
- [Предварительные требования](#предварительные-требования)
- [Использование](#использование)
- [Настройка](#настройка)
- [Структура проекта](#структура-проекта)
- [Примеры](#примеры)
- [Разработка](#разработка)
- [Заметки по разработке](#заметки-по-разработке)
- [Устранение неполадок](#устранение-неполадок)
- [❤️ Support](#-support)
- [Дорожная карта](#дорожная-карта)
- [Участие](#участие)
- [Лицензия](#лицензия)

## Возможности

- Три варианта обработки с единым шаблоном использования Web Audio.
- Экспорт точки входа библиотеки в форматах ESM + CJS.
- Отдельные пути экспорта для JS-файлов Worklet и бинарных файлов wasm.
- Поддержка определения SIMD в RNNoise через `loadRnnoise`.
- Демо-приложение (`demo/`) с маршрутизацией микрофона в реальном времени и визуализатором.

## Установка

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## Предварительные требования

- Браузер или runtime со поддержкой `AudioWorklet`.
- Безопасный контекст для захвата микрофона (`https://` или localhost).
- Стратегия сборки, которая умеет подставлять URL для JS-файлов Worklet и бинарников wasm.
  - Примеры в документации ориентированы на Vite.

## Использование

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

Подробности в [исходном коде демо](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## Настройка

### Опции нод

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

### Загрузка RNNoise

`loadRnnoise` принимает non-SIMD и SIMD URL и выбирает один из них во время выполнения:

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
│   ├── noiseGate/          # логика gate + worklet node/processor
│   ├── rnnoise/            # интеграция rnnoise + worklet node/processor
│   ├── speex/              # интеграция speex preprocess + worklet node/processor
│   ├── utils/              # общие утилиты
│   └── index.ts            # экспорт публичного API
├── demo/                   # демо-приложение Vite
├── patches/                # pnpm patch для rnnoise-wasm
├── i18n/                   # артефакты локализации
├── tsdown.config.ts        # конфиг сборки
└── package.json
```

## Разработка

### Подготовка

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

Из `demo/package.json` также доступен `build:all`:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Заметки по разработке

- Нода RNNoise worklet предполагает частоту 48kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` и `SpeexWorkletNode` имеют `destroy()` для завершения ресурсов на стороне wasm.
- Выходные артефакты сборки генерируются через `tsdown` и включают скопированные wasm-ассеты в `dist/`.
- Локальный патч применяется к `@shiguredo/rnnoise-wasm` через patched dependencies в pnpm.

## Устранение неполадок

- `AudioWorklet is not defined`
  - Проверьте поддержку в браузере и корректность безопасного контекста.
- `Failed to execute 'addModule'`
  - Проверьте, что путь к worklet JS корректно разрешается в валидный URL во время выполнения.
| Симптом | Что проверить |
| --- | --- |
| `AudioWorklet is not defined` | Проверьте поддержку в браузере и корректность безопасного контекста. |
| `Failed to execute 'addModule'` | Проверьте, что путь к worklet JS корректно разрешается в валидный URL во время выполнения. |
| Сбои загрузки wasm (`fetch`/404/CORS) | Убедитесь, что wasm-ресурсы скопированы/доступны, а стратегия импорта URL в вашем сборщике настроена правильно. |
| Нет слышимого эффекта | Проверьте цепочку графа (`source -> node -> destination`) и включены ли ограничения WebRTC. |
| Поведение RNNoise нестабильно | Используйте/проверяйте аудио контекст с частотой 48kHz. |

## ❤️ Support

| Donate | PayPal | Stripe |
|---|---|---|
| [![Donate](https://img.shields.io/badge/Donate-LazyingArt-0EA5E9?style=for-the-badge&logo=ko-fi&logoColor=white)](https://chat.lazying.art/donate) | [![PayPal](https://img.shields.io/badge/PayPal-RongzhouChen-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen) | [![Stripe](https://img.shields.io/badge/Stripe-Donate-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## Дорожная карта

- Расширить документацию по интеграции без привязки к Vite.
- Добавить версии README на разных языках в `i18n/`.
- Подробнее описать характеристики производительности и компромиссы между процессорами.

## Участие

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
