## 中文语音
kokoro支持中文语音 support Chinese voice

## 用法
首先安装 `kokoro-js-zh` 包 [NPM](https://npmjs.com/package/kokoro-js-zh):

```bash
npm i kokoro-js-zh
```
### 单独使用示例：

```html
<script src="dist/kokoro.umd.js"></script>
<script>
  (async () => {
    const { KokoroTTS } = window.Kokoro;
    const tts = await KokoroTTS.from_pretrained(
      'onnx-community/Kokoro-82M-v1.0-ONNX-timestamped',
      { dtype: 'q8', device: 'wasm' }
    );
    console.log(tts.list_voices());
    const text = "¡Hola, mundo!";
    const audioBuffer = await tts.generate(text, { voice: "em_santa" });
  })();
</script>
```
### 使用包示例

```js
import { KokoroTTS } from "kokoro-js-zh";

const model_id = "onnx-community/Kokoro-82M-v1.0-ONNX";
const tts = await KokoroTTS.from_pretrained(model_id, {
  dtype: "q8", // Options: "fp32", "fp16", "q8", "q4", "q4f16"
  device: "wasm", // Options: "wasm", "webgpu" (web) or "cpu" (node). If using "webgpu", we recommend using dtype="fp32".
});

const text = "Life is like a box of chocolates. You never know what you're gonna get.";
const audio = await tts.generate(text, {
  // Use `tts.list_voices()` to list all available voices
  voice: "zf_xiaobei",
});
audio.save("audio.wav");
```

Or if you'd prefer to stream the output, you can do that with:

```js
import { KokoroTTS, TextSplitterStream } from "kokoro-js";

const model_id = "onnx-community/Kokoro-82M-v1.0-ONNX";
const tts = await KokoroTTS.from_pretrained(model_id, {
  dtype: "fp32", // Options: "fp32", "fp16", "q8", "q4", "q4f16"
  // device: "webgpu", // Options: "wasm", "webgpu" (web) or "cpu" (node).
});

// First, set up the stream
const splitter = new TextSplitterStream();
const stream = tts.stream(splitter, {
  voice: "zf_xiaobei",
});
(async () => {
  let i = 0;
  for await (const { text, phonemes, audio } of stream) {
    console.log({ text, phonemes });
    audio.save(`audio-${i++}.wav`);
  }
})();

// Next, add text to the stream. Note that the text can be added at different times.
// For this example, let's pretend we're consuming text from an LLM, one word at a time.
const text = "你好，我是中文语音!";
const tokens = text.match(/\s*\S+/g);
for (const token of tokens) {
  splitter.push(token);
  await new Promise((resolve) => setTimeout(resolve, 10));
}

// Finally, close the stream to signal that no more text will be added.
splitter.close();

// Alternatively, if you'd like to keep the stream open, but flush any remaining text, you can use the `flush` method.
// splitter.flush();
```

### Mandarin Chinese 支持的中文语音
这里都是四川话的中文语音

| Name | Traits | Target Quality | Training Duration | Overall Grade | SHA256 |
| ---- | ------ | -------------- | ----------------- | ------------- | ------ |
| zf_xiaobei | 🚺 | C | MM minutes | D | `9b76be63` |
| zf_xiaoni | 🚺 | C | MM minutes | D | `95b49f16` |
| zf_xiaoxiao | 🚺 | C | MM minutes | D | `cfaf6f2d` |
| zf_xiaoyi | 🚺 | C | MM minutes | D | `b5235dba` |
| zm_yunjian | 🚹 | C | MM minutes | D | `76cbf8ba` |
| zm_yunxi | 🚹 | C | MM minutes | D | `dbe6e1ce` |
| zm_yunxia | 🚹 | C | MM minutes | D | `bb2b03b0` |
| zm_yunyang | 🚹 | C | MM minutes | D | `5238ac22` |

Folders:
- `kokoro.js`: Reference copy of [kokoro.js](https://github.com/hexgrad/kokoro/tree/main/kokoro.js) with the changes mentioned in [https://github.com/hexgrad/kokoro/issues/65#issuecomment-2848869611](https://github.com/hexgrad/kokoro/issues/65#issuecomment-2848869611).
- `dist`: Ready-to-use packaged version of kokoro.js that includes Spanish voices.

### 常见错误
1. 如果用vite 本地dev报错，需要把espeak-ng.wasm 复制到 node_modules/.vite/deps 目录下
2. 语音可以替换，我还在尝试不同的中文语音, 目前中文口音的问题需要重新编译WASM

#### Build WASM

```sh
# Docker (optional)
docker run -it -v $(pwd):/wasm -w /wasm debian:12.5
apt-get update
apt-get install --yes --no-install-recommends build-essential cmake ca-certificates curl pkg-config git python3 autogen automake autoconf libtool

# Emscripten
git clone --depth 1 https://github.com/emscripten-core/emsdk.git /wasm/modules/emsdk
cd /wasm/modules/emsdk
./emsdk install 4.0.14
./emsdk activate 4.0.14
source ./emsdk_env.sh
TOOLCHAIN_FILE=$EMSDK/upstream/emscripten/cmake/Modules/Platform/Emscripten.cmake
sed -i -E 's/int\s+(iswalnum|iswalpha|iswblank|iswcntrl|iswgraph|iswlower|iswprint|iswpunct|iswspace|iswupper|iswxdigit)\(wint_t\)/\/\/\0/g' ./upstream/emscripten/cache/sysroot/include/wchar.h

# espeak-ng
# https://github.com/ianmarmour/espeak-ng.js
git clone --depth 1 https://github.com/espeak-ng/espeak-ng.git /wasm/modules/espeak-ng
cd /wasm/modules/espeak-ng
./autogen.sh
./configure --without-async --without-mbrola --without-sonic --without-pcaudiolib --without-klatt --without-speechplayer --with-extdict-cmn
make

cd /wasm/modules/espeak-ng/src/ucd-tools
mv CHANGELOG.md CHANGELOG.tmp
mv CHANGELOG.tmp ChangeLog.md
./autogen.sh
./configure
make clean
emconfigure ./configure
emmake make clean
emmake make

cd /wasm/modules/espeak-ng
emconfigure ./configure --without-async --without-mbrola --without-sonic --without-pcaudiolib --without-klatt --without-speechplayer --with-extdict-cmn
emmake make clean
emmake make src/espeak-ng
emcc -O3 -s INVOKE_RUN=0 -s EXIT_RUNTIME=0 -s MODULARIZE=1 -s EXPORT_NAME='createESpeakNg' -s EXPORTED_FUNCTIONS='[_free, _malloc, _espeak_Initialize, _espeak_TextToPhonemes, _espeak_SetVoiceByName, _espeak_ListVoices]' -s EXPORTED_RUNTIME_METHODS='[lengthBytesUTF8, stringToUTF8, UTF8ToString, setValue, getValue, FS]' -s INITIAL_MEMORY=64MB -s STACK_SIZE=5MB -s DEFAULT_PTHREAD_STACK_SIZE=2MB src/.libs/libespeak-ng.so src/espeak-ng.o -o /wasm/espeakng.js --embed-file espeak-ng-data@/usr/local/share/espeak-ng-data/
```

Produces `espeakng.js` and `espeakng.wasm` in root.

## 原项目 Kokoro TTS 

<p align="center">
    <a href="https://www.npmjs.com/package/kokoro-js"><img alt="NPM" src="https://img.shields.io/npm/v/kokoro-js"></a>
    <a href="https://www.npmjs.com/package/kokoro-js"><img alt="NPM Downloads" src="https://img.shields.io/npm/dw/kokoro-js"></a>
    <a href="https://www.jsdelivr.com/package/npm/kokoro-js"><img alt="jsDelivr Hits" src="https://img.shields.io/jsdelivr/npm/hw/kokoro-js"></a>
    <a href="https://github.com/hexgrad/kokoro/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/github/license/hexgrad/kokoro?color=blue"></a>
    <a href="https://huggingface.co/spaces/webml-community/kokoro-webgpu"><img alt="Demo" src="https://img.shields.io/badge/Hugging_Face-demo-green"></a>
</p>

Kokoro is a frontier TTS model for its size of 82 million parameters (text in/audio out). This JavaScript library allows the model to be run 100% locally in the browser thanks to [🤗 Transformers.js](https://huggingface.co/docs/transformers.js). Try it out using our [online demo](https://huggingface.co/spaces/webml-community/kokoro-webgpu)!

## Usage

First, install the `kokoro-js` library from [NPM](https://npmjs.com/package/kokoro-js) using:

```bash
npm i kokoro-js
```

You can then generate speech as follows:

```js
import { KokoroTTS } from "kokoro-js";

const model_id = "onnx-community/Kokoro-82M-v1.0-ONNX";
const tts = await KokoroTTS.from_pretrained(model_id, {
  dtype: "q8", // Options: "fp32", "fp16", "q8", "q4", "q4f16"
  device: "wasm", // Options: "wasm", "webgpu" (web) or "cpu" (node). If using "webgpu", we recommend using dtype="fp32".
});

const text = "Life is like a box of chocolates. You never know what you're gonna get.";
const audio = await tts.generate(text, {
  // Use `tts.list_voices()` to list all available voices
  voice: "af_heart",
});
audio.save("audio.wav");
```

Or if you'd prefer to stream the output, you can do that with:

```js
import { KokoroTTS, TextSplitterStream } from "kokoro-js";

const model_id = "onnx-community/Kokoro-82M-v1.0-ONNX";
const tts = await KokoroTTS.from_pretrained(model_id, {
  dtype: "fp32", // Options: "fp32", "fp16", "q8", "q4", "q4f16"
  // device: "webgpu", // Options: "wasm", "webgpu" (web) or "cpu" (node).
});

// First, set up the stream
const splitter = new TextSplitterStream();
const stream = tts.stream(splitter);
(async () => {
  let i = 0;
  for await (const { text, phonemes, audio } of stream) {
    console.log({ text, phonemes });
    audio.save(`audio-${i++}.wav`);
  }
})();

// Next, add text to the stream. Note that the text can be added at different times.
// For this example, let's pretend we're consuming text from an LLM, one word at a time.
const text = "Kokoro is an open-weight TTS model with 82 million parameters. Despite its lightweight architecture, it delivers comparable quality to larger models while being significantly faster and more cost-efficient. With Apache-licensed weights, Kokoro can be deployed anywhere from production environments to personal projects. It can even run 100% locally in your browser, powered by Transformers.js!";
const tokens = text.match(/\s*\S+/g);
for (const token of tokens) {
  splitter.push(token);
  await new Promise((resolve) => setTimeout(resolve, 10));
}

// Finally, close the stream to signal that no more text will be added.
splitter.close();

// Alternatively, if you'd like to keep the stream open, but flush any remaining text, you can use the `flush` method.
// splitter.flush();
```

## Voices/Samples

> [!TIP]
> You can find samples for each of the voices in the [model card](https://huggingface.co/onnx-community/Kokoro-82M-v1.0-ONNX#samples) on Hugging Face.

### American English

| Name         | Traits | Target Quality | Training Duration | Overall Grade |
| ------------ | ------ | -------------- | ----------------- | ------------- |
| **af_heart** | 🚺❤️   |                |                   | **A**         |
| af_alloy     | 🚺     | B              | MM minutes        | C             |
| af_aoede     | 🚺     | B              | H hours           | C+            |
| af_bella     | 🚺🔥   | **A**          | **HH hours**      | **A-**        |
| af_jessica   | 🚺     | C              | MM minutes        | D             |
| af_kore      | 🚺     | B              | H hours           | C+            |
| af_nicole    | 🚺🎧   | B              | **HH hours**      | B-            |
| af_nova      | 🚺     | B              | MM minutes        | C             |
| af_river     | 🚺     | C              | MM minutes        | D             |
| af_sarah     | 🚺     | B              | H hours           | C+            |
| af_sky       | 🚺     | B              | _M minutes_ 🤏    | C-            |
| am_adam      | 🚹     | D              | H hours           | F+            |
| am_echo      | 🚹     | C              | MM minutes        | D             |
| am_eric      | 🚹     | C              | MM minutes        | D             |
| am_fenrir    | 🚹     | B              | H hours           | C+            |
| am_liam      | 🚹     | C              | MM minutes        | D             |
| am_michael   | 🚹     | B              | H hours           | C+            |
| am_onyx      | 🚹     | C              | MM minutes        | D             |
| am_puck      | 🚹     | B              | H hours           | C+            |
| am_santa     | 🚹     | C              | _M minutes_ 🤏    | D-            |

### British English

| Name        | Traits | Target Quality | Training Duration | Overall Grade |
| ----------- | ------ | -------------- | ----------------- | ------------- |
| bf_alice    | 🚺     | C              | MM minutes        | D             |
| bf_emma     | 🚺     | B              | **HH hours**      | B-            |
| bf_isabella | 🚺     | B              | MM minutes        | C             |
| bf_lily     | 🚺     | C              | MM minutes        | D             |
| bm_daniel   | 🚹     | C              | MM minutes        | D             |
| bm_fable    | 🚹     | B              | MM minutes        | C             |
| bm_george   | 🚹     | B              | MM minutes        | C             |
| bm_lewis    | 🚹     | C              | H hours           | D+            |
