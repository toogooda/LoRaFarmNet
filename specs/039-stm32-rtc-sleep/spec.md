# Feature Specification: 039 - STM32 RTC Timed Sleep for STM32LoRaNode

**Feature Branch**: `039-stm32-rtc-sleep`  
**Created**: 2026-09-27  
**Status**: Draft  
**Target Repository**: `Nodes/STM32LoRaNode`  
**MCU Target**: STM32U073C8T6 (Cortex-M0+, 64KB Flash, 40KB SRAM, 8MHz HSE)  
**Input**: User description: "replace delay in loop with sleep initially 60 seconds starting with just no pin interrupts just using the RTC"

---

## Context & Motivation
The `STM32LoRaNode` currently executes a blocking `delay(60000)` in `loop()` between consecutive sensor read and LoRa transmission cycles. 

### Current Limitations:
1. **Excessive Power Consumption**:
   - During `delay(60000)`, the STM32U073 CPU remains fully active in Run mode at 8MHz, drawing several milliamps continuously.
   - For a supercapacitor/solar-powered field node, this rapid energy drain exhausts supercapacitor reserves quickly during night or low-light conditions.
2. **Missing Low-Power Transition**:
   - While the SX1262 LoRa radio is powered down via the high-side PNP switch (`LPWR = HIGH`), the microcontroller itself never enters low-power Stop mode.
3. **Hardware Power Gating (`LPWR`)**:
   - `LPWR` (PB10) controls the high-side B772 PNP power switch supplying the Ra-01S (SX1262) LoRa module.
   - The hardware includes a **100kΩ pull-up resistor to 3.3V** on the `LPWR` line.
   - Driving `LPWR = LOW` turns the PNP switch ON; driving `LPWR = HIGH` (and letting the 100kΩ pull-up hold the gate during sleep) turns the radio power completely OFF.
4. **Objective**:
   - Replace the blocking `delay(60000)` with hardware RTC-based low-power sleep (`deepSleep` / Stop mode) for an initial duration of 60 seconds.
   - Keep the initial implementation focused and minimal: **wake up solely on the internal RTC timer**, with no external GPIO pin interrupts enabled.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - RTC Timed Deep Sleep Wakeup (Priority: P1)
As a solar/supercapacitor field node, the device must transition into low-power Stop mode after transmitting its LoRa telemetry, remain asleep for 60 seconds, and automatically wake up using the internal RTC timer to execute the next loop cycle.

**Why this priority**: Core requirement for energy conservation on battery-free / supercapacitor hardware.

**Independent Test**:
- Flash the firmware to `STM32LoRaNode`.
- Observe the serial output and LoRa transmissions.
- Verify that a packet is transmitted, the node enters sleep, and exactly 60 seconds later wakes up, prints diagnostic/telemetry logs, and transmits the next message.

**Acceptance Scenarios**:
1. **Given** the node has completed its LoRa transmission and powered down the radio, **When** it reaches the end of `loop()`, **Then** it flushes `Serial1` and enters low-power Stop mode (`deepSleep`) configured for a 60-second RTC alarm.
2. **Given** the node is asleep in Stop mode, **When** the 60-second RTC timer expires, **Then** the STM32 wakes up, re-arms its system clocks if needed, and proceeds to the top of `loop()` for the next sample/transmission cycle.

---

### User Story 2 - State & Telemetry Continuity Across Sleep (Priority: P2)
As a field node, variables in SRAM (such as `LoraMsg::messageCounter` / message ID, unique MAC address, and device type) must be preserved across RTC sleep cycles so that consecutive transmissions show incrementing or distinct telemetry without rebooting from scratch.

**Why this priority**: Stop mode preserves SRAM, whereas Standby/Shutdown resets the MCU. Maintaining Stop mode ensures smooth continuous operation without re-running `setup()` or losing sensor state.

**Independent Test**:
- Observe consecutive LoRa packets over multiple 60-second wake cycles.
- Verify that the node does not print the boot banner (`STM32LoRaNode Hello World`) on each wake-up, and that message counters increment across cycles.

**Acceptance Scenarios**:
1. **Given** the MCU enters 60-second Stop mode, **When** it wakes up, **Then** execution resumes directly after the sleep call in `loop()`, without restarting `setup()`.

---

## Edge Cases

- **Radio Power Gate (`LPWR`) Stability in Sleep**: Because PB10 is equipped with a 100kΩ hardware pull-up to 3.3V, the B772 PNP high-side switch is securely held OFF during Stop mode even if the GPIO enters high-impedance, preventing any accidental power drain into the SX1262 transceiver.
- **Serial Flush Before Sleep**: If `Serial1` has pending bytes in its TX FIFO/buffer when Stop mode is entered, the output becomes corrupted or truncated. All serial buffers must be flushed (`Serial1.flush()`) before entering sleep.
- **Clock Restoration After Stop Mode**: When waking from Stop mode on STM32, the core clock typically defaults to the internal MSI/HSI oscillator. The system clock configuration must be verified or re-engaged so peripherals (SPI, I2C, USART1) continue operating at their expected frequencies.
- **ST-Link SWD Debug Disconnect**: Entering Stop mode disables the CPU debug clock by default. During bring-up, flashing or attaching a debugger while the MCU is asleep may require hardware reset (`connect under reset`) or a brief initial delay in `setup()` before entering sleep.

---

## Requirements *(mandatory)*

### Functional Requirements
- **FR-001**: The system MUST integrate the STM32Duino `STM32LowPower` and `STM32RTC` libraries (or equivalent STM32 HAL RTC Wakeup Timer).
- **FR-002**: The system MUST configure the RTC using the internal LSI (Low Speed Internal oscillator, ~32kHz) or LSE if present, ensuring reliable clocking without external dependencies.
- **FR-003**: The system MUST replace `delay(60000)` in `loop()` with an RTC-backed sleep call configured for 60 seconds (`60000 ms`).
- **FR-004**: The system MUST NOT configure or attach any GPIO pin wake-up interrupts during this initial phase (wake-up is exclusively triggered by the RTC timer).
- **FR-005**: All peripheral and radio power gates (specifically `LPWR = HIGH`, assisted by the onboard 100kΩ pull-up) MUST remain powered OFF during sleep to eliminate standby power leakage.
- **FR-006**: Serial communication output MUST be completely flushed via `Serial1.flush()` prior to entering sleep.
- **FR-007**: Upon RTC wakeup, the MCU MUST resume execution in `loop()` and successfully execute the subsequent sensor readings and LoRa transmission.

### Non-Functional Requirements
- **NFR-001**: In Stop mode, power consumption of the STM32U073 MCU MUST drop significantly below Run mode current (sub-milliamp range).
- **NFR-002**: Firmware MUST compile cleanly with `pio run -d Nodes/STM32LoRaNode` without warnings or linker errors.
- **NFR-003**: Flash memory usage MUST remain comfortably within the 64KB Flash boundary of the STM32U073C8.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes
- **SC-001**: Node transmits LoRa packet, enters sleep, and wakes up after 60 seconds ($\pm 5\%$ using LSI or $\pm 0.1\%$ using LSE).
- **SC-002**: Node successfully performs at least 5 consecutive 60-second sleep and transmit cycles without hanging, crashing, or bus fault.
- **SC-003**: `Serial1` output remains clean and readable before sleep and after wakeup.
- **SC-004**: Compilation succeeds with PlatformIO.
