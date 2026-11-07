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
