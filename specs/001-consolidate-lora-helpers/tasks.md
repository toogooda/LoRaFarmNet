# Tasks: Consolidate LoRa Helpers into Shared Library Repository

**Input**: Design documents from `specs/001-consolidate-lora-helpers/`  
**Prerequisites**: `plan.md` (required), `spec.md` (required), `research.md` (required)  

## Format: `[ID] [P?] [Story] Description`
- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: User story identifier (US1, US2, US3, US4)

---

## Phase 1: User Story 1 - Pairwise Diff Audit & Selection Approval (Priority: P1) 🎯 MVP

- [x] T001 [US1] Complete pairwise diff audit of `Ra01S.h` across Gateway and 6 Node repos.
- [x] T002 [US1] Complete pairwise diff audit of `LoRaHelper.h` across Gateway and 6 Node repos.
- [x] T003 [US1] Complete pairwise diff audit of `LoraMsg.h` & `LoraMsg.cpp` across Gateway and 6 Node repos.
- [x] T004 [US1] Confirm unified API signatures (`getFromByte()` + `getMessageID()`) with maintainer.

**Checkpoint**: Story 1 Audit complete and approved by maintainer.

---

## Phase 2: User Story 2 - Dedicated Shared Library Repository Assembly (Priority: P2)

**Goal**: Create standalone library repo `Libraries/LoRaNetLibrary` with `library.json`, `src/LoRaHelper.h`, `src/LoRaHelper.cpp`, and push `v1.0.1` release to GitHub (`toogooda/LoRaNetLibrary`).

**Independent Test**: Library unit structure is valid and published to GitHub.

- [x] T005 [US2] Create directory structure `Libraries/LoRaNetLibrary/src`.
- [x] T006 [US2] Create `library.json` manifest with metadata for Arduino framework, `atmelavr` and `espressif32` platforms.
- [x] T007 [US2] Create unified `Libraries/LoRaNetLibrary/src/LoRaHelper.h` with `#pragma once`, `Ra01S` register definitions, default RF parameters, `PortValue` struct, `SX126x` class, `LoraMsg` class, and `extern SX126x lora;` declaration.
- [x] T008 [US2] Create unified `Libraries/LoRaNetLibrary/src/LoRaHelper.cpp` containing member implementations for `SX126x` transceiver methods and `LoraMsg` message serialization/encryption/CRC methods.
- [x] T009 [US2] Add `README.md` to `Libraries/LoRaNetLibrary` with usage documentation.

**Checkpoint**: Standalone `LoRaNetLibrary` repository published to GitHub (`toogooda/LoRaNetLibrary.git#v1.0.1`).

---

## Phase 3: User Story 3 - Local Code Cleanup & platformio.ini Dependency Integration (Priority: P3)

**Goal**: Remove duplicate local helper files from `src/` of all 7 device repos, add `https://github.com/toogooda/LoRaNetLibrary.git#v1.0.1` to `lib_deps` in `platformio.ini`, and verify clean `pio run` builds.

**Independent Test**: `pio run` executes cleanly with zero errors on all 7 device projects.

- [x] T010 [P] [US3] Remove local `LoRaHelper.h`/`LoRaHelper.cpp` from `Gateway/LoRaNetGateway/src/`, add `lib_deps` entry in `platformio.ini`, and verify with `pio run`.
- [x] T011 [P] [US3] Remove local `LoRaHelper.h`/`LoRaHelper.cpp` from `Nodes/LoraNodeButton/src/`, add `lib_deps` entry in `platformio.ini`, and verify with `pio run`.
- [x] T012 [P] [US3] Remove local `LoRaHelper.h`/`LoRaHelper.cpp` from `Nodes/LoraNodeDualGateController/src/`, add `lib_deps` entry in `platformio.ini`, and verify with `pio run`.
- [x] T013 [P] [US3] Remove local `LoRaHelper.h`/`LoRaHelper.cpp` from `Nodes/LoraNodeDualPIR/src/`, add `lib_deps` entry in `platformio.ini`, and verify with `pio run`.
- [x] T014 [P] [US3] Remove local `LoRaHelper.h`/`LoRaHelper.cpp` from `Nodes/LoraNodeRepeater/src/`, add `lib_deps` entry in `platformio.ini`, and verify with `pio run`.
- [x] T015 [P] [US3] Remove local `LoRaHelper.h`/`LoRaHelper.cpp` from `Nodes/LoraNodeVictron/src/`, add `lib_deps` entry in `platformio.ini`, and verify with `pio run`.
- [x] T016 [P] [US3] Remove local `LoRaHelper.h`/`LoRaHelper.cpp` from `Nodes/LoraNodeWaterTankLevel/src/`, add `lib_deps` entry in `platformio.ini`, and verify with `pio run`.

**Checkpoint**: All 7 repositories compile cleanly using `LoRaNetLibrary` from `platformio.ini`.

---

## Phase 4: User Story 4 - Manual Hardware Verification & GitHub PR Delivery (Priority: P4)

**Goal**: Perform manual hardware testing, push feature branches (including new library repo), and submit GitHub Pull Requests.

**Independent Test**: Dedicated PR opened for `LoRaNetLibrary` and each modified device repository linked to Issue #1.

- [x] T017 [US4] Pause for manual hardware testing across physical gateway and field nodes.
- [x] T018 [US4] Ask user permission to initialize git repo for `LoRaNetLibrary` and push feature branches to remote repositories.
- [x] T019 [US4] Open individual GitHub Pull Requests for `LoRaNetLibrary`, all modified device repos, and meta-repo.
- [x] T020 [US4] Update and close GitHub Issue #1 upon PR approval.

