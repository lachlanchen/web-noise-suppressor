[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# @sapphi-red/web-noise-suppressor

**خيارات اللغة:** الإنجليزية (`README.md`) | دليل i18n: `i18n/` (لا توجد ملفات README مترجمة حاليًا)

[![npm version](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor.svg)](https://badge.fury.io/js/@sapphi-red%2Fweb-noise-suppressor)
![CI](https://github.com/sapphi-red/web-noise-suppressor/workflows/CI/badge.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Web Audio](https://img.shields.io/badge/Web%20Audio-Worklet-0A84FF)
![License](https://img.shields.io/badge/License-MIT-22C55E)

عُقد كتم الضوضاء لـ Web Audio API.

[🎧 العرض التجريبي](https://web-noise-suppressor.sapphi.red)

توفر هذه الحزمة ثلاث عُقد لكتم الضوضاء.

| العقدة | الوصف | المحرك الخلفي |
| --- | --- | --- |
| `NoiseGateWorkletNode` | تنفيذ بسيط لبوابة الضوضاء | منطق أصلي |
| `RnnoiseWorkletNode` | كاتم ضوضاء مبني على RNNoise | [xiph/rnnoise](https://github.com/xiph/rnnoise) عبر [shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) |
| `SpeexWorkletNode` | كاتم ضوضاء Speex preprocess | `preprocess` من [xiph/speexdsp](https://github.com/xiph/speexdsp) عبر [sapphi-red/speex-preprocess-wasm](https://github.com/sapphi-red/speex-preprocess-wasm) |

> [!IMPORTANT]
> تتطلب هذه الحزمة دعم `AudioWorklet` حتى تعمل.

## نظرة عامة

`@sapphi-red/web-noise-suppressor` هي مكتبة TypeScript موجهة للمتصفح وتوفر أغلفة `AudioWorkletNode` لتنقية الإدخال في الزمن الحقيقي. صُممت لخطوط أنابيب الميكروفون، ويمكن استخدامها مع قيود WebRTC في المتصفح (`noiseSuppression` و`echoCancellation`) عند الحاجة.

التدفق الأساسي هو:

1. تحميل ملف wasm الثنائي (`loadSpeex` / `loadRnnoise`).
2. تسجيل وحدة معالج worklet (`audioWorklet.addModule(...)`).
3. إنشاء العقدة (`new SpeexWorkletNode(...)`، إلخ).
4. توصيل مخطط الصوت.

## الميزات

- ثلاثة خيارات للمعالجة بنمط استخدام موحد لـ Web Audio.
- تصدير نقطة دخول المكتبة بصيغة ESM + CJS.
- مسارات تصدير مخصصة لملفات worklet JS وملفات wasm الثنائية.
- دعم اكتشاف RNNoise SIMD عبر `loadRnnoise`.
- تطبيق عرض تجريبي (`demo/`) مع توجيه مباشر للميكروفون ومصوّر مرئي.

## التثبيت

```shell
npm i @sapphi-red/web-noise-suppressor # yarn add @sapphi-red/web-noise-suppressor
```

## المتطلبات المسبقة

- متصفح/بيئة تشغيل تدعم `AudioWorklet`.
- سياق آمن لالتقاط الميكروفون (`https://` أو localhost).
- نهج bundler يمكنه توفير مراجع URL لملفات worklet JS وملفات wasm.
  - الأمثلة الموثقة موجهة إلى Vite.

## الاستخدام

هذا القسم مكتوب فقط لمستخدمي vite.

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

لمزيد من التفاصيل، راجع [الشيفرة المصدرية للعرض التجريبي](https://github.com/sapphi-red/web-noise-suppressor/blob/main/demo/src/index.ts).

## الضبط

### خيارات العقدة

- `SpeexWorkletNode` (`SpeexProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (من `loadSpeex`)
- `RnnoiseWorkletNode` (`RnnoiseProcessorOptions`)
  - `maxChannels: number`
  - `wasmBinary: ArrayBuffer` (من `loadRnnoise`)
- `NoiseGateWorkletNode` (`NoiseGateProcessorOptions`)
  - `openThreshold: number` (dB)
  - `closeThreshold?: number` (dB، القيمة الافتراضية `openThreshold`)
  - `holdMs: number`
  - `maxChannels: number`

### تحميل RNNoise

يقبل `loadRnnoise` كلًا من روابط non-SIMD وSIMD، ويختار أحدهما وقت التشغيل:

```ts
const rnnoiseWasmBinary = await loadRnnoise({
  url: rnnoiseWasmPath,
  simdUrl: rnnoiseWasmSimdPath
})
```

### المسارات الفرعية المُصدَّرة

تُصدِّر الحزمة:

- `@sapphi-red/web-noise-suppressor`
- `@sapphi-red/web-noise-suppressor/noiseGateWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoiseWorklet.js`
- `@sapphi-red/web-noise-suppressor/speexWorklet.js`
- `@sapphi-red/web-noise-suppressor/rnnoise.wasm`
- `@sapphi-red/web-noise-suppressor/rnnoise_simd.wasm`
- `@sapphi-red/web-noise-suppressor/speex.wasm`

## أمثلة

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

## هيكل المشروع

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

## التطوير

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

## ملاحظات التطوير

- تتضمن عقدة worklet الخاصة بـ RNNoise افتراض 48kHz (`src/rnnoise/workletNode.ts`).
- يوفّر كل من `RnnoiseWorkletNode` و`SpeexWorkletNode` الدالة `destroy()` لإنهاء الموارد على جانب wasm.
- يتم توليد ناتج البناء بواسطة `tsdown` ويتضمن نسخ أصول wasm إلى `dist/`.
- يتم تطبيق تصحيح محلي على `@shiguredo/rnnoise-wasm` عبر تبعيات pnpm المصححة.

## استكشاف الأخطاء وإصلاحها

- `AudioWorklet is not defined`
  - تحقّق من دعم المتصفح والسياق الآمن.
- `Failed to execute 'addModule'`
  - تأكد من أن مسار worklet JS يُحل إلى URL صالح وقت التشغيل.
- فشل تحميل wasm (`fetch`/404/CORS)
  - تأكد من نسخ/تقديم أصول wasm وأن استراتيجية استيراد URL في bundler صحيحة.
- لا يوجد تأثير مسموع
  - تحقّق من توصيل مخطط الصوت (`source -> node -> destination`) وما إذا كانت قيود WebRTC مفعلة أيضًا.
- يبدو سلوك RNNoise غير مستقر
  - استخدم أو اختبر مع سياق/معدل عينة صوت 48kHz.

## خارطة الطريق

- توسيع توثيق التكامل لغير Vite.
- إضافة نسخ README مترجمة ضمن `i18n/`.
- توثيق خصائص الأداء ومفاضلات المعالجات بمزيد من التفصيل.

## المساهمة

البلاغات وطلبات السحب مرحب بها:

- Issues: <https://github.com/sapphi-red/web-noise-suppressor/issues>
- Repository: <https://github.com/sapphi-red/web-noise-suppressor>

قبل فتح PR، شغّل:

```shell
pnpm run lint
pnpm run test
pnpm run build
pnpm run type-check
```

## الترخيص

رخصة MIT. راجع [LICENSE](./LICENSE).
