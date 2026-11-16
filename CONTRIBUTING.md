# Contributing to DXVK (ARM Mali / tiler fork)

Thank you for helping improve DXVK on ARM Mali / Immortalis and other tile-based (TBDR) GPUs!

## Scope

This repository is strictly for **DXVK** (D3D8/9/10/11 → Vulkan, Mali/tiler tuning, and COFF-ARM64X build support). Documentation or scripts for Android launchers, Steam/Proton packaging, or specific game setups belong in [JDis03/MaliBridge](https://github.com/JDis03/MaliBridge) or downstream projects, not here.

## Repository layout & remotes

```bash
git clone --recursive https://github.com/JDis03/dxvk.git
cd dxvk
git remote add upstream https://github.com/doitsujin/dxvk.git
git fetch upstream --tags
```

- `master`: Fork branch based on upstream DXVK `v3.1.1` with Mali/tiler patches and ARM64X support.
- `upstream`: `doitsujin/dxvk` (keep this fork easy to rebase/merge and easy to cherry-pick from).

## Guidelines for pull requests

1. **One change per commit / PR.** Keep unrelated refactors, formatting changes, or personal build/log files out of feature commits so individual fixes can be reviewed and cherry-picked cleanly to `doitsujin/dxvk`.
2. **Do not regress desktop GPUs or upstream conformance.** Gate mobile/TBDR behavior behind `DxvkAdapter::isArmMali()`, `DxvkAdapter::isTiler()`, or `DxvkDevicePerfHints` (`tilerMode`), leaving NVIDIA, AMD, and Intel desktop paths untouched unless the fix is genuinely universal (such as `maxFrameLatency` clamping).
3. **Verify both targets.** Changes should build cleanly for both ARM64X (`build-win-arm64ec.txt`) and standard x86-64 (`build-win64.txt`) without introducing errors.
4. **Measure on real hardware.** Performance claims in commit messages or PRs must include real before/after measurements on target hardware (e.g., `DXVK_LOG_LEVEL=info`, `DXVK_HUD=devinfo,fps,frametimes`, GPU profiler counters, and exact resolution/scene).

## Upstreaming to `doitsujin/dxvk`

To keep contributing back to upstream DXVK straightforward, changes in this fork fall into two buckets:

### 1. Upstream-candidate patches (cherry-pickable onto `upstream/master`)
These fixes are self-contained and gated safely for tilers or general correctness:
- `75a26c6f` — Render pass `storeOp = DONT_CARE` for resolved multisample attachments on tilers (`preferRenderPassOps`).
- `dca68fcf` — Throttle `vkGetPhysicalDeviceMemoryProperties2` calls in `getMemoryHeapInfo` (250 ms cache).
- `b3f09b27` — Respect `maxFrameLatency` cap without clamping to `BufferCount = 1` in D3D11/D3D9 swapchains.
- `c54299bb` — `preferTransferClears`: use `vkCmdClearColorImage` for whole storage-only color subresource clears on tilers.

To prepare an upstream PR from one of these commits:

```bash
git fetch upstream
git checkout -b upstream-pr/<topic> upstream/master
git cherry-pick <commit-sha>
# Build & verify against upstream/master before opening the PR
```

### 2. Fork-specific defaults & build targets
These stay in `JDis03/dxvk` unless upstream requests them:
- `getArmMaliConfig()` automatic profile in `src/util/config/config.cpp` (`isArmMali()`).
- `dxvk.enableAsync = true` default and `DXVK_MALI_PROFILE` / `DXVK_ASYNC` env overrides.
- Non-desktop vendor ID masking (`AMD Radeon RX 6700 XT` fallback in `dxgi_adapter.cpp`).
- `build-win-arm64ec.txt` / `build-win-arm64.txt` llvm-mingw cross files.
