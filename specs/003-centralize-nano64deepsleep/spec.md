# Feature Specification: Centralize nano64DeepSleep Library for Field Nodes

**Feature Branch**: `003-centralize-nano64deepsleep`  
**Created**: 2026-07-29  
**Status**: Draft  
**Input**: GitHub Issue #3: "Move the many copies of nano64DeepSleep to central library for all nodes that sleep"

---

## User Scenarios & Testing

### User Story 1 - Baseline Library Extraction from WaterTankLevel (Priority: P1)
As a maintainer of LoRaFarmNet, I want the most up-to-date implementation of `nano64DeepSleep` from `LoraNodeWaterTankLevel` extracted as the starting baseline into a dedicated shared PlatformIO library repository (`Libraries/nano64DeepSleep`) without changing its initial functionality, so that a clean, standardized library module with `library.json`, `#pragma once`, `.h`, and `.cpp` structure is established.

**Why this priority**: Issue #3 explicitly specifies starting with `LoraNodeWaterTankLevel` because it contains the most recent debouncing, interrupt, and packaging toggle enhancements.

**Independent Test**: Verify `Libraries/nano64DeepSleep` contains `library.json`, `src/nano64DeepSleep.h`, and `src/nano64DeepSleep.cpp` with proper header guards and compiles as a PlatformIO library unit.

**Acceptance Scenarios**:
1. **Given** `LoraNodeWaterTankLevel/include/nano64DeepSleep.h`, **When** extracted into `Libraries/nano64DeepSleep`, **Then** the initial functionality is preserved exactly, converted to standard `.h` / `.cpp` files with `#pragma once` guards and a `library.json` manifest.

---

### User Story 2 - Comprehensive Multi-Repo Diff Audit & Interactive Reconciliation (Priority: P2)
As a maintainer, I want an extensive pairwise diff audit comparing all copies of `nano64DeepSleep.h` across `LoraNodeWaterTankLevel`, `LoraNodeDualPIR`, `LoraNodeVictron`, `LoraNodeButton`, and `LoraNodeRepeater`, with each difference presented and agreed upon one-at-a-time before incorporating into the unified implementation plan.

**Why this priority**: Node types have accumulated specific pin definitions, interrupt mode handlers, or wake pin logic. Each variation must be evaluated interactively to ensure no node functionality breaks.

**Independent Test**: Generate a complete diff matrix comparing function signatures, interrupt handlers, and macros across all sleeping node repositories.

**Acceptance Scenarios**:
1. **Given** the 5 node copies of `nano64DeepSleep.h`, **When** audited during planning, **Then** all differences are presented for maintainer review and agreement before finalizing `plan.md`.

---

### User Story 3 - Node Deprecation & platformio.ini Dependency Integration (Priority: P3)
As a developer, I want local copies of `nano64DeepSleep.h` removed from all sleeping node repositories (`LoraNodeButton`, `LoraNodeDualPIR`, `LoraNodeRepeater`, `LoraNodeVictron`, and `LoraNodeWaterTankLevel`) and replaced with `lib_deps = symlink://../../Libraries/nano64DeepSleep` in `platformio.ini`.

**Why this priority**: Eliminates code duplication across field node repositories and establishes a single point of maintenance for deep sleep logic.

**Independent Test**: Run `pio run` on all sleeping node repositories to confirm they compile cleanly against the shared library.

**Acceptance Scenarios**:
1. **Given** local `nano64DeepSleep.h` files, **When** removed and replaced with `lib_deps` referencing `Libraries/nano64DeepSleep`, **Then** `pio run` builds successfully for each sleeping node.

---

### User Story 4 - Build Verification & User Hardware Flashing Handoff (Priority: P4)
As a maintainer, I want the AI agent to only execute build verification (`pio run`) and explicitly pause to ask me to take over for flashing firmware to physical hardware and conducting manual testing (verifying sleep entry, WDT wake timing, and external pin interrupts) before any commits or Pull Requests are created.

**Why this priority**: Physical device flashing and hardware testing are strictly handled by the maintainer. The AI agent must never attempt to program physical devices or proceed without explicit maintainer confirmation post-build.

**Independent Test**: AI agent runs `pio run` on all target sleeping nodes, confirms zero compilation errors, and then pauses execution with a prompt handing over to the maintainer for physical hardware flashing and testing.

**Acceptance Scenarios**:
1. **Given** all sleeping nodes compiled cleanly via `pio run`, **When** build verification completes, **Then** the AI agent stops and asks the user to take over for physical hardware flashing and manual testing.

---

### User Story 5 - Pull Requests & GitHub Issue Closure (Priority: P5)
As a maintainer, I want feature branches/PRs submitted for user review after manual hardware verification succeeds, and GitHub Issue #3 closed on `toogooda/LoRaFarmNet` as the final task in the implementation plan upon completion.

**Why this priority**: Fulfills Constitution (v1.1.0) requirements for PR delivery and issue resolution.

**Independent Test**: Confirm GitHub Issue #3 is marked closed on `toogooda/LoRaFarmNet` after implementation and merge.

**Acceptance Scenarios**:
1. **Given** manual testing confirmation, **When** PRs are created and merged, **Then** GitHub Issue #3 on `toogooda/LoRaFarmNet` is closed.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST create a new standalone PlatformIO library in `Libraries/nano64DeepSleep` containing `library.json`, `src/nano64DeepSleep.h`, and `src/nano64DeepSleep.cpp`.
- **FR-002**: The baseline library code MUST be copied from `LoraNodeWaterTankLevel/include/nano64DeepSleep.h` without altering existing logic.
- **FR-003**: Header files MUST use `#pragma once` include guards instead of `#ifndef` macro wrappers.
- **FR-004**: System MUST perform an extensive diff audit comparing `nano64DeepSleep.h` across all 5 sleeping nodes (`WaterTankLevel`, `DualPIR`, `Victron`, `Button`, `Repeater`).
- **FR-005**: All implementation differences MUST be reviewed and agreed upon interactively with the maintainer during planning prior to code changes.
- **FR-006**: Sleeping node repositories MUST remove local copies of `nano64DeepSleep.h` from `src/` and `include/`.
- **FR-007**: Sleeping node repositories MUST specify `nano64DeepSleep` via `lib_deps` in `platformio.ini`.
- **FR-008**: All sleeping node firmwares MUST compile without errors via `pio run`.
- **FR-009**: The AI agent MUST only perform build verification (`pio run`) and MUST pause execution after a successful build to hand over to the maintainer for physical hardware flashing and manual device testing.
- **FR-010**: Implementation plan MUST include closing GitHub Issue #3 on `toogooda/LoRaFarmNet` as the final task.

### Key Entities

- **nano64DeepSleep Library**: Centralized PlatformIO library handling watchdog timer (WDT) sleep cycles, external pin interrupt wake handling, pin debouncing, and sleep state transitions for ATmega644PA nodes.

---

## Success Criteria

### Measurable Outcomes

- **SC-001**: Single centralized `nano64DeepSleep` library created under `Libraries/nano64DeepSleep`.
- **SC-002**: 100% of sleeping node projects (`LoraNodeButton`, `LoraNodeDualPIR`, `LoraNodeRepeater`, `LoraNodeVictron`, `LoraNodeWaterTankLevel`) build cleanly (`pio run`) using the shared library.
- **SC-003**: 0 local duplicate copies of `nano64DeepSleep.h` remain in individual node repositories.
- **SC-004**: GitHub Issue #3 on `toogooda/LoRaFarmNet` is closed upon task completion.

---

## Assumptions

- `LoRaNetGateway` and `LoraNodeDualGateController` do not use deep sleep functionality and do not contain `nano64DeepSleep.h`, so they are **OUT OF SCOPE**.
- `LoraNodeVictron` and `LoraNodeRepeater` both include and instantiate `nano64DeepSleep.h` in their `main.cpp`, so they are **IN SCOPE** alongside `LoraNodeWaterTankLevel`, `LoraNodeDualPIR`, and `LoraNodeButton` (total 5 node repos).
- `LoraNodeWaterTankLevel` contains the most refined version of `nano64DeepSleep.h`.
- The library targets `atmelavr` / ATmega644PA nodes running MightyCore framework.
