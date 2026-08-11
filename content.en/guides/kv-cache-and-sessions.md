---
title: "KV cache and sessions"
weight: 6
---

KV cache reuse is one of the most important performance features for chat-like experiences.
Instead of rebuilding the full prompt state from scratch for every turn, the model can continue from what it has already processed.

There are three ways to use the KV cache:

- **`LlamaSession`** — for streaming multi-turn conversations with an in-memory KV cache that persists across `stream()` calls. Recommended for chat UIs that do not need cross-process persistence.
- **`generateContinueStream`** — streaming multi-turn generation with the global KV cache. Supports `sessionSave`/`sessionLoad` for persistence across restarts. The correct choice when you need both streaming and disk-serialized session state.
- **`generateContinue`** — blocking (non-streaming) continuation. Use when you do not need incremental token delivery.

## Core methods (global)

```kotlin
LlamaBridge.sessionReset()
LlamaBridge.sessionSave(path)
LlamaBridge.sessionLoad(path)
LlamaBridge.generateContinue(prompt)          // blocking
LlamaBridge.generateContinueStream(prompt, callback) // streaming
```

## Fresh turn vs continued turn

### Fresh generation

```kotlin
val answer = LlamaBridge.generate("Explain Kotlin coroutines.")
```

Use this when the request is independent and you do not care about previous turns.

### Continued generation

```kotlin
val answer2 = LlamaBridge.generateContinue("Now show a short example.")
```

Use this when you want the next prompt to continue from the current session state.

## Typical chat flow

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val first = LlamaBridge.generate("Explain Kotlin coroutines in simple words.")
val second = LlamaBridge.generateContinue("Now give one practical example.")
val third = LlamaBridge.generateContinue("Summarize both answers in 3 bullets.")
```

## Saving a session

```kotlin
val sessionPath = "/tmp/demo.session"
val saved = LlamaBridge.sessionSave(sessionPath)
check(saved)
```

This lets you persist the current conversation state across app restarts or later reuse.

## Loading a session

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val loaded = LlamaBridge.sessionLoad(sessionPath)
check(loaded)

val resumed = LlamaBridge.generateContinue("Continue from where we stopped.")
```

## Resetting the session

```kotlin
LlamaBridge.sessionReset()
```

This is useful when the user starts a new conversation but you want to keep the model loaded.

## Streaming with `generateContinueStream`

`generateContinueStream` is the streaming equivalent of `generateContinue`. It uses the global KV cache and is the right choice when you also need `sessionSave`/`sessionLoad` for cross-restart persistence.

Under the hood the C++ implementation:
1. Tokenizes the full new prompt (including BOS).
2. Finds the longest common token prefix between the new prompt and the cached context.
3. Trims the KV cache to that prefix (discards only the diverging tail).
4. Decodes only the new suffix tokens — prior turns are never re-encoded.

This means the performance gain grows with each additional turn, not just the first one.

```kotlin
LlamaBridge.initGenerateModel(modelPath)

// Turn 1 — no prior session, falls back to a fresh stream
LlamaBridge.generateContinueStream("Explain Kotlin coroutines.", object : GenStream {
    override fun onDelta(text: String) = print(text)
    override fun onComplete() {
        println()
        LlamaBridge.sessionSave(sessionPath)   // persist after each turn
    }
    override fun onError(message: String) = println("Error: $message")
})

// Turn 2 — restore the saved session and stream the continuation
LlamaBridge.sessionLoad(sessionPath)
LlamaBridge.generateContinueStream("Now give a short example.", object : GenStream {
    override fun onDelta(text: String) = print(text)
    override fun onComplete() {
        println()
        LlamaBridge.sessionSave(sessionPath)
    }
    override fun onError(message: String) = println("Error: $message")
})
```

`generateContinueStream` falls back to a fresh `generateStream` when no session is active, so it is safe to call for every turn without checking session state first.

## Streaming multi-turn chat with `LlamaSession`

For in-process streaming conversations without disk persistence, `LlamaSession` is the simpler option.
Each `session.stream(...)` call appends the new prompt tokens to the session's existing in-memory KV cache,
so the model retains full memory of previous turns without any explicit save/load.

```kotlin
LlamaBridge.initGenerateModel(modelPath)

val session = LlamaBridge.createSession(name = "chat")
    ?: error("sessions not supported on this platform")

// Turn 1
session.stream("What is Kotlin?", callback)

// Turn 2 — model remembers what was said in turn 1
session.stream("Give me an example of a coroutine.", callback)

// Explicitly reset the KV state when the conversation is over
session.close()
```

The global `LlamaBridge.generateStream(...)` is stateless and does not carry KV context across calls.

See [Concurrent Sessions]({{< relref "concurrent-sessions" >}}) for running multiple independent sessions in parallel.

## Choosing between `generateContinueStream` and `LlamaSession`

| | `generateContinueStream` | `LlamaSession` |
|---|---|---|
| Streaming | Yes | Yes |
| KV cache persists across app restarts | Yes (via `sessionSave`/`sessionLoad`) | No |
| Multiple concurrent streams | No (global state) | Yes |
| WASM support | No (session persistence unavailable) | No |

## Important limitations

- Session persistence (`sessionSave`/`sessionLoad`) is currently unavailable on WASM.
- Session files must be used with the same model that created them.
- If no active session exists, both `generateContinue()` and `generateContinueStream()` fall back to fresh generation.
- `LlamaSession` is not supported on WASM — use `LlamaBridge.generateStream(...)` or `LlamaBridge.generateContinueStream(...)` there instead.

## When this feature is worth using

Use KV sessions when:
- you are building a chat app
- the user asks follow-up questions
- you want faster continuation across turns
- you are streaming and need the model to remember previous turns

Skip it when:
- every request is independent
- you deliberately want each run to start with a clean state
