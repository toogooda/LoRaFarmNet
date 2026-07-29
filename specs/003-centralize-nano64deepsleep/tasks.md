# Tasks: Centralize nano64DeepSleep Library for Field Nodes

**Input**: Feature Specification (`specs/003-centralize-nano64deepsleep/spec.md`) & Implementation Plan (`specs/003-centralize-nano64deepsleep/plan.md`)

---

## Phase 1: Foundational Setup (Shared Library Repository)

- [x] T001 Create feature branch `feature/issue-3-centralize-nano64deepsleep` across `LoRaFarmNet` parent repo and the 5 sleeping node repos before making code changes
- [x] T002 Update GitHub Issue #3 on `toogooda/LoRaFarmNet` with "Started" status, feature branch name, and plan link
- [x] T003 Create `Libraries/nano64DeepSleep` directory structure with `src/` folder
- [x] T004 Create `Libraries/nano64DeepSleep/library.json` manifest configured for `atmelavr` / ATmega644PA
- [x] T005 Create `Libraries/nano64DeepSleep/README.md` with library usage instructions

---

## Phase 2: User Story 1 - Baseline Library Assembly (Priority: P1)

- [x] T006 [US1] Copy baseline logic from `Nodes/LoraNodeWaterTankLevel/include/nano64DeepSleep.h` into `Libraries/nano64DeepSleep/src/nano64DeepSleep.h` and `src/nano64DeepSleep.cpp`
- [x] T007 [US1] Add `#pragma once` header guard to `Libraries/nano64DeepSleep/src/nano64DeepSleep.h`
- [x] T008 [US1] Separate class declarations (`n64DS`) into `.h` and method implementations / ISR handlers into `.cpp`

---

## Phase 3: User Story 2 - Diff Audit Enhancements Integration (Priority: P2)

- [x] T009 [US2] Add `bool usePullup = true` parameter and `_pullupPinsMask` tracking to `enableWakeExternal()` in `Libraries/nano64DeepSleep/src/nano64DeepSleep.h` (Item 1 - `DualPIR`)
- [x] T010 [US2] Retain EIFR clearing (`clearPendingExternalInterrupt`), reed switch debouncing (`recordPackagingToggle`), and `wakeExternalSawLow` in `Libraries/nano64DeepSleep/src/nano64DeepSleep.cpp` (Item 2 - `WaterTankLevel`)
- [x] T011 [US2] Cleanly omit unused legacy functions (`wakeFromTimer`, `wakeFromExternal`, `setHigh`) from header (Item 3 - Header Cleanup)
- [x] T012 [US2] Add `lightSleep()` method and `unsigned long inMs = 0` prescaler parameter to `enableWakeTimer()` in `Libraries/nano64DeepSleep/src/nano64DeepSleep.h` and `.cpp` (Item 4 - `Repeater`)

---

## Phase 4: User Story 3 - Node Deprecation & platformio.ini Integration (Priority: P3)

- [x] T013 [US3] Remove local `nano64DeepSleep.h` from `Nodes/LoraNodeWaterTankLevel/include/`
- [x] T014 [US3] Remove local `nano64DeepSleep.h` from `Nodes/LoraNodeDualPIR/include/`
- [x] T015 [US3] Remove local `nano64DeepSleep.h` from `Nodes/LoraNodeVictron/include/`
- [x] T016 [US3] Remove local `nano64DeepSleep.h` from `Nodes/LoraNodeButton/src/`
- [x] T017 [US3] Remove local `nano64DeepSleep.h` from `Nodes/LoraNodeRepeater/src/`
- [x] T018 [US3] Add `lib_deps = symlink://../../Libraries/nano64DeepSleep` to `platformio.ini` across all 5 sleeping node repositories

---

## Phase 5: User Story 4 - Build Verification & User Hardware Flashing Handoff (Priority: P4)

- [x] T019 [US4] Run `pio run` build verification on all 5 sleeping node firmwares (`WaterTankLevel`, `DualPIR`, `Victron`, `Button`, `Repeater`)
- [x] T020 [US4] Confirm zero compilation errors and PAUSE execution to hand over to maintainer for physical hardware flashing & testing

---

## Phase 6: User Story 5 - Pull Requests & GitHub Issue Closure (Priority: P5)

- [x] T021 [US5] Initialize git repository in `Libraries/nano64DeepSleep` and create initial commit
- [x] T022 [US5] Push feature branches and create Pull Requests for `Libraries/nano64DeepSleep` and all 5 sleeping node repositories
- [x] T023 [US5] Close GitHub Issue #3 on `toogooda/LoRaFarmNet` as the final plan task upon PR merge
