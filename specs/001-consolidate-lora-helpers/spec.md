# Feature Specification: Consolidate LoRa Helpers into Shared Library Repository

**Feature Branch**: `001-consolidate-lora-helpers`  
**Created**: 2026-07-28  
**Status**: Complete  
**Input**: GitHub Issue #1: "Consolidate LoRa helpers to a new shared library for all LoRaFarmNet Devices"

---

## User Scenarios & Testing

### User Story 1 - Feasibility & Pairwise Diff Audit (Priority: P1)
As a developer maintaining LoRaFarmNet, I want a complete diff audit of `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, and `LoraMsg.cpp` across all 7 device repositories before consolidation so that all repo-specific variations are identified and presented for maintainer selection.

**Why this priority**: Must happen first to evaluate how the 4 separate helper files across 7 repos can be cleanly merged into one unified Header and one C++ file without dropping features.

**Independent Test**: Generate a diff matrix comparing helper implementations across `Gateway` and all `Nodes`, highlighting divergent macros, pin imports, and method access.

**Acceptance Scenarios**:
1. **Given** helper implementations across `LoRaNetGateway`, `LoraNodeButton`, `LoraNodeDualGateController`, `LoraNodeDualPIR`, `LoraNodeRepeater`, `LoraNodeVictron`, and `LoraNodeWaterTankLevel`, **When** compared, **Then** all variations are documented and presented to the maintainer for selection.

---

### User Story 2 - Dedicated Shared Library Repository Assembly (Priority: P2)
As a developer, I want `Ra01S`, `LoRaHelper`, and `LoraMsg` combined into a **dedicated standalone library repository (`LoRaNetLibrary`)** containing `library.json`, `src/LoRaHelper.h`, and `src/LoRaHelper.cpp` with `#pragma once` include guards.

**Why this priority**: Solves Issue #1 Problem 2 and avoids copying library code into individual application repositories by maintaining a single reusable PlatformIO library repository (`toogooda/LoRaNetLibrary`).

**Independent Test**: Verify that the new `Libraries/LoRaNetLibrary` repo structure (`library.json`, `src/LoRaHelper.h`, `src/LoRaHelper.cpp`) is valid and compiles cleanly as a PlatformIO library unit.

**Acceptance Scenarios**:
1. **Given** the 4 separate helper files, **When** combined into a standalone library repository `LoRaNetLibrary` with `library.json` and `#pragma once` include guards, **Then** a single version-controlled library handles radio initialization, frame serialization, encryption, and CRC methods.

---

### User Story 3 - Local Code Cleanup & platformio.ini Dependency Integration (Priority: P3)
As a developer, I want each node and gateway repository updated to **remove local duplicate helper files** and include `LoRaNetLibrary` via `lib_deps` in `platformio.ini` so that local builds pass cleanly without duplicated code.

**Why this priority**: Replaces duplicated local files in `src/` across all 7 repos with a central library dependency link in `platformio.ini`.

**Independent Test**: Execute `pio run` on each of the 7 device repositories after removing local helper files and adding `lib_deps` entry.

**Acceptance Scenarios**:
1. **Given** any device repository in `LoRaFarmNet`, **When** local helper files are removed, `platformio.ini` includes `lib_deps = symlink://../../Libraries/LoRaNetLibrary` (or `https://github.com/toogooda/LoRaNetLibrary.git`), and `pio run` is executed, **Then** compilation succeeds cleanly.

---

### User Story 4 - Device Hardware Testing & GitHub PR Delivery (Priority: P4)
As a maintainer, I want individual GitHub Pull Requests opened for each updated device repository, the new `LoRaNetLibrary` repository, and the root meta-repository after manual hardware verification.

**Why this priority**: Respects the multi-repository structure of LoRaFarmNet while delivering the new library repo and updated application repos.

**Independent Test**: Dedicated PR opened for `LoRaNetLibrary` and each modified device repo linked to Issue #1.

**Acceptance Scenarios**:
1. **Given** completed changes, **When** verified via hardware testing, **Then** Pull Requests are opened for user review and approval (never auto-merged).

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST perform a complete pairwise diff audit across all 7 repositories before making source code edits.
- **FR-002**: System MUST combine `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, and `LoraMsg.cpp` into a dedicated standalone PlatformIO library repository (`LoRaNetLibrary`).
- **FR-003**: The library MUST include `library.json` with metadata for `atmelavr` and `espressif32` platforms.
- **FR-004**: The consolidated header MUST include proper `#pragma once` directives to prevent multi-definition compilation errors.
- **FR-005**: Target application repositories (`Gateway` and 6 `Nodes`) MUST include `LoRaNetLibrary` in `platformio.ini` via `lib_deps`.
- **FR-006**: Target application repositories MUST NOT contain local copies of `LoRaHelper.h` or `LoRaHelper.cpp` in their `src/` directories.
- **FR-007**: Each node and gateway firmware MUST compile without errors using `pio run`.
- **FR-008**: All repositories (new library repo and modified device repos) MUST be submitted via individual Pull Requests linked to GitHub Issue #1.

