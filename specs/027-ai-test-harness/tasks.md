# Tasks: AI Test Harness API & Autonomous Gateway Verification

**Input**: Design documents from `specs/027-ai-test-harness/`  
**Prerequisites**: [spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/027-ai-test-harness/spec.md), [plan.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/027-ai-test-harness/plan.md)

---

## Phase 1: Setup & Foundational Infrastructure

**Purpose**: Create dedicated test harness header, declare global in-memory state, and hook into Gateway web server.

- [x] T001 Create `Gateway/LoRaNetGateway/src/AITestHarness.h` with test mode variables (`testModeActive`, `testModeExpiry`) and authorization helper `isTestModeAuthorized(request)`.
- [x] T002 In `Gateway/LoRaNetGateway/src/main.cpp`, include `AITestHarness.h` and call `setupAITestHarnessRoutes(server)` inside `setupWebServer()`.

---

## Phase 2: User Story 1 - Test Mode Activation & Security Guarding (Priority: P1) 🎯 MVP

**Goal**: Implement test mode toggle guarded by `isauthorized()` with 1-hour auto-timeout and strict 403 protection on all test endpoints.

- [x] T003 [US1] Implement `POST /settings/testmode` in `AITestHarness.h` (accepting `enable=1/0`, verifying `authorized(request)`, and arming the 3600000ms safety timer).
- [x] T004 [US1] Add visual Test Mode status card and toggle switch in `Gateway/LoRaNetGateway/src/WebHelper.h` under the `/settings` page.
- [x] T005 [US1] Verify that all `/api/test/*` endpoints enforce `isTestModeAuthorized()` and immediately return HTTP `403 Forbidden` when Test Mode is disabled or expired.

---

## Phase 3: User Story 2 - Synthetic LoRa Message Injection (Priority: P1) 🎯 MVP

**Goal**: Implement `POST /api/test/message` accepting JSON and raw hex frames to inject directly into `farmNet->processNewMessage(*lMsg, rssi, snr)`.

- [x] T006 [US2] Implement binary packet builder helper in `AITestHarness.h` that converts JSON (`from`, `to`, `ports` map) into a binary byte buffer with `MI` header and calculated 16-bit `CS` CRC.
- [x] T007 [US2] Implement `POST /api/test/message` endpoint in `AITestHarness.h` supporting both JSON (`{"from":"...", "to":"010101010101", "rssi":-110, "snr":-4, "ports":{...}}`) and raw hex (`{"rawHex":"..."}`).
- [x] T008 [US2] Feed constructed `LoraMsg` directly into `farmNet->processNewMessage(*lMsg, rssi, snr)`, triggering sensor updates, translations, highlights, new device detection, and MQTT dispatch.

---

## Phase 4: User Story 3 - Device State Inspection & Network Snapshot/Restore (Priority: P1)

**Goal**: Implement full JSON object tree inspection (`GET /api/test/device`), device deletion (`DELETE /api/test/device`), and atomic snapshot/restore (`POST /api/test/snapshot`, `POST /api/test/restore`).

- [x] T009 [US3] Implement `GET /api/test/device?id=<hex>` in `AITestHarness.h` using `ArduinoJson` (v7) to serialize device metadata, sensor values, child entities, translation rules, and highlight rules.
- [x] T010 [US3] Implement `DELETE /api/test/device?id=<hex>` in `AITestHarness.h` to invoke `farmNet->removeDevice(mac)` and `sdHelper.saveNetwork(farmNet)`.
- [x] T011 [US3] Implement `POST /api/test/snapshot` in `AITestHarness.h` to atomically copy `/lfm/data/network.dat` to `/lfm/data/test_snapshot.bak`.
- [x] T012 [US3] Implement `POST /api/test/restore` in `AITestHarness.h` to restore `/lfm/data/test_snapshot.bak` to `network.dat` and call `farmNet->loadNetwork()` to reset PSRAM memory state.

---

## Phase 5: User Story 4 - System Telemetry, Clock Simulation & Mock OTA (Priority: P2)

**Goal**: Implement system status querying, clock offset simulation for timeouts/heartbeats, and mock OTA test hooks.

- [x] T013 [US4] Implement `GET /api/test/status` in `AITestHarness.h` returning free heap, free PSRAM, uptime, SD status, device counts, and telemetry counters.
- [x] T014 [US4] Implement `POST /api/test/time` in `AITestHarness.h` with simulated time offset (`simulatedTimeOffsetSeconds`) for validating device heartbeat decay and offline indicators.
- [x] T015 [US4] Implement `POST /api/test/trigger_ota` in `AITestHarness.h` for simulated OTA firmware upgrade evaluations.

---

## Phase 6: User Story 5 - Automated Test Client & AI Autonomous Testing Rules (Priority: P2)

**Goal**: Create command-line test runner script and update AI instructions with test harness protocols and serial monitoring.

- [x] T016 [US5] Create `Gateway/LoRaNetGateway/scripts/test_gateway_harness.py` supporting command-line test execution (`--ip`, `--enable-test-mode`, `--inject`, `--query`, `--snapshot`, `--restore`, `--run-suite`).
- [x] T017 [US5] Create `.agents/rules/ai_testing_workflow.md` detailing autonomous testing procedures, test snapshot/restore rules, and checking `platformio.ini` for serial port (`upload_port`) and baud rate (`monitor_speed = 115200`).
- [x] T018 [US5] Update `.specify/memory/constitution.md` with the autonomous testing quality gate.

---

## Phase 7: Build Verification, Automated Test Run & Release

**Goal**: Compile gateway firmware, run automated regression tests via test harness, submit Pull Request, and close Issue #27.

- [x] T019 Run local build verification (`pio run`) in `Gateway/LoRaNetGateway`.
- [ ] T020 Flash firmware (`pio run -t upload`), monitor serial port from `platformio.ini` at 115200 baud until `IP Address:<ip>` is captured, wait 3 seconds, then execute automated regression test suite using `test_gateway_harness.py` against live Gateway to verify end-to-end injection, query, and cleanup.
- [ ] T021 Push feature branch `027-ai-test-harness` and create Pull Requests on `toogooda/LoRaNetGateway` and `toogooda/LoRaFarmNet`.
- [ ] T022 Post final completion report to GitHub Issue #27 and close the issue upon PR merge.
