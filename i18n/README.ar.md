[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# @sapphi-red/web-noise-suppressor

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red/web-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)
[![Stars](https://img.shields.io/github/stars/sapphi-red/web-noise-suppressor?style=flat-square)](https://github.com/sapphi-red/web-noise-suppressor/stargazers)

عقد عمل Web Audio API لإزالة الضوضاء — مخصصة لتنظيف الميكروفون في الزمن الحقيقي داخل المتصفح.

[![🎧 جرب العرض المباشر](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| التركيز | ما الذي يغطيه هذا الملف |
| --- | --- |
| وقت التشغيل | خط أنابيب Web Audio Worklet لإزالة الضوضاء في المتصفح |
| المخرجات الأساسية | `NoiseGateWorkletNode`، `RnnoiseWorkletNode`، `SpeexWorkletNode` |
| نمط الاستخدام | تحميل WASM → تسجيل المعالج → إنشاء العقدة → توصيل الرسم البياني |

يوفر هذا الحزمة ثلاثة عقد لإخماد الضوضاء.

| العقدة | الوصف | الخلفية |
| --- | --- | --- |
| `NoiseGateWorkletNode` | تنفيذ بسيط لبوابة الضوضاء | منطق أصلي |
| `RnnoiseWorkletNode` | كابتض للضوضاء يعتمد على RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) عبر [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | كابتض ما قبل المعالجة من Speex | `preprocess` من [xiph/speexdsp](https://github.com/xiph/speexdsp) عبر [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> هذه الحزمة تتطلب `AudioWorklet` للعمل.

## جدول المحتويات

- [نظرة عامة](#overview)
- [الميزات](#features)
- [التثبيت](#installation)
- [المتطلبات المسبقة](#prerequisites)
- [الاستخدام](#usage)
- [الإعداد](#configuration)
- [هيكل المشروع](#project-structure)
- [الأمثلة](#examples)
- [التطوير](#development)
- [ملاحظات التطوير](#development-notes)
- [استكشاف الأخطاء وإصلاحها](#troubleshooting)
- [❤️ Support](#-support)
- [خارطة الطريق](#roadmap)
- [المساهمة](#contributing)
- [الترخيص](#license)

## <a id="overview"></a>نظرة عامة

`@sapphi-red/web-noise-suppressor` هي مكتبة TypeScript موجهة للمتصفح تعرض طبقات تغليف `AudioWorkletNode` لتنظيف المدخلات في الزمن الحقيقي. وهي مصممة لمسارات الميكروفون ويمكن استخدامها بجانب قيود WebRTC في المتصفح (`noiseSuppression` و`echoCancellation`) عند الحاجة.

التدفق الأساسي هو:

1. تحميل ثنائي wasm (`loadSpeex` / `loadRnnoise`).
2. تسجيل وحدة المعالج (`audioWorklet.addModule(...)`).
3. إنشاء العقدة (`new SpeexWorkletNode(...)` وغيرها).
4. توصيل مخطط الصوت.

## <a id="features"></a>الميزات

- ثلاثة خيارات للمعالجة مع نمط استخدام مشترك لـ Web Audio.
- تصدير نقطة دخول مكتبة بنسختي ESM وCJS.
- مسارات تصدير مخصصة لملفات worklet JS وملفات wasm.
- دعم كشف SIMD الخاص بـ RNNoise عبر `loadRnnoise`.
- تطبيق تجريبي (`demo/`) مع توجيه ميكروفون مباشر ومرئي صوتي.

## <a id="installation"></a>التثبيت

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## <a id="prerequisites"></a>المتطلبات المسبقة

- متصفح/زمن تشغيل بدعم `AudioWorklet`.
- سياق آمن لالتقاط الصوت من الميكروفون (`https://` أو localhost).
- استراتيجية bundler قادرة على تزويد مراجع URL لملفات worklet JS وملفات wasm.
  - الأمثلة الموثقة موجهة إلى Vite.

## <a id="usage"></a>الاستخدام

هذا القسم مكتوب فقط لمستخدمي Vite.

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

للتفاصيل الإضافية راجع [كود العرض التجريبي](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## <a id="configuration"></a>الإعداد

### خيارات العقدة

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (من `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (من `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB, الافتراضي `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### تحميل RNNoise

`loadRnnoise` يقبل عناوين URL غير SIMD وSIMD ويختار أحدهما أثناء التشغيل:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### المسارات الفرعية المصدرة

تصدّر الحزمة:

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## <a id="examples"></a>الأمثلة

### مثال Speex

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### مثال RNNoise

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

### مثال Noise Gate

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

## <a id="project-structure"></a>هيكل المشروع

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

## <a id="development"></a>التطوير

### الإعداد

```shell
pnpm install
```

### سكربتات الجذر

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### سكربتات العرض التجريبي

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo dev
pnpm --filter @sapphi-red/web-noise-suppressor-demo build
pnpm --filter @sapphi-red/web-noise-suppressor-demo preview
```

من `demo/package.json`، يتوفر أيضًا `build:all`:

```shell
pnpm --filter @sapphi-red/web-noise-suppressor-demo run build:all
```

## <a id="development-notes"></a>ملاحظات التطوير

- عقدة worklet الخاصة بـ RNNoise تتضمن افتراضًا تردد 48kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` و`SpeexWorkletNode` يقدمان `destroy()` لإنهاء موارد طرف wasm.
- يتم إنشاء مخرجات البناء عبر `tsdown` وتشمل أصول wasm المنسوخة إلى `dist/`.
- يتم تطبيق تصحيح محلي على `@shiguredo/rnnoise-wasm` عبر اعتماديات pnpm المصححة.

## <a id="troubleshooting"></a>استكشاف الأخطاء وإصلاحها

- `AudioWorklet is not defined`
  - تحقق من دعم المتصفح وسياقه الآمن.
- `Failed to execute 'addModule'`
  - تأكد أن مسار JS الخاص بالـ worklet يحل إلى URL صالح أثناء التشغيل.
| العرض | ما يجب التحقق منه |
| --- | --- |
| `AudioWorklet is not defined` | تحقق من دعم المتصفح وسياقه الآمن. |
| `Failed to execute 'addModule'` | تأكد أن مسار JS الخاص بالـ worklet يحل إلى URL صالح أثناء التشغيل. |
| فشل تحميل wasm (`fetch`/404/CORS) | تأكد أن أصول wasm منسوخة/مُقدّمة بشكل صحيح وأن استراتيجية استيراد URL في bundler مناسبة. |
| لا يوجد تأثير مسموع | افحص توصيل الرسم البياني (`source -> node -> destination`) وما إذا كانت قيود WebRTC مفعلة أيضًا. |
| سلوك RNNoise يبدو غير مستقر | استخدم أو اختبر بسياق ومعدل عينات 48kHz. |

## ❤️ Support

| Donate | PayPal | Stripe |
|---|---|---|
| [![Donate](https://img.shields.io/badge/Donate-LazyingArt-0EA5E9?style=for-the-badge&logo=ko-fi&logoColor=white)](https://chat.lazying.art/donate) | [![PayPal](https://img.shields.io/badge/PayPal-RongzhouChen-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/RongzhouChen) | [![Stripe](https://img.shields.io/badge/Stripe-Donate-635BFF?style=for-the-badge&logo=stripe&logoColor=white)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## <a id="roadmap"></a>خريطة الطريق

- توسيع وثائق التكامل خارج Vite.
- إضافة نسخ مترجمة من ملف README تحت `i18n/`.
- توثيق خصائص الأداء ومفاضلات المعالج بمزيد من التفصيل.

## <a id="contributing"></a>المساهمة

الـ issues و pull requests مرحّبة:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

قبل فتح PR شغّل:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## <a id="license"></a>الترخيص

رخصة MIT. انظر [LICENSE](./LICENSE).
