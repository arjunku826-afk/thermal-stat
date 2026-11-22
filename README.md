# Thermal States

A HUD mod for CaramelClient / GlowberryClient. It shows the device's thermal state
(Nominal / Fair / Serious / Critical / Shutdown) in the top-left corner, colour-coded, with a
headroom bar when the OS reports one.

## Build

Needs the Android NDK (arm64-v8a), CMake 3.22+, and network access for FetchContent.

    cmake -B build -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK_HOME/build/cmake/android.toolchain.cmake \
          -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-28
    cmake --build build

## Package

The directory name is the mod id. Put the built `libthermal_states.so` next to `manifest.json`:

    thermal-states/
    ├── manifest.json
    └── libthermal_states.so

Zip that folder (a bare `.zip` is an accepted import) or copy it into the launcher's native mod
location.

## How it reads thermal state

Android's NDK thermal API (`AThermal_*`), looked up with `dlsym` so the mod still loads on old
devices. Android's seven statuses are folded onto five levels:

| Android | Shown as |
| --- | --- |
| NONE | Nominal |
| LIGHT, MODERATE | Fair |
| SEVERE | Serious |
| CRITICAL, EMERGENCY | Critical |
| SHUTDOWN | Shutdown |

If the API is missing the HUD says "Unavailable" rather than showing a made-up value. The headroom
bar needs Android 12+ and a device that provides a forecast; otherwise it is hidden.

## Not verified

- It has not been compiled or run on a device.
- `pl::modmenu::DrawCommand` field names (`color`, `size`, `text`) and the `RectFilled` enum value
  are taken from the launcher's Java mirror, because the SweetLoader headers are a git submodule.
  If the compiler disagrees, fix them against `include/pl/ModMenu.hpp`.
- Colours are written as ARGB.
- The overlay is submitted from a background thread once per second. If SweetLoader requires
  `submitDrawCommands` on a specific thread, move the call there.

## Reading BerryAPI's governor instead

BerryAPI has a performance governor that reports thermal state and quality knobs, but its C header
(`perf.h` / the governor C ABI) is not in the client repo, so this version does not call it. With
that header, replace `ThermalReader` with a reader over `BERRY_GetGovernorApi()` (struct size 48,
ABI version 1) and keep the HUD code unchanged.
