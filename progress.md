## 2026-11-11 07:35 — dxvk
**Summary**: Contexto guardado: Wow.exe x64 70862 del PC copiado al teléfono (md5 f9313e2a verificado), ntdll.dll de Proton y prefijo restaurados a originales (md5 82967168). Pendiente probar el juego y, si falla por versión, conseguir x64 70888.
**Verified**: Se verificaron por adb y md5: Wow.exe del teléfono idéntico al del PC, y ntdll.dll de Proton y del prefijo idénticos al original. El juego no se lanzó tras estos cambios, así que no está verificado que arranque.
**Completed**: none
---
## 2026-11-11 07:29 — dxvk
**Summary**: Battle.net login arreglado (logs de depuración apagados, IPv4, RAM liberada); WoW registrado tras corregir .product.db y actualizado a 70888. Wow-ARM64.exe probado con parche de ntdll: avanza pero se cuelga en excepciones ILLEGAL_INSTRUCTION bajo proot. Wow.exe actual es copia del ARM; respaldo x64 70755 en _forever_.
**Verified**: Verificado por adb en el teléfono: logs de Agent, .build.info 70888, Wow-ARM64.exe machine 0xaa64 build 70888, excepciones en log de Wine, ntdll.dll parcheado md5 c6463579 en Proton y prefijo. El cliente ARM NO llegó a mostrar ventana; no se verificó el juego en funcionamiento.
**Completed**: none
---
## 2026-11-09 15:54 — dxvk
**Summary**: Corregido bloqueo de paralelismo CPU-GPU en D3D11SwapChain: GetActualFrameLatency respetando dxgi.maxFrameLatency=2 sin clampeo por BufferCount=1.
**Verified**: Compilacion ARM64X limpia con ninja en build.arm64ec, commit b3f09b27 empujado a github.com:JDis03/dxvk.git. DLLs desplegadas a todos los prefijos en Xiaomi 15T con MD5 45a4d90c0ba3711e5fb88a8af0123b3c. WTF y HUD actualizados.
**Completed**: none
---
## 2026-11-07 11:54 — dxvk
**Summary**: Configuradas optimizaciones de IPC de sincronizacion en MaliBridge y desactivado MSAA en DXVK para escenas 3D.
**Verified**: Librerias compiladas y sincronizadas con git ffd262ca y desplegadas en 192.168.50.4. Entorno de MaliBridge y WTF actualizados.
**Completed**: none
---
## 2026-11-07 10:19 — dxvk
**Summary**: Identificado cuello de botella de CPU core 0 en hilo principal y submit, configurado aislamiento de nucleos de alto rendimiento y distancia de dibujado.
**Verified**: Comprobado en /proc que hilos criticos corrian en core 0. Configurados WTF y game-environment.json.
**Completed**: none
---
## 2026-11-07 10:05 — dxvk
**Summary**: Verificado en vivo el rendimiento de la sesion 10: tasa de cuadros duplicada de 28 a 58.8 FPS estables en Xiaomi 15T.
**Verified**: Telemetria de MaliBridge (vkbridge-frames-11203.txt) reporta fps_p50=58.8 y ft_p50=17ms con 57 segundos continuos entre 45-60 FPS.
**Completed**: none
---
## 2026-11-07 09:59 — dxvk
**Summary**: Corregido despliegue al prefijo real 2951205796, habilitado maxFrameLatency=2 para solapamiento TBDR y corregido limite de teselacion a 1.
**Verified**: MD5sums verificados ac8cacdc9f08fe77682b29021b6eb067 en todos los prefijos activos en Xiaomi 15T. Commit d735902e en master.
**Completed**: none
---
## 2026-11-07 09:37 — dxvk
**Summary**: Identificado y eliminado cuello de botella critico de vkGetPhysicalDeviceMemoryProperties2 en DXGI (22% de tiempo de CPU) y desplegado en Xiaomi 15T.
**Verified**: Librerias compiladas y verificadas con llvm-strip y desplegadas a DroidDeck-Proton en 192.168.50.4. Codigo sincronizado en git dca68fcf.
**Completed**: none
---
## 2026-11-07 08:58 — dxvk
**Summary**: Desplegadas librerias ARM64X con compilacion asincrona en el Xiaomi 15T (192.168.50.4) y configurado dxvk.conf para deslimitar FPS.
**Verified**: Librerias copiadas con exito via ADB Wi-Fi a lib/wine/dxvk/aarch64-windows y drive_c/windows/system32. dxvk.conf actualizado con syncInterval=0 y enableAsync=true.
**Completed**: none
---
## 2026-11-07 08:44 — dxvk
**Summary**: Implementada compilacion asincrona de pipelines graficos (dxvk.enableAsync) y soporte de build nativo ARM64X.
**Verified**: Compilacion nativa de build.arm64ec ejecutada con exito usando ninja produciendo dxgi.dll, d3d11.dll, d3d10core.dll, d3d9.dll y d3d8.dll en formato coff-arm64x. Verificado con llvm-objdump -p y llvm-strip sin errores.
**Completed**: none
---
## 2026-11-07 06:34 — dxvk
**Summary**: Implementada auto-deteccion y perfil optimizado TBDR para ARM Mali en DXVK y optimizacion de StoreOp DONT_CARE en renders passes resueltos.
**Verified**: Compilacion MinGW-w64 de 64 bits ejecutada con exito usando ninja -C build.w64 con 0 errores y 0 warnings. Instalacion verificada con DESTDIR=/tmp/dxvk-install ninja -C build.w64 install generando d3d11.dll, dxgi.dll, d3d9.dll, d3d8.dll y d3d10core.dll.
**Completed**: none
---
## 2026-11-07 06:35 — dxvk
**Summary**: Implementada auto-detección y perfil optimizado de ARM Mali / TBDR en DxvkAdapter, DxvkInstance, DxgiAdapter y DxvkDeviceInfo. Optimizada gestión de RenderPass StoreOp DONT_CARE en DxvkContext para adjuntos con resolve en tilers.
**Verified**: Compilación completa de 64 bits con MinGW-w64 (`ninja -C build.w64`) completada con 0 errores y 0 advertencias. Instalación verificada con `DESTDIR=/tmp/dxvk-install ninja -C build.w64 install` generando d3d11.dll, dxgi.dll, d3d9.dll, d3d8.dll y d3d10core.dll en /tmp/dxvk-install/tmp/dxvk-install/bin.
**Completed**: 1246, 1248
---
## 2026-11-07 05:59 — dxvk
**Summary**: Contexto y roadmap inicial para optimizacion de DXVK en arquitecturas moviles TBDR y ARM Mali creado con exito.
**Verified**: Creado documento MALI_TBDR_ROADMAP.md en /home/dark/Project/dxvk. Registrada decision tecnica y 3 TODOs prioritarios en el servidor de memoria. El entorno de build en build.w64 verificado listo para compilar con ninja.
**Completed**: none
---
## 2026-09-12 02:36 — dxvk
**Summary**: Sesión enfocada en contribuir a DXVK: se configuró el entorno de compilación (meson, submodules incluyendo el anidado subprojects/dxbc-spirv/submodules/spirv_headers) y se logró un build exitoso de DXVK 64-bit. Luego la conversación pivotó a diagnosticar problemas del entorno de desarrollo Unreal Engine del usuario en Linux: se investigó un crash de nvidia-open-dkms, se configuró un kernel alternativo (linux estándar además de linux-zen) generando un UKI adicional y regenerando grub.cfg para dual-boot de kernels. Se diagnosticó que los crashes reales ocurrían con Sunshine/Moonlight (streaming GPU) y que el problema de "mouse loco" en RustDesk durante Play-In-Editor de Unreal se debe a que RustDesk bloquea deliberadamente su función 'relative mouse mode' cuando el host remoto reporta platform==Linux (ver flutter/lib/models/relative_mouse_model.dart, isSupported), sin distinguir X11 de Wayland. Se decidió forkear rustdesk/rustdesk (fork creado: JDis03/rustdesk) y clonarlo en /home/dark/Project/rustdesk para intentar relajar esa restricción para hosts X11 como contribución real de código abierto a la compatibilidad Linux gaming.
**Verified**: Verificado: build DXVK 64-bit completado con ninja install generando d3d8/d3d9/d3d10core/d3d11/dxgi.dll en /tmp/dxvk-install/bin. Verificado: nuevo UKI arch-linux.efi generado con mkinitcpio y grub.cfg regenerado con grub-mkconfig. Verificado: servicio systemd rustdesk reiniciado (nuevo PID). Verificado: fork JDis03/rustdesk creado vía gh y clonado exitosamente en /home/dark/Project/rustdesk con remotes origin/upstream configurados.
**Completed**: none
