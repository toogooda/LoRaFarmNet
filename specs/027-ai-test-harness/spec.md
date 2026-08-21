# Feature Specification: AI Test Harness API & Autonomous Gateway Verification

**Feature Branch**: `027-ai-test-harness`  
**Created**: 2026-08-21  
**Status**: Draft  
**Input**: GitHub Issue #27 ("Test harness API so AI can fully test new features going forward")

---

## Overview

Provide a dedicated, secure HTTP test harness API (`AITestHarness`) on the ESP32 LoRa Gateway. This allows the AI coding assistant (or automated test scripts) to inject synthetic LoRa radio frames, inspect internal device/entity state, simulate system time/heartbeats, trigger mock OTA flows, and clean up test data without requiring physical node hardware.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Test Mode Activation & Security Guarding (Priority: P1) 🎯 MVP

As an administrator or developer on the local network, I want to enable and disable Test Mode from the Gateway Settings page so that test harness endpoints can only be accessed intentionally and remain strictly blocked in normal operation.

**Why this priority**: Core safety invariant. Prevents unauthorized or accidental mock data injection in production environments.

**Independent Test**:
Can be fully tested by attempting to access `/api/test/*` endpoints while Test Mode is OFF (verifying `403 Forbidden` response), enabling Test Mode via Settings page (or authorized endpoint), and verifying endpoints become accessible.

**Acceptance Scenarios**:
1. **Given** Test Mode is OFF, **When** any request is made to `/api/test/*`, **Then** the Gateway responds with `403 Forbidden` and does not process the request.
2. **Given** an authorized user is on local WiFi, **When** they toggle Test Mode ON in Settings (`POST /settings/testmode?enable=1`), **Then** Test Mode is enabled in memory with a 1-hour safety auto-timeout.
3. **Given** Test Mode is ON, **When** the Gateway is rebooted or 1 hour elapses, **Then** Test Mode automatically reverts to OFF.

---

### User Story 2 - Synthetic LoRa Message Injection (Priority: P1) 🎯 MVP

As an AI developer or automated test suite, I want to send synthetic LoRa messages to the Gateway via HTTP POST (in structured JSON or raw binary Hex format) so that the Gateway processes the packet through its exact radio reception pipeline (`LoraMsg` decoding, sensor value updates, pipeline execution, new device detection, and MQTT publishing).

**Why this priority**: Fundamental requirement for testing Gateway features (online device types, entity rules, translations, pipelines, and routing) without physical LoRa nodes.

**Independent Test**:
Send a POST request with a realistic LoRa message payload matching the on-air binary protocol (e.g., Source `FC0FE71463C5`, `MI=14`, `IV=325`, `SM=15`, `DT=0`, `RSSI=-110`, `SNR=-4`) and verify the Gateway creates/updates the device, updates sensor values, and surfaces it on the web UI and MQTT.

**Acceptance Scenarios**:
1. **Given** Test Mode is ON, **When** a POST request is made to `/api/test/message` with JSON payload:
   ```json
   {
     "from": "FC0FE71463C5",
     "to": "010101010101",
     "rssi": -110,
     "snr": -4,
     "ports": {
       "MI": 14,
       "IV": 325,
       "SM": 15,
       "DT": 0
     }
   }
   ```
   *(Note: `"to": "010101010101"` is the LoRaFarmNet standard Gateway destination address, enabling nodes to deliver telemetry to the Gateway without individual MAC pairing).*
   **Then** the Gateway automatically wraps the ports with the `MI` header and calculated `CS` CRC checksum, constructs a `LoraMsg` instance, passes it to `farmNet->processNewMessage(*lMsg, rssi, snr)`, and returns `{"success": true, "deviceId": "fc0fe71463c5", "portsUpdated": 4}`.
2. **Given** Test Mode is ON, **When** a POST request is made to `/api/test/message` with multi-port telemetry (e.g., Victron `DT=2`, `MI=3563`, `V_=13680`, `I_=170`, `VV=47870`, `PV=4`, `SM=15`, `CS=6811`), **Then** all 20+ sensor values are updated in `FarmNetwork` and corresponding translation/highlight rules fire.
3. **Given** Test Mode is ON, **When** a POST request is made to `/api/test/message` with a raw binary frame hex string `{"rawHex": "010101010101fc0fe71463c5..."}`, **Then** the Gateway deserializes the binary stream, validates CRC `CS`, and processes the packet through the exact physical reception path.
4. **Given** an injected message with an unknown `DT` (e.g., `DT=99`), **When** the message is processed, **Then** the Gateway registers the device as `DeviceType::New` with the "Unknown Device Type" status and provides the "Check Online" action.

---

### User Story 3 - Device State Inspection & Deletion (Priority: P1)

As an AI developer, I want to query the full object hierarchy of a device (SensorValues, Entities, Translations, Highlights, Actions, Pipelines) and delete test devices after verification so that I can validate state assertions and perform clean tear-downs.

**Why this priority**: Enables deep programmatic verification of internal gateway state beyond HTML screen scraping and provides cleanup mechanisms to prevent test pollution.

**Independent Test**:
Query a device via `GET /api/test/device?id=010203040506` to inspect JSON entity/translation structures, then send `DELETE /api/test/device?id=010203040506` and verify the device is removed from memory and SD.

**Acceptance Scenarios**:
1. **Given** a device exists in `FarmNetwork`, **When** a `GET /api/test/device?id=<hex_mac>` request is made, **Then** the Gateway returns a JSON object detailing device metadata, sensor values, linked entities, translation rules, highlight rules, and configured actions/pipelines.
2. **Given** a test device exists, **When** a `DELETE /api/test/device?id=<hex_mac>` request is made, **Then** the Gateway removes the device from memory, updates `network.dat` atomically, and returns `{"success":true}`.

---

### User Story 4 - System Telemetry, Clock Simulation & Mock OTA (Priority: P2)

As an AI developer, I want to inspect system metrics (heap, uptime, SD status, device count), simulate system time for heartbeat/timeout checks, and trigger mock OTA flows so that edge-case timing and upgrade logic can be validated autonomously.

**Why this priority**: Allows verification of time-dependent features (heartbeat decay, offline timeouts) and OTA upgrade flows without waiting for real-world clock progression.

**Independent Test**:
Call `POST /api/test/time` to advance simulated clock, verify device heartbeat indicators change on UI, and query `GET /api/test/status` for telemetry.

**Acceptance Scenarios**:
1. **Given** Test Mode is ON, **When** `GET /api/test/status` is called, **Then** the Gateway returns JSON with free heap, PSRAM, uptime, SD status, device counts (configured, new, ignored), and current telemetry counters.
2. **Given** Test Mode is ON, **When** `POST /api/test/time` is called with timestamp offset `{"offsetSeconds": 900}`, **Then** the system calculates last seen intervals and health metrics using the simulated time.
3. **Given** Test Mode is ON, **When** `POST /api/test/trigger_ota` is called with simulated version `{"simulatedVersion": "1.4.0"}`, **Then** the Gateway executes the OTA check/upgrade cycle with mock parameters.

---

### User Story 5 - AI Workflow Rules, Autonomous Verification & Serial Debugging (Priority: P2)

As a developer working with the AI coding assistant, I want the AI instructions (`.agents/rules/` and constitution) updated so that the AI autonomously uses `AITestHarness` and checks `platformio.ini` for the active serial port (`upload_port` / `monitor_port`) and `monitor_speed` (115200 baud) to monitor live serial logs and validate Gateway changes before pausing for manual human confirmation.

**Why this priority**: Fulfills the core objective of Issue #27 — enabling the AI to compile, flash, monitor serial output for panics/logs, inject test cases via HTTP API, self-heal issues, and present fully verified work to the human user.

**Independent Test**:
Review `.agents/rules/` and verify that the mandatory checklist includes reading `platformio.ini` for serial monitoring parameters and running automated test injections via `AITestHarness` during the verification phase.

**Acceptance Scenarios**:
1. **Given** an AI agent implementing a Gateway feature, **When** code compilation passes, **Then** the AI inspects `platformio.ini` to discover the configured serial port (e.g., `COM3`) and `monitor_speed` (e.g., `115200`), asks for the Gateway IP address (if not already known), enables Test Mode, executes test injections via `AITestHarness`, observes serial logs, validates JSON responses, and restores network state before requesting final human verification.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The Gateway MUST provide a `TestMode` state (defaulting to `OFF` on boot) that guards all `/api/test/*` endpoints.
- **FR-002**: Enabling Test Mode MUST be restricted to authorized users via `isauthorized()` from `/settings` and MUST include an in-memory auto-disable safety timer (1 hour).
- **FR-003**: The Gateway MUST provide `POST /api/test/message` accepting JSON payloads `{"from":"<6B_HEX>","to":"<6B_HEX>","rssi":<INT>,"snr":<INT>,"ports":{"<PORT_KEY>": <VALUE_INT>}}` to construct binary `LoraMsg` frames (with `MI` header and calculated `CS` CRC) and inject them into `farmNet->processNewMessage(*lMsg, rssi, snr)`.
- **FR-004**: The Gateway MUST support raw binary hex string injection via `POST /api/test/message` `{"rawHex":"<HEX_STREAM>"}` for validating exact binary stream parsing and CRC verification.
- **FR-005**: The Gateway MUST provide `GET /api/test/device?id=<6B_HEX>` returning complete JSON representation of device state (name, uniqueId, type, routerId, sensorValues, entities, translations, highlights, actions, pipelines).
- **FR-006**: The Gateway MUST provide `DELETE /api/test/device?id=<6B_HEX>` to safely remove test devices from memory and SD card.
- **FR-007**: The Gateway MUST provide `GET /api/test/status` returning system health, heap memory, PSRAM memory, SD mount status, network device counts, and telemetry counters.
- **FR-008**: The Gateway MUST provide `POST /api/test/time` to simulate timestamp offsets for testing device heartbeats, timeouts, and offline indicators.
- **FR-009**: The Gateway MUST provide `POST /api/test/trigger_ota` to trigger simulated OTA version checks without requiring physical release tagging.
- **FR-010**: All `/api/test/*` endpoints MUST return standard JSON responses with status codes (`200 OK`, `400 Bad Request`, `403 Forbidden`, `404 Not Found`).
- **FR-011**: Project instructions in `.agents/rules/` and `.specify/memory/constitution.md` MUST be updated to guide the AI on utilizing `AITestHarness` during development.

---

### Key Endpoints & Architecture

#### Dedicated Test Harness Endpoints (`AITestHarness.h`)
All test harness logic is cleanly isolated in `AITestHarness.h` (registered via `setupAITestHarnessRoutes(server)`):

| Method | Endpoint | Description | Security |
|---|---|---|---|
| `POST` | `/settings/testmode` | Toggle Test Mode ON/OFF (1-hour auto-timeout) | `isauthorized()` |
| `POST` | `/api/test/message` | Inject synthetic LoRa message (JSON or Raw Hex) into radio pipeline | Test Mode ON |
| `GET` | `/api/test/device` | Query device full state hierarchy in JSON for programmatic assertions | Test Mode ON |
| `DELETE` | `/api/test/device` | Delete test device by MAC (targeted teardown) | Test Mode ON |
| `POST` | `/api/test/snapshot` | Create atomic pre-test backup snapshot of `network.dat` | Test Mode ON |
| `POST` | `/api/test/restore` | Restore pre-test snapshot and reload `FarmNetwork` in memory (instant full cleanup) | Test Mode ON |
| `GET` | `/api/test/status` | Query system status, heap, PSRAM, and telemetry | Test Mode ON |
| `POST` | `/api/test/time` | Apply simulated clock offset for heartbeat/timeout checks | Test Mode ON |
| `POST` | `/api/test/trigger_ota` | Mock OTA check & upgrade flow | Test Mode ON |

#### Reuse of Existing Gateway Web Endpoints
For UI operations, backup/restore, and configuration interactions, automated tests leverage the existing production web endpoints directly:
- **Pre-Test Backup / Post-Test Restore**: `GET /backup?file=/lfm/data/network.dat` and `GET /restore?file=...`
- **Device Template Setup**: `GET /device?deviceid=<hex>&setup=1`
- **Check Online Resolution**: `GET /checkonline?dtid=<dt>&hw=<hw>`
- **Entity Creation & Editing**: `GET /entity?deviceid=<hex>&port=<port>&newentity=1`
- **Translation & Highlight Configuration**: Existing `/entity` form submission endpoints
- **Device Renaming & Router Assignment**: Existing `/device` configuration parameters
- **Ignore / Unignore**: `GET /device?deviceid=<hex>&ignore=1`

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: When Test Mode is OFF, 100% of calls to `/api/test/*` receive `403 Forbidden`.
- **SC-002**: Injecting a synthetic LoRa message via `POST /api/test/message` processes through `FarmNetwork` within 50ms, updating sensor values and entity states identically to real radio packets.
- **SC-003**: Querying `GET /api/test/device` returns complete, well-formed JSON matching the exact in-memory representation.
- **SC-004**: Test devices created during automated verification can be cleanly purged via `DELETE /api/test/device` without corrupting `network.dat` or leaving orphan entities.
- **SC-005**: The AI assistant can autonomously execute end-to-end testing of new device templates and firmware flows in silo prior to human handoff.

---

## Assumptions

- Test harness endpoints are strictly encapsulated in `AITestHarness.h`, keeping `WebHelper.h` clean and focused on standard web routing.
- The Gateway has sufficient PSRAM/heap to serialize JSON device graphs via `ArduinoJson` (v7).
- Device configuration (creating entities, setting translations, template application) reuses existing production web endpoints (`/device`, `/entity`, `/settings`), ensuring test scripts exercise the exact production handlers.
- Automated test scripts run from the developer workstation over local HTTP WiFi connection to the Gateway IP.
