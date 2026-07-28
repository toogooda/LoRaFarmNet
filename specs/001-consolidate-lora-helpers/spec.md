# Feature Specification: Consolidate LoRa Helpers into Shared Library

**Feature Branch**: `001-consolidate-lora-helpers`  
**Created**: 2026-07-28  
**Status**: Draft  
**Input**: GitHub Issue #1: "Consolidate LoRa helpers to a new shared library for all LoRaFarmNet Devices"

---

## User Scenarios & Testing

### User Story 1 - Feasibility & Pairwise Diff Audit (Priority: P1)
As a developer maintaining LoRaFarmNet, I want a complete diff audit of `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, and `LoraMsg.cpp` across all 7 device repositories before consolidation so that all repo-specific variations are identified and approved by the maintainer.

**Why this priority**: Must happen first. Consolidation cannot safely occur without knowing existing variations and resolving discrepancies with the maintainer.

**Independent Test**: Generate a comprehensive diff report comparing helper implementations across `Gateway` and all `Nodes`, highlighting any divergent macros, pin definitions, or logic.

**Acceptance Scenarios**:
1. **Given** helper implementations across `LoRaNetGateway`, `LoraNodeButton`, `LoraNodeDualGateController`, `LoraNodeDualPIR`, `LoraNodeRepeater`, `LoraNodeVictron`, and `LoraNodeWaterTankLevel`, **When** compared, **Then** all variations are documented and presented to the maintainer for selection.

---

### User Story 2 - Dedicated Private GitHub Repository & Shared Library Setup (Priority: P2)
As a maintainer, I want the consolidated LoRa helper library stored in its own **new private GitHub repository** (e.g., `toogooda/LoRaFarmNetCore` or `toogooda/LoRaFarmNetLibrary`) structured as a PlatformIO-compatible library so that all gateway and node repos can cleanly reference or submodule it.

**Why this priority**: Centralizes code ownership in a dedicated repository, making updates, versioning, and linking across all node/gateway projects seamless.

**Independent Test**: Verify that the new private GitHub repository is created, contains the merged headers (`Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, `LoraMsg.cpp`) with `#pragma once` guards and `library.json`, and compiles cleanly.

**Acceptance Scenarios**:
1. **Given** approved merged sources, **When** pushed to a new private GitHub repository (e.g. `toogooda/LoRaFarmNetCore`), **Then** it is accessible to all device repos via PlatformIO dependency (`lib_deps`) or Git submodules.
2. **Given** `#pragma once` guards and standard C++ declarations in the new library, **When** compiled, **Then** zero symbol duplication errors occur.

---

### User Story 3 - Repository Migration & Compilation Verification (Priority: P3)
As a developer, I want each node and gateway repository updated to reference the new private shared library repository so that local files are replaced and builds pass cleanly.

**Why this priority**: Replaces duplicated local files across all 7 repos and verifies compilation compatibility.

**Independent Test**: Execute `pio run` on each of the 7 device repositories after linking to the new shared library.

**Acceptance Scenarios**:
1. **Given** any device repository in `LoRaFarmNet`, **When** updated to link to the new shared library repo and `pio run` is executed, **Then** compilation succeeds with zero errors or warnings related to missing or duplicate LoRa headers.

---

### User Story 4 - Device Hardware Testing & GitHub PR Delivery (Priority: P4)
As a maintainer, I want individual GitHub Pull Requests opened for each updated device repository plus the root meta-repository and shared library repo after manual hardware verification so that each repo is cleanly versioned.

**Why this priority**: Respects the multi-repository structure of LoRaFarmNet while linking all changes to Issue #1.

**Independent Test**: Every modified repository has a dedicated branch, successful build check, and an open GitHub PR linked to Issue #1.

**Acceptance Scenarios**:
1. **Given** completed changes in a device repository, **When** verified via hardware testing, **Then** a Pull Request is opened for user review and approval (never auto-merged).

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST perform a complete pairwise diff audit across all 7 repositories before making source code edits.
- **FR-002**: System MUST create a new **private GitHub repository** (e.g. `toogooda/LoRaFarmNetCore`) structured as a PlatformIO library to host the consolidated helpers.
- **FR-003**: System MUST consolidate `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, and `LoraMsg.cpp` into the new shared library repository based on maintainer-approved variations.
- **FR-004**: All shared headers MUST include proper `#pragma once` directives to prevent multi-definition compilation errors.
- **FR-005**: System MUST preserve binary frame protocol invariants (`MI` first pair, `CS` checksum last pair, 6-byte hardware addressing).
- **FR-006**: Each node and gateway firmware MUST compile without errors using `pio run` after linking to the new shared library repository.
- **FR-007**: All device repositories MUST be updated via individual Pull Requests linked to GitHub Issue #1.

---

## Technical Feasibility & Audit Plan

1. **Step 1 - Diff Audit**: Perform pairwise diff comparison of `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, `LoraMsg.cpp` across all 7 projects.
2. **Step 2 - New Repository & Shared Library Creation**: Create new private GitHub repo (`toogooda/LoRaFarmNetCore`), assemble merged files, add `#pragma once`, and create PlatformIO library structure.
3. **Step 3 - Incremental Replacement & Compilation**: Link each device repo to the new shared library repo and run `pio run`.
4. **Step 4 - User Testing & Manual Verification**: Pause for user hardware testing per device.
5. **Step 5 - Multi-Repo PR Delivery**: Open PRs for all affected repositories (including new library repo & meta-repo) and close Issue #1 upon final merge.
