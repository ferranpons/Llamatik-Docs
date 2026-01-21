---
title: "LlamaBridge"
weight: 1
---
`LlamaBridge` is an `expect object` with platform-specific native implementations.

## Model path

```kotlin
@Composable
fun getModelPath(modelFileName: String): String
```

On Android, this helper typically copies an asset to a readable path and returns it.

## Embeddings

```kotlin
fun initModel(modelPath: String): Boolean
fun embed(input: String): FloatArray
```

## Generation

```kotlin
fun initGenerateModel(modelPath: String): Boolean
fun generate(prompt: String): String
fun generateWithContext(systemPrompt: String, contextBlock: String, userPrompt: String): String
```

## JSON / JSON Schema

```kotlin
fun generateJson(prompt: String, jsonSchema: String? = null): String
fun generateJsonWithContext(
  systemPrompt: String,
  contextBlock: String,
  userPrompt: String,
  jsonSchema: String? = null
): String
```

## Streaming

```kotlin
fun generateStream(prompt: String, callback: GenStream)
fun generateStreamWithContext(systemPrompt: String, contextBlock: String, userPrompt: String, callback: GenStream)

fun generateJsonStream(prompt: String, jsonSchema: String? = null, callback: GenStream)
fun generateJsonStreamWithContext(
  systemPrompt: String,
  contextBlock: String,
  userPrompt: String,
  jsonSchema: String? = null,
  callback: GenStream
)
```

## Runtime control

```kotlin
fun nativeCancelGenerate()
fun updateGenerateParams(temperature: Float, maxTokens: Int, topP: Float, topK: Int, repeatPenalty: Float)
fun shutdown()
```

Call `shutdown()` when your app is terminating (or when you know you will not use the model anymore) to release native resources.
