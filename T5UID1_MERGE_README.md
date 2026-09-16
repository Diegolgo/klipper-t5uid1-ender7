# Klipper mainline + soporte DGUS T5UID1 (fork de Desuuuu)

Este repositorio es Klipper mainline actualizado, con el soporte de
pantallas DGUS T5UID1 (proyecto `Desuuuu/klipper`, rama `dgus-reloaded`)
fusionado encima.

## Origen

- Base: Klipper mainline, commit `72b3cdb4e` (2026-08-31)
- Fork fusionado: `Desuuuu/klipper`, rama `dgus-reloaded`, último commit
  `baae7f3a7` (2023-05-26) — el fork está archivado y no recibe updates
  desde esa fecha.
- Punto de divergencia común: `f7e29b276` (2022-08-24)

## Qué se hizo

1. `git merge` directo del último commit del fork sobre el mainline
   actual (commit `1644ada86`).
2. Conflictos reales solo en `src/atsamd/*` y `src/lpc176x/Kconfig`
   (mainline reestructuró las opciones de interfaz de comunicación
   para esas arquitecturas después de 2022). Se resolvieron tomando
   la versión de mainline **sin** el soporte T5UID1 para esas dos
   arquitecturas — no se usan en la MKS Robin Nano V3.1.
3. **`src/stm32/*` se fusionó sin ningún conflicto** — es la
   arquitectura relevante para este proyecto (STM32F407).
4. Fix de compatibilidad en `klippy/extras/t5uid1/t5uid1.py`
   (commit `f68e69037`): la API de `MCU` en klippy cambió desde 2023:
   - `mcu.register_response(cb, msg)` → ya no existe.
   - Reemplazado por `mcu.register_serial_response(cb, msg_format, oid=None)`,
     que además requiere el string de formato completo del mensaje
     (`"t5uid1_received command=%c data=%*s"`), no solo el nombre.

## Qué se validó

- `make menuconfig` + `make` compila limpio para:
  - `MACH_STM32F407`
  - Bootloader offset 48KiB (`STM32_FLASH_START_C000`)
  - Comunicación USB (`STM32_USB_PA11_PA12`)
  - `T5UID1_SERIAL=y`, pantalla en **USART3 (PB11/PB10)**
    (`STM32_T5UID1_SERIAL_USART3_PB11_PB10`) — coincide con los pines
    físicos usados (P10/P11 en la MKS Robin Nano V3.1).
  - El firmware compilado reserva correctamente esos pines:
    `RESERVE_PINS_t5uid1=PB11,PB10`.
- `klippy.py` corrido en modo debug contra el dictionary generado y un
  `printer.cfg` de prueba con `[t5uid1]` — carga sin errores ni
  tracebacks después del fix de la API.

## Qué NO se validó

- No se probó en hardware real (impresora física).
- No se revisaron a fondo las funciones interactivas del `[t5uid1]`
  (nivelación manual, control de temperatura desde pantalla, sonidos,
  etc.) más allá de que el módulo carga y responde al protocolo MCU
  correctamente.
- Soporte T5UID1 para `atsamd` (SAMD21/51) y `lpc176x` fue
  intencionalmente descartado en este merge.

## Uso

```bash
cd ~/klipper
make menuconfig
```
- Micro-controller Architecture: STMicroelectronics STM32
- Processor model: STM32F407
- Bootloader offset: 48KiB bootloader
- Communication interface: USB (on PA11/PA12)
- (con LOW_LEVEL_OPTIONS activado) Screen serial interface: USART3 (on PB11/PB10)

```bash
make clean
make
```

Copia `out/klipper.bin` a la SD como `Robin_nano_v3.bin`, e instala como
de costumbre.

En tu `printer.cfg`, agrega la sección `[t5uid1]` (ver
`config/sample-t5uid1.cfg` en este repo para todas las opciones
disponibles).

## Mantenimiento

Este repo es un snapshot puntual pensado para **no actualizarse más**.
Si en algún momento decides volver a sincronizar con Klipper mainline,
vas a tener que repetir el merge manual — el soporte T5UID1 no está
mantenido río arriba (upstream) en Klipper oficial.
