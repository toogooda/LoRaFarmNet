# Implementation Plan: AI Test Harness API & Autonomous Gateway Verification

**Branch**: `027-ai-test-harness` | **Date**: 2026-08-21 | **Spec**: [specs/027-ai-test-harness/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/027-ai-test-harness/spec.md)

**Input**: Feature specification from [specs/027-ai-test-harness/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/027-ai-test-harness/spec.md)

---

## Summary

Implement a dedicated `AITestHarness` subsystem for the ESP32 Gateway in `Gateway/LoRaNetGateway/src/AITestHarness.h` that enables programmatic simulation and verification of the LoRa message pipeline, device state inspection, atomic network snapshot/restore, simulated time progression, and telemetry querying. Integrate test mode control on `/settings` guarded by `isauthorized()`, and update AI workflow rules in `.agents/rules/` and `constitution.md` to establish autonomous testing protocols using both HTTP API and serial monitor debugging (`platformio.ini`).

---

## Technical Context

**Language/Version**: C++11 / Arduino Framework (ESP32 core 2.0.0+)  
**Primary Dependencies**: `ESPAsyncWebServer`, `ArduinoJson` (v7.4.2), `LoRaNetLibrary` (`LoraMsg.h`, `LoRaHelper.h`), `SD`  
**Storage**: SD Card FAT filesystem (`/lfm/data/network.dat`, `/lfm/data/test_snapshot.bak`)  
**Target Platform**: ESP32-WROVER with 16MB Flash and 8MB PSRAM  
**Project Type**: Embedded Gateway Firmware & Autonomous AI Testing Workflow  
**Constraints**: 
- Non-blocking asynchronous HTTP handling.
- Strict security isolation: 100% of `/api/test/*` endpoints rejected with `403 Forbidden` unless Test Mode is actively enabled.
- Zero footprint on physical radio interrupts during normal operation.
- In-memory auto-disable safety timer (1 hour).

---

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Evaluation |
|---|---|---|
| **I. Hardware Addressing & MCU Targets** | PASS | Targets ESP32-WROVER with PSRAM. No hardcoded MAC addresses. Standard Gateway broadcast destination `010101010101` supported. |
| **II. Power Integrity & Airtime** | PASS | Gateway is always-on; does not sleep. Test harness runs entirely over WiFi/HTTP and memory, requiring 0 radio airtime. |
| **III. Binary Frame Protocol** | PASS | Synthetic message injection constructs compliant binary frames (`MI` header, port-value pairs, terminating `CS` CRC). |
| **IV. Core Shared Libraries** | PASS | Builds upon `LoraMsg.h` and `FarmNetwork.h` without modifying core binary packet invariants. |
| **V. Build Verification & PR Workflow** | PASS | Local build verification via `pio run`. Delivery via feature branch `027-ai-test-harness` and GitHub PR. Issue #27 lifecycle tracked. |

---

## Proposed Project Structure

```text
specs/027-ai-test-harness/
├── spec.md                     # Feature specification
├── plan.md                     # Technical implementation plan
└── tasks.md                    # Actionable task breakdown

Gateway/LoRaNetGateway/
├── src/
│   ├── AITestHarness.h         # [NEW] Dedicated test harness routes and logic
│   ├── main.cpp                # [MODIFY] Register AITestHarness routes on webserver
│   ├── WebHelper.h             # [MODIFY] Add Test Mode toggle UI in /settings
│   └── SDHelper.h              # [MODIFY] Support snapshot and reload integration
└── scripts/
    └── test_gateway_harness.py # [NEW] Python automated test client for AI/developer

.agents/rules/
└── ai_testing_workflow.md      # [NEW] AI autonomous testing rules & serial debugging guide
```

---

## Phase 0: Research & Technical Architecture

### 1. Ingestion Pipeline Hook (`AITestHarness.h`)
- **Radio Pipeline Injection**:
  When `/api/test/message` receives a JSON payload:
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
  The handler converts hex MACs to 6-byte arrays, packs the port-value stream with `MI` header, computes the 16-bit CRC checksum for `CS`, creates a dynamic `LoraMsg`, and directly invokes:
  ```cpp
  farmNet->processNewMessage(*lMsg, rssi, snr);
  ```
  This triggers identical execution: sensor values update, translations execute, highlight states compute, and new devices are registered.

### 2. Full Device State Serialization (`GET /api/test/device?id=...`)
The Gateway data model contains multi-level **1-to-many linked list relationships**:
- **Device &rarr; Many `SensorValue` objects** (`svHead`, traversed via `sv->next`)
  - **SensorValue &rarr; Many `Entity` objects** (`_head`, traversed via `e->next`)
    - **Entity &rarr; Many `Translation` objects** (`_transHead`, traversed via `t->next`)
    - **Entity &rarr; Many `HighlightRule` objects** (`_hlHead`, traversed via `hl->next`)
  - **SensorValue &rarr; Many `EndPoint` objects** (`_headEndPoint`, traversed via `ep->next`)
  - **SensorValue &rarr; Many `PipeLine` objects** (`_headPipeLine`, traversed via `pl->next`)
- **Device &rarr; Many `RoutedNode` objects** (`routedNodesHead`, traversed via `rn->next`)

`GET /api/test/device` recursively traverses every linked list and serializes the complete tree using `ArduinoJson` (v7):
```json
{
  "id": "fc0fe71463c5",
  "name": "Custom 1",
  "type": "Configured",
  "hasData": true,
  "lastSeenSecondsAgo": 12,
  "routerId": 0,
  "sensorValues": [
    {
      "port": "IV",
      "value": 325.0,
      "entities": [
        {
          "id": 1,
          "name": "Battery Voltage",
          "order": 1,
          "icon": "f240",
          "uom": "V",
          "computedValue": 3.25,
          "translations": [
            { "id": 1, "type": "Scale", "name": "Div100", "valFrom": 0.01, "valTo": 0.0, "visible": true, "changeable": true }
          ],
          "highlightRules": [
            { "id": 1, "name": "Low Batt", "test": "LessThan", "valFrom": 3.0, "valTo": 0.0, "highlight": "Yellow" }
          ]
        }
      ],
      "endPoints": [],
      "pipeLines": []
    }
  ],
  "routedNodes": []
}
```

### 3. Snapshot & Restore Cleanup Mechanism
- `POST /api/test/snapshot`: Atomically copies `/lfm/data/network.dat` to `/lfm/data/test_snapshot.bak`.
- `POST /api/test/restore`: Restores `/lfm/data/test_snapshot.bak` back to `/lfm/data/network.dat` and calls `farmNet->loadNetwork()` to reset PSRAM memory state.

### 4. Serial Debugging Protocol
- `platformio.ini` is inspected for:
  - `upload_port = COM3`
  - `monitor_speed = 115200`
  - `monitor_filters = direct, time, esp32_exception_decoder`
- AI/scripts can open serial listeners during test runs to verify console logs (`MSG: To: ... From: ...`, `RSSI: ... SNR: ...`) and monitor for ESP32 panics.

---

## Phase 1: Implementation Phases & Component Design

### Component 1: `AITestHarness.h` & Route Registration
1. Define test mode state variables:
   ```cpp
   extern bool testModeActive;
   extern unsigned long testModeExpiry;
   ```
2. Implement route handlers:
   - `POST /settings/testmode`
   - `POST /api/test/message`
   - `GET /api/test/device`
   - `DELETE /api/test/device`
   - `POST /api/test/snapshot`
   - `POST /api/test/restore`
   - `GET /api/test/status`
   - `POST /api/test/time`
   - `POST /api/test/trigger_ota`
3. Hook `setupAITestHarnessRoutes(server)` inside `main.cpp` `setupWebServer()`.

### Component 2: Settings UI Integration (`WebHelper.h`)
1. Add Test Mode card in `/settings`:
   - Visual badge displaying "Test Mode: ACTIVE (Expires in X min)" or "Test Mode: DISABLED".
   - Toggle switch submitting `POST /settings/testmode?enable=1` or `0`.

### Component 3: Automated Test Client Script (`test_gateway_harness.py`)
1. Python client script supporting:
   - `--ip <gateway_ip>`: Gateway target address.
   - `--enable-test-mode`: Enable test mode.
   - `--inject <json_or_preset>`: Send synthetic LoRa frame.
   - `--query <mac>`: Query full device state.
   - `--snapshot` / `--restore`: Capture and restore network state.
   - `--run-suite`: Execute full end-to-end regression test suite.

### Component 4: AI Workflow Documentation & Rules
1. Create `.agents/rules/ai_testing_workflow.md` documenting:
   - Discovery of Gateway IP and serial port (`platformio.ini`).
   - Standard automated verification procedure using `AITestHarness`.
   - Tear-down and snapshot restore protocol.
2. Update `.specify/memory/constitution.md` with the autonomous testing gate.

---

## Phase 2: Verification Plan

### Automated Regression Verification:
1. `pio run` build compilation check in `Gateway/LoRaNetGateway`.
2. Execute `test_gateway_harness.py` against live Gateway:
   - Validate `403 Forbidden` when Test Mode is disabled.
   - Enable Test Mode and verify `200 OK`.
   - Snapshot network baseline.
   - Inject simulated device `FC0FE71463C5` (`DT=0`, `IV=325`, `SM=15`, `MI=14`).
   - Query `GET /api/test/device?id=fc0fe71463c5` and assert `IV == 325`, `DT == 0`.
   - Apply template and verify entities created.
   - Restore snapshot and verify device is purged.

### Manual Hardware Sanity:
- Verify normal LoRa reception from physical field nodes is completely unaffected.
- Verify Settings page UI toggle works smoothly on mobile and desktop browsers.

---

## Phase 3: Finalization & GitHub Issue Lifecycle
1. Execute `pio run` verification build.
2. Push commits to `027-ai-test-harness`.
3. Create Pull Request on `toogooda/LoRaNetGateway` and `toogooda/LoRaFarmNet`.
4. Update GitHub Issue #27 with completion report and close issue upon PR merge.
