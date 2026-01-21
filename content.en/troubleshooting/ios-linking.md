---
title: "iOS linking issues"
weight: 1
---
If your iOS build fails during the **link** step, it’s usually one of these:

## 1) Relative paths inside published artifacts

If you see errors like “search path not found” or “library not found”, check whether any `.def` / `linkerOpts` are using **relative** paths that don’t exist in the consumer project.

**Fix:** ensure published artifacts don’t encode build-machine-relative paths. Prefer packaging the needed native libs inside the artifact (or use absolute paths generated at build time).

## 2) Missing symbols from static archives

If you see “undefined symbol” for wrapper functions, the archive may be present but not **force-loaded**.

**Fix:** add force-load for the wrapper archive to the framework linker options:

- `-Wl,-force_load,<path-to-libllama_wrapper.a>` (device)
- `-Wl,-force_load,<path-to-libllama_wrapper_sim.a>` (simulator)

(Exact filenames depend on your setup.)

## 3) Mismatched simulator / device architectures

Make sure you’re linking:
- `iosArm64` for devices
- `iosSimulatorArm64` for Apple Silicon simulators
- `iosX64` for Intel simulators

## Debug tips

- Confirm the symbol exists in the archive with `nm -gU <archive>.a | grep <symbol>`.
- Confirm the archive is actually passed to the final link command (Gradle verbose logs help).

If you can share the full linker error and your target (device vs simulator), it’s usually straightforward to pinpoint.
