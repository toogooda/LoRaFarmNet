# Implementation Plan: Centralize nano64DeepSleep Library for Field Nodes

**Branch**: `003-centralize-nano64deepsleep` | **Date**: 2026-07-29 | **Spec**: [specs/003-centralize-nano64deepsleep/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/003-centralize-nano64deepsleep/spec.md)

**Input**: Feature specification from `specs/003-centralize-nano64deepsleep/spec.md` and GitHub Issue #3: "Move the many copies of nano64DeepSleep to central library for all nodes that sleep".

---

## Summary

Centralize `nano64DeepSleep` into a dedicated shared PlatformIO library repository (`Libraries/nano64DeepSleep`). Use `LoraNodeWaterTankLevel/include/nano64DeepSleep.h` as the starting baseline, converting it to standard `library.json`, `src/nano64DeepSleep.h`, `src/nano64DeepSleep.cpp` with `#pragma once` header guards. Reconcile differences across `LoraNodeDualPIR`, `LoraNodeVictron`, `LoraNodeButton`, and `LoraNodeRepeater`. Deprecate local `nano64DeepSleep.h` copies across the 5 sleeping nodes, update `platformio.ini` dependencies, run build verification (`pio run`), hand off to the maintainer for hardware flashing/testing, and close GitHub Issue #3.

---

## Technical Context

**Language/Version**: C++11 / Arduino Framework / PlatformIO  
**Primary Dependencies**: `Arduino.h`, `<avr/sleep.h>`, `<avr/wdt.h>`  
**Library Format**: PlatformIO Shared Library (`library.json`, `src/nano64DeepSleep.h`, `src/nano64DeepSleep.cpp`)  
**Testing**: Build verification (`pio run`) across 5 sleeping node repos + Maintainer Hardware Flashing  
**Target Platform**: ATmega644PA (MightyCore framework)  
**Dependency Declaration**: `lib_deps = symlink://../../Libraries/nano64DeepSleep` (or `https://github.com/toogooda/nano64DeepSleep.git`)  
**Out of Scope**: `LoRaNetGateway` (ESP32) and `LoraNodeDualGateController` (Always-On node)  

---

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

* **MCU Target Invariants**: Firmware explicitly targets ATmega644PA via Arduino framework in PlatformIO (Passed).
* **Supercapacitor Power Integrity**: Preserves 15-minute default deep sleep, GPIO power-gating, and zero blocking delays without explicit `// AI-PROTECT:` comments (Passed).
* **Core Shared Libraries**: Consolidates fragmented copies into `Libraries/nano64DeepSleep` with `#pragma once` include guards (Passed).
* **Build & Flashing Gate**: AI agent performs build verification (`pio run`) only and hands over to maintainer for physical hardware flashing/testing (Passed).
* **Pull Request Policy**: Delivered via feature branch and GitHub PRs without AI auto-merging (Passed).
* **Issue Resolution**: Closing GitHub Issue #3 on `toogooda/LoRaFarmNet` scheduled as the final plan task (Passed).

---

## Shared Library Repository Structure

```text
Libraries/nano64DeepSleep/
├── library.json              # PlatformIO Library Manifest
├── README.md                 # API documentation & usage instructions
└── src/
    ├── nano64DeepSleep.h     # Single Header containing class n64DS (#pragma once)
    └── nano64DeepSleep.cpp   # Implementation file containing sleep state machine & ISRs
```

---

## Pairwise Diff Audit Findings (For Maintainer Review)

The following differences were identified during comparison of `nano64DeepSleep.h` across the 5 node repositories:

### Item 1: Optional Internal Pull-Up Control (`DualPIR` Enhancement)
- **`DualPIR`**: `enableWakeExternal(byte xint, unsigned long cooldownMs = 50, bool usePullup = true)` tracks `_pullupPinsMask` so external wake pins can optionally run without internal pull-ups (`INPUT` vs `INPUT_PULLUP`).
- **`WaterTankLevel` Baseline**: Always applies `pinMode(xint, INPUT_PULLUP)`.
- **Proposed Integration**: Add `bool usePullup = true` (default `true`) to `enableWakeExternal` in the central library.

### Item 2: Reed Switch Debouncing & Flag Clearing (`WaterTankLevel` Enhancement)
- **`WaterTankLevel` Baseline**: Includes `clearPendingExternalInterrupt(int irq)`, `recordPackagingToggle()`, and `wakeExternalSawLow` to debounce magnetic packaging toggles and clear pending EIFR flags.
- **Other Nodes**: Do not implement packaging debounce functions.
- **Proposed Integration**: Retain these helper methods in the central library so all nodes benefit from clean EIFR interrupt clearing without breaking nodes that don't invoke packaging toggles.

### Item 3: Unused Legacy Functions (`Button` & `Repeater` Header Cleanup)
- **Code Inspection**: Analysis confirmed `wakeFromTimer()`, `wakeFromExternal()`, and `setHigh()` were internal helper functions in older header copies, but are **never called** in `main.cpp` of `Button`, `Repeater`, or any other node (`Repeater` already uses `wakeWDT` and `wakeExternal` directly).
- **Agreed Integration**: Omit `wakeFromTimer()`, `wakeFromExternal()`, and `setHigh()` completely from the centralized library to keep the header clean and modern. No application code changes are required.

### Item 4: Millisecond Watchdog Intervals & `lightSleep` (`Repeater` Specifics)
- **`Repeater`**: Includes `lightSleep()` (idle sleep keeping SPI active) and `enableWakeTimer` with millisecond prescaler support (`unsigned long inMs` decomposing into 16ms WDT cycles).
- **Proposed Integration**: Include `lightSleep()` and `inMs` optional parameter (`unsigned long inMs = 0`) in `enableWakeTimer` in the central library so `LoraNodeRepeater` retains full functionality.

---

## Phased Implementation Plan

### Phase 1: Interactive Diff Review & Consensus (Story 2) - COMPLETE
- Review Pairwise Diff Audit Items 1–4 with the maintainer and agree on final library API signatures (All items 1–4 agreed and approved).

### Phase 2: Central Shared Library Creation (`Libraries/nano64DeepSleep`)
- Create `Libraries/nano64DeepSleep` repository directory with `library.json`, `src/nano64DeepSleep.h`, `src/nano64DeepSleep.cpp`, and `README.md`.
- Convert `LoraNodeWaterTankLevel` header into clean `.h` / `.cpp` files with `#pragma once` guards and merged items.

### Phase 3: Local File Removal & `platformio.ini` Integration
- Remove local `nano64DeepSleep.h` from:
  - `Nodes/LoraNodeWaterTankLevel/include/`
  - `Nodes/LoraNodeDualPIR/include/`
  - `Nodes/LoraNodeVictron/include/`
  - `Nodes/LoraNodeButton/src/`
  - `Nodes/LoraNodeRepeater/src/`
- Add `lib_deps = symlink://../../Libraries/nano64DeepSleep` to `platformio.ini` across all 5 sleeping node repositories.
- Execute `pio run` build verification on all 5 sleeping nodes.

### Phase 4: User Hardware Flashing & Testing Handoff
- Pause execution and hand over to the maintainer for physical hardware flashing and testing.

### Phase 5: GitHub PR Delivery & Issue Closure
- Commit/push feature branches and open GitHub Pull Requests for `Libraries/nano64DeepSleep` and the 5 node repos.
- Close GitHub Issue #3 on `toogooda/LoRaFarmNet`.
