---
title: "Building from source"
weight: 12
---

The pre-built Maven Central artifacts include CPU-only native binaries everywhere except macOS, where Metal is enabled by default. If you need a GPU backend such as Vulkan or CUDA you must build Llamatik from source and pass the appropriate CMake flags.

## Extra CMake flags

The `llamatik.cmake.args` Gradle property (or the `LLAMATIK_CMAKE_ARGS` environment variable as a fallback) lets you append flags to the CMake configure step without editing the build script.

Values are split on whitespace. Flags containing spaces are not supported.

### Vulkan (Android / Desktop Linux)

```bash
./gradlew :library:build -Pllamatik.cmake.args="-DGGML_VULKAN=ON"
```

### CUDA (Desktop Linux / Windows)

```bash
./gradlew :library:build -Pllamatik.cmake.args="-DGGML_CUDA=ON"
```

### Using the environment variable

```bash
LLAMATIK_CMAKE_ARGS="-DGGML_VULKAN=ON" ./gradlew :library:build
```

### Multiple flags

```bash
./gradlew :library:build -Pllamatik.cmake.args="-DGGML_VULKAN=ON -DGGML_VULKAN_VALIDATE=ON"
```

## Scope

The extra flags are forwarded to these targets:

| Target | Receives flags |
|---|---|
| Apple wrapper (macOS / iOS simulator / iOS device) | Yes |
| Desktop JNI (macOS, Linux, Windows) | Yes |
| Android (`externalNativeBuild`) | Yes |
| WASM / Emscripten | **No** — GPU backends do not apply to WebAssembly |

## iOS limitation

The `mergeLlamaStatic*` step that produces the iOS fat static library merges a hardcoded list of ggml archives. If a GPU backend produces additional archives (for example a Vulkan MoltenVK layer), they will **not** be included in the merged binary. A build warning is emitted when extra flags are passed to an iOS target as a reminder. Desktop and Android GPU builds are not affected.

## Enabling GPU layers at runtime

Once you have built with a GPU backend enabled, set `gpuLayers` in `GenerateParams` to offload computation:

```kotlin
LlamaBridge.updateGenerateParams(
    gpuLayers = -1  // offload all layers
)
```

See [Generation parameters]({{< ref "guides/generation-parameters" >}}) for the full `gpuLayers` reference.
