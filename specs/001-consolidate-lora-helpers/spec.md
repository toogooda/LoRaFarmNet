# Feature Specification: Consolidate LoRa Helpers into Single Header & Cpp Library

**Feature Branch**: `001-consolidate-lora-helpers`  
**Created**: 2026-07-28  
**Status**: Draft  
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

### User Story 2 - Consolidated Single Header & Cpp Library with Hardware Pin Safety (Priority: P2)
As a developer, I want `Ra01S`, `LoRaHelper`, and `LoraMsg` combined and simplified into **one single Header file (`.h`) and one single Implementation file (`.cpp`)** with `#pragma once` include guards while preserving hardware-specific `Pinout.h` pin mapping in each node/gateway.

**Why this priority**: Solves Issue #1 Problem 2 (no pragma/cpp files for most helpers) by replacing 4 scattered files per project with 1 pair of `.h`/`.cpp` files without breaking target-specific hardware pins (Gateway ESP32 vs ATmega644PA Nodes).

**Independent Test**: Verify that the combined `.h`/`.cpp` library compiles cleanly as a unified library unit and correctly accepts device-specific pins from each node's local `Pinout.h`.

**Acceptance Scenarios**:
1. **Given** the 4 separate files (`Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, `LoraMsg.cpp`), **When** combined into a single header and single `.cpp` file with `#pragma once` include guards, **Then** all radio initialization, frame serialization, encryption, and helper methods function in a single library pair.
2. **Given** device-specific hardware pin definitions in local `Pinout.h` (e.g. `LCSS`, `LRST`, `LBSY`, `LPWR`), **When** compiled for any node/gateway, **Then** device-specific pin mappings remain intact.

---

### User Story 3 - Incremental Replacement & Compilation Verification (Priority: P3)
As a developer, I want each node and gateway repository updated to replace the 4 legacy helper files with the new single-pair library so that local builds pass cleanly.

**Why this priority**: Replaces duplicated local files across all 7 repos and verifies compilation compatibility.

**Independent Test**: Execute `pio run` on each of the 7 device repositories.

**Acceptance Scenarios**:
1. **Given** any device repository in `LoRaFarmNet`, **When** updated to use the consolidated single-pair library and `pio run` is executed, **Then** compilation succeeds with zero errors or warnings.

---

### User Story 4 - Device Hardware Testing & GitHub PR Delivery (Priority: P4)
As a maintainer, I want individual GitHub Pull Requests opened for each updated device repository plus the root meta-repository after manual hardware verification so that each repo is cleanly versioned.

**Why this priority**: Respects the multi-repository structure of LoRaFarmNet while linking all changes to Issue #1.

**Independent Test**: Every modified repository has a dedicated branch, successful build check, and an open GitHub PR linked to Issue #1.

**Acceptance Scenarios**:
1. **Given** completed changes in a device repository, **When** verified via hardware testing, **Then** a Pull Request is opened for user review and approval (never auto-merged).

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST perform a complete pairwise diff audit across all 7 repositories before making source code edits.
- **FR-002**: System MUST combine `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, and `LoraMsg.cpp` into **one single Header file and one single C++ implementation file**.
- **FR-003**: The consolidated header MUST include proper `#pragma once` directives to prevent multi-definition compilation errors.
- **FR-004**: System MUST preserve device-specific hardware pin definitions in each repo's local `Pinout.h` (e.g. ESP32 Gateway vs ATmega644PA nodes).
- **FR-005**: System MUST preserve binary frame protocol invariants (`MI` first pair, `CS` checksum last pair, 6-byte hardware addressing).
- **FR-006**: Each node and gateway firmware MUST compile without errors using `pio run` after replacing legacy files with the single-pair library.
- **FR-007**: All device repositories MUST be updated via individual Pull Requests linked to GitHub Issue #1.
