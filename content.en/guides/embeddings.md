---
title: "Embeddings"
weight: 1
---
Embeddings convert text into a vector (`FloatArray`) you can use for similarity search, clustering, and retrieval.

## Init + embed

```kotlin
val ok = LlamaBridge.initModel(modelPath)
check(ok)

val v1 = LlamaBridge.embed("What is Llamatik?")
val v2 = LlamaBridge.embed("Kotlin llama.cpp bindings")
```

## Cosine similarity (example)

```kotlin
fun cosine(a: FloatArray, b: FloatArray): Float {
  require(a.size == b.size)
  var dot = 0.0
  var na = 0.0
  var nb = 0.0
  for (i in a.indices) {
    val x = a[i].toDouble()
    val y = b[i].toDouble()
    dot += x * y
    na += x * x
    nb += y * y
  }
  return (dot / (kotlin.math.sqrt(na) * kotlin.math.sqrt(nb))).toFloat()
}

println(cosine(v1, v2))
```

## Tips

- Use the same model for all embeddings in a given index.
- Normalize vectors if your search library expects it.
- Cache embeddings for repeated documents.
