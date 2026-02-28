[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# @sapphi-red/web-noise-suppressor

**Opciones de idioma:** Inglés (`README.md`) | directorio i18n: `i18n/` (actualmente no hay archivos README traducidos)

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)

Nodos de supresión de ruido para la API Web Audio.

[🎧 Demo](https://web-noise-suppressor.sapphi.red)

Este paquete ofrece tres nodos de supresión de ruido.

| Nodo | Descripción | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Implementación simple de una puerta de ruido | Lógica nativa |
| `RnnoiseWorkletNode` | Supresor basado en RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) mediante [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Supresor Speex preprocess | `preprocess` de [xiph/speexdsp](https://github.com/xiph/speexdsp) mediante [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Este paquete requiere `AudioWorklet` para funcionar.

## Descripción general

`@sapphi-red/web-noise-suppressor` es una biblioteca TypeScript enfocada en navegador que expone wrappers de `AudioWorkletNode` para limpiar entrada en tiempo real. Está diseñada para flujos de micrófono y puede usarse junto con restricciones WebRTC del navegador (`noiseSuppression`, `echoCancellation`) cuando sea necesario.

El flujo principal es:

1. Cargar el binario wasm (`loadSpeex` / `loadRnnoise`).
2. Registrar el módulo del procesador worklet (`audioWorklet.addModule(...)`).
3. Crear el nodo (`new SpeexWorkletNode(...)`, etc.).
4. Conectar el grafo de audio.

## Características

- Tres opciones de procesamiento con un patrón de uso compartido de Web Audio.
- Exportaciones de entrada de biblioteca en formato ESM + CJS.
- Rutas de exportación dedicadas para archivos JS de worklet y binarios wasm.
- Soporte de detección SIMD de RNNoise mediante `loadRnnoise`.
- App de demostración (`demo/`) con enrutamiento de micrófono en vivo y visualizador.

## Instalación

```shell
npm i @sapphi-red/web-noise-suppressor # yarn add @sapphi-red/web-noise-suppressor
```

## Requisitos previos

- Un navegador/runtime con soporte para `AudioWorklet`.
- Un contexto seguro para captura de micrófono (`https://` o localhost).
- Una estrategia de bundler que pueda proporcionar referencias URL para archivos JS de worklet y archivos wasm.
  - Los ejemplos documentados están orientados a Vite.

## Uso

Esta sección está escrita solo para usuarios de vite.

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

## Configuración

### Opciones del nodo

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

### Carga de RNNoise

`loadRnnoise` acepta tanto URLs non-SIMD como SIMD y selecciona una en tiempo de ejecución:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Subrutas exportadas

El paquete exporta:

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## Ejemplos

### Ejemplo de Speex

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### Ejemplo de RNNoise

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

### Ejemplo de Noise Gate

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

## Estructura del proyecto

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

## Desarrollo

### Configuración

```shell
pnpm install
```

### Scripts raíz

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### Scripts de la demo

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

Desde `demo/package.json`, también está disponible `build:all`:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Notas de desarrollo

- El nodo worklet de RNNoise incluye una suposición de 48 kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` y `SpeexWorkletNode` exponen `destroy()` para finalizar recursos del lado wasm.
- La salida de build se genera con `tsdown` e incluye assets wasm copiados en `dist/`.
- Se aplica un parche local a `@shiguredo/rnnoise-wasm` mediante dependencias parcheadas de pnpm.

## Solución de problemas

- `AudioWorklet is not defined`
  - Verifica el soporte del navegador y el contexto seguro.
- `Failed to execute 'addModule'`
  - Asegúrate de que la ruta JS del worklet se resuelva a una URL válida en tiempo de ejecución.
- Fallos al cargar wasm (`fetch`/404/CORS)
  - Confirma que los assets wasm se copian/sirven y que tu estrategia de importación de URL del bundler es correcta.
- Sin efecto audible
  - Revisa el cableado del grafo (`source -> node -> destination`) y si también están habilitadas las restricciones de WebRTC.
- El comportamiento de RNNoise parece inestable
  - Usa o prueba con contexto/tasa de muestreo de audio de 48 kHz.

## Hoja de ruta

- Ampliar la documentación de integración para entornos no Vite.
- Agregar variantes traducidas del README en `i18n/`.
- Documentar con más detalle las características de rendimiento y los tradeoffs entre procesadores.

## Contribuciones

Se aceptan issues y pull requests:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

Antes de abrir un PR, ejecuta:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## Licencia

Licencia MIT. Consulta [LICENSE](./LICENSE).
