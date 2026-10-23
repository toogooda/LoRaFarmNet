# Implementation Tasks: 039 - STM32 RTC Timed Sleep for STM32LoRaNode

**Branch**: `039-stm32-rtc-sleep` | **Spec**: [specs/039-stm32-rtc-sleep/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/039-stm32-rtc-sleep/spec.md) | **Plan**: [specs/039-stm32-rtc-sleep/plan.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/039-stm32-rtc-sleep/plan.md)

## Tasks

- [x] **Task 1: Setup Feature Branch**
  - Create and switch to feature branch `039-stm32-rtc-sleep` in `Nodes/STM32LoRaNode` and `LoRaFarmNet`.

- [x] **Task 2: Add Dependencies to `platformio.ini`**
  - Update `Nodes/STM32LoRaNode/platformio.ini`:
    - Add `https://github.com/stm32duino/STM32LowPower.git`
    - Add `https://github.com/stm32duino/STM32RTC.git`

- [x] **Task 3: Implement LowPower Timed Sleep in `src/main.cpp`**
  - Include `<STM32LowPower.h>` and `<STM32RTC.h>`.
  - In `setup()`: invoke `LowPower.begin()`, unlock backup domain, initialize RTC on LSI, enable `HAL_DBGMCU_EnableDBGStopMode()` for debugger retention, and attach alarm callback.
  - Implement `sleepSeconds(uint32_t sec)` using direct STM32U0 RTC Alarm A (`MATCH_HHMMSS`), internal wakeup line (`HAL_PWREx_EnableInternalWakeUpLine`), EXTI Line 28, SysTick suspension (`HAL_SuspendTick`), and STOP 1 low-power entry (`HAL_PWR_EnterSTOPMode`).
  - In `loop()`: execute sensor reading and LoRa packet transmission (`sendMsg(lMsg)`), followed by `sleepSeconds(60)`.

- [x] **Task 4: Build Verification**
  - Run `pio run -d Nodes/STM32LoRaNode`.
  - Verify zero compilation/linker errors and check Flash/SRAM bounds (RAM: 2,188 B [5.3%], Flash: 62,352 B [95.1%]).

- [x] **Task 5: Device Verification & Testing Pause**
  - Flash firmware to `STM32LoRaNode`.
  - Monitor serial output at 115200 baud to verify:
    - Successful sensor read and LoRa packet transmission.
    - Clean entry into STOP sleep.
    - Automatic RTC wakeup after 60 seconds (verified: 12:00:00 -> 12:01:00 -> 12:02:00).
    - SRAM preservation: `MI` counter incremented from 0 to 1 across sleep cycles.
    - Subsequent cycles continue smoothly without hangs.
  - Paused for user manual confirmation before pushing or creating Pull Request.
