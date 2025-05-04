Folders:
- `kokoro.js`: Reference copy of [kokoro.js](https://github.com/hexgrad/kokoro/tree/main/kokoro.js) with the changes mentioned in [https://github.com/hexgrad/kokoro/issues/65#issuecomment-2848869611](https://github.com/hexgrad/kokoro/issues/65#issuecomment-2848869611).
- `dist`: Ready-to-use packaged version of kokoro.js that includes Spanish voices.

Use example:
```html
<script src="http://localhost/proyectos/pruebas/kokoro/out/kokoro-vite/dist/kokoro.umd.js"></script>
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