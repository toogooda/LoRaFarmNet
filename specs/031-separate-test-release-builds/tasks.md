# Tasks: Move to Separate Test and Release Builds

**Input**: Design documents from `specs/031-separate-test-release-builds/` (`spec.md`, `plan.md`)  
**Feature Branch**: `031-separate-test-release-builds`  
**GitHub Issue**: [#31](https://github.com/toogooda/LoRaFarmNet/issues/31)

---

## Phase 1: Setup & PlatformIO Environment Architecture

**Purpose**: Configure `gateway-test` (default) and `gateway-release` environments in `platformio.ini`.

- [x] T001 Restructure `Gateway/LoRaNetGateway/platformio.ini`:
  - Extract common board, framework, partition, and upload settings to base `[env]`.
  - Set `default_envs = gateway-test`.
  - Define `[env:gateway-test]` with `build_flags = -DBOARD_HAS_PSRAM -mfix-esp32-psram-cache-issue -DGATEWAY_TEST_BUILD`.
  - Define `[env:gateway-release]` with `build_flags = -DBOARD_HAS_PSRAM -mfix-esp32-psram-cache-issue` and `extra_scripts = post:scripts/publish_release.py`.

---

## Phase 2: User Story 1 (Priority: P1 🎯 MVP) - Test Build Always-Active Test APIs

**Goal**: In test builds, `/api/test/*` endpoints are compiled in and permanently active without requiring manual arming or timing out.

- [x] T002 Wrap `setupTestEndpoints()` and all `/api/test/*` route handlers in `Gateway/LoRaNetGateway/src/WebHelper.h` with `#ifdef GATEWAY_TEST_BUILD`.
- [x] T003 Remove `testModeActive` check inside test handlers so test APIs are always active in test builds.
- [x] T004 Wrap `setupTestEndpoints(server)` invocation in `Gateway/LoRaNetGateway/src/main.cpp` with `#ifdef GATEWAY_TEST_BUILD`.

---

## Phase 3: User Story 2 (Priority: P1 🎯 MVP) - Visual "TEST BUILD" Badge & Blue Brand Title

**Goal**: Style the brand title in blue and display the test flask badge in the navbar across all pages on test builds.

- [x] T005 Update `sendPageHeader()` in `Gateway/LoRaNetGateway/src/WebHelper.h` to apply `text-info` (blue) brand title and render `<span class='badge bg-warning text-dark me-2 font-monospace'><i class='fa-solid fa-flask-vial me-1'></i>TEST BUILD</span>` when `#ifdef GATEWAY_TEST_BUILD` is defined.

---

## Phase 4: User Story 3 (Priority: P1 🎯 MVP) - Production Release Stripping of Test UI Artifacts & Debug Logs

**Goal**: Omit all demo simulation buttons, developer test controls, devtype delete capabilities, and message/ACK Serial debug logs in production release builds.

- [x] T006 Enclose `[🧪 Simulate Router Response (Demo)]` button and simulator JS on `/adddevice` in `Gateway/LoRaNetGateway/src/WebHelper.h` with `#ifdef GATEWAY_TEST_BUILD`.
- [x] T007 Enclose developer test controls and test mode switches in `sendSettingsPage()` and `sendFirmwarePage()` in `Gateway/LoRaNetGateway/src/WebHelper.h` with `#ifdef GATEWAY_TEST_BUILD`.
- [x] T008 Enclose Device Types and Device Categories delete buttons and delete action URL handlers in `Gateway/LoRaNetGateway/src/WebHelper.h` with `#ifdef GATEWAY_TEST_BUILD`.
- [x] T009 Enclose message transmission and return ACK `Serial.println()` / `Serial.printf()` logs in `Gateway/LoRaNetGateway/src/FarmNetwork.cpp` with `#ifdef GATEWAY_TEST_BUILD`.

---

## Phase 5: User Story 4 (Priority: P2) - Release Automation & Harness Synchronization

**Goal**: Ensure `pio run -t release` packages the clean release binary and regression tests assert the test build state.

- [x] T010 Update `Gateway/LoRaNetGateway/scripts/publish_release.py` to package `gateway-release` binary from `.pio/build/gateway-release/firmware.bin`.
- [x] T011 Update `Gateway/LoRaNetGateway/scripts/test_gateway_harness.py` to assert `TEST BUILD` badge and always-active status.

---

## Phase 6: Automated Dual-Surface Verification & Quality Gates

**Goal**: End-to-end automated testing across both build environments, live hardware flashing, and manual testing pause.

- [x] T012 Compile test environment with `pio run -e gateway-test` and verify zero errors.
- [x] T013 Compile release environment with `pio run -e gateway-release` and verify zero errors and stripped size.
- [x] T014 Flash test build to Gateway hardware (`pio run -t upload`) and run automated dual-surface regression suite.
- [x] T015 Pause for user manual device testing and verification before requesting commit and Pull Request creation.
