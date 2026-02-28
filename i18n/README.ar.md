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

عقدات كتم الضوضاء لواجهة Web Audio API — صممت لتنظيف الميكروفون في المتصفح في الزمن الحقيقي.

[![🎧 Try Demo](https://img.shields.io/badge/🎧-Live_Demo-0EA5E9?style=for-the-badge)](https://web-noise-suppressor.sapphi.red)
[![📦 npm](https://img.shields.io/badge/📦-npm_Install-18181B?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor)

| وصول سريع | الرابط |
| --- | --- |
| 🎧 تجربة مباشرة | [web-noise-suppressor.sapphi.red](https://web-noise-suppressor.sapphi.red) |
| 📦 حزمة npm | [@sapphi-red/web-noise-suppressor](https://www.npmjs.com/package/@sapphi-red/web-noise-suppressor) |
| 📄 نقاط دخول المصدر | [src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/src/index.ts) |
| 🧪 مصدر العرض التوضيحي | [demo/src/index.ts](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts) |

| مجال التركيز | ما الذي يغطيه هذا الدليل |
| --- | --- |
| وقت التشغيل | خط أنابيب Web Audio Worklet لكتم الضوضاء في المتصفح |
| الناتج الأساسي | `NoiseGateWorkletNode` و `RnnoiseWorkletNode` و `SpeexWorkletNode` |
| نمط الاستخدام | تحميل WASM → تسجيل المعالج → إنشاء العقدة → ربط الرسم البياني |

توفر هذه الحزمة ثلاث عقد لكتم الضوضاء.

| العقدة | الوصف | الخلفية |
| --- | --- | --- |
| `NoiseGateWorkletNode` | تنفيذ بسيط لباب الضوضاء | منطق أصلي |
| `RnnoiseWorkletNode` | كابح ضوضاء مبني على RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) عبر [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | كابح المعالجة المسبقة لـ Speex | `preprocess` من [xiph/speexdsp](https://github.com/xiph/speexdsp) عبر [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> هذه الحزمة تحتاج وجود `AudioWorklet` لتعمل.

## <a id="overview"></a>نظرة عامة

`@sapphi-red/web-noise-suppressor` هي مكتبة TypeScript موجهة للمتصفح تعرض غلافات `AudioWorkletNode` لتنظيف الصوت الوارد في الزمن الفعلي. وهي مصممة لمسارات الميكروفون وتستطيع العمل مع قيود WebRTC الخاصة بالمتصفح (`noiseSuppression`، `echoCancellation`) عند الحاجة.

التدفق الأساسي هو:

1. تحميل ملف wasm (`loadSpeex` / `loadRnnoise`).
2. تسجيل وحدة معالجة الـ worklet (`audioWorklet.addModule(...)`).
3. إنشاء العقدة (`new SpeexWorkletNode(...)` وغيرها).
4. توصيل الرسم البياني الصوتي.

## <a id="table-of-contents"></a>جدول المحتويات

- [نظرة عامة](#overview)
- [الميزات](#features)
- [التثبيت](#installation)
- [المتطلبات المسبقة](#prerequisites)
- [الاستخدام](#usage)
- [الإعدادات](#configuration)
- [بنية المشروع](#project-structure)
- [الأمثلة](#examples)
- [التطوير](#development)
- [ملاحظات التطوير](#development-notes)
- [استكشاف الأخطاء وإصلاحها](#troubleshooting)
- [خطة التطوير](#roadmap)
- [المساهمة](#contributing)
- [❤️ الدعم](#-support)
- [الترخيص](#license)

## <a id="features"></a>الميزات

- ثلاثة خيارات للمعالجة بنمط استخدام واحد في Web Audio.
- نقطة دخول مكتبة تدعم ESM و CJS.
- مسارات تصدير مخصصة لملفات worklet JS وملفات wasm.
- دعم كشف دعم SIMD في RNNoise عبر `loadRnnoise`.
- تطبيق توضيحي (`demo/`) مع توجيه مباشر للميكروفون ومُصوّر صوتي.

## <a id="installation"></a>التثبيت

```shell
npm i @sapphi-red/web-noise-suppressor
# yarn add @sapphi-red/web-noise-suppressor
# pnpm add @sapphi-red/web-noise-suppressor
```

## <a id="prerequisites"></a>المتطلبات المسبقة

- متصفح أو بيئة تشغيل تدعم `AudioWorklet`.
- سياق آمن لالتقاط الميكروفون (`https://` أو localhost).
- استراتيجية bundler قادرة على توفير مراجع URL لملفات worklet JS و wasm.
  - الأمثلة الموثقة موجهة نحو Vite.

## <a id="usage"></a>الاستخدام

هذا القسم مكتوب لمستخدمي vite فقط.

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

لمزيد من التفاصيل، راجع [كود مصدر العرض التوضيحي](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## <a id="configuration"></a>الإعدادات

### <a id="node-options"></a>خيارات العقدة

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (من `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (من `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (ديسيبل)
  - `closeThreshold?: number` (ديسيبل، القيمة الافتراضية `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### <a id="rnnoise-loading"></a>تحميل RNNoise

`loadRnnoise` يقبل روابط لاستخدام SIMD وبدون SIMD ويختار المناسبة أثناء التشغيل:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### <a id="exported-subpaths"></a>المسارات الفرعية المصدرة

الحزمة تصدر:

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## <a id="examples"></a>الأمثلة

### <a id="speex-example"></a>مثال Speex

```ts
import { SpeexWorkletNode, loadSpeex } from '@sapphi-red/web-noise-suppressor'
import speexWorkletPath from '@sapphi-red/web-noise-suppressor/speexWorklet.js?url'
import speexWasmPath from '@sapphi-red/web-noise-suppressor/speex.wasm?url'

const ctx = new AudioContext()
const wasmBinary = await loadSpeex({ url: speexWasmPath })
await ctx.audioWorklet.addModule(speexWorkletPath)

const node = new SpeexWorkletNode(ctx, { wasmBinary, maxChannels: 2 })
```

### <a id="rnnoise-example"></a>مثال RNNoise

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

### <a id="noise-gate-example"></a>مثال Noise Gate

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

## <a id="project-structure"></a>بنية المشروع

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

### <a id="setup"></a>الإعداد

```shell
pnpm install
```

### <a id="root-scripts"></a>سكربتات الجذر

```shell
pnpm run build
pnpm run lint
pnpm run format
pnpm run type-check
pnpm run test
```

### <a id="demo-scripts"></a>سكربتات الـ demo

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

- node worklet الخاص بـ RNNoise يتضمن افتراضًا 48kHz (`src/rnnoise/workletNode.ts`).
- `RnnoiseWorkletNode` و`SpeexWorkletNode` تعرضان `destroy()` لإنهاء موارد wasm.
- ناتج البناء يتم إنشاؤه بواسطة `tsdown` ويضم أصول wasm المنسوخة داخل `dist/`.
- يتم تطبيق patch محلي إلى `@shiguredo/rnnoise-wasm` عبر تبعيات pnpm المعدّلة.

## <a id="troubleshooting"></a>استكشاف الأخطاء وإصلاحها

| عرض العَرَض | ما يجب التحقق منه |
| --- | --- |
| `AudioWorklet is not defined` | تحقق من دعم المتصفح وسياق التشغيل الآمن. |
| `Failed to execute 'addModule'` | تأكد من أن مسار ملف worklet يحل إلى رابط URL صالح أثناء التشغيل. |
| فشل تحميل wasm (`fetch`/404/CORS) | تأكد من نسخ/تقديم أصول wasm وأن استراتيجية استيراد روابط الـ bundler صحيحة. |
| لا يوجد تأثير مسموع | تحقق من توصيل الرسم البياني (`source -> node -> destination`) ومن تمكين قيود WebRTC أيضًا. |
| سلوك RNNoise يبدو غير مستقر | جرّب العمل على سياق صوتي/معدل أخذ عينات 48kHz. |

## <a id="roadmap"></a>خطة الطريق

- توسيع توثيق الدمج غير المعتمد على Vite.
- إضافة نسخ README مترجمة تحت `i18n/`.
- توثيق خصائص الأداء وتبادلات أداء المعالِجات بشكل أعمق.

## <a id="contributing"></a>المساهمة

الـ issues وطلبات السحب مرحب بها:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

قبل فتح PR، شغّل:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## <a id="license"></a>الترخيص

رخصة MIT. انظر [LICENSE](./LICENSE).


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
