# DXVK — edición ARM Mali / tilers

[English](README.md) · **Español**

Fork de [DXVK](https://github.com/doitsujin/dxvk) (Direct3D 8/9/10/11 → Vulkan) ajustado para
**GPU de renderizado por tiles, en especial Arm Mali / Immortalis**, y compilado como DLL híbridas
**COFF-ARM64X** para Wine / Proton en ARM64 (Android incluido).

Basado en DXVK 3.1 de upstream. La documentación de upstream (wiki, referencia de configuración)
sigue valiendo: <https://github.com/doitsujin/dxvk/wiki>.

> **Alcance.** Este repositorio solo contiene DXVK. Cómo ejecutar Steam, Battle.net o juegos
> concretos en Android con GPU Mali está en
> [JDis03/MaliBridge](https://github.com/JDis03/MaliBridge), que incluye DXVK dentro de DroidDeck Mali.

---

## Qué cambia respecto a upstream

| # | Cambio | Dónde | Qué hace |
|---|---|---|---|
| 1 | **Detección de Mali** | `dxvk_adapter.cpp`, `dxvk_instance.cpp` | `DxvkAdapter::isArmMali()` reconoce el driver propietario de Arm, PanVK, el vendor ID `0x13b5` o un nombre de dispositivo con "Mali". |
| 2 | **Perfil de configuración Mali** | `config.cpp` (`getArmMaliConfig`) | Se aplica solo en Mali (o con `DXVK_MALI_PROFILE=1`). Ver tabla más abajo. |
| 3 | **Modo tiler** | `dxvk_adapter.cpp`, `dxvk_device.cpp` | `isTiler()` es verdadero en Mali, Adreno (Turnip / Qualcomm), Apple, PanVK, V3D, PowerVR y similares. Activa `preferRenderPassOps` y `preferCachedMemory`. Se puede forzar con `dxvk.tilerMode`. |
| 4 | **`storeOp` en resolve** | `dxvk_context.cpp` | En tilers, un adjunto multisample que se resuelve dentro del render pass usa `storeOp = DONT_CARE`, así no se escribe a RAM el MSAA sin resolver. |
| 5 | **Compilación asíncrona de pipelines** | `dxvk_graphics.cpp`, `dxvk_context.cpp` | Un draw cuyo pipeline aún no está compilado se encola a los hilos de compilación y se omite, en vez de bloquear el hilo de render. Solo con render targets ya usados en frames anteriores. `dxvk.enableAsync` (por defecto `true`); se desactiva con `DXVK_ASYNC=0`. |
| 6 | **Corrección de latencia de frames** | `d3d11_swapchain.cpp`, `d3d9_swapchain.cpp` | Upstream limita la latencia a `BufferCount`. Los juegos con `DXGI_SWAP_EFFECT_DISCARD` declaran `BufferCount = 1`, lo que forzaba latencia 1 y serializaba CPU y GPU. Ahora se respeta el `maxFrameLatency` configurado. |
| 7 | **Caché de consultas de memoria** | `dxvk_adapter.cpp` | `getMemoryHeapInfo` guarda 250 ms el resultado de `vkGetPhysicalDeviceMemoryProperties2`. Los juegos que consultan la VRAM cada frame ya no hacen una llamada al driver cada vez. |
| 8 | **Parche de driver** | `dxvk_device_info.cpp` | `VK_EXT_multi_draw` se desactiva en el driver propietario de Arm (upstream ya lo hace con Qualcomm). |
| 9 | **Enmascarar el vendor ID** | `dxgi_adapter.cpp` | Una GPU cuyo vendor ID no es NVIDIA / AMD / Intel se reporta al juego como *AMD Radeon RX 6700 XT*, para que los juegos no rechacen una GPU móvil. Se puede cambiar con `dxgi.customVendorId` / `customDeviceId` / `customDeviceDesc`. |
| 10 | **Compilación COFF-ARM64X** | `build-win-arm64ec.txt` | Un solo juego de DLL válido para procesos ARM64 nativos y ARM64EC/x64 emulados. |

### El perfil Mali

Estos son los valores que establece `getArmMaliConfig()`. Son **valores por defecto**: lo que pongas
en tu `dxvk.conf` o en la configuración por aplicación tiene prioridad, porque el perfil se mezcla
sin sobrescribir claves existentes.

| Opción | Valor | Motivo |
|---|---|---|
| `d3d11.relaxedBarriers` / `relaxedGraphicsBarriers` | `True` | menos vaciados de pipeline y de tiles |
| `d3d11.cachedDynamicResources` | `a` (todos) | buffers dinámicos en memoria cacheada en SoC con memoria unificada |
| `d3d9.cachedWriteOnlyBuffers` | `True` | lo mismo para D3D9 |
| `d3d9.allowDirectBufferMapping` | `False` | |
| `d3d11.constantBufferRangeCheck` | `False` | shaders más baratos |
| `d3d11.zeroInitWorkgroupMemory` | `False` | compute más barato |
| `d3d11.samplerAnisotropy` / `d3d9.samplerAnisotropy` | `0` | **desactiva el filtrado anisotrópico** |
| `d3d11.maxTessFactor` | `1` | **desactiva en la práctica la teselación** |
| `d3d11.disableMsaa` | `True` | **desactiva MSAA** |
| `dxgi.syncInterval` | `0` | el present no se frena por el intervalo de vsync |
| `dxgi.maxFrameLatency` | `2` | |
| `dxgi.maxDeviceMemory` / `maxSharedMemory` / `d3d9.maxAvailableMemory` | `2048` | la memoria que se reporta al juego se limita a 2 GiB |
| `dxvk.numCompilerThreads` | `2` | deja libres los núcleos grandes para el juego |
| `dxvk.enableAsync` / `dxvk.tilerMode` | `true` / `True` | |

Las filas en negrita cambian calidad de imagen por velocidad. Si un juego se ve mal
(geometría que falta, sin antialiasing, texturas borrosas), sobrescribe el valor:

```ini
d3d11.maxTessFactor = 0        # usa los factores de teselación del propio juego
d3d11.disableMsaa = False
d3d11.samplerAnisotropy = 16
```

Cuando se aplica el perfil, DXVK escribe en el log `ARM Mali / TBDR mobile architecture detected`
seguido de los valores efectivos; así lo compruebas en el archivo `*_d3d11.log`.

### Otras GPU

- **GPU de escritorio** (NVIDIA, AMD, Intel): el perfil Mali y el modo tiler quedan desactivados.
  En este fork `dxvk.enableAsync` es `true` por defecto para todas las GPU; pon
  `dxvk.enableAsync = False` (o `DXVK_ASYNC=0`) para volver al comportamiento de upstream.
  La corrección de latencia de frames se aplica en todas.
- **Adreno y otros tilers**: el modo tiler (puntos 3 y 4) se activa solo, pero el perfil Mali
  completo **no**. Puedes probarlo con `DXVK_MALI_PROFILE=1`.

---

## Instalación

Descarga `dxvk-mali-arm64x-v3.1.0.zip` o `.tar.gz` desde
[Releases](https://github.com/JDis03/dxvk/releases). Dentro hay una carpeta `arm64x/` con
`d3d8.dll`, `d3d9.dll`, `d3d10core.dll`, `d3d11.dll` y `dxgi.dll`.

```bash
export WINEPREFIX=/ruta/al/prefijo
cp arm64x/*.dll "$WINEPREFIX/drive_c/windows/system32/"
```

Después configura los reemplazos `native,builtin` para `d3d8 d3d9 d3d10core d3d11 dxgi`
(`winecfg` → Bibliotecas, o `WINEDLLOVERRIDES="d3d8,d3d9,d3d10core,d3d11,dxgi=n,b"`).

Estas DLL son **ARM64X**, así que sirven para Wine/Proton compilados para ARM64 (también cargan en
procesos ARM64EC / x64). **No** sirven para Wine x86-64 en un PC: allí usa DXVK de upstream.

Si usas [DroidDeck Mali / MaliBridge](https://github.com/JDis03/MaliBridge) no instalas nada a mano.

### Comprobar que está activo

```bash
DXVK_HUD=devinfo,fps,frametimes DXVK_LOG_LEVEL=info  wine juego.exe
```

`devinfo` muestra la GPU y el driver; el log contiene la línea "ARM Mali / TBDR" y las opciones efectivas.

### Variables de entorno añadidas por este fork

| Variable | Efecto |
|---|---|
| `DXVK_MALI_PROFILE=1` | Fuerza el perfil Mali en cualquier GPU |
| `DXVK_ASYNC=0` | Desactiva la compilación asíncrona de pipelines |

### Aviso sobre la compilación asíncrona

Mientras un pipeline se compila en segundo plano, ese draw se omite. En la práctica puede verse como
unos frames sin geometría o efectos la primera vez que algo aparece. Cambia un congelamiento visible
por una omisión breve. Desactívala con `DXVK_ASYNC=0` si un juego falla.

---

## Compilar

El archivo `build-win-arm64ec.txt` llama a `arm64x-clang`, `arm64x-clang++`, `arm64x-ar`,
`arm64x-strip` y `arm64x-windres`. Son **scripts envoltorio pequeños que no están en este
repositorio**: cada uno ejecuta el `clang` de llvm-mingw con `-target aarch64-w64-mingw32`, un
sysroot de mingw y `-marm64x`. Las rutas del archivo apuntan a la máquina del autor, así que
**edítalas** antes de usarlo.

Necesitas:

- llvm-mingw / LLVM Clang (se usó 22.x) capaz de generar `-marm64x`, con `lld`
- Meson ≥ 0.58 y Ninja
- un sysroot de mingw-w64 para aarch64

```bash
git clone --recursive https://github.com/JDis03/dxvk.git
cd dxvk
meson setup --cross-file build-win-arm64ec.txt --buildtype release build.arm64ec
ninja -C build.arm64ec
llvm-strip --strip-debug build.arm64ec/src/*/*.dll    # opcional, archivos más pequeños
```

Salida: `build.arm64ec/src/{dxgi,d3d11,d3d10,d3d9,d3d8}/*.dll`. Comprueba el formato:

```bash
llvm-readobj --coff-load-config build.arm64ec/src/dxgi/dxgi.dll | grep Format
# Format: COFF-ARM64X
```

`build-win-arm64.txt` compila DLL ARM64 simples (no híbridas) con llvm-mingw.
Los destinos estándar de upstream (`build-win64.txt`, `build-win32.txt`) no se tocaron.

La toolchain ARM64X emite muchos avisos de las cabeceras de libc++; son inofensivos.

---

## Estado y pruebas

Compilado y probado en un Xiaomi 15T (Dimensity 8400 / Mali-G720 MC7, Android 16) con Wine/Proton y
una capa de traducción Vulkan para Mali. No se han vuelto a probar GPU de escritorio ni la
conformidad con upstream en este fork, así que considéralo experimental fuera de Mali.

Las notas de diseño y la hoja de ruta están en [`MALI_TBDR_ROADMAP.md`](MALI_TBDR_ROADMAP.md).

## Licencia

zlib, igual que upstream. Ver [LICENSE](LICENSE). Proyecto original: <https://github.com/doitsujin/dxvk>.
