---
title: "WhisperBridge"
weight: 4
---

`WhisperBridge` provides on-device speech-to-text using Whisper models.
Its API is intentionally small: initialize a model, transcribe a WAV file, and release resources when done.

## Platform support

| Platform | Status |
|---|---|
| Android | supported |
| iOS | supported |
| JVM / Desktop | supported |
| WASM | not implemented |

## Public API

```kotlin
expect object WhisperBridge {
    fun getModelPath(modelFileName: String): String
    fun initModel(modelPath: String): Boolean
    fun transcribeWav(
        wavPath: String,
        language: String? = null,
        initialPrompt: String? = null,
    ): String
    fun transcribeWavSegments(
        wavPath: String,
        language: String? = null,
        initialPrompt: String? = null,
        translate: Boolean = false,
        diarize: Boolean = false,
    ): String
    fun release()
}
```

## `getModelPath(modelFileName)`
Returns a platform-usable path for the Whisper model file.

```kotlin
val modelPath = WhisperBridge.getModelPath("ggml-base.en.bin")
```

## `initModel(modelPath)`
Loads the Whisper model and prepares the transcription context.

```kotlin
val ok = WhisperBridge.initModel(modelPath)
check(ok) { "Failed to initialize Whisper model" }
```

Call this once before transcription.

## `transcribeWav(wavPath, language, initialPrompt)`
Transcribes a WAV audio file and returns all segments joined into a single flat string.

```kotlin
val text = WhisperBridge.transcribeWav(
    wavPath = "/path/to/sample.wav",
    language = "en",
    initialPrompt = "The following is a technical discussion about Kotlin."
)
println(text)
```

### Parameters

- `wavPath`: path to the WAV file to transcribe
- `language`: optional language hint such as `"en"`, `"es"`, or `"fr"`. Providing it improves predictability and reduces ambiguity when you already know the input language.
- `initialPrompt`: optional text to prime the model before transcription begins. Use this to bias the output toward specific vocabulary, domain terms, or formatting conventions. The model treats this as prior context without transcribing it literally.

### Return value

Returns the transcription as a `String`. If transcription fails, the result may be empty.

## `transcribeWavSegments(wavPath, language, initialPrompt, translate, diarize)`

Segment-aware transcription. Returns a JSON document that exposes what `transcribeWav` discards: per-segment text with start/end timestamps (milliseconds), the tinydiarize speaker-turn flag, and the whole-audio detected language code.

### JSON shape

```json
{
  "language": "de",
  "segments": [
    {"text": "Guten Morgen.", "t0": 0, "t1": 1200, "speaker_turn_next": false},
    {"text": "Wie geht es Ihnen?", "t0": 1200, "t1": 2800, "speaker_turn_next": true}
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `language` | `String` | ISO language code auto-detected by Whisper (e.g. `"en"`, `"de"`) |
| `segments[].text` | `String` | Transcribed (or translated) text for this segment |
| `segments[].t0` | `Long` | Segment start time in **milliseconds** |
| `segments[].t1` | `Long` | Segment end time in **milliseconds** |
| `segments[].speaker_turn_next` | `Boolean` | `true` if a speaker change follows this segment (only meaningful with a `-tdrz` model) |

### Parameters

- `wavPath`: path to the WAV file to transcribe.
- `language`: optional language hint (same as `transcribeWav`).
- `initialPrompt`: optional priming text (same as `transcribeWav`).
- `translate`: pass `true` to run Whisper's built-in ORIGINAL→ENGLISH translation task. Segment text becomes the English translation regardless of the spoken language. Default `false` = language-preserving.
- `diarize`: pass `true` **only** with a tinydiarize (`…-tdrz`) model to enable `tdrz_enable` — this makes `speaker_turn_next` return real speaker-turn boundaries and also injects `[SPEAKER_TURN]` markers into segment text. Leave `false` for regular models.

### Example

```kotlin
import com.llamatik.library.platform.WhisperBridge
import kotlinx.serialization.json.*

WhisperBridge.initModel(modelPath)

val json = WhisperBridge.transcribeWavSegments(
    wavPath  = "/path/to/recording.wav",
    language = null,   // auto-detect
)

val root     = Json.parseToJsonElement(json).jsonObject
val language = root["language"]?.jsonPrimitive?.content
val segments = root["segments"]!!.jsonArray

for (seg in segments) {
    val obj  = seg.jsonObject
    val text = obj["text"]?.jsonPrimitive?.content
    val t0   = obj["t0"]?.jsonPrimitive?.long    // milliseconds
    val t1   = obj["t1"]?.jsonPrimitive?.long
    val turn = obj["speaker_turn_next"]?.jsonPrimitive?.boolean
    println("[$t0–$t1 ms] $text  (turn=$turn)")
}
```

### Translation example

```kotlin
val json = WhisperBridge.transcribeWavSegments(
    wavPath   = "/path/to/german_audio.wav",
    translate = true,   // segment text will be English regardless of spoken language
)
```

### Diarization example

```kotlin
// Use a tinydiarize model: e.g. ggml-small.en-tdrz.bin
WhisperBridge.initModel(WhisperBridge.getModelPath("ggml-small.en-tdrz.bin"))

val json = WhisperBridge.transcribeWavSegments(
    wavPath = "/path/to/two_speakers.wav",
    diarize = true,
)
```

## Supported input expectations

The bridge API is WAV-oriented, so the simplest and most reliable flow is:

1. record or load audio
2. convert it to WAV if necessary (16 kHz, mono, 16-bit PCM recommended)
3. call `transcribeWav(...)` or `transcribeWavSegments(...)`

If your app records audio in another format, perform a conversion step before calling the bridge.

## `release()`
Releases native Whisper resources.

```kotlin
WhisperBridge.release()
```

## Practical recommendations

- Keep audio clean and reasonably short when testing.
- Normalize your pipeline around WAV files for fewer surprises.
- Reuse the initialized model when transcribing multiple files.
- Use `transcribeWav` when you only need the full transcript text.
- Use `transcribeWavSegments` when you need timestamps, the detected language, or speaker turns.
- Pass `diarize = true` only with a `-tdrz` model — it has no effect (and no error) on regular models, but `speaker_turn_next` will always be `false`.
- Release resources when leaving the speech feature or shutting the app down.
