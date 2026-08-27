# Implementation Plan: Fix Div Nesting and Layout for Devices with No Data

**Branch**: `036-fix-no-data-div-nesting` | **Date**: 2026-08-27 | **Spec**: [specs/036-fix-no-data-div-nesting/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/036-fix-no-data-div-nesting/spec.md)

**Input**: Feature specification from `specs/036-fix-no-data-div-nesting/spec.md`

## Summary

Fix `sendDevice()` in `Gateway/LoRaNetGateway/src/WebHelper.h` to balance `<div>` open/close tags across all branches (`DeviceType::New`, `!d->getHasData()`, and established data), preventing extra `</div>` emissions that break category cards and displace elements on the Home Page (`/`).

---

## Technical Context

- **Platform**: ESP32-WROVER (PlatformIO `gateway-test` / `gateway-release` environments)
- **Language**: C++11 (Arduino Framework)
- **Core Components**:
  - `Gateway/LoRaNetGateway/src/WebHelper.h`: `sendDevice()`
  - `Gateway/LoRaNetGateway/src/HomePageChunkedResponse.cpp`: `_fillBuffer()` State 1, 2, 3, 4
- **Testing**:
  - Compilation: `pio run -e gateway-test`
  - Automated Regression Harness: `python scripts/test_gateway_harness.py --ip 192.168.68.104 --run-suite`
  - Live DOM Inspection: `GET http://192.168.68.104/`

---

## Constitution Check

*GATE: All changes comply with project constitution.*
- **C-001**: Single-responsibility HTML generator in `sendDevice()`.
- **C-002**: Regression test integrity preserved.
- **C-003**: Mandatory verification gate before git commit.

---

## Project Structure & Planned Changes

### Modified Files

#### [`Gateway/LoRaNetGateway/src/WebHelper.h`](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h)
- In `sendDevice(Print *response, Device *d)`:
  - **Line 828**: `<div class='container border shadow-lg rounded'...>` opens outer container.
  - **Lines 891–925 (`DeviceType::New`)**: Change `</div></div></div>` to `</div></div>` across all 4 sub-branches so outer container is not closed inside the branch.
  - **Lines 928–940 (`!s || !d->getHasData()`)**: Change `</div></div>` to `</div>` so only the inner `position:relative` wrapper is closed.
  - **Lines 963–1071 (`else` established data)**: Closes `row no-pad` (`</div>`).
  - **Line 1073**: `response->print("</div>"); // Closes main container` cleanly and universally closes the outer container for all branches.

---

## Verification Plan

### 1. Build Verification
```powershell
& "C:\Users\USER\.platformio\penv\Scripts\platformio.exe" run -e gateway-test
```

### 2. Hardware Upload & Live DOM Inspection
```powershell
& "C:\Users\USER\.platformio\penv\Scripts\platformio.exe" run -e gateway-test -t upload
```
Fetch `GET http://192.168.68.104/` and assert:
- `Router Top` is inside `.full-view-container-7`.
- `Test Router` is inside `.full-view-container-7`.
- `.mini-view-container-7` is strictly inside `#cat-card-body-7` and `<div class='card'>`.
- Toggling Mini/Full switches displays and hides cards without visual spill.

### 3. Regression Suite
```powershell
python scripts/test_gateway_harness.py --ip 192.168.68.104 --run-suite
```
