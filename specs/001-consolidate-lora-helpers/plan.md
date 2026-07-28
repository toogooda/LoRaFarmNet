# Implementation Plan: Consolidate LoRa Helpers into Shared Library

**Branch**: `001-consolidate-lora-helpers` | **Date**: 2026-07-28 | **Spec**: [specs/001-consolidate-lora-helpers/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/001-consolidate-lora-helpers/spec.md)

**Input**: Feature specification from `specs/001-consolidate-lora-helpers/spec.md` and research in `specs/001-consolidate-lora-helpers/research.md`.

---

## Summary

Consolidate duplicate radio communication helper files (`Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, `LoraMsg.cpp`) across 7 gateway and node repositories into a new dedicated private GitHub repository (`toogooda/LoRaFarmNetCore`). Add `#pragma once` guards, resolve interface variations, decouple hardware pinouts, and verify `pio run` builds across all projects.

---

## Technical Context

**Language/Version**: C++11 / Arduino Framework / PlatformIO  
**Primary Dependencies**: `Ra01S` (SX126x driver), `SPI.h`, `Arduino.h`  
**Storage**: N/A (Memory & Radio Serialization)  
**Testing**: `pio run` compilation check per node + physical hardware testing  
**Target Platform**: ESP32-WROVER (Gateway) & ATmega644PA (Nodes)  
**Project Type**: Shared C++ Firmware Library (`PlatformIO` package / Git repository)  

---

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

* **Hardware Addressing**: Preserves 6-byte EEPROM MAC extraction rule (Passed).
* **Power Topology**: No delay additions; preserves `nano64DeepSleep.h` integration (Passed).
* **Binary Stream Markers**: Preserves `MI` (Message ID) initial pair & `CS` (Checksum) stream termination (Passed).
* **Core Shared Libraries**: Replaces fragmented local helper files with a single centralized shared library (Passed).
* **Build Verification**: `pio run` required for each device repository before delivery (Passed).

---

## Project Structure

```text
LoRaFarmNet/                           # Meta-Repository (001-consolidate-lora-helpers branch)
├── .agents/
├── .specify/
└── specs/001-consolidate-lora-helpers/
    ├── spec.md
    ├── research.md
    └── plan.md

# Dedicated Shared Library Repository (toogooda/LoRaFarmNetCore)
LoRaFarmNetCore/
├── library.json
├── README.md
├── src/
│   ├── Ra01S.h
│   ├── Ra01S.cpp
│   ├── LoRaHelper.h
│   ├── LoRaHelper.cpp
│   ├── LoraMsg.h
│   └── LoraMsg.cpp
```

**Structure Decision**: Multi-repository architecture. Shared firmware library hosted in dedicated repository `toogooda/LoRaFarmNetCore`. Node/Gateway projects link via PlatformIO library reference or Git submodule.

---

## Phased Implementation Plan

### Phase 1: User Audit & Selection Approval (Story 1)
- Present research findings on `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h` differences to maintainer.
- Confirm maintainer selection on unified API signatures (`getFromByte` + `getMessageID`).

### Phase 2: Dedicated Private Repo & Library Assembly (Story 2)
- Create private GitHub repository `toogooda/LoRaFarmNetCore`.
- Populate library with unified `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, `LoraMsg.cpp`.
- Add `#pragma once` guards and `library.json` for PlatformIO support.

### Phase 3: Repository Migration & `pio run` Verification (Story 3)
- Update each of the 7 device repositories to reference `LoRaFarmNetCore`.
- Remove redundant local helper files.
- Execute `pio run` across all repositories.

### Phase 4: Manual Device Testing & GitHub PRs (Story 4)
- Pause for user manual device testing.
- Push branches and create GitHub Pull Requests for all modified repositories.
