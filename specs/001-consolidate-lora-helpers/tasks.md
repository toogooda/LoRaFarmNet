# Tasks: Consolidate LoRa Helpers into Single Header & Cpp Library

**Input**: Design documents from `specs/001-consolidate-lora-helpers/`  
**Prerequisites**: `plan.md` (required), `spec.md` (required), `research.md` (required)  

## Format: `[ID] [P?] [Story] Description`
- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: User story identifier (US1, US2, US3, US4)

---

## Phase 1: User Story 1 - Pairwise Diff Audit & Selection Approval (Priority: P1) 🎯 MVP

**Goal**: Document all variations in `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, `LoraMsg.cpp` across 7 repos and confirm merged signatures with maintainer.

**Independent Test**: Complete diff matrix verifying every macro, method signature, and pin import divergence.

- [x] T001 [US1] Complete pairwise diff audit of `Ra01S.h` across Gateway and 6 Node repos.
- [x] T002 [US1] Complete pairwise diff audit of `LoRaHelper.h` across Gateway and 6 Node repos.
- [x] T003 [US1] Complete pairwise diff audit of `LoraMsg.h` & `LoraMsg.cpp` across Gateway and 6 Node repos.
- [x] T004 [US1] Confirm unified API signatures (`getFromByte()` + `getMessageID()`) with maintainer.

**Checkpoint**: Story 1 Audit complete and approved by maintainer.

---

## Phase 2: User Story 2 - Consolidated Single Header & Cpp Library Assembly (Priority: P2)

**Goal**: Create a single `LoRaHelper.h` and `LoRaHelper.cpp` pair with `#pragma once` guards while preserving local `Pinout.h` hardware pin mapping.

**Independent Test**: Consolidated `.h`/`.cpp` files compile as a unified library unit.

- [ ] T005 [US2] Assemble unified `LoRaHelper.h` with `#pragma once`, `Ra01S` register definitions, default RF parameters, `PortValue` struct, `SX126x` class, and `LoraMsg` class.
- [ ] T006 [US2] Assemble unified `LoRaHelper.cpp` containing member implementations for `SX126x` transceiver methods and `LoraMsg` message serialization/encryption/CRC methods.
- [ ] T007 [US2] Verify `LoRaHelper.h` includes local `"Pinout.h"` to inherit target-specific hardware pins (`LCSS`, `LRST`, `LBSY`, `LPWR`, etc.).

**Checkpoint**: Unified library pair complete and ready for repository migration.

---

## Phase 3: User Story 3 - Repository Migration & Compilation Verification (Priority: P3)

**Goal**: Replace 4 legacy helper files in each of the 7 repositories with the unified `LoRaHelper.h`/`LoRaHelper.cpp` pair and verify clean builds with `pio run`.

**Independent Test**: `pio run` executes cleanly with zero errors on all 7 device projects.

- [ ] T008 [P] [US3] Replace legacy files in `Gateway/LoRaNetGateway/` and verify with `pio run`.
- [ ] T009 [P] [US3] Replace legacy files in `Nodes/LoraNodeButton/` and verify with `pio run`.
- [ ] T010 [P] [US3] Replace legacy files in `Nodes/LoraNodeDualGateController/` and verify with `pio run`.
- [ ] T011 [P] [US3] Replace legacy files in `Nodes/LoraNodeDualPIR/` and verify with `pio run`.
- [ ] T012 [P] [US3] Replace legacy files in `Nodes/LoraNodeRepeater/` and verify with `pio run`.
- [ ] T013 [P] [US3] Replace legacy files in `Nodes/LoraNodeVictron/` and verify with `pio run`.
- [ ] T014 [P] [US3] Replace legacy files in `Nodes/LoraNodeWaterTankLevel/` and verify with `pio run`.

**Checkpoint**: All 7 repositories compile cleanly using the consolidated library pair.

---

## Phase 4: User Story 4 - Manual Hardware Verification & GitHub PR Delivery (Priority: P4)

**Goal**: Perform manual hardware testing, push feature branches, and submit GitHub Pull Requests.

**Independent Test**: Dedicated PR opened for each modified repository and linked to Issue #1.

- [ ] T015 [US4] Pause for manual hardware testing across physical gateway and field nodes.
- [ ] T016 [US4] Ask user permission to push feature branches to remote repositories.
- [ ] T017 [US4] Open individual GitHub Pull Requests for all modified device repos and meta-repo.
- [ ] T018 [US4] Update and close GitHub Issue #1 upon PR approval.
