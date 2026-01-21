---
title: "Text generation"
weight: 2
---
## Basic generation

```kotlin
val ok = LlamaBridge.initGenerateModel(modelPath)
check(ok)

val answer = LlamaBridge.generate("Explain Llamatik in one sentence.")
```

## With context (system + context + user)

Use `generateWithContext` when you want a chat-style prompt:

```kotlin
val system = "You are a helpful assistant."
val context = "Project: Llamatik is a Kotlin-first llama.cpp integration."
val user = "Write a short README intro."

val text = LlamaBridge.generateWithContext(system, context, user)
```

## Cancel generation

If you need to stop generation (for example on screen close):

```kotlin
LlamaBridge.nativeCancelGenerate()
```

(See also **Streaming** for incremental output.)
