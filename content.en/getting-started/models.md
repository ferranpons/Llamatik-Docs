---
title: "Models"
weight: 3
---
Llamatik uses **GGUF** models (the standard file format used by `llama.cpp`).

## Picking a model

- Choose a model that has a **GGUF** release.
- Choose a **quantization** that matches your device constraints:
  - Smaller quantizations run faster and use less memory, with some quality tradeoff.
  - Larger quantizations can be higher quality but require more RAM.

## Where to get GGUF models

Common places include Hugging Face model pages that provide `.gguf` files.

## Recommended first tests

- Tiny/mini models (very small) to validate that your build + loading works.
- Then move to your target model size/quality once everything is stable.

## Shipping models

Models can be large. Consider:
- **Android:** ship a smaller default model, or download on first run.
- **iOS:** Apple size limits apply; use on-demand resources or download after install.
- **Desktop:** bundle or download, depending on your distribution model.
