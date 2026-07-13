---
title: "Whisper speech-to-text"
weight: 8
---

`WhisperBridge` is the speech-to-text entry point in Llamatik.
The library exposes two transcription methods: `transcribeWav` for a simple flat string, and `transcribeWavSegments` for structured per-segment output with timestamps, detected language, and speaker-turn boundaries.

## Initialize the model

```kotlin
val modelPath = WhisperBridge.getModelPath("ggml-base.en.bin")
val ok = WhisperBridge.initModel(modelPath)
check(ok)
```

## Transcribe a file

```kotlin
val text = WhisperBridge.transcribeWav(
    wavPath = "/path/to/audio.wav",
    language = "en"
)
println(text)
```

## Language hint

The `language` parameter is optional, but supplying it can improve reliability when you already know the input language.

## Initial prompt

The `initialPrompt` parameter primes the model before transcription begins.
Use it to bias output toward specific vocabulary, domain terms, or formatting conventions.

```kotlin
val text = WhisperBridge.transcribeWav(
    wavPath = audioPath,
    language = "en",
    initialPrompt = "The following is a developer podcast about Kotlin Multiplatform."
)
```

The model uses the prompt as prior context — it does not transcribe it literally.
This is useful when your audio contains technical terms or proper nouns that the model might otherwise mis-transcribe.

## Segment-aware transcription

`transcribeWavSegments` returns a JSON document exposing what `transcribeWav` discards: per-segment text with start/end timestamps (milliseconds), the auto-detected language, and speaker-turn boundaries.

```kotlin
val json = WhisperBridge.transcribeWavSegments(
    wavPath  = "/path/to/audio.wav",
    language = null,   // auto-detect
)
```

The JSON shape:

```json
{
  "language": "de",
  "segments": [
    {"text": "Guten Morgen.", "t0": 0, "t1": 1200, "speaker_turn_next": false},
    {"text": "Wie geht es Ihnen?", "t0": 1200, "t1": 2800, "speaker_turn_next": true}
  ]
}
```

- `language` — ISO language code auto-detected by Whisper.
- `t0` / `t1` — segment start and end in **milliseconds**.
- `speaker_turn_next` — `true` if a speaker change follows (only meaningful with a `-tdrz` model).

Parse it with any JSON library:

```kotlin
import kotlinx.serialization.json.*

val root     = Json.parseToJsonElement(json).jsonObject
val language = root["language"]?.jsonPrimitive?.content
val segments = root["segments"]!!.jsonArray

for (seg in segments) {
    val obj  = seg.jsonObject
    val text = obj["text"]?.jsonPrimitive?.content
    val t0   = obj["t0"]?.jsonPrimitive?.long
    val t1   = obj["t1"]?.jsonPrimitive?.long
    println("[$t0–$t1 ms] $text")
}
```

## Translation to English

Pass `translate = true` to run Whisper's built-in translate task. Segment text becomes the English translation of the audio, regardless of the spoken language — this is a real Whisper translation, not a post-processing step.

```kotlin
val json = WhisperBridge.transcribeWavSegments(
    wavPath   = "/path/to/german_audio.wav",
    translate = true,
)
```

`translate = false` (the default) keeps the transcription in the original spoken language.

## Speaker diarization

Pass `diarize = true` **only** with a tinydiarize (`…-tdrz`) model to enable speaker-turn detection. This sets Whisper's `tdrz_enable` flag, which makes `speaker_turn_next` return real boundaries and also injects `[SPEAKER_TURN]` markers into segment text.

```kotlin
// Load a tinydiarize model first
WhisperBridge.initModel(WhisperBridge.getModelPath("ggml-small.en-tdrz.bin"))

val json = WhisperBridge.transcribeWavSegments(
    wavPath = "/path/to/two_speakers.wav",
    diarize = true,
)
```

Leave `diarize = false` (the default) for regular models — `speaker_turn_next` will always be `false` and there is no error.

## Recommended workflow

1. record or obtain audio
2. convert it to WAV if needed (16 kHz, mono, 16-bit PCM)
3. initialize the model once
4. call `transcribeWav(...)` for plain text, or `transcribeWavSegments(...)` for structured output
5. release the model when done

## Cleanup

```kotlin
WhisperBridge.release()
```

## Practical notes

- For best results, keep input audio clear and reasonably clean.
- Reuse the initialized model if you transcribe multiple files.
- Use `transcribeWav` when you only need the full transcript string.
- Use `transcribeWavSegments` when you need timestamps, the detected language, or speaker turns.
- WASM support is currently not available for WhisperBridge.
