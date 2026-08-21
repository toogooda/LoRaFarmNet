---
trigger: always_on
---

# Global Network Operational Rules & Framing Invariants

## Hardware & System Constraints
* **MCU Targets:**
  * **ESP32-WROVER:** Gateway & heavy nodes (`PSRAM` mandatory).
  * **ATmega644PA:** Low-power field nodes & repeaters (Arduino framework via PlatformIO).
* **Power Topology:** Some nodes use sleep and some are always on for sleep nodes 10–15F Supercapacitors (battery-free). Awake time and airtime minimization is CRITICAL to prevent power loss.
* **Sleep Management:**
  * Periodic field nodes MUST use `nano64DeepSleep.h` and enter deep sleep for 15 minutes by default.
  * Always-On nodes (`LoRaGateway`, `LoRaNodeDualGateController`, `LoRaNetRepeaterNode`) MUST NEVER enter deep sleep.
  * Peripheral/sensor power MUST be switched off via GPIO power gates prior to sleep entry.
  * NO blocking `delay()` calls on sleeping nodes unless protected by an explicit inline `// AI-PROTECT:` hardware-settling comment.

---

## Hardware Addressing
* Every node extracts a unique **6-byte hardware address** directly from its onboard I2C EEPROM/MAC chip at boot.
* Do not hardcode device IDs or MAC addresses in codebase files.

---

## Binary Message Frame Protocol
Over-the-air messages follow a fixed-pair binary stream (no length byte). Stream parsing terminates strictly at the mandatory `CS` checksum key pair.

### Frame Layout
| Offset | Field | Size | Requirement |
|---|---|---|---|
| `0x00` | Source Address | 6 Bytes | Source device hardware MAC |
| `0x06` | Destination Address | 6 Bytes | Target device hardware MAC |
| `0x0C` | Payload Stream | N * 4 Bytes | Stream of `[2-Byte ASCII Key]` + `[2-Byte uint16_t Value]` |
| End | Checksum (`CS`) | 4 Bytes | Mandatory `CS` key + `uint16_t` calculated CRC (terminates stream) |

### Key & Payload Conventions
* **`MI`** (`uint16_t`): Message ID — **MUST be the first pair** in the payload stream.
* **`CS`** (`uint16_t`): Checksum — **MUST be the final pair** in the payload stream.
* **Port Identifiers:** 2-character ASCII identifiers map directly to `SensorValues` (e.g., `VB` = voltage, `TK` = tank level, `G1` = gate position).

---

## Messaging & ACK Enforcement
* **Default Sensor Policy (Best-Effort):** Transmit once, **NO ACK expected**, immediately return to `nano64DeepSleep` to conserve supercapacitor charge.
* **ACK Exception 1 (Always-On Devices):** Gateway configuration payloads targeting Always-On nodes require an explicit response ACK.
* **ACK Exception 2 (Remote Buttons):** Button nodes MUST wait for a Gateway ACK before returning to deep sleep.

---

## Shared Core Libraries
All radio and power logic must build upon these shared headers:
* `Ra01S.h`: Core driver for SX126x/Ra-01S transceivers.
* `LoRaHelper.h`: Transmission, reception state machine, and retries.
* `LoraMsg.h`: Binary serialization, parsing, encryption, and CRC validation.
* `nano64DeepSleep.h`: Hardware sleep timers and wake interrupts.

---

## Subsystem & Gateway Testing Workflows
* **Gateway Autonomous Testing & Hardware Verification Workflow**: See [`Gateway/LoRaNetGateway/.agent/rules/ai_testing_workflow.md`](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/.agent/rules/ai_testing_workflow.md) for full test harness, dynamic serial IP discovery, and dual-surface verification rules.