[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red/web-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)
[![Stars](https://img.shields.io/github/stars/sapphi-red/web-noise-suppressor?style=flat-square)](https://github.com/sapphi-red/web-noise-suppressor/stargazers)

Nœuds de suppression du bruit pour Web Audio API — conçus pour le nettoyage micro en temps réel côté navigateur.

[![🎧 Démo en direct](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Focus | Ce que couvre ce README |
| --- | --- |
| Exécution | Pipeline Web Audio Worklet pour la suppression du bruit dans le navigateur |
| Sortie principale | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Modèle d’utilisation | Charger WASM → enregistrer le processeur → instancier le nœud → connecter le graphe |

Ce package fournit trois nœuds de suppression du bruit.

| Nœud | Description | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Une implémentation simple de noise gate | Logique native |
| `RnnoiseWorkletNode` | Suppresseur basé sur RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Suppresseur de prétraitement Speex | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Ce package requiert `AudioWorklet` pour fonctionner.

## Aperçu

`@sapphi-red/web-noise-suppressor` est une bibliothèque TypeScript orientée navigateur qui expose des wrappers `AudioWorkletNode` pour la suppression audio en temps réel des entrées. Elle est conçue pour les pipelines de microphone et peut être utilisée en complément des contraintes WebRTC du navigateur (`noiseSuppression`, `echoCancellation`) quand nécessaire.

Le flux principal est :

1. Charger le binaire wasm (`loadSpeex` / `loadRnnoise`).
2. Enregistrer le module processeur du worklet (`audioWorklet.addModule(...)`).
3. Construire le nœud (`new SpeexWorkletNode(...)`, etc.).
4. Connecter le graphe audio.

## Table des matières

- [Aperçu](#aperçu)
- [Fonctionnalités](#fonctionnalités)
- [Installation](#installation)
- [Prérequis](#prérequis)
- [Utilisation](#utilisation)
- [Configuration](#configuration)
- [Structure du projet](#structure-du-projet)
- [Exemples](#exemples)
- [Développement](#développement)
- [Notes de développement](#notes-de-développement)
- [Dépannage](#dépannage)
- [❤️ Support](#-support)
- [Feuille de route](#feuille-de-route)
- [Contribuer](#contribuer)
- [Licence](#licence)

## Fonctionnalités

- Trois options de traitement avec un modèle d’usage Web Audio partagé.
- Exports du point d’entrée de bibliothèque en ESM + CJS.
- Chemins d’export dédiés pour les fichiers JS du worklet et les binaires wasm.
- Détection SIMD de RNNoise supportée via `loadRnnoise`.
- Application de démo (`demo/`) avec routage micro en direct et visualiseur.

## Installation

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## Prérequis

- Un navigateur/environnement d’exécution avec prise en charge de `AudioWorklet`.
- Un contexte sécurisé pour la capture micro (`https://` ou localhost).
- Une stratégie de bundler capable de fournir des références d’URL pour les fichiers JS worklet et wasm.
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

### Options des nœuds

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (from `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (from `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, `openThreshold` par défaut)
  - `holdMs: number`
  - `maxChannels: number`

### Chargement RNNoise

`loadRnnoise` accepte les URL non-SIMD et SIMD et en sélectionne une à l’exécution :

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

- Le nœud worklet RNNoise inclut une hypothèse de 48 kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` et `SpeexWorkletNode` exposent `destroy()` pour terminer les ressources côté wasm.
- La sortie du build est générée par `tsdown` et inclut les actifs wasm copiés dans `dist/`.
- Un patch local est appliqué à `@shiguredo/rnnoise-wasm` via les dépendances `pnpm` patchées.

## Dépannage

- `AudioWorklet is not defined`
  - Vérifiez la prise en charge du navigateur et le contexte sécurisé.
- `Failed to execute 'addModule'`
  - Assurez-vous que le chemin JS du worklet résout vers une URL valide à l’exécution.
| Symptôme | À vérifier |
| --- | --- |
| `AudioWorklet is not defined` | Vérifiez la prise en charge du navigateur et le contexte sécurisé. |
| `Failed to execute 'addModule'` | Assurez-vous que le chemin JS du worklet résout vers une URL valide à l’exécution. |
| Échecs de chargement wasm (`fetch`/404/CORS) | Confirmez que les actifs wasm sont copiés/servis et que votre stratégie d’import URL de bundler est correcte. |
| Aucun effet audible | Vérifiez le câblage du graphe (`source -> node -> destination`) et si les contraintes WebRTC sont aussi activées. |
| Le comportement de RNNoise semble instable | Utilisez ou testez avec un taux d’échantillonnage/context audio à 48 kHz. |

## ❤️ Support

| Donate | PayPal | Stripe |
|---|---|---|
| [![Donate](https://img.shields.io/badge/Donate-LazyingArt-0EA5E9?style=for-the-badge&logo=ko-fi&logoColor=white)](https://chat.lazying.art/donate) | [![PayPal](https://img.shields.io/badge/PayPal-RongzhouChen-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen) | [![Stripe](https://img.shields.io/badge/Stripe-Donate-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## Feuille de route

- Étendre la documentation d’intégration hors Vite.
- Ajouter des variantes de README traduites dans `i18n/`.
- Documenter plus en détail les caractéristiques de performance et les compromis des processeurs.

## Contribuer

Les issues et pull requests sont les bienvenues :

- Issues : <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository : <https://github.com/sapphi-red/web-noise-suppressor>

Avant d’ouvrir une PR, exécutez :

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## Licence

MIT License. Voir [LICENSE](./LICENSE).
