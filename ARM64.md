# ARM64 Status — SeedMapper

This is a fork of [`xpple/SeedMapper`](https://github.com/xpple/SeedMapper) by
**Frederik van der Els (xpple)** — an in-game Minecraft Fabric mod that lets you
do various things with the world seed (locate structures, biomes, loot, render
seed maps, …).

- **License:** LGPL-3.0 (inherited from upstream; `LICENSE` and the cubiomes
  `LICENSE` under `src/main/c/cubiomes/` are preserved).
- **Credit:** the mod and its cubiomes core (submodule `src/main/c/cubiomes`)
  belong entirely to xpple.

## Good news: SeedMapper already supports ARM64

Unlike NetherBedrockCracker, SeedMapper's `master` **already builds and ships
ARM64 natives** — no code changes were required:

- **Runtime loader** (`src/main/java/dev/xpple/seedmapper/SeedMapper.java`)
  already detects `os.arch` and selects `cubiomes_x86` / `cubiomes_arm` (and the
  universal `libcubiomes.dylib` on macOS).
- **CI** (`.github/workflows/build.yml`, `compile-cubiomes` job) already
  cross-compiles:
  - Linux: `libcubiomes_arm.so` via `gcc-aarch64-linux-gnu` (+ `libcubiomes_x86.so`),
  - macOS: a universal `libcubiomes.dylib` (`arm64;x86_64`),
  - Windows: `cubiomes_arm.dll` via MinGW (+ `cubiomes_x86.dll`).

This fork therefore requires **no ARM64 code changes**; it exists to give you a
self-controlled copy that builds ARM64 artifacts through GitHub Actions, with the
verification recorded below.

## Native verification (aarch64, performed locally)

- Built `libcubiomes_arm.so` natively on aarch64 (cmake):
  `ELF64, Machine: AArch64`; exports `getBiomeAt`, `getStructurePos`,
  `setupGenerator`, `locateBiome`, `getStructureConfig_default`, …
- cubiomes official test suite on aarch64 — `test_biomes`, `test_structures`,
  `test_loot`, `test_pieces`, `test_carvers` → **5/5 PASS** (biome / structure /
  loot generation matches the embedded reference values).

## Build

Native library (aarch64 Linux):
```bash
cd src/main/c/cubiomes
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DINFER_POSTFIX=ON
cmake --build build -j            # -> build/libcubiomes_arm.so
```
GitHub Actions builds the full mod jar on push (JDK 25 / Minecraft 26.3,
downloads the cubiomes natives into `src/main/resources/`).

## Optional follow-up

The cubiomes CMake now supports `INFER_POSTFIX=ON` (auto-suffixes `_x86` / `_arm`
from `CMAKE_SYSTEM_PROCESSOR`); the CI still passes explicit postfixes. Simplifying
to `INFER_POSTFIX=ON` is a cosmetic cleanup best left for upstream.
