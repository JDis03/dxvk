# DXVK Fork: Mobile TBDR & ARM Mali Architecture Roadmap

## 1. Contexto y Objetivos

Este fork de **DXVK** (`https://github.com/JDis03/dxvk.git`) tiene como objetivo optimizar la traducción de Direct3D 9/10/11 a Vulkan específicamente para arquitecturas móviles **Tile-Based Deferred Rendering (TBDR)** como **ARM Mali** (Mali-G720, Immortalis, Valhall) y Qualcomm Adreno, cuando se ejecutan bajo entornos como **DroidDeck-Proton**, FEX-Emu y capas de puente Vulkan (MaliBridge).

### El problema en GPUs móviles:
1. **Pérdida de rendimiento por barreras de pipeline**: Las GPUs de PC (IMR) ejecutan comandos de forma inmediata. Las GPUs móviles procesan la pantalla dividida en bloques (tiles en memoria rápida on-chip). Las barreras agresivas de Direct3D obligan a la GPU Mali a vaciar y recargar la memoria de tiles a la RAM del teléfono (*tile writebacks* masivos), hundiendo los FPS.
2. **Tráfico de memoria unificada (LPDDR5X)**: DXVK desktop asigna recursos dinámicos en memoria no cacheada (*write-combining*). En arquitecturas móviles con memoria unificada compartida entre CPU y GPU, esto satura el bus de memoria y dispara el *kswapd* del kernel de Android.
3. **Tirones por compilación síncrona**: Cuando un juego emite una llamada de dibujo con un shader nuevo, el hilo de render se congela hasta que Vulkan termina de compilarlo. En CPUs emuladas (x86 sobre ARM vía FEX), esta pausa dura hasta 1.5 segundos.
4. **Falta de detección de perfiles móviles**: DXVK upstream no detecta controladores móviles (`VK_DRIVER_ID_ARM_PROPRIETARY`), tratando a la GPU móvil como si fuera una RTX de escritorio.

---

## 2. Puntos Clave de Modificación en el Código C++ de DXVK

### A. Detección automática de GPU ARM Mali y Tilers (`src/dxvk/dxvk_adapter.cpp`, `dxvk_instance.cpp` y `dxgi_adapter.cpp`) - [COMPLETADO]
* **Implementado**:
  * Métodos `isArmMali()` e `isTiler()` en `DxvkAdapter`.
  * Vendor IDs `Arm` (0x13b5) y `Qualcomm` (0x5143) añadidos a `DxvkGpuVendor`.
  * Detección automática en `dxvk_instance.cpp` (y soporte de flag de entorno `DXVK_MALI_PROFILE=1`).
  * Enmascaramiento de vendor no clásico en `dxgi_adapter.cpp` reportando fallback AMD RX 6700 XT para evitar cierres en juegos.
  * Quirk `extMultiDraw = false` para driver propietario de ARM Mali en `dxvk_device_info.cpp`.
  * Perfil móvil optimizado en `Config::getArmMaliConfig()`:
    * `d3d11.relaxedBarriers = True`
    * `d3d11.relaxedGraphicsBarriers = True`
    * `d3d11.samplerAnisotropy = 0`
    * `d3d11.maxTessFactor = 8`
    * `d3d11.cachedDynamicResources = a`
    * `dxgi.maxDeviceMemory = 2048`
    * `dxgi.maxSharedMemory = 2048`
    * `dxvk.numCompilerThreads = 2`
    * `dxvk.tilerMode = True`
    * `d3d9.cachedWriteOnlyBuffers = True`
    * `d3d9.allowDirectBufferMapping = False`
    * `d3d9.samplerAnisotropy = 0`
    * `d3d9.maxAvailableMemory = 2048`

### B. Recursos dinámicos en memoria cacheada (`src/util/config/config.cpp`) - [COMPLETADO]
* **Implementado**: Integrado en el perfil automático `Config::getArmMaliConfig()` forzando `d3d11.cachedDynamicResources = a` y `d3d9.cachedWriteOnlyBuffers = True` para evitar retrasos de escritura no cacheada en memoria unificada LPDDR5.

### C. Compilación Asíncrona de Pipelines (`src/dxvk/dxvk_pipemanager.cpp`) - [PENDIENTE]
* **Ubicación**: `src/dxvk/dxvk_pipemanager.cpp` y `src/dxvk/dxvk_graphics.cpp`.
* **Mejora**: Integrar compilación asíncrona no bloqueante (estilo `dxvk-async`): si un pipeline gráfico aún no está listo en el caché de Vulkan, emitir temporalmente un shader dummy o posponer la llamada de dibujo en vez de congelar el hilo principal de renderizado.

### D. Optimización de Render Passes para TBDR (LoadOp / StoreOp DONT_CARE) - [COMPLETADO PARCIALMENTE]
* **Implementado**: En `dxvk_context.cpp` (`finalizeLoadStoreOps`), para arquitecturas tiler (`preferRenderPassOps`), cuando un adjunto (color, profundidad o stencil) tiene un resolve dentro del render pass, demotar `storeOp` a `VK_ATTACHMENT_STORE_OP_DONT_CARE` para evitar volcar gigabytes de datos multisample redundantes a la memoria principal.

---

## 3. Entorno de Compilación

* **Sistema de build**: Meson + Ninja con toolchain cruzado MinGW-w64 (`x86_64-w64-mingw32`).
* **Directorio de compilación existente**: `/home/dark/Project/dxvk/build.w64`.
* **Comandos de compilación**:
  ```bash
  cd /home/dark/Project/dxvk
  ninja -C build.w64
  DESTDIR=/tmp/dxvk-install ninja -C build.w64 install
  ```
  Genera:
  * `d3d11.dll`
  * `dxgi.dll`
  * `d3d9.dll`
  * `d3d10core.dll`
