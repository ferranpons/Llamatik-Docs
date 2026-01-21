---
title: "GenStream"
weight: 2
---
`GenStream` is a minimal callback interface used by streaming generation functions.

```kotlin
interface GenStream {
  fun onDelta(text: String)
  fun onComplete()
  fun onError(message: String)
}
```

### Typical usage

- Append `onDelta` chunks to your UI buffer.
- Use `onComplete` to re-enable UI actions.
- Surface `onError` to logs and/or a snackbar/toast.
