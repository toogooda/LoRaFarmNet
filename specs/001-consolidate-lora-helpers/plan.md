# Implementation Plan: Consolidate LoRa Helpers into Shared Library Repository

**Branch**: `001-consolidate-lora-helpers` | **Date**: 2026-07-28 | **Spec**: [specs/001-consolidate-lora-helpers/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/001-consolidate-lora-helpers/spec.md)

**Input**: Feature specification from `specs/001-consolidate-lora-helpers/spec.md` and user directive to create a standalone library repository rather than duplicating files locally.

---

## Summary

Consolidate duplicate radio communication helper files (`Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, `LoraMsg.cpp`) into a **dedicated standalone PlatformIO library repository (`LoRaNetLibrary`)** located under `Libraries/LoRaNetLibrary` (target remote `https://github.com/toogooda/LoRaNetLibrary.git`). Include `library.json`, `src/LoRaHelper.h`, and `src/LoRaHelper.cpp`. Remove local `LoRaHelper.h`/`LoRaHelper.cpp` copies from all 7 device repositories (`Gateway` + 6 `Nodes`), update each repository's `platformio.ini` to declare `LoRaNetLibrary` in `lib_deps`, and verify clean `pio run` builds.

---

## Technical Context

**Language/Version**: C++11 / Arduino Framework / PlatformIO  
**Primary Dependencies**: `SPI.h`, `Arduino.h`  
**Library Format**: PlatformIO Shared Library (`library.json`, `src/LoRaHelper.h`, `src/LoRaHelper.cpp`)  
**Testing**: `pio run` compilation check per node + physical hardware testing  
**Target Platform**: ESP32-WROVER (Gateway) & ATmega644PA (Nodes)  
**Dependency Declaration**: `lib_deps = symlink://../../Libraries/LoRaNetLibrary` (local dev) / `https://github.com/toogooda/LoRaNetLibrary.git`  

---

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

* **Hardware Addressing**: Preserves 6-byte EEPROM MAC extraction rule (Passed).
* **Power Topology**: No delay additions; preserves `nano64DeepSleep.h` integration (Passed).
* **Binary Stream Markers**: Preserves `MI` (Message ID) initial pair & `CS` (Checksum) stream termination (Passed).
* **Core Shared Libraries**: Replaces fragmented 4-file setup and local duplicated files with a single version-controlled library repo `LoRaNetLibrary` (Passed).
* **Hardware Pin Safety**: Preserves local `Pinout.h` in each repo so ESP32 Gateway and ATmega644PA Node pin assignments are 100% maintained (Passed).
* **Build Verification**: `pio run` required for each device repository before delivery (Passed).

---

## Shared Library Repository Structure

```text
Libraries/LoRaNetLibrary/
├── library.json          # PlatformIO Library Manifest
├── README.md             # Library Usage & API Documentation
└── src/
    ├── LoRaHelper.h      # Single Header containing Ra01S registers, RF settings, PortValue, SX126x, and LoraMsg (#pragma once)
    └── LoRaHelper.cpp    # Single C++ Implementation containing SX126x and LoraMsg methods
```

---

## Phased Implementation Plan

### Phase 1: User Audit & Selection Approval (Story 1) - COMPLETE
- Present research findings on `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h` differences to maintainer.
- Confirm maintainer selection on unified API signatures (`getFromByte` + `getMessageID`).

### Phase 2: Dedicated Shared Library Repository Assembly (Story 2)
- Create `Libraries/LoRaNetLibrary` repository directory with `library.json`, `src/LoRaHelper.h`, `src/LoRaHelper.cpp`, and `README.md`.
- Ensure library is self-contained with `#pragma once` include guards.

### Phase 3: Local Code Cleanup & platformio.ini Dependency Integration (Story 3)
- Remove local `LoRaHelper.h` and `LoRaHelper.cpp` from `Gateway/LoRaNetGateway/src/` and each node's `src/` directory.
- Update `platformio.ini` in `Gateway` and all 6 `Nodes` projects to add `LoRaNetLibrary` to `lib_deps`.
- Execute `pio run` across all 7 device repositories to verify dependency resolution and clean compilation.

### Phase 4: Manual Device Testing & GitHub PRs (Story 4)
- Pause for user manual device testing.
- Initialize/commit the new `LoRaNetLibrary` git repository.
- Push feature branches and create GitHub Pull Requests for `LoRaNetLibrary`, all 7 device repositories, and the root meta-repository.

