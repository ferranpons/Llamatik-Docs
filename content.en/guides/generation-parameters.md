---
title: "Generation parameters"
weight: 4
---
Llamatik exposes a small set of runtime generation parameters.

```kotlin
LlamaBridge.updateGenerateParams(
  temperature = 0.7f,
  maxTokens = 256,
  topP = 0.95f,
  topK = 40,
  repeatPenalty = 1.1f,
)
```

### What they do (quick intuition)

- **temperature:** higher → more random; lower → more deterministic
- **maxTokens:** hard cap for output length
- **topP / topK:** sampling constraints that trade creativity vs consistency
- **repeatPenalty:** discourages repeating the same phrases

If you’re tuning, change one parameter at a time and test on a fixed prompt set.
