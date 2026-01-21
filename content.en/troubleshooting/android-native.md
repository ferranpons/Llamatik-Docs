---
title: "Android native loading issues"
weight: 2
---
If you get `UnsatisfiedLinkError` on Android, it usually means:

## 1) The native library is missing for the ABI

Make sure the correct `.so` is packaged for your device ABI (arm64-v8a, etc.).

## 2) Loading happens too early / in the wrong classloader

Llamatik’s Android implementation loads its JNI library. If you have a custom setup, ensure the load happens in a place that runs once and before any native calls.

## 3) Proguard / R8 stripping

If you minify, confirm you’re not stripping required JNI entry points.

## Debug tips

- Inspect the APK/AAB and confirm the `.so` exists under `lib/<abi>/`.
- Use `adb logcat` to see the exact library name it tries to load.
