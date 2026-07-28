# Implementation Plan: Consolidate LoRa Helpers into Single Header & Cpp Library

**Branch**: `001-consolidate-lora-helpers` | **Date**: 2026-07-28 | **Spec**: [specs/001-consolidate-lora-helpers/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/001-consolidate-lora-helpers/spec.md)

**Input**: Feature specification from `specs/001-consolidate-lora-helpers/spec.md` and research in `specs/001-consolidate-lora-helpers/research.md`.

---

## Summary

Consolidate duplicate radio communication helper files (`Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, `LoraMsg.cpp`) across 7 gateway and node repositories into **one single Header file (`.h`) and one single Implementation file (`.cpp`)** with `#pragma once` guards. Resolve interface variations, preserve target-specific `Pinout.h` hardware pin definitions, and verify `pio run` builds across all projects.

---

## Technical Context

**Language/Version**: C++11 / Arduino Framework / PlatformIO  
**Primary Dependencies**: `SPI.h`, `Arduino.h`  
**Storage**: N/A (Memory & Radio Serialization)  
**Testing**: `pio run` compilation check per node + physical hardware testing  
**Target Platform**: ESP32-WROVER (Gateway) & ATmega644PA (Nodes)  
**Project Type**: Single Header/Cpp Library Pair  

---

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

* **Hardware Addressing**: Preserves 6-byte EEPROM MAC extraction rule (Passed).
* **Power Topology**: No delay additions; preserves `nano64DeepSleep.h` integration (Passed).
* **Binary Stream Markers**: Preserves `MI` (Message ID) initial pair & `CS` (Checksum) stream termination (Passed).
* **Core Shared Libraries**: Replaces fragmented 4-file setup with a single unified `.h`/`.cpp` library pair (Passed).
* **Hardware Pin Safety**: Preserves local `Pinout.h` in each repo so ESP32 Gateway and ATmega644PA Node pin assignments are 100% maintained (Passed).
* **Build Verification**: `pio run` required for each device repository before delivery (Passed).

---

## Single Pair Library Structure

```text
# Unified Library Pair (to replace legacy 4 files in each project)
LoRaHelper.h     # Single Header containing Ra01S registers, RF settings, PortValue, SX126x, and LoraMsg (#pragma once, includes local Pinout.h)
LoRaHelper.cpp   # Single C++ Implementation containing SX126x and LoraMsg methods
```

---

## Phased Implementation Plan

### Phase 1: User Audit & Selection Approval (Story 1)
- Present research findings on `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h` differences to maintainer.
- Confirm maintainer selection on unified API signatures (`getFromByte` + `getMessageID`).

### Phase 2: Single Header & Cpp Library Assembly with Hardware Safety (Story 2)
- Combine `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, and `LoraMsg.cpp` into a single `LoRaHelper.h` and `LoRaHelper.cpp` pair.
- Add `#pragma once` include guards and ensure `"Pinout.h"` resolves to local device hardware definitions (`LCSS`, `LRST`, `LBSY`, `LPWR`, etc.).

### Phase 3: Repository Migration & `pio run` Verification (Story 3)
- Replace legacy 4 files in each of the 7 device repositories with the new `LoRaHelper.h` and `LoRaHelper.cpp`.
- Execute `pio run` across all repositories.

### Phase 4: Manual Device Testing & GitHub PRs (Story 4)
- Pause for user manual device testing.
- Push branches and create GitHub Pull Requests for all modified repositories.
