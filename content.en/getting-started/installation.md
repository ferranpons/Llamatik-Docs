---
title: "Installation"
weight: 1
---
## Gradle (Kotlin DSL)

Add the dependency to the module where you want to use Llamatik:

```kotlin
dependencies {
    implementation("com.llamatik:library:0.x.y")
}
```

### Kotlin Multiplatform

In a KMP module, add it to the relevant source sets:

```kotlin
kotlin {
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("com.llamatik:library:0.x.y")
            }
        }
    }
}
```

### Android notes

- **Min SDK:** 26 (Android 8.0)
- Llamatik loads a native library at runtime on Android.
- Put your `.gguf` model under `app/src/main/assets/` and use `getModelPath(...)` (see Quickstart).

### iOS notes

- **Minimum iOS deployment target:** 16.6
- Llamatik is distributed as a Kotlin/Native framework via KMP. If you hit linking issues, see Troubleshooting → iOS.

### Desktop / JVM notes

- Requires a recent JDK (the repo uses toolchain 21).
- The JVM target loads the native library (`llama_jni`) on startup.
