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

Nodos de supresión de ruido para Web Audio API, diseñados para limpiar el micrófono en tiempo real en el navegador.

[![🎧 Probar demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Acceso rápido | Enlace |
| --- | --- |
| 🎧 Demo en vivo | [web-noise-suppressor.sapphi.red](https://web-noise-suppressor.sapphi.red) |
| 📦 Paquete npm | [@sapphi-red/web-noise-suppressor](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor) |
| 📄 Puntos de entrada | [src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/src/index.ts) |
| 🧪 Fuente del demo | [demo/src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) |

| Enfoque | Qué cubre este README |
| --- | --- |
| Ejecución | Pipeline de Web Audio Worklet para supresión de ruido en navegador |
| Salida principal | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Patrón de uso | Cargar WASM → registrar procesador → instanciar nodo → conectar grafo |

Este paquete proporciona tres nodos de supresión de ruido.

| Nodo | Descripción | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Implementación simple de una puerta de ruido | Lógica nativa |
| `RnnoiseWorkletNode` | Supresor basado en RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) a través de [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Supresor de preprocesamiento Speex | `preprocess` de [xiph/speexdsp](https://github.com/xiph/speexdsp) mediante [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Este paquete requiere `AudioWorklet` para funcionar.

## Visión general

`@sapphi-red/web-noise-suppressor` es una librería de TypeScript centrada en navegador que expone envoltorios de `AudioWorkletNode` para limpieza de entrada en tiempo real. Está diseñada para pipelines de micrófono y puede usarse junto a las restricciones WebRTC del navegador (`noiseSuppression`, `echoCancellation`) cuando sea necesario.

El flujo principal es:

1. Cargar el binario wasm (`loadSpeex` / `loadRnnoise`).
2. Registrar el módulo del procesador del worklet (`audioWorklet.addModule(...)`).
3. Construir el nodo (`new SpeexWorkletNode(...)`, etc.).
4. Conectar el grafo de audio.

## Tabla de contenidos

- [Visión general](#visión-general)
- [Características](#características)
- [Instalación](#instalación)
- [Requisitos previos](#requisitos-previos)
- [Uso](#uso)
- [Configuración](#configuración)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Ejemplos](#ejemplos)
- [Desarrollo](#desarrollo)
- [Notas de desarrollo](#notas-de-desarrollo)
- [Solución de problemas](#solución-de-problemas)
- [❤️ Support](#-support)
- [Hoja de ruta](#hoja-de-ruta)
- [Contribuir](#contribuir)
- [Licencia](#licencia)

## Características

- Tres opciones de procesamiento con un patrón de uso de Web Audio compartido.
- Exportaciones de punto de entrada de librería ESM + CJS.
- Rutas de exportación dedicadas para archivos JS de worklet y binarios wasm.
- Soporte de detección SIMD de RNNoise mediante `loadRnnoise`.
- Aplicación de demostración (`demo/`) con enrutamiento en vivo de micrófono y visualizador.

## Instalación

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## Requisitos previos

- Un navegador/entorno de ejecución con soporte de `AudioWorklet`.
- Un contexto seguro para captura de micrófono (`https://` o localhost).
- Estrategia de bundler capaz de proporcionar referencias URL para archivos JS y wasm del worklet.
  - Los ejemplos documentados están orientados a Vite.

## Uso

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

Para más detalles, consulta el [código fuente del demo](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

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
  - `closeThreshold?: number` (`openThreshold` por defecto)
  - `holdMs: number`
  - `maxChannels: number`

### Carga de RNNoise

`loadRnnoise` acepta URLs no SIMD y SIMD y selecciona una en tiempo de ejecución:

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

### Ejemplo Speex

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### Ejemplo RNNoise

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

### Ejemplo Noise Gate

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
│   ├── noiseGate/          # lógica de puerta + nodo/procesador del worklet
│   ├── rnnoise/            # integración de rnnoise + nodo/procesador del worklet
│   ├── speex/              # integración de preprocess de Speex + nodo/procesador del worklet
│   ├── utils/              # utilidades compartidas
│   └── index.ts            # exportaciones públicas de la API
├── demo/                   # aplicación demo de Vite
├── patches/                # parche de pnpm para rnnoise-wasm
├── i18n/                   # artefactos de traducción
├── tsdown.config.ts        # configuración de build
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

### Scripts de demo

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

- El nodo worklet de RNNoise incluye una suposición de 48kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` y `SpeexWorkletNode` exponen `destroy()` para liberar recursos del lado wasm.
- La salida de compilación es generada por `tsdown` e incluye activos wasm copiados en `dist/`.
- Se aplica un parche local a `@shiguredo/rnnoise-wasm` mediante dependencias parcheadas de pnpm.

## Solución de problemas

| Síntoma | Qué verificar |
| --- | --- |
| `AudioWorklet is not defined` | Verifica el soporte del navegador y el contexto seguro. |
| `Failed to execute 'addModule'` | Asegúrate de que la ruta JS del worklet resuelva una URL válida en tiempo de ejecución. |
| Fallos al cargar wasm (`fetch`/404/CORS) | Confirma que los activos wasm se copien/serven y que tu estrategia de URL import del bundler sea correcta. |
| Sin efecto audible | Revisa el cableado del grafo (`source -> node -> destination`) y si las restricciones de WebRTC también están activadas. |
| El comportamiento de RNNoise parece inestable | Usa o prueba con un contexto de audio/tasa de muestreo de 48kHz. |

## Roadmap

- Ampliar la documentación de integración fuera de Vite.
- Añadir variantes traducidas de README en `i18n/`.
- Documentar con más detalle las características de rendimiento y los compromisos de los procesadores.

## Contribuir

Se aceptan issues y pull requests:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repositorio: <https://github.com/sapphi-red/web-noise-suppressor>

Antes de abrir un PR, ejecuta:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## Licencia

MIT License. Ver [LICENSE](./LICENSE).


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
