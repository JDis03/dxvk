# DXVK (ARM Mali & ARM64X Edition)

A Vulkan-based translation layer for Direct3D 8/9/10/11 optimized specifically for **ARM Mali GPUs** (TBDR architecture) and compiled as hybrid **COFF-ARM64X** binaries for Windows-on-ARM, Proton, and Wine on Android (DroidDeck, Termux, Box64, FEX-Emu).

---

## 🚀 Key Improvements in this Fork

### 1. Hybrid COFF-ARM64X Binaries
Standard DXVK produces separate `x64` and `x32` DLLs. On Windows-on-ARM or Wine ARM64, running native ARM64 games and x64 emulated games previously required swapping DLL sets.
- This fork compiles hybrid **COFF-ARM64X** DLLs using LLVM/Clang.
- **A single set of DLLs** (`d3d11.dll`, `dxgi.dll`, `d3d9.dll`, `d3d8.dll`, `d3d10core.dll`) runs **both native ARM64 applications and emulated x64 / ARM64EC applications** without conflict or manual switching.

### 2. ARM Mali (TBDR) Render Pass Optimizations
Tile-Based Deferred Rendering (TBDR) architectures (ARM Mali, Immortalis, Adreno) suffer significant memory bandwidth penalties when resolved render targets are written back to DRAM unnecessarily.
- **Auto-Detection:** Detects ARM Mali GPUs at runtime via Vulkan device properties.
- **Zero Bandwidth Waste:** Sets `storeOp = VK_ATTACHMENT_STORE_OP_DONT_CARE` on multisample render targets that have already been resolved, dramatically reducing thermal throttling and memory bus congestion.

### 3. Smooth Frame Pacing (Fixed Frame Latency Clamping)
- In upstream DXVK, swapchain latency in D3D11 and D3D9 could artificially clamp `GetActualFrameLatency` down to `BufferCount`, causing frame queue starvation and jerky rendering on mobile display compositors (such as Gamescope / Wayland).
- This fork respects the user's `dxgi.maxFrameLatency` setting without clamping to `BufferCount`, producing consistent frame delivery.

### 4. Memory Heap Query Throttling
- Frequent calls to `vkGetPhysicalDeviceMemoryProperties2` inside DXGI created severe kernel context switch storms under Linux proot and Wine.
- Added a 250 ms cache on device memory heap queries, freeing CPU time for game logic and draw submission.

### 5. Async Pipeline Compilation
- Graphics pipeline compilation is decoupled from the main thread (`dxvk.enableAsync`), eliminating shader compilation micro-stutter common on big.LITTLE / heterogeneous mobile CPUs.

---

## 🎮 Tested In-Game Performance

- **Hardware:** Xiaomi 15T (MediaTek Dimensity 9300+, ARM Mali-G720 MC7)
- **Environment:** DroidDeck / DroidDeck-Proton 11-12.1 / Android 15
- **Verified Games:**
  - **World of Warcraft Retail (build 70888):** Native ARM64 client running at a locked **55–60 FPS** at ~600 MB–1 GB RAM consumption.
  - **World of Warcraft: Classic Forever:** Running smoothly at **59.9 FPS** with full touchscreen controller overlays.

---

## 📦 How to Install (For Users)

### Option 1: Automatic / DroidDeck
If you are using DroidDeck on Android:
1. Download the latest release from the [Releases](https://github.com/JDis03/dxvk/releases) page (`dxvk-mali-arm64x-v3.1.0.zip` or `.tar.gz`).
2. Extract the DLLs from the `arm64x/` folder:
   - `d3d11.dll`
   - `dxgi.dll`
   - `d3d9.dll`
   - `d3d10core.dll`
   - `d3d8.dll`
3. Copy them into your Wine/Proton prefix `system32` folder:
   ```bash
   cp arm64x/*.dll <WINEPREFIX>/drive_c/windows/system32/
   ```
4. (Optional) Also copy them into your Proton installation under `files/lib/wine/aarch64-windows/` to make them default for all new prefixes.

### Option 2: Wine / Proton on Android (Manual)
1. Open `winecfg` for your prefix.
2. In the **Libraries** tab, ensure the following DLL overrides are set to **Native, Builtin**:
   - `d3d11`
   - `dxgi`
   - `d3d9`
   - `d3d10core`
   - `d3d8`

---

## ⚙️ Recommended Configuration (`dxvk.conf`)

For optimal performance on ARM Mali devices, create or edit `dxvk.conf` next to your game executable (or set via `DXVK_CONFIG` environment variable):

```ini
# Async shader compilation (reduces shader stutter)
dxvk.enableAsync = true
dxvk.numCompilerThreads = 2

# Frame pacing
dxgi.syncInterval = 0
dxgi.maxFrameLatency = 1

# Barriers & memory optimization for Mali
d3d11.relaxedBarriers = True
d3d11.relaxedGraphicsBarriers = True
d3d11.constantBufferRangeCheck = False
d3d11.zeroInitWorkgroupMemory = False

# Memory caps for mobile RAM envelopes
dxgi.maxDeviceMemory = 2048
dxgi.maxSharedMemory = 2048
```

---

## 🛠️ Build Instructions (For Developers)

### Requirements
- Linux build host (x86_64 or aarch64)
- LLVM / Clang (at least version 17) with ARM64 and ARM64EC target support
- `lld` (LLVM linker) with COFF-ARM64X support
- Meson (>= 0.58) and Ninja
- Windows SDK / MinGW-w64 headers for ARM64

### Compiling ARM64X DLLs
Use the included cross-compilation definition `build-win-arm64ec.txt`:

```bash
git clone --recursive https://github.com/JDis03/dxvk.git
cd dxvk

meson setup --cross-file build-win-arm64ec.txt --buildtype release build.arm64ec
ninja -C build.arm64ec
```

The resulting hybrid binaries will be generated in:
- `build.arm64ec/src/dxgi/dxgi.dll`
- `build.arm64ec/src/d3d11/d3d11.dll`
- `build.arm64ec/src/d3d10/d3d10core.dll`
- `build.arm64ec/src/d3d9/d3d9.dll`
- `build.arm64ec/src/d3d8/d3d8.dll`

Verify that they are true COFF-ARM64X binaries:
```bash
llvm-readobj --coff-load-config build.arm64ec/src/dxgi/dxgi.dll | grep "Format:"
# Output: Format: COFF-ARM64X
```

---

## 📊 HUD & Telemetry
To monitor framerate and GPU metrics in-game, set the environment variable:
```bash
DXVK_HUD=devinfo,fps,frametimes,memory
```
Or for full telemetry:
```bash
DXVK_HUD=full
```

---

## 📄 License
DXVK is licensed under the **zlib License**. See [LICENSE](LICENSE) for details.
Upstream project: [doitsujin/dxvk](https://github.com/doitsujin/dxvk).
