[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)

Rauschunterdrückungs-Nodes für die Web Audio API.

[🎧 Demo](https://web-noise-suppressor.sapphi.red)

Dieses Paket stellt drei Nodes zur Rauschunterdrückung bereit.

| Node | Beschreibung | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Eine einfache Noise-Gate-Implementierung | Native Logik |
| `RnnoiseWorkletNode` | RNNoise-basierter Unterdrücker | [xiph/rnnoise](https://github.com/xiph/rnnoise) über [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex-Preprocess-Unterdrücker | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` über [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Dieses Paket benötigt `AudioWorklet`, um zu funktionieren.

## Überblick

`@sapphi-red/web-noise-suppressor` ist eine browserorientierte TypeScript-Bibliothek, die `AudioWorkletNode`-Wrapper für die Echtzeit-Bereinigung von Eingangssignalen bereitstellt. Sie ist für Mikrofon-Pipelines ausgelegt und kann bei Bedarf zusammen mit Browser-WebRTC-Constraints (`noiseSuppression`, `echoCancellation`) verwendet werden.

Der grundlegende Ablauf ist:

1. Wasm-Binärdatei laden (`loadSpeex` / `loadRnnoise`).
2. Worklet-Processor-Modul registrieren (`audioWorklet.addModule(...)`).
3. Node erzeugen (`new SpeexWorkletNode(...)` usw.).
4. Audio-Graph verbinden.

## Funktionen

- Drei Verarbeitungsoptionen mit einem gemeinsamen Web-Audio-Nutzungsmuster.
- Bibliotheks-Entrypoint-Exporte für ESM + CJS.
- Dedizierte Exportpfade für Worklet-JS-Dateien und Wasm-Binärdateien.
- RNNoise-SIMD-Erkennungsunterstützung über `loadRnnoise`.
- Demo-App (`demo/`) mit Live-Mikrofon-Routing und Visualizer.

## Installation

```shell
npm i @sapphi-red/web-noise-suppressor # yarn add @sapphi-red/web-noise-suppressor
```

## Voraussetzungen

- Ein Browser/eine Runtime mit `AudioWorklet`-Unterstützung.
- Ein sicherer Kontext für Mikrofonzugriff (`https://` oder localhost).
- Eine Bundler-Strategie, die URL-Referenzen für Worklet-JS- und Wasm-Dateien bereitstellen kann.
  - Die dokumentierten Beispiele sind auf Vite ausgerichtet.

## Verwendung

Dieser Abschnitt ist nur für Vite-Nutzer geschrieben.

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

Weitere Details findest du im [Demo-Quellcode](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## Konfiguration

### Node-Optionen

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (von `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (von `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, Standardwert ist `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise laden

`loadRnnoise` akzeptiert sowohl Nicht-SIMD- als auch SIMD-URLs und wählt zur Laufzeit eine davon aus:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Exportierte Subpaths

Das Paket exportiert:

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## Beispiele

### Speex-Beispiel

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### RNNoise-Beispiel

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

### Noise-Gate-Beispiel

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

## Projektstruktur

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

## Entwicklung

### Setup

```shell
pnpm install
```

### Root-Skripte

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### Demo-Skripte

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

Aus `demo/package.json` ist auch `build:all` verfügbar:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Hinweise zur Entwicklung

- Der RNNoise-Worklet-Node enthält eine 48-kHz-Annahme (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` und `SpeexWorkletNode` bieten `destroy()`, um Ressourcen auf Wasm-Seite zu beenden.
- Das Build-Output wird von `tsdown` erzeugt und enthält kopierte Wasm-Assets in `dist/`.
- Ein lokaler Patch wird über gepatchte pnpm-Abhängigkeiten auf `@shiguredo/rnnoise-wasm` angewendet.

## Fehlerbehebung

- `AudioWorklet is not defined`
  - Browser-Unterstützung und sicheren Kontext prüfen.
- `Failed to execute 'addModule'`
  - Sicherstellen, dass der Worklet-JS-Pfad zur Laufzeit auf eine gültige URL auflöst.
- Wasm-Ladefehler (`fetch`/404/CORS)
  - Prüfen, ob Wasm-Assets kopiert/ausgeliefert werden und ob die Bundler-URL-Importstrategie korrekt ist.
- Kein hörbarer Effekt
  - Graph-Verdrahtung prüfen (`source -> node -> destination`) und ob zusätzlich WebRTC-Constraints aktiviert sind.
- RNNoise-Verhalten wirkt instabil
  - Einen 48-kHz-Audiokontext/eine entsprechende Sample-Rate verwenden oder damit testen.

## Roadmap

- Dokumentation für Nicht-Vite-Integrationen erweitern.
- Übersetzte README-Varianten unter `i18n/` ergänzen.
- Performance-Eigenschaften und Processor-Trade-offs detaillierter dokumentieren.

## Mitwirken

Issues und Pull Requests sind willkommen:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

Vor dem Öffnen eines PRs ausführen:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## Lizenz

MIT License. Siehe [LICENSE](./LICENSE).
