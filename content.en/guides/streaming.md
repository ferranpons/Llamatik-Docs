---
title: "Streaming"
weight: 3
---

Streaming lets your app consume generation output incrementally.
This usually feels much better than waiting for the full answer, especially on-device where responses may take time.

## Plain streaming

```kotlin
LlamaBridge.generateStream(
    prompt = "Stream a short poem.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

## Streaming with context

```kotlin
LlamaBridge.generateStreamWithContext(
    systemPrompt = "You are concise.",
    contextBlock = "Topic: on-device LLMs.",
    userPrompt = "Give me 3 bullet points.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

## Streaming JSON

```kotlin
LlamaBridge.generateJsonStream(
    prompt = "Return a JSON object for one task.",
    jsonSchema = schema,
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println("\nDone")
        override fun onError(message: String) = println("Error: $message")
    }
)
```

## Streaming with KV cache continuity

Every streaming function on `LlamaBridge` (`generateStream`, `generateStreamWithContext`, etc.) resets the KV cache before each call by design. To keep the model's memory across turns while streaming, you have two options.

### Option 1 — `generateContinueStream` (global KV cache, supports disk persistence)

`generateContinueStream` is the streaming equivalent of `generateContinue`. It reuses the existing global KV cache instead of resetting it. The C++ layer finds the longest common token prefix between the new prompt and the cached context, trims only the diverging tail from the KV cache, and decodes just the new tokens — so prior turns are never re-encoded.

```kotlin
LlamaBridge.initGenerateModel(modelPath)

// Turn 1 — no prior session, behaves like generateStream
LlamaBridge.generateContinueStream(
    prompt = "Explain Kotlin coroutines.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() {
            println()
            LlamaBridge.sessionSave(sessionPath)
        }
        override fun onError(message: String) = println("Error: $message")
    }
)

// Turn 2 — load the saved session and stream the continuation
LlamaBridge.sessionLoad(sessionPath)
LlamaBridge.generateContinueStream(
    prompt = "Now show a short example.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() {
            println()
            LlamaBridge.sessionSave(sessionPath)
        }
        override fun onError(message: String) = println("Error: $message")
    }
)
```

Use this when you need **streaming + disk-serialized session persistence** across app restarts.

### Option 2 — `LlamaSession` (in-memory KV cache, supports concurrency)

For in-process conversations without disk persistence, `LlamaSession` is simpler. Each `session.stream(...)` call appends to the session's in-memory KV cache automatically.

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val session = LlamaBridge.createSession(name = "chat")
    ?: error("sessions not supported on this platform")

// Turn 1 — KV state is built from this prompt
session.stream(
    prompt = "Explain Kotlin coroutines.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println()
        override fun onError(message: String) = println("Error: $message")
    }
)

// Turn 2 — continues from the existing KV state; model remembers the previous turn
session.stream(
    prompt = "Now show a short example.",
    callback = object : GenStream {
        override fun onDelta(text: String) = print(text)
        override fun onComplete() = println()
        override fun onError(message: String) = println("Error: $message")
    }
)

session.close()
```

Use this when you need **multiple concurrent independent streams** or no cross-restart persistence is required.

See [KV cache and sessions]({{< relref "kv-cache-and-sessions" >}}) for a full comparison.

## Lifecycle tips

- Cancel generation when the user navigates away.
- Buffer deltas and render efficiently.
- Use `generateContinueStream` when you need KV-cache reuse with disk persistence.
- Use `LlamaSession` when you need concurrent independent streams.
- Use `LlamaBridge.generateStream(...)` only for independent one-shot requests.
- Do not start multiple heavy streams on the same small device unless you have tested the behavior carefully.
- On WASM, prefer streaming APIs when running in worker-only mode.
