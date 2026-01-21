---
title: "Quickstart"
weight: 2
---
## 1) Add a model

Download a `.gguf` model (see **Models** page). Then:

### Android
Place it in your app assets, for example:
`composeApp/src/androidMain/assets/tinyllama.gguf`

### Desktop / JVM
Place it somewhere on disk and pass the absolute path.

### iOS
Bundle the model with the app (for example as a resource) and pass the resolved path.

## 2) Initialize

```kotlin
import com.llamatik.library.platform.LlamaBridge

val modelPath = /* platform-specific path to .gguf */
val ok = LlamaBridge.initGenerateModel(modelPath)
check(ok) { "Failed to init model at $modelPath" }
```

## 3) Generate

```kotlin
val text = LlamaBridge.generate("Write a 2 sentence story about a llama.")
println(text)
```

## 4) Embeddings

```kotlin
val ok2 = LlamaBridge.initModel(modelPath)
check(ok2) { "Failed to init embedding model" }

val vector: FloatArray = LlamaBridge.embed("hello embeddings")
println("dims = ${vector.size}")
```

## 5) Streaming

```kotlin
LlamaBridge.generateStream(
  prompt = "Stream a short haiku.",
  callback = object : GenStream {
    override fun onDelta(text: String) = print(text)
    override fun onComplete() = println("\n✅ done")
    override fun onError(message: String) = println("❌ $message")
  }
)
```
