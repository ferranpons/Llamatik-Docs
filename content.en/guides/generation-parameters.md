---
title: "Generation parameters"
weight: 4
---

Llamatik exposes runtime generation parameters through `updateGenerateParams(...)`.

```kotlin
LlamaBridge.updateGenerateParams(
    temperature = 0.7f,
    maxTokens = 256,
    topP = 0.95f,
    topK = 40,
    repeatPenalty = 1.1f,
    contextLength = 4096,
    numThreads = 4,
    useMmap = true,
    flashAttention = false,
    batchSize = 512,
    gpuLayers = 0,
)
```

## Sampling parameters

### temperature
Controls randomness.

- lower values: more deterministic
- higher values: more varied and creative

### maxTokens
Sets the maximum number of tokens the model may generate.
Use this to control response length and latency.

### topP
Nucleus sampling threshold.
The model samples from the smallest set of likely tokens whose cumulative probability reaches `topP`.

### topK
Limits sampling to the `K` most likely next tokens.

### repeatPenalty
Discourages repeated phrases and loops.
Useful for chat, summaries, and structured outputs.

## Backend and memory parameters

### contextLength
The model's context window in tokens.
Larger values allow longer conversations but require more memory.
Must not exceed what the loaded model was compiled to support.

### numThreads
Number of CPU threads used during inference.
Pass `-1` to let the platform choose a sensible default.
On mobile, matching the number of performance cores is a good starting point.

### useMmap
Whether to use memory-mapped I/O for the model weights.
Enabled by default on most platforms.
Turn it off if you encounter issues with certain file systems or need full memory ownership.

### flashAttention
Enables Flash Attention when supported by the backend.
Can reduce memory usage and improve throughput on large context windows.
Not supported on all platforms — verify with the underlying llama.cpp version in use.

### batchSize
The batch size used during prompt processing (the prefill phase).
Larger values can improve throughput on long prompts at the cost of memory.
A value of `512` works well for most use cases.

### gpuLayers
The number of transformer layers to offload to the GPU accelerator.

- `0` — all computation runs on CPU (default)
- `-1` — all layers are offloaded to GPU (Metal on iOS/macOS, CUDA or Vulkan on Android/JVM where supported)
- `N` (positive integer) — exactly N layers are offloaded; the rest remain on CPU

Offloading more layers increases throughput significantly on devices with a capable GPU.
Start with `-1` to offload everything, then reduce if you hit memory limits.

This parameter requires a model reload to take effect.
On WASM, `gpuLayers` is silently ignored — there is no GPU offload path in the WebAssembly target.

> **Note:** Vulkan and CUDA are not enabled in the pre-built Maven Central artifacts. To use them you must build Llamatik from source with the appropriate CMake flag. See [Building from source]({{< ref "guides/building-from-source" >}}) for instructions.

## Tuning advice

- Start from moderate values and test on a fixed prompt set.
- Change one parameter at a time.
- For extraction or JSON, lean toward lower temperature.
- For brainstorming or creative tasks, slightly higher temperature can help.
- For long chat history, increase `contextLength` and consider enabling `flashAttention`.
- If memory is tight on mobile, reduce `batchSize` and `contextLength`.
- On Metal-capable iOS/macOS devices, set `gpuLayers = -1` to offload all layers and gain significant throughput.
- On Android with a CUDA or Vulkan-capable GPU, the same `-1` value applies; fall back to a specific layer count if you hit OOM errors.
