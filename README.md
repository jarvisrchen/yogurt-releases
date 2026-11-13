# yogurt releases

This repo holds the published builds of yogurt, a local-first meeting copilot for macOS, and its Android companion app.
It has no source code.

Install on a Mac with Homebrew:

```sh
brew install jarvisrchen/yogurt/yogurt
```

The Android app is the `yogurt-android.apk` asset on the [latest release](https://github.com/jarvisrchen/yogurt-releases/releases/latest).

The `models-v1` release mirrors the speech models yogurt can transcribe with, byte for byte, so installs work on networks that cannot reach Hugging Face.
That tag never moves, because released builds resolve their models against it.
