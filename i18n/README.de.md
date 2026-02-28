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

Noise-Suppressor-Nodes für die Web Audio API — für die Echtzeit-Bereinigung von Mikrofon-Eingaben im Browser.

[![🎧 Try Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Schneller Zugriff | Link |
| --- | --- |
| 🎧 Live-Demo | [web-noise-suppressor.sapphi.red](https://web-noise-suppressor.sapphi.red) |
| 📦 npm-Paket | [@sapphi-red/web-noise-suppressor](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor) |
| 📄 Einstiegspunkte | [src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/src/index.ts) |
| 🧪 Demo-Quellcode | [demo/src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) |

| Fokus | Was dieses README abdeckt |
| --- | --- |
| Laufzeit | Web Audio Worklet-Pipeline für Browser-Rauschunterdrückung |
| Kernausgabe | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Nutzungsablauf | WASM laden → Prozessor registrieren → Node instanziieren → Graph verbinden |

Dieses Paket stellt drei Noise-Suppressor-Nodes bereit.

| Node | Beschreibung | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Eine einfache Noise-Gate-Implementierung | Native Logik |
| `RnnoiseWorkletNode` | RNNoise-basierte Rauschunterdrückung | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Speex-Preprocess-Unterdrücker | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Dieses Paket benötigt `AudioWorklet`.

## Überblick

`@sapphi-red/web-noise-suppressor` ist eine browserfokussierte TypeScript-Bibliothek, die `AudioWorkletNode`-Wrapper für die Echtzeitbereinigung von Eingaben bereitstellt. Sie ist für Mikrofon-Pipelines ausgelegt und kann bei Bedarf zusammen mit den WebRTC-Constraints (`noiseSuppression`, `echoCancellation`) im Browser verwendet werden.

Der grundlegende Ablauf ist:

1. WASM-Binärdatei laden (`loadSpeex` / `loadRnnoise`).
2. Worklet-Prozessor-Modul registrieren (`audioWorklet.addModule(...)`).
3. Node erstellen (`new SpeexWorkletNode(...)` usw.).
4. Audio-Graph verbinden.

## Inhaltsverzeichnis

- [Überblick](#überblick)
- [Funktionen](#funktionen)
- [Installation](#installation)
- [Voraussetzungen](#voraussetzungen)
- [Nutzung](#nutzung)
- [Konfiguration](#konfiguration)
- [Projektstruktur](#projektstruktur)
- [Beispiele](#beispiele)
- [Entwicklung](#entwicklung)
- [Entwicklungsnotizen](#entwicklungsnotizen)
- [Fehlerbehebung](#fehlerbehebung)
- [Roadmap](#roadmap)
- [Mitwirken](#mitwirken)
- [❤️ Support](#-support)
- [Lizenz](#lizenz)

## Funktionen

- Drei Verarbeitungsoptionen mit einem gemeinsamen Web-Audio-Nutzungsmuster.
- Exporte des Bibliotheks-Einstiegspunkts für ESM + CJS.
- Eigene Exportpfade für Worklet-JS-Dateien und WASM-Binärdateien.
- RNNoise-SIMD-Erkennung über `loadRnnoise`.
- Demo-App (`demo/`) mit Live-Mikrofon-Routing und Visualisierung.

## Installation

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## Voraussetzungen

- Ein Browser/Runtime mit Unterstützung für `AudioWorklet`.
- Ein sicherer Kontext für die Mikrofon-Aufnahme (`https://` oder localhost).
- Eine Bundler-Strategie, die URL-Referenzen für Worklet-JS- und WASM-Dateien bereitstellt.
  - Die dokumentierten Beispiele sind auf Vite ausgelegt.

## Nutzung

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
  - `wasmBinary: ArrayBuffer` (aus `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (aus `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, standardmäßig `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### RNNoise-Laden

`loadRnnoise` akzeptiert sowohl non-SIMD- als auch SIMD-URLs und wählt zur Laufzeit eine davon aus:

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
│   ├── noiseGate/          # Gate-Logik + Worklet Node/Processor
│   ├── rnnoise/            # RNNoise-Integration + Worklet Node/Processor
│   ├── speex/              # Speex-Preprocess-Integration + Worklet Node/Processor
│   ├── utils/              # Gemeinsame Hilfsfunktionen
│   └── index.ts            # Öffentliche API-Exporte
├── demo/                   # Vite Demo-Anwendung
├── patches/                # pnpm-Patch für rnnoise-wasm
├── i18n/                   # Übersetzungsdateien
├── tsdown.config.ts        # Build-Konfiguration
└── package.json
```

## Entwicklung

### Einrichtung

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

Aus `demo/package.json` ist außerdem `build:all` verfügbar:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Entwicklungsnotizen

- Der RNNoise-Worklet-Node enthält eine 48-kHz-Annahme (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` und `SpeexWorkletNode` stellen `destroy()` bereit, um WASM-Ressourcen auf der nativen Seite freizugeben.
- Build-Output wird über `tsdown` erzeugt und kopiert WASM-Assets nach `dist/`.
- Ein lokaler Patch wird auf `@shiguredo/rnnoise-wasm` über pnpm-patch dependencies angewendet.

## Fehlerbehebung

| Symptom | Was zu prüfen ist |
| --- | --- |
| `AudioWorklet is not defined` | Browser-Unterstützung und sicheren Kontext prüfen. |
| `Failed to execute 'addModule'` | Sicherstellen, dass der Worklet-JS-Pfad zur Laufzeit eine gültige URL auflöst. |
| Fehler beim Laden von WASM (`fetch`/404/CORS) | Prüfen, dass WASM-Dateien kopiert/ausgeliefert werden und die Bundle-Strategie für URL-Importe korrekt ist. |
| Keine hörbare Wirkung | Prüfen der Graphverkabelung (`source -> node -> destination`) und ob WebRTC-Constraints ebenfalls aktiv sind. |
| RNNoise-Verhalten wirkt instabil | Mit einem 48-kHz-AudioContext/Sample-Rate testen. |

## Roadmap

- Dokumentation für Nicht-Vite-Integrationen erweitern.
- README-Übersetzungen unter `i18n/` ergänzen.
- Leistungskennzahlen und Verarbeitungs-Tradeoffs detaillierter dokumentieren.

## Mitwirken

Issues und Pull Requests sind willkommen:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

Bevor du einen PR öffnest, führe aus:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## Lizenz

MIT-Lizenz. Siehe [LICENSE](./LICENSE).


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
