# ARM64 Port — SeedMapper (Minecraft 1.21.11 / mod v2.21.2)

ARM64 (aarch64) build fork of [`xpple/SeedMapper`](https://github.com/xpple/SeedMapper)
(tag `v2.21.2`, Minecraft **1.21.11**), by Frederik van der Els (xpple).

- License: **LGPL-3.0** (inherited; `LICENSE` + `src/main/c/cubiomes/LICENSE` preserved).
- This branch (`mc1.21.11-arm64`) backports ARM64 support to the 1.21.11 line.

## The ARM64 gap (present in v2.21.2)

Unlike newer 26.x releases, **SeedMapper v2.21.2 did NOT yet support ARM64** for
Linux/Windows:
- Loader used `System.mapLibraryName("cubiomes")` with **no architecture detection**.
- CI compiled a single x86-64 `libcubiomes.so` / `cubiomes.dll` (only macOS was a
  universal `arm64;x86_64` dylib).

So on aarch64 Linux/Windows the mod failed to load the native. (ARM64 support was
only added upstream in the 26.x line.)

## Changes

1. **Loader** (`SeedMapper.java`): added `os.arch` detection — macOS keeps the
   universal `libcubiomes.dylib`; Linux/Windows select `cubiomes_x86` / `cubiomes_arm`.
2. **CI** (`.github/workflows/build.yml` `compile-cubiomes` job): cross-compile
   aarch64 Linux → `libcubiomes_arm.so` (`gcc-aarch64-linux-gnu`) and aarch64
   Windows → `cubiomes_arm.dll` (MinGW cross, best-effort); `fail-fast: false`;
   macOS universal unchanged. Outputs use the `_x86`/`_arm` rename approach
   (independent of the cubiomes CMake `RELEASE_POSTFIX` feature).
3. `ARM64.md` (this doc).

## Verified (aarch64, local)

- `libcubiomes.so` built natively: `ELF64, Machine: AArch64`, exports
  `getBiomeAt`/`getStructurePos`/`setupGenerator`/`locateBiome`.
- Loader resolves to `libcubiomes_arm.so` on an aarch64 JVM.
- cubiomes official test suite (biomes/structures/loot/pieces/carvers) → PASS.

## Build

GitHub Actions (`.github/workflows/build.yml`) on push to `mc1.21.11-arm64`:
`compile-cubiomes` (native libs, all platforms) + `build` (mod jar, JDK 23 / MC 1.21.11).
