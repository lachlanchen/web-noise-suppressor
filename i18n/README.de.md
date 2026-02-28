[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red/web-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)
[![Stars](https://img.shields.io/github/stars/sapphi-red/web-noise-suppressor?style=flat-square)](https://github.com/sapphi-red/web-noise-suppressor/stargazers)

Rauschunterdrückungsknoten für die Web Audio API — entwickelt für Echtzeit-Entstörung von Mikrofoneingängen im Browser.

[![🎧 Demo testen](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Fokus | Inhalt dieses READMEs |
| --- | --- |
| Runtime | Web Audio Worklet-Pipeline für Browser-Rauschunterdrückung |
| Kernausgabe | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Nutzungsmuster | WASM laden → Prozessor registrieren → Node instanziieren → Graph verbinden |

Dieses Paket stellt drei Knoten für Rauschunterdrückung bereit.

| Knoten | Beschreibung | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Eine einfache Noise-Gate-Implementierung | Native Logik |
| `RnnoiseWorkletNode` | RNNoise-basierter Unterdrücker | [xiph/rnnoise](https://github.com/xiph/rnnoise) über [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex Preprocess-Unterdrücker | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` über [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Dieses Paket benötigt `AudioWorklet`, um zu funktionieren.

## Überblick

`@sapphi-red/web-noise-suppressor` ist eine browserorientierte TypeScript-Bibliothek, die `AudioWorkletNode`-Wrappers für die Echtzeitbereinigung von Eingaben bereitstellt. Sie ist für Mikrofon-Pipelines ausgelegt und kann bei Bedarf zusammen mit WebRTC-Constraints des Browsers (`noiseSuppression`, `echoCancellation`) verwendet werden.

Der Kernablauf ist:

1. WASM-Binärdatei laden (`loadSpeex` / `loadRnnoise`).
2. Worklet-Processor-Modul registrieren (`audioWorklet.addModule(...)`).
3. Node erstellen (`new SpeexWorkletNode(...)` usw.).
4. Audiograph verbinden.

## Inhaltsverzeichnis

- [Überblick](#überblick)
- [Funktionen](#funktionen)
- [Installation](#installation)
- [Voraussetzungen](#voraussetzungen)
- [Verwendung](#verwendung)
- [Konfiguration](#konfiguration)
- [Projektstruktur](#projektstruktur)
- [Beispiele](#beispiele)
- [Entwicklung](#entwicklung)
- [Entwicklungsnotizen](#entwicklungsnotizen)
- [Fehlerbehebung](#fehlerbehebung)
- [❤️ Support](#-support)
- [Roadmap](#roadmap)
- [Mitwirken](#mitwirken)
- [Lizenz](#lizenz)

## Funktionen

- Drei Verarbeitungsoptionen mit einem einheitlichen Web-Audio-Nutzungsschema.
- ESM- + CJS-Entrypoints als Export.
- Dedizierte Exportpfade für Worklet-JS-Dateien und WASM-Binärdateien.
- RNNoise-SIMD-Erkennung über `loadRnnoise`.
- Demo-App (`demo/`) mit Live-Mikrofonrouting und Visualisierung.

## Installation

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## Voraussetzungen

- Ein Browser bzw. Runtime mit Unterstützung für `AudioWorklet`.
- Ein sicherer Kontext für die Mikrofonaufnahme (`https://` oder localhost).
- Eine Bundler-Strategie, die URL-Referenzen für Worklet-JS und WASM-Dateien bereitstellen kann.
  - Die dokumentierten Beispiele sind Vite-orientiert.

## Verwendung

Dieser Abschnitt ist nur für Vite-Nutzer*innen geschrieben.

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
  - `wasmBinary: ArrayBuffer` (aus `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (aus `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, standardmäßig `openThreshold`)
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

### Exportierte Unterpfade

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

### Noise Gate-Beispiel

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

### Skripte im Root

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

Aus `demo/package.json` ist zusätzlich `build:all` verfügbar:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Entwicklungsnotizen

- Der RNNoise Worklet Node enthält die Annahme einer 48-kHz-Samplerate (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` und `SpeexWorkletNode` stellen `destroy()` bereit, um Ressourcen auf der WASM-Seite zu terminieren.
- Das Build-Output wird von `tsdown` erzeugt und enthält kopierte WASM-Assets in `dist/`.
- Ein lokaler Patch wird auf `@shiguredo/rnnoise-wasm` über `pnpm`-gepannte Abhängigkeiten angewendet.

## Fehlerbehebung

- `AudioWorklet is not defined`
  - Browser-Unterstützung und sicheren Kontext prüfen.
- `Failed to execute 'addModule'`
  - Sicherstellen, dass der Worklet-JS-Pfad zur Laufzeit auf eine gültige URL auflöst.
| Symptom | Was prüfen |
| --- | --- |
| `AudioWorklet is not defined` | Browser-Unterstützung und sicheren Kontext prüfen. |
| `Failed to execute 'addModule'` | Sicherstellen, dass der Worklet-JS-Pfad zur Laufzeit auf eine gültige URL auflöst. |
| Fehler beim Laden von WASM (`fetch`/404/CORS) | Bestätigen, dass WASM-Assets kopiert/ausgeliefert werden und deine Bundler-URL-Import-Strategie korrekt ist. |
| Kein hörbarer Effekt | Prüfen der Graphverdrahtung (`source -> node -> destination`) und ob WebRTC-Constraints ebenfalls aktiviert sind. |
| RNNoise-Verhalten wirkt instabil | Mit einem AudioContext/einer Abtastrate von 48 kHz testen oder verwenden. |

## ❤️ Support

| Donate | PayPal | Stripe |
|---|---|---|
| [![Donate](https://img.shields.io/badge/Donate-LazyingArt-0EA5E9?style=for-the-badge&logo=ko-fi&logoColor=white)](https://chat.lazying.art/donate) | [![PayPal](https://img.shields.io/badge/PayPal-RongzhouChen-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen) | [![Stripe](https://img.shields.io/badge/Stripe-Donate-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## Roadmap

- Dokumentation für Nicht-Vite-Integrationen erweitern.
- Übersetzte README-Varianten in `i18n/` ergänzen.
- Leistungsdaten und Abwägungen der Verarbeitung genauer dokumentieren.

## Mitwirken

Issues und Pull Requests sind willkommen:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

Bevor du eine PR eröffnest, führe aus:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## Lizenz

MIT-Lizenz. Siehe [LICENSE](./LICENSE).
