# Spec Track: LoRaNetLibrary Reconciliation & Audit Decisions

**Branch**: `001-consolidate-lora-helpers` | **Status**: **COMPLETED** | **Date**: 2026-07-28

This specification records the agreed reconciliation decisions across all 20 code and hardware differences identified between the original 7 device repositories (`Gateway` + 6 `Nodes`) and the shared `LoRaNetLibrary`.

---

## Decision Log

### Item 1: SPI Clock Speed Configuration (`SPI_Speed`)
* **Status**: **AGREED & RECORDED**
* **Decision**: Adopt **Option B (Automatic Architecture Detection)** in `LoRaNetLibrary/src/LoRaHelper.h`.
* **Code Specification**:
  ```cpp
  #ifndef SPI_Speed
    #if defined(ARDUINO_ARCH_ESP32) || defined(ESP32)
      #define SPI_Speed 2000000 // 2 MHz for ESP32 Gateway
    #else
      #define SPI_Speed 500000  // 500 kHz for ATmega644PA Field Nodes
    #endif
  #endif
  ```
* **Hardware Rationale**:
  - ESP32 Gateway runs at 240 MHz CPU clock speed and requires 2 MHz SPI throughput to process high network traffic without blocking.
  - ATmega644PA field nodes run at 8 MHz CPU clock speed on 3.3V supercapacitor power, where 2 MHz ($F_{CPU}/4$) causes SPI timing distortion over wire runs. 500 kHz guarantees AVR bus stability.
  - Option B detects target architecture compiler flags (`ARDUINO_ARCH_ESP32` vs `AVR`) automatically, ensuring correct clock rates for both chipsets with zero `platformio.ini` edits.

---

### Item 2: `ON_AIR_TIMEOUT` and `BUSY_WAIT` Header Macros
* **Status**: **AGREED & RECORDED**
* **Decision**: Define `#ifndef ON_AIR_TIMEOUT #define ON_AIR_TIMEOUT 1000 #endif` and `#ifndef BUSY_WAIT #define BUSY_WAIT 5000 #endif` in `LoRaNetLibrary/src/LoRaHelper.h`.
* **Code Specification**:
  ```cpp
  #ifndef ON_AIR_TIMEOUT
  #define ON_AIR_TIMEOUT 1000 // 1000 ms max transmit timeout
  #endif

  #ifndef BUSY_WAIT
  #define BUSY_WAIT 5000      // 5000 ms max BUSY wait timeout
  #endif
  ```
* **Hardware Rationale**:
  - `LoraNodeVictron` and complex telemetry nodes transmit large payloads containing multiple `PortValue`s (battery voltage, current, solar yield, state of charge). Longer payloads take more airtime and require extended timeouts to avoid premature transmission cancellation.
  - Gateway primarily receives packets, so its local `Ra01S.h` was never updated. Defining these macros under `#ifndef` in `LoRaNetLibrary` protects all nodes and the Gateway when transmitting large frames.

---

### Item 3: `FREQ_DIV_2_25` Macro in `Ra01S.h`
* **Status**: **AGREED & RECORDED**
* **Decision**: Include `#define FREQ_DIV_2_25 FREQ_DIV` and `#define FREQ_STEP ( double )( XTAL_FREQ / FREQ_DIV_2_25 )` in `LoRaNetLibrary/src/LoRaHelper.h`.
* **Code Specification**:
  ```cpp
  #define XTAL_FREQ                       ( double )32000000
  #define FREQ_DIV                        ( double )pow( 2.0, 25.0 )
  #define FREQ_DIV_2_25                   FREQ_DIV
  #define FREQ_STEP                       ( double )( XTAL_FREQ / FREQ_DIV_2_25 )
  ```
* **Hardware Rationale**:
  - `LoraNodeDualPIR` defined `FREQ_DIV_2_25` to explicitly document the 2^25 crystal clock divisor in SX126x frequency step calculations (32 MHz / 2^25 = 0.95367431640625 Hz).
  - Mathematically identical across all node architectures; including the macro preserves build compatibility for `DualPIR` without altering frequency calculations for Gateway or other nodes.

---

### Item 4: Local `#include "Pinout.h"` Decoupling
* **Status**: **AGREED & RECORDED**
* **Decision**: Do not include `#include "Pinout.h"` inside `LoRaNetLibrary/src/LoRaHelper.h`. Each project includes its local `Pinout.h` in `main.cpp`.
* **Hardware Rationale**:
  - `Pinout.h` contains board-specific GPIO mappings unique to each microcontroller hardware layout (ESP32 Gateway vs ATmega644PA Nodes).
  - `LoRaNetLibrary` receives pin numbers dynamically via the `SX126x` constructor arguments (`spiSelect`, `reset`, `busy`, `txen`, `rxen`). Decoupling `Pinout.h` from the shared library header ensures the library is self-contained while preserving 100% of each project's local pin definitions.

---

### Item 7: `WaitForIdle` Internal Private Helper Function
* **Status**: **AGREED & RECORDED**
* **Decision**: Keep `WaitForIdle` as a `private` internal helper method inside `SX126x` class within `LoRaHelper.cpp`.
* **Rationale**:
  - `WaitForIdle` is never called by application code in any repository (0 calls in `main.cpp`).
  - The signature divergence came from upstream third-party `Arduino-Ra01S` library version updates. Making it an internal private helper standardizes SPI BUSY pin handling internally without exposing or affecting application-level code.

---


### Item 5: `extern SX126x lora;` Declaration in Header
* **Status**: **AGREED & RECORDED**
* **Decision**: Declare `extern SX126x lora;` at the bottom of `LoRaNetLibrary/src/LoRaHelper.h`.
* **Rationale**:
  - `LoRaNetGateway` has a multi-file architecture (`main.cpp`, `FarmNetwork.cpp`, `MQTTHelper.cpp`) where `FarmNetwork.cpp` accesses the global `lora` transceiver instance to send ACK frames and pairing responses.
  - Harmless to single-file field nodes while satisfying the Gateway's multi-file compilation requirement.

---

### Item 6: Centralized Network RF Parameters & Device Repo Standardization
* **Status**: **AGREED & RECORDED**
* **Decision**: 
  1. Define central `LoRaFarmNet` network RF parameter constants in `LoRaNetLibrary/src/LoRaHelper.h`.
  2. Add helper method `lora.beginFarmDefaults()` to `SX126x`.
  3. Standardize each device repository (`Gateway` + 6 `Nodes`) to use `LoRaNetLibrary` central defaults, eliminating local duplicate `#define` statements.
* **Code Specification**:
  In `LoRaNetLibrary/src/LoRaHelper.h`:
  ```cpp
  #define RF_FREQUENCY          915000000UL // 915 MHz Center Frequency
  #define TX_OUTPUT_POWER       22          // 22 dBm Tx Power
  #define LORA_BANDWIDTH        6           // 500 kHz Bandwidth
  #define LORA_SPREADING_FACTOR 11          // SF11
  #define LORA_CODINGRATE       1           // 4/5 Coding Rate
  #define LORA_PREAMBLE_LENGTH  8           // 8 Preamble Symbols
  #define LORA_PAYLOADLENGTH    0           // 0 = Variable Length (Explicit Header)
  ```
  In `LoRaNetLibrary/src/LoRaHelper.cpp`:
  ```cpp
  int16_t SX126x::beginFarmDefaults() {
    int16_t err = begin(RF_FREQUENCY, TX_OUTPUT_POWER);
    if (err == ERR_NONE) {
      LoRaConfig(LORA_SPREADING_FACTOR, LORA_BANDWIDTH, LORA_CODINGRATE, LORA_PREAMBLE_LENGTH, LORA_PAYLOADLENGTH, true, false);
    }
    return err;
  }
  ```
* **Rationale**:
  - Guarantees 100% RF parameter consistency across every single device on the farm network.
  - Eliminates local parameter drift or human error when adding new nodes.
  - Updating `LoRaNetLibrary` tunes the entire farm network identically.

---

### Item 8: Boot Diagnostic Logging in `begin()`
* **Status**: **AGREED & RECORDED**
* **Decision**: Keep boot diagnostic `Serial.print` statements commented out inside `SX126x::begin(...)` in `LoRaHelper.cpp`.
* **Rationale**:
  - `DualPIR` had boot pin logging temporarily uncommented during PCB bring-up. Keeping these statements commented out keeps boot output clean across all Gateway and Node projects.

---

### Item 9: Maximum Message Buffer Size (`MAX_MSG_SIZE = 128`)
* **Status**: **AGREED & RECORDED**
* **Decision**: Define `static const int MAX_MSG_SIZE = 128;` as the standard message buffer size in `LoRaNetLibrary/src/LoRaHelper.h`.
* **Hardware Rationale**:
  - 128 bytes is the standard over-the-air frame size limit for the network. Transmitting 128-byte payloads at SF11 / 500 kHz takes ~400-500 ms airtime, matching the 1000 ms `ON_AIR_TIMEOUT` transmit window.
  - `LoraNodeRepeater` allocates historical message arrays (`lastMessage[10]`, `repeatMessage[5]`) in RAM. Restricting `MAX_MSG_SIZE` to 128 bytes conserves SRAM on ATmega644P field nodes (4KB RAM budget) and prevents memory fragmentation.

---

### Item 10: Source Address Extraction Methods (`getFromByte` & `getFromAddress`)
* **Status**: **AGREED & RECORDED**
* **Decision**: Expose both `getFromByte(const uint8_t byteNumber)` and `getFromAddress(uint8_t* address)` as public methods of `LoraMsg` in `LoRaNetLibrary`.
* **Code Specification**:
  ```cpp
  uint8_t getFromByte(const uint8_t byteNumber) const;
  void getFromAddress(uint8_t* address) const;
  ```
* **Rationale**:
  - `Gateway` requires `getFromByte` to extract individual MAC bytes for web UI rendering and MQTT topic publishing.
  - `LoraNodeDualGateController` requires `getFromAddress` to copy the 6-byte MAC array for button authorization checks.
  - Including both methods provides a complete MAC inspection API across all projects with zero overhead.

---

### Item 11: In-Flight Payload Modification Method (`setPortValue`)
* **Status**: **AGREED & RECORDED**
* **Decision**: Include `bool setPortValue(const char type[2], uint16_t newValue)` as a public member function of `LoraMsg` in `LoRaNetLibrary`.
* **Code Specification**:
  ```cpp
  bool setPortValue(const char type[2], uint16_t newValue);
  ```
* **Rationale**:
  - `LoraNodeRepeater` requires `setPortValue` to modify incoming packet payloads in-flight (updating Message Reason `MR` or Router ID `RI`) before re-transmitting frames over multi-hop paths to the Gateway.

---

### Item 12: `getMessageID()` Public Method Visibility
* **Status**: **AGREED & RECORDED**
* **Decision**: Keep `uint16_t getMessageID() const` as a public member function of `LoraMsg` in `LoRaNetLibrary`.
* **Code Specification**:
  ```cpp
  uint16_t getMessageID() const;
  ```
* **Rationale**:
  - Message ID (`MI`) is the mandatory initial key pair in every binary frame. Exposing `getMessageID()` allows `Gateway` and `LoraNodeRepeater` to inspect packet sequence numbers for duplicate filtering and ACK matching.

---

### Item 13: SX126x `SetTx` Timeout Calculation (15.625 µs Ticks Conversion)
* **Status**: **AGREED & RECORDED**
* **Decision**: Retain the exact `uint32_t tout = timeoutInMs * 1000 / 15.625;` RTC tick conversion formula in `SX126x::SetTx` in `LoRaNetLibrary/src/LoRaHelper.cpp`.
* **Code Specification**:
  ```cpp
  void SX126x::SetTx(uint32_t timeoutInMs) {
    uint32_t tout = timeoutInMs * 1000 / 15.625;
    uint8_t buf[3];
    buf[0] = (tout >> 16) & 0xFF;
    buf[1] = (tout >> 8) & 0xFF;
    buf[2] = tout & 0xFF;
    SetTxEnable();
    WriteCommand(SX126X_CMD_SET_TX, buf, 3);
  }
  ```
* **Hardware Rationale**:
  - SX126x opcode `0x83` (`SetTx`) expects a 24-bit timeout parameter in 15.625 µs RTC clock steps (1 / 64 kHz = 15.625 µs). Converting ms to 15.625 µs ticks guarantees accurate radio hardware transmit duration.

---

### Item 14: `GetPacketStatus` SPI Response Offsets & RSSI/SNR Formula
* **Status**: **AGREED & RECORDED**
* **Decision**: Retain the exact 4-byte `GetPacketStatus` reading and RSSI/SNR decoding formula in `LoRaNetLibrary/src/LoRaHelper.cpp`.
* **Code Specification**:
  ```cpp
  uint8_t SX126x::GetPacketStatus(int8_t *rssiPacket, int8_t *snrPacket) {
    uint8_t buf[4];
    ReadCommand(SX126X_CMD_GET_PACKET_STATUS, buf, 4);
    *rssiPacket = (buf[3] >> 1) * -1;
    *snrPacket = buf[2] < 128 ? buf[2] >> 2 : ((buf[2] - 256) >> 2);
    return buf[0];
  }
  ```
* **Hardware Rationale**:
  - Opcode `0x14` returns 4 response bytes over SPI (`buf[0]` status, `buf[1]` RSSI, `buf[2]` 2's complement SNR/4, `buf[3]` signal RSSI / -2). This exact offset and math decodes negative RSSI (dBm) and SNR values.

---

### Item 15: `LoraMsg` Debug Output & Blocking Delays
* **Status**: **AGREED & RECORDED**
* **Decision**: Remove blocking `delay(100)` calls and comment out `Serial.print` debug lines inside `LoraMsg::decryptMessage()` in `LoRaHelper.cpp`.
* **Rationale**:
  - Eliminates unnecessary 100 ms blocking delays during message parsing, reducing awake time and conserving power on supercapacitor-powered field nodes.

---

### Item 16: Standard Framework Inclusion (`#include <Arduino.h>`)
* **Status**: **AGREED & RECORDED**
* **Decision**: Include `#include <Arduino.h>` at the top of `LoRaNetLibrary/src/LoRaHelper.h`.
* **Rationale**:
  - Ensures standard Arduino core types (`uint8_t`, `uint16_t`, `uint32_t`, `millis()`, `delay()`, `digitalRead()`) resolve cleanly across both ESP32 and ATmega644P toolchains.

---

### Item 17: `SetPaConfig` Power Amplifier Register Configuration (+22 dBm Output)
* **Status**: **AGREED & RECORDED**
* **Decision**: Retain `SetPaConfig(0x04, 0x07, 0x00, 0x01);` inside `SX126x::begin(...)` in `LoRaNetLibrary/src/LoRaHelper.cpp`.
* **Hardware Rationale**:
  - Configures internal SX1262 power amplifier duty cycles (`paDutyCycle=0x04`, `hpDutyCycle=0x07`) according to Semtech Datasheet Section 13.1.14 to operate safely at max +22 dBm output power for farm coverage.

---

### Item 18: Overcurrent Protection (`SetOvercurrentProtection(60.0)`)
* **Status**: **AGREED & RECORDED**
* **Decision**: Retain `SetOvercurrentProtection(60.0);` inside `SX126x::begin(...)` in `LoRaNetLibrary/src/LoRaHelper.cpp`.
* **Hardware Rationale**:
  - Caps maximum radio power draw at 60 mA, protecting supercapacitor power supplies on field nodes from brownout voltage drops during RF transmit bursts.

---

### Item 19: Inverted IQ Silicon Errata Fix (`FixInvertedIQ`)
* **Status**: **AGREED & RECORDED**
* **Decision**: Retain `FixInvertedIQ(PacketParams[5]);` inside `SX126x::LoRaConfig(...)` in `LoRaNetLibrary/src/LoRaHelper.cpp`.
* **Hardware Rationale**:
  - Implements Semtech SX1261/SX1262 Errata Section 15.4 silicon fix for register `0x0736` bit 2 to ensure stable reception and repeat operation across IQ polarity modes.

---

### Item 20: SPI Opcode Status Verification & Command Retries (`WriteCommand2`)
* **Status**: **AGREED & RECORDED**
* **Decision**: Retain the exact SPI status byte verification and command retry logic inside `SX126x::WriteCommand2` in `LoRaNetLibrary/src/LoRaHelper.cpp`.
* **Hardware Rationale**:
  - Verifies SX126x hardware status bytes over SPI to protect against bus bit noise glitches on 8 MHz ATmega644P field nodes, guaranteeing reliable command execution.

---















