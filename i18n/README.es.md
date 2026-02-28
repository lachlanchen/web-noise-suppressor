[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red/web-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)
[![Stars](https://img.shields.io/github/stars/sapphi-red/web-noise-suppressor?style=flat-square)](https://github.com/sapphi-red/web-noise-suppressor/stargazers)

Nodos de supresión de ruido para la API Web Audio, pensados para la limpieza del micrófono en tiempo real en el navegador.

[![🎧 Probar demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Enfoque | Qué cubre este README |
| --- | --- |
| Runtime | Flujo de Web Audio Worklet para supresión de ruido en el navegador |
| Core output | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Usage pattern | Cargar WASM → registrar processor → instanciar nodo → conectar grafo |

Este paquete ofrece tres nodos de supresión de ruido.

| Nodo | Descripción | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Implementación sencilla de una puerta de ruido | Lógica nativa |
| `RnnoiseWorkletNode` | Supresor basado en RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) a través de [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Supresor Speex preprocess | `preprocess` de [xiph/speexdsp](https://github.com/xiph/speexdsp) a través de [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Este paquete requiere `AudioWorklet` para funcionar.

## Overview

`@sapphi-red/web-noise-suppressor` es una biblioteca TypeScript enfocada en navegador que expone envoltorios (`wrapper`) de `AudioWorkletNode` para limpieza de entrada en tiempo real. Está diseñada para pipelines de micrófono y puede usarse junto con restricciones WebRTC del navegador (`noiseSuppression`, `echoCancellation`) cuando sea necesario.

El flujo central es:

1. Cargar el binario wasm (`loadSpeex` / `loadRnnoise`).
2. Registrar el módulo del procesador worklet (`audioWorklet.addModule(...)`).
3. Construir el nodo (`new SpeexWorkletNode(...)`, etc.).
4. Conectar el grafo de audio.

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
- [❤️ Support](#-support)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

## Features

- Tres opciones de procesamiento con un patrón de uso compartido en Web Audio.
- Punto de entrada de librería ESM + CJS para exportaciones.
- Rutas de exportación dedicadas para archivos JS del worklet y binarios wasm.
- Detección de SIMD de RNNoise vía `loadRnnoise`.
- Aplicación de demo (`demo/`) con enrutado de micrófono en vivo y visualizador.

## Installation

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## Prerequisites

- Un navegador/entorno con soporte para `AudioWorklet`.
- Un contexto seguro para captura del micrófono (`https://` o localhost).
- Una estrategia de bundler que pueda proporcionar referencias URL para archivos JS del worklet y binarios wasm.
  - Los ejemplos documentados están orientados a Vite.

## Usage

Esta sección está escrita solo para usuarios de Vite.

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

Para más detalles, consulta el [código fuente de la demo](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## Configuration

### Node options

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (de `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (de `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, por defecto `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise loading

`loadRnnoise` acepta tanto URLs non-SIMD como SIMD y selecciona una en tiempo de ejecución:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Exported subpaths

El paquete exporta:

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

De `demo/package.json` también está disponible `build:all`:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Development Notes

- El nodo worklet de RNNoise incluye una suposición de 48 kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` y `SpeexWorkletNode` exponen `destroy()` para finalizar recursos del lado de wasm.
- La salida de build se genera por `tsdown` e incluye activos wasm copiados en `dist/`.
- Se aplica un parche local a `@shiguredo/rnnoise-wasm` mediante dependencias parcheadas de pnpm.

## Troubleshooting

- `AudioWorklet is not defined`
  - Verifica el soporte del navegador y el contexto seguro.
- `Failed to execute 'addModule'`
  - Asegúrate de que la ruta JS del worklet se resuelva a una URL válida en tiempo de ejecución.
| Symptom | Qué verificar |
| --- | --- |
| `AudioWorklet is not defined` | Verifica el soporte del navegador y el contexto seguro. |
| `Failed to execute 'addModule'` | Asegúrate de que la ruta JS del worklet se resuelva a una URL válida en tiempo de ejecución. |
| Fallos al cargar wasm (`fetch`/404/CORS) | Confirma que los activos wasm se copien/sirvan y tu estrategia de importación de URL del bundler sea correcta. |
| Sin efecto audible | Comprueba el cableado del grafo (`source -> node -> destination`) y si también están habilitadas las restricciones de WebRTC. |
| El comportamiento de RNNoise parece inestable | Usa o prueba con un contexto/tasa de muestreo de audio de 48kHz. |

## ❤️ Support

| Donate | PayPal | Stripe |
|---|---|---|
| [![Donate](https://img.shields.io/badge/Donate-LazyingArt-0EA5E9?style=for-the-badge&logo=ko-fi&logoColor=white)](https://chat.lazying.art/donate) | [![PayPal](https://img.shields.io/badge/PayPal-RongzhouChen-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen) | [![Stripe](https://img.shields.io/badge/Stripe-Donate-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## Roadmap

- Ampliar la documentación de integración para entornos no Vite.
- Agregar variantes de README traducidas en `i18n/`.
- Documentar con más detalle las características de rendimiento y los tradeoffs entre procesadores.

## Contributing

Las issues y pull requests son bienvenidas:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repositorio: <https://github.com/sapphi-red/web-noise-suppressor>

Antes de abrir un PR, ejecuta:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## License

Licencia MIT. Consulta [LICENSE](./LICENSE).
