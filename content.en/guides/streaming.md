---
title: "Streaming"
weight: 3
---
Streaming lets you receive tokens as they are generated.

## Plain streaming

```kotlin
LlamaBridge.generateStream(
  prompt = "Stream a short poem.",
  callback = object : GenStream {
    override fun onDelta(text: String) = print(text)
    override fun onComplete() = println("\n✅ done")
    override fun onError(message: String) = println("❌ $message")
  }
)
```

## Streaming with context

```kotlin
LlamaBridge.generateStreamWithContext(
  systemPrompt = "You are concise.",
  contextBlock = "Topic: on-device LLMs.",
  userPrompt = "Give me 3 bullet points.",
  callback = object : GenStream { /* ... */ }
)
```

## Lifecycle tips

- Call `nativeCancelGenerate()` when leaving the screen to stop background work.
- If you hold a reference to the callback, avoid leaking UI scope objects.
