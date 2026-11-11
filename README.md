# DXVK — ARM Mali / tiler edition

**English** · [Español](README.es.md)

A fork of [DXVK](https://github.com/doitsujin/dxvk) (Direct3D 8/9/10/11 → Vulkan) tuned for
**tile-based GPUs, in particular Arm Mali / Immortalis**, and built as hybrid **COFF-ARM64X** DLLs
for Wine / Proton on ARM64 (Android included).

Based on upstream DXVK 3.1. Upstream documentation (wiki, configuration reference) still applies:
<https://github.com/doitsujin/dxvk/wiki>.

> **Scope.** This repository only contains DXVK. How to run Steam, Battle.net or specific games on
> Android with a Mali GPU is documented in
> [JDis03/MaliBridge](https://github.com/JDis03/MaliBridge), which ships DXVK as part of DroidDeck Mali.

---

## What is different from upstream

| # | Change | Where | What it does |
|---|---|---|---|
| 1 | **Mali detection** | `dxvk_adapter.cpp`, `dxvk_instance.cpp` | `DxvkAdapter::isArmMali()` matches the Arm proprietary driver, PanVK, Arm vendor ID `0x13b5`, or a device name containing "Mali". |
| 2 | **Mali config profile** | `config.cpp` (`getArmMaliConfig`) | Applied automatically on Mali (or with `DXVK_MALI_PROFILE=1`). See the table below. |
| 3 | **Tiler mode** | `dxvk_adapter.cpp`, `dxvk_device.cpp` | `isTiler()` is true for Mali, Adreno (Turnip / Qualcomm), Apple, PanVK, V3D, PowerVR and similar. It turns on `preferRenderPassOps` and `preferCachedMemory`. Override with `dxvk.tilerMode`. |
| 4 | **Resolve store-op demotion** | `dxvk_context.cpp` | On tilers, a multisampled attachment that is resolved inside the render pass gets `storeOp = DONT_CARE`, so the raw MSAA tile data is not written back to RAM. |
| 5 | **Asynchronous pipeline compilation** | `dxvk_graphics.cpp`, `dxvk_context.cpp` | A draw whose pipeline is not compiled yet is queued to the compiler workers and skipped instead of stalling the render thread. Only used for render targets already drawn to in previous frames. `dxvk.enableAsync` (default `true`); disable with `DXVK_ASYNC=0`. |
| 6 | **Frame-latency fix** | `d3d11_swapchain.cpp`, `d3d9_swapchain.cpp` | Upstream clamps the latency to `BufferCount`. Games using `DXGI_SWAP_EFFECT_DISCARD` report `BufferCount = 1`, which forced a latency of 1 and serialised CPU and GPU. The configured `maxFrameLatency` cap is now respected. |
| 7 | **Memory-heap query cache** | `dxvk_adapter.cpp` | `getMemoryHeapInfo` caches `vkGetPhysicalDeviceMemoryProperties2` for 250 ms. Games that poll VRAM every frame no longer cost a driver call each time. |
| 8 | **Driver quirks** | `dxvk_device_info.cpp` | `VK_EXT_multi_draw` is disabled on the Arm proprietary driver (as upstream already does for Qualcomm). |
| 9 | **Vendor-ID masking** | `dxgi_adapter.cpp` | A GPU whose vendor ID is not NVIDIA / AMD / Intel is reported to the game as an *AMD Radeon RX 6700 XT*, so games do not reject a mobile GPU. Overridable with `dxgi.customVendorId` / `customDeviceId` / `customDeviceDesc`. |
| 10 | **COFF-ARM64X build** | `build-win-arm64ec.txt` | One DLL set that works for both native ARM64 and ARM64EC/x64-emulated processes. |

### The Mali profile

These are the values `getArmMaliConfig()` sets. They are **defaults**: anything in your `dxvk.conf`
or in a per-application config wins, because the profile is merged without overwriting existing keys.

| Option | Value | Reason |
|---|---|---|
| `d3d11.relaxedBarriers` / `relaxedGraphicsBarriers` | `True` | fewer pipeline flushes, fewer tile write-backs |
| `d3d11.cachedDynamicResources` | `a` (all) | dynamic buffers in cached memory on unified-memory SoCs |
| `d3d9.cachedWriteOnlyBuffers` | `True` | same, for D3D9 |
| `d3d9.allowDirectBufferMapping` | `False` | |
| `d3d11.constantBufferRangeCheck` | `False` | cheaper shaders |
| `d3d11.zeroInitWorkgroupMemory` | `False` | cheaper compute |
| `d3d11.samplerAnisotropy` / `d3d9.samplerAnisotropy` | `0` | **forces anisotropic filtering off** |
| `d3d11.maxTessFactor` | `1` | **effectively disables tessellation** |
| `d3d11.disableMsaa` | `True` | **disables MSAA** |
| `dxgi.syncInterval` | `0` | present is not throttled by vsync interval |
| `dxgi.maxFrameLatency` | `2` | |
| `dxgi.maxDeviceMemory` / `maxSharedMemory` / `d3d9.maxAvailableMemory` | `2048` | the memory budget reported to the game is capped at 2 GiB |
| `dxvk.numCompilerThreads` | `2` | leaves the big cores for the game |
| `dxvk.enableAsync` / `dxvk.tilerMode` | `true` / `True` | |

The rows in bold trade image quality for speed. If a game looks wrong
(missing geometry, no AA, blurry textures), override the value, for example:

```ini
d3d11.maxTessFactor = 0        # use the game's own tessellation factors
d3d11.disableMsaa = False
d3d11.samplerAnisotropy = 16
```

When the profile is applied, DXVK logs `ARM Mali / TBDR mobile architecture detected` followed by the
effective values, so you can confirm it in the `*_d3d11.log` file.

### Other GPUs

- **Desktop GPUs** (NVIDIA, AMD, Intel): the Mali profile and tiler mode stay off. `dxvk.enableAsync`
  is `true` by default for every GPU in this fork; set `dxvk.enableAsync = False` (or `DXVK_ASYNC=0`)
  to get upstream behaviour. The frame-latency fix applies everywhere.
- **Adreno / other tilers**: tiler mode (items 3 and 4) is enabled automatically, but the full Mali
  profile is **not**. You can opt in with `DXVK_MALI_PROFILE=1` if you want to try it.

---

## Installing

Download `dxvk-mali-arm64x-v3.1.0.zip` or `.tar.gz` from the
[Releases](https://github.com/JDis03/dxvk/releases) page. Inside there is an `arm64x/` folder with
`d3d8.dll`, `d3d9.dll`, `d3d10core.dll`, `d3d11.dll` and `dxgi.dll`.

```bash
export WINEPREFIX=/path/to/prefix
cp arm64x/*.dll "$WINEPREFIX/drive_c/windows/system32/"
```

Then set `native,builtin` DLL overrides for `d3d8 d3d9 d3d10core d3d11 dxgi`
(`winecfg` → Libraries, or `WINEDLLOVERRIDES="d3d8,d3d9,d3d10core,d3d11,dxgi=n,b"`).

These DLLs are **ARM64X**, so they are meant for Wine/Proton builds for ARM64 (they also load in
ARM64EC / x64 processes). They are **not** for x86-64 Wine on a PC: use upstream DXVK there.

If you use [DroidDeck Mali / MaliBridge](https://github.com/JDis03/MaliBridge) you do not install
anything by hand.

### Checking that it is active

```bash
DXVK_HUD=devinfo,fps,frametimes DXVK_LOG_LEVEL=info  wine game.exe
```

`devinfo` shows the GPU and driver; the log file contains the "ARM Mali / TBDR" line and the
effective options.

### Environment variables added by this fork

| Variable | Effect |
|---|---|
| `DXVK_MALI_PROFILE=1` | Force the Mali profile on any GPU |
| `DXVK_ASYNC=0` | Disable asynchronous pipeline compilation |

### Async compilation caveat

While a pipeline compiles in the background, that draw is skipped. In practice this can show up as
a few frames of missing geometry or effects the first time something appears. It trades a visible
freeze for a brief omission. Turn it off with `DXVK_ASYNC=0` if a game misbehaves.

---

## Building

The provided cross file, `build-win-arm64ec.txt`, calls `arm64x-clang`, `arm64x-clang++`, `arm64x-ar`,
`arm64x-strip` and `arm64x-windres`. These are **small wrapper scripts that are not in this
repository**: each one runs llvm-mingw's `clang` with `-target aarch64-w64-mingw32`, a mingw sysroot
and `-marm64x`. The paths in the file point at the author's machine, so **edit them** before use.

You need:

- llvm-mingw / LLVM Clang (22.x was used) able to produce `-marm64x` output, with `lld`
- Meson ≥ 0.58 and Ninja
- an aarch64 mingw-w64 sysroot

```bash
git clone --recursive https://github.com/JDis03/dxvk.git
cd dxvk
meson setup --cross-file build-win-arm64ec.txt --buildtype release build.arm64ec
ninja -C build.arm64ec
llvm-strip --strip-debug build.arm64ec/src/*/*.dll    # optional, smaller files
```

Output: `build.arm64ec/src/{dxgi,d3d11,d3d10,d3d9,d3d8}/*.dll`. Check the format:

```bash
llvm-readobj --coff-load-config build.arm64ec/src/dxgi/dxgi.dll | grep Format
# Format: COFF-ARM64X
```

`build-win-arm64.txt` builds plain ARM64 (non-hybrid) DLLs with llvm-mingw.
The standard upstream targets (`build-win64.txt`, `build-win32.txt`) are unchanged.

The ARM64X toolchain compiles with many warnings from the libc++ headers; they are harmless.

---

## Status and testing

Built and run on a Xiaomi 15T (Dimensity 8400 / Mali-G720 MC7, Android 16) under Wine/Proton with a
Vulkan translation layer for Mali. Desktop GPUs and upstream conformance have not been re-tested
for this fork, so treat it as experimental outside Mali.

Design notes and the optimisation roadmap are in [`MALI_TBDR_ROADMAP.md`](MALI_TBDR_ROADMAP.md)
(Spanish).

## License

zlib, as upstream. See [LICENSE](LICENSE). Upstream project: <https://github.com/doitsujin/dxvk>.
