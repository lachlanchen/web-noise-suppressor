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

Nœuds de suppression du bruit pour la Web Audio API — conçus pour le nettoyage en temps réel du microphone côté navigateur.

[![🎧 Démo en direct](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| Accès rapide | Lien |
| --- | --- |
| 🎧 Démo en direct | [web-noise-suppressor.sapphi.red](https://web-noise-suppressor.sapphi.red) |
| 📦 package npm | [@sapphi-red/web-noise-suppressor](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor) |
| 📄 Points d’entrée source | [src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/src/index.ts) |
| 🧪 Source de la démo | [demo/src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) |

| Focus | Contenu de ce README |
| --- | --- |
| Exécution | Pipeline Web Audio Worklet pour la suppression de bruit dans le navigateur |
| Sortie principale | `NoiseGateWorkletNode`, `RnnoiseWorkletNode`, `SpeexWorkletNode` |
| Modèle d’utilisation | Charger WASM → enregistrer le processeur → instancier le nœud → connecter le graphe |

Ce package fournit trois nœuds de suppression du bruit.

| Nœud | Description | Backend |
| --- | --- | --- |
| `NoiseGateWorkletNode` | Implémentation simple d’une noise gate | Logique native |
| `RnnoiseWorkletNode` | Suppresseur basé sur RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) via [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | Suppresseur de prétraitement Speex | [xiph/speexdsp](https://github.com/xiph/speexdsp) `preprocess` via [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> Ce package nécessite `AudioWorklet` pour fonctionner.

## Aperçu

`@sapphi-red/web-noise-suppressor` est une bibliothèque TypeScript orientée navigateur qui expose des wrappers `AudioWorkletNode` pour nettoyer les entrées en temps réel. Elle est conçue pour les pipelines de microphone et peut être utilisée avec les contraintes WebRTC du navigateur (`noiseSuppression`, `echoCancellation`) si nécessaire.

Le flux principal est :

1. Charger le binaire wasm (`loadSpeex` / `loadRnnoise`).
2. Enregistrer le module du processeur de worklet (`audioWorklet.addModule(...)`).
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
- [Feuille de route](#feuille-de-route)
- [Contribuer](#contribuer)
- [❤️ Support](#-support)
- [Licence](#licence)

## Fonctionnalités

- Trois options de traitement avec un modèle d’utilisation Web Audio commun.
- Exports de point d’entrée de bibliothèque ESM + CJS.
- Chemins d’export dédiés pour les fichiers JS de worklet et les binaires wasm.
- Détection SIMD RNNoise via `loadRnnoise`.
- Application de démonstration (`demo/`) avec routage microphone en direct et visualiseur.

## Installation

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## Prérequis

- Un navigateur/environnement d’exécution avec prise en charge de `AudioWorklet`.
- Un contexte sécurisé pour la capture du microphone (`https://` ou localhost).
- Une stratégie de bundler capable de fournir des références d’URL pour les fichiers JS du worklet et du wasm.
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
  - `wasmBinary: ArrayBuffer` (via `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (via `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (`openThreshold` par défaut)
  - `holdMs: number`
  - `maxChannels: number`

### Chargement RNNoise

`loadRnnoise` accepte des URL non-SIMD et SIMD et en sélectionne une à l’exécution :

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

### Installation

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
- `RnnoiseWorkletNode` et `SpeexWorkletNode` exposent `destroy()` pour libérer les ressources côté wasm.
- Le build est généré par `tsdown` et inclut les actifs wasm copiés dans `dist/`.
- Un patch local est appliqué à `@shiguredo/rnnoise-wasm` via les dépendances patchées de `pnpm`.

## Dépannage

| Symptôme | À vérifier |
| --- | --- |
| `AudioWorklet is not defined` | Vérifiez la prise en charge du navigateur et le contexte sécurisé. |
| `Failed to execute 'addModule'` | Vérifiez que le chemin JS du worklet résout vers une URL valide à l’exécution. |
| Échecs de chargement wasm (`fetch`/404/CORS) | Vérifiez que les actifs wasm sont copiés/servis et que votre stratégie d’import d’URL du bundler est correcte. |
| Aucun effet audible | Vérifiez le câblage du graphe (`source -> node -> destination`) et si les contraintes WebRTC sont aussi activées. |
| Le comportement de RNNoise semble instable | Utilisez ou testez avec un contexte audio/un taux d’échantillonnage de 48 kHz. |

## Feuille de route

- Étendre la documentation d’intégration hors Vite.
- Ajouter des variantes de README traduites dans `i18n/`.
- Décrire plus en détail les caractéristiques de performance et les compromis des processeurs.

## Contribuer

Issues et pull requests sont les bienvenus :

- Issues : <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Dépôt : <https://github.com/sapphi-red/web-noise-suppressor>

Avant d’ouvrir une PR, exécutez :

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## Licence

MIT License. Voir [LICENSE](./LICENSE).


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
