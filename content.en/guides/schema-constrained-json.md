---
title: "Schema-constrained JSON"
weight: 5
---
Llamatik can generate **valid JSON** and (optionally) constrain it to a **JSON Schema**.

This is useful for:
- typed outputs you can parse into Kotlin data classes
- calling tools / functions from structured responses
- building reliable pipelines (no “almost JSON”)

## Basic JSON generation (no schema)

```kotlin
val json = LlamaBridge.generateJson(
  prompt = "Return a JSON object with fields: name (string), year (int)."
)
println(json)
```

## Constrain to a JSON Schema

Pass a schema string to `jsonSchema`.

```kotlin
val schema = """{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "name": { "type": "string" },
    "year": { "type": "integer" }
  },
  "required": ["name", "year"]
}"""

val json = LlamaBridge.generateJson(
  prompt = "Generate an example object.",
  jsonSchema = schema
)
```

## With context

```kotlin
val json = LlamaBridge.generateJsonWithContext(
  systemPrompt = "You output only JSON.",
  contextBlock = "The product is Llamatik.",
  userPrompt = "Return product metadata.",
  jsonSchema = schema
)
```

## Streaming JSON

If you want incremental output:

```kotlin
LlamaBridge.generateJsonStream(
  prompt = "Return JSON for a grocery list with 3 items.",
  jsonSchema = schema,
  callback = object : GenStream { /* ... */ }
)
```

### Parsing

Once you get the string, parse it with your JSON library of choice (Kotlinx Serialization, Moshi, Jackson, etc.).
