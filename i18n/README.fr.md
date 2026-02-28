[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)

Nœuds de suppression du bruit pour l’API Web Audio.

[🎧 Démo](https://web-noise-suppressor.sapphi.red)

Ce package fournit trois nœuds de suppression du bruit.

| Nœud | Description | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Une implémentation simple de noise gate | Logique native |
| `RnnoiseWorkletNode` | Suppresseur basé sur RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Suppresseur de prétraitement Speex | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Ce package nécessite `AudioWorklet` pour fonctionner.

## Vue d’ensemble

`@sapphi-red/web-noise-suppressor` est une bibliothèque TypeScript orientée navigateur qui expose des wrappers `AudioWorkletNode` pour le nettoyage en temps réel des entrées audio. Elle est conçue pour les pipelines microphone et peut être utilisée en complément des contraintes WebRTC du navigateur (`noiseSuppression`, `echoCancellation`) si nécessaire.

Le flux principal est :

1. Charger le binaire wasm (`loadSpeex` / `loadRnnoise`).
2. Enregistrer le module du processeur worklet (`audioWorklet.addModule(...)`).
3. Construire le nœud (`new SpeexWorkletNode(...)`, etc.).
4. Connecter le graphe audio.

## Fonctionnalités

- Trois options de traitement avec un modèle d’usage Web Audio commun.
- Exports d’entrypoint de bibliothèque en ESM + CJS.
- Chemins d’export dédiés pour les fichiers JS worklet et les binaires wasm.
- Prise en charge de la détection SIMD RNNoise via `loadRnnoise`.
- Application de démo (`demo/`) avec routage micro en direct et visualiseur.

## Installation

```shell
npm i @sapphi-red/web-noise-suppressor # yarn add @sapphi-red/web-noise-suppressor
```

## Prérequis

- Un navigateur/runtime avec prise en charge de `AudioWorklet`.
- Un contexte sécurisé pour la capture microphone (`https://` ou localhost).
- Une stratégie bundler capable de fournir des références URL pour les fichiers JS worklet et wasm.
  - Les exemples documentés sont orientés Vite.

## Utilisation

Cette section est rédigée uniquement pour les utilisateurs de Vite.

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

Pour plus de détails, voir le [code source de la démo](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## Configuration

### Options de nœud

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (from `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (from `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, defaults to `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### Chargement de RNNoise

`loadRnnoise` accepte les URL non-SIMD et SIMD et sélectionne la bonne à l’exécution :

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### Sous-chemins exportés

Le package exporte :

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## Exemples

### Exemple Speex

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### Exemple RNNoise

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

### Exemple Noise Gate

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

## Structure du projet

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

## Développement

### Configuration

```shell
pnpm install
```

### Scripts racine

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### Scripts de démo

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

Depuis `demo/package.json`, `build:all` est également disponible :

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## Notes de développement

- Le nœud worklet RNNoise inclut une hypothèse 48kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` et `SpeexWorkletNode` exposent `destroy()` pour terminer les ressources côté wasm.
- La sortie de build est générée par `tsdown` et inclut les assets wasm copiés dans `dist/`.
- Un patch local est appliqué à `@shiguredo/rnnoise-wasm` via les dépendances pnpm patchées.

## Dépannage

- `AudioWorklet is not defined`
  - Vérifiez la prise en charge navigateur et le contexte sécurisé.
- `Failed to execute 'addModule'`
  - Assurez-vous que le chemin JS du worklet est résolu en URL valide à l’exécution.
- Échecs de chargement wasm (`fetch`/404/CORS)
  - Vérifiez que les assets wasm sont copiés/servis et que votre stratégie d’import URL bundler est correcte.
- Aucun effet audible
  - Vérifiez le câblage du graphe (`source -> node -> destination`) et si les contraintes WebRTC sont aussi activées.
- Le comportement RNNoise semble instable
  - Utilisez ou testez avec un contexte/taux d’échantillonnage audio à 48kHz.

## Feuille de route

- Étendre la documentation d’intégration hors Vite.
- Ajouter des variantes de README traduites sous `i18n/`.
- Documenter plus en détail les caractéristiques de performance et les compromis entre processeurs.

## Contribution

Les issues et pull requests sont les bienvenues :

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

Avant d’ouvrir une PR, exécutez :

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## Licence

Licence MIT. Voir [LICENSE](./LICENSE).
