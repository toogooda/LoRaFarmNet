# Implementation Plan: Unified Add Device Experience & Enhanced `sendDevice()` Not-Setup View

**Branch**: `037-unified-add-device-senddevice` | **Date**: 2026-08-28 | **Spec**: [specs/037-unified-add-device-senddevice/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/037-unified-add-device-senddevice/spec.md)

**Input**: Feature specification from `specs/037-unified-add-device-senddevice/spec.md`

## Summary
Harmonize the **Dynamic Add Device Page** ([009](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/009-dynamic-add-device-pairing/plan.md)) and the **Online Device Type Resolution & Auto-Upgrade** ([008](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/008-online-device-types-and-auto-upgrade/plan.md)) into a single consistent architecture powered by an enhanced [`sendDevice()`](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h#L827-L941).

---

## Technical Context

- **Platform**: ESP32-WROVER with 8MB PSRAM (PlatformIO `gateway-test` / `gateway-release` environments)
- **Language**: C++11 (Arduino Framework)
- **Core Files**:
  - [`Gateway/LoRaNetGateway/src/WebHelper.h`](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h)
  - [`Gateway/LoRaNetGateway/src/FarmNetwork.cpp`](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/FarmNetwork.cpp)
  - [`Gateway/LoRaNetGateway/src/main.cpp`](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/main.cpp)
- **Testing**:
  - PlatformIO test build compilation: `pio run -e gateway-test`
  - Automated Regression Harness: `python scripts/test_gateway_harness.py --ip <ip_address> --run-suite`
  - SSE Pairing Test Suite: `python scripts/test_pair_sse.py <ip_address>`

---

## Project Structure & Planned Changes

### Component 1: `sendDevice()` Enhancement in `WebHelper.h`
#### [MODIFY] [`Gateway/LoRaNetGateway/src/WebHelper.h`](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h)
- In `sendDevice(Print *response, Device *d)`:
  - Add `id='device-card-%s'` to the container `<div>` using `d`'s 12-char hex MAC address.
  - **Header Row (Row 1)**:
    - Check if `d->getType() == DeviceType::New` and `d` has an `MR` port (`MR >= 100`).
    - Look up router device name via `farmNet->getDeviceByRouterID(mrId)` (or fallback `Router <mrId>`).
    - Output blue badge `<span class='badge bg-info text-dark'><i class='fa-solid fa-satellite-dish me-1'></i>Via [Router Name]</span>` between device name and ID.
  - **Status & RSSI Row (Row 2)**:
    - Retrieve `SensorValue *rsSV = d->getSensorValueByPort("RS")`.
    - Compute `rssi = rsSV ? -(int)rsSV->getValue() : 0`.
    - Retrieve category icon from `DeviceCategory *cat` if template is known (`cat->icon`).
    - Render `Device Icon + "New " + Device Name` on the left.
    - Render `<span class='badge [bg-secondary|bg-warning] font-monospace'>[rssi] dBm</span>` on the right below the ID.
  - **Bottom Row (Row 3: Countdown & Action Buttons with Icons)**:
    - Retrieve `SensorValue *smSV = d->getSensorValueByPort("SM")`.
    - If present, calculate `dueInMinutes = max(0, sleepMinutes - elapsedMinutes)`.
    - Render `<div class='d-flex justify-content-between align-items-center mt-2'>`:
      - Left: Next message badge (`Next ~X min` or `Due anytime`).
      - Right: Icon-consistent action buttons:
        - Auto-configured: `<i class='fa-solid fa-wrench me-1'></i> Setup` + `<i class='fa-solid fa-eye-slash me-1'></i> Ignore`
        - Unknown DT: `<i class='fa-solid fa-cloud-arrow-down me-1'></i> Check Online` + `<i class='fa-solid fa-eye-slash me-1'></i> Ignore`
        - Needs FW: `<i class='fa-solid fa-rotate me-1'></i> Update Firmware` + `<i class='fa-solid fa-eye-slash me-1'></i> Ignore`

---

### Component 2: Unified Add Device Page (`/adddevice`) & Dynamic Card Replacement
#### [MODIFY] [`Gateway/LoRaNetGateway/src/WebHelper.h`](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h)
- In `sendAddDevicePage(AsyncWebServerRequest *request)`:
  - Retain Radar Hero visual animation and filter switches (`Older`, `Ignored`, `Query Router`).
  - Directly render unsetup devices in C++ via `sendDevice()` inside `#device-grid`.
  - Add card snippet endpoint: `GET /api/device/card?id=<mac>` which streams single-device HTML via `sendDevice()`.
  - In `evtSource.addEventListener('new_device')`:
    - When a packet arrives (initial direct packet or subsequent routed packet), fetch `/api/device/card?id=<mac>` and insert or replace `document.getElementById('device-card-' + mac).outerHTML`.
    - If the device was previously in the "Remote Router Candidates" list, remove it from candidates.
- In `setupDeviceRoutes(server)`:
  - Add `redirect` query parameter support for `checkonline=1` (e.g. `/device?deviceid=<mac>&checkonline=1&redirect=adddevice`):
    - When `redirect=adddevice`, redirect back to `/adddevice` after downloading and auto-configuring the template so the user stays on the Add Device page with the device ready to setup.

---

### Component 3: Route Discovery Candidates
#### [MODIFY] [`Gateway/LoRaNetGateway/src/WebHelper.h`](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h)
- Refactor Remote Router Candidates section to match the clean card styling of `sendDevice()`, with an icon-enabled button: `<i class='fa-solid fa-plus me-1'></i> Add via [Router]` and the arrival countdown timer.

---

## Verification Plan

### 1. Build Verification
```powershell
& "C:\Users\USER\.platformio\penv\Scripts\platformio.exe" run -e gateway-test
```

### 2. Hardware Upload & Live Verification
```powershell
& "C:\Users\USER\.platformio\penv\Scripts\platformio.exe" run -e gateway-test -t upload
```

### 3. Automated Regression Suites
```powershell
python scripts/test_gateway_harness.py --ip <ip_address> --run-suite
python scripts/test_pair_sse.py <ip_address>
```
