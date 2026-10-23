# Implementation Plan: 039 - STM32 RTC Timed Sleep for STM32LoRaNode

**Feature Branch**: `039-stm32-rtc-sleep` | **Date**: 2026-09-27 | **Spec**: [specs/039-stm32-rtc-sleep/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/039-stm32-rtc-sleep/spec.md)  
**Target Repository**: `Nodes/STM32LoRaNode`  
**MCU Target**: STM32U073C8T6 (Cortex-M0+, 40KB RAM, 64KB Flash, 8MHz HSE)

---

## Technical Approach & Architecture

### 1. Library Dependencies
In `Nodes/STM32LoRaNode/platformio.ini`:
Add official STMicroelectronics low-power and RTC libraries:
```ini
lib_deps =
    https://github.com/wollewald/INA226_WE.git
    https://github.com/toogooda/LoRaNetLibrary.git
    stm32duino/STM32LowPower@^1.3.0
    stm32duino/STM32RTC@^1.6.0
```
*Note*: `STM32LowPower` v1.3.0 and `STM32RTC` v1.6.0 provide native support for the `STM32U0xx` architecture.

### 2. Initialization in `setup()`
In `Nodes/STM32LoRaNode/src/main.cpp`:
* Include `<STM32LowPower.h>`.
* In `setup()`, call `LowPower.begin();` after initializing Serial and I2C.
* Retain the existing `delay(1000)` boot delay to ensure ST-Link/SWD has an attachment window upon power-up/reset.

### 3. Sleep Execution in `loop()`
In `Nodes/STM32LoRaNode/src/main.cpp`:
* Replace the blocking `delay(60000)` at the end of `loop()` with:
  ```cpp
  Serial1.println(F("Sleeping for 60 seconds (RTC)..."));
  Serial1.flush();
  LowPower.deepSleep(60000);
  ```
* `LowPower.deepSleep(60000)` puts the STM32U0 into Stop mode (Stop 1/2) with SRAM preserved and arms the internal RTC Wakeup Timer for 60,000 ms.
* Upon wakeup, `STM32LowPower` invokes `SystemClock_Config()` (defined in `STM32Helpers.h`) to automatically restore the 8MHz HSE clock, then returns execution directly to `loop()`.

### 4. Hardware & Power State Invariants
* **Radio Power Gate (`LPWR`)**: Already set to `HIGH` (OFF) by `sendMsg()` before sleep. The onboard **100kΩ hardware pull-up** holds the B772 PNP base at 3.3V, ensuring zero radio standby draw.
* **INA226 ADCs**: Each ADC is already put into `powerDown()` mode during `configureADCs()` and immediately following each measurement (`getP2Value()`, `getP3Value()`, `getInternalVoltage()`).
* **Interrupt Scope**: Strictly no external GPIO pin interrupts are configured; wake-up is solely triggered by the internal RTC timer.
* **Serial Buffer**: `Serial1.flush()` is called immediately before `LowPower.deepSleep()` so no TX bytes are truncated.

---

## File Changes & Modifications

| File | Proposed Changes |
| :--- | :--- |
| [`platformio.ini`](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/STM32LoRaNode/platformio.ini) | Add `stm32duino/STM32LowPower@^1.3.0` and `stm32duino/STM32RTC@^1.6.0` to `lib_deps`. |
| [`src/main.cpp`](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/STM32LoRaNode/src/main.cpp) | `#include <STM32LowPower.h>`, call `LowPower.begin()` in `setup()`, and replace `delay(60000)` in `loop()` with `LowPower.deepSleep(60000)`. |

---

## Verification Strategy

1. **Build Verification**:
   Execute `pio run -d Nodes/STM32LoRaNode` to verify clean compilation with the new libraries and Flash/SRAM bounds.
2. **Serial & Timing Verification**:
   Flash firmware and monitor output at 115200 baud:
   * Observe packet transmission log.
   * Observe `"Sleeping for 60 seconds (RTC)..."` log.
   * Verify that exactly 60 seconds later, the node wakes up, prints the next readings, and transmits.
3. **Continuity Verification**:
   * Verify `setup()` is NOT re-executed on wake (boot banner does not repeat).
   * Verify message counter / telemetry increments across wakeups.
