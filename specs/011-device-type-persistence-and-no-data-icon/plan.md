# Implementation Plan: Device Type Persistence and No Data Icon

**Branch**: `011-device-type-persistence-and-no-data-icon` | **Date**: 2026-07-29 | **Spec**: [specs/011-device-type-persistence-and-no-data-icon/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/011-device-type-persistence-and-no-data-icon/spec.md)

**Input**: Feature specification from `/specs/011-device-type-persistence-and-no-data-icon/spec.md` and GitHub Issue #11.

---

## Summary

1. **Device Type Persistence**: Ensure the `DT` (Device Type) `SensorValue` port and Device Type configuration are saved to SD card storage (`network.dat`) whenever a device is set up or updated, and verified upon loading.
2. **Homepage Mini View "No Data Yet" Icon**: Update `HomePageChunkedResponse.cpp` mini view renderer to check `!d->getHasData()`. When a device has not yet transmitted sensor payload data, display an orange pending icon badge (e.g., `&#xf119;` / `&#xf05a;`) with a "No data received yet" tooltip instead of rendering incomplete entity values.

---

## Technical Context

**Language/Version**: C++11 / Arduino framework on ESP32-WROVER  
**Primary Dependencies**: `AsyncTCP`, `ESPAsyncWebServer`, `FarmNetwork` models, `SD`  
**Storage**: SD card file `/lfm/data/network.dat`  
**Testing**: Build verification (`pio run`) + Web UI manual verification on `http://192.168.68.104/`  
**Target Platform**: ESP32-WROVER (Gateway)  
**Project Type**: Embedded Web Application / Gateway Firmware  

---

## Constitution Check

- **Hardware Target Invariant**: ESP32-WROVER Gateway (PSRAM enabled). **PASS**
- **Always-On Device**: Gateway never sleeps. **PASS**
- **Core Shared Libraries**: Built on standard `FarmNetwork` models. **PASS**
- **Build Verification & PR Workflow**: `pio run` verification required; PR created without auto-merging. **PASS**

---

## Project Structure

### Documentation (this feature)

```text
specs/011-device-type-persistence-and-no-data-icon/
├── spec.md              # Feature specification
└── plan.md              # This technical implementation plan
```

### Source Code (Gateway/LoRaNetGateway repository)

```text
Gateway/LoRaNetGateway/src/
├── HomePageChunkedResponse.cpp # Mini View device icon rendering logic
├── SDHelper.h                  # Network loading and saving to network.dat
└── WebHelper.h                 # Device setup handler & saveNetwork calls
```

---

## Detailed Technical Approach

### Phase 1: Device Type (DT) Persistence Audit & Enforcement
- Audit `saveNetwork()` in `SDHelper.h` to verify that all `SensorValue` ports (specifically `DT`) and `DeviceType` status (`dStatus`) are serialized.
- Ensure that whenever a device is set up via `/device?deviceid=<MAC>&setup=1`, the assigned `DT` port value is created/updated and `saveNetwork()` is executed immediately.

### Phase 2: Mini View "No Data Yet" Rendering
- In `HomePageChunkedResponse.cpp` (case 3: Render Mini View Devices):
  - Check `if (!d->getHasData())`:
    - Render an orange pending icon card (`<i class='fas' style='color:orange;'>&#xf119;</i>`) with tooltip `"No data received yet"`.
  - Otherwise (if `d->getHasData()` is true):
    - Render `sendEntity(&bp, eOrd1, IconSize::SmallIcon)` as normal.

---

## Planned Code Changes

#### [MODIFY] [HomePageChunkedResponse.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/HomePageChunkedResponse.cpp)
- Update mini view device card rendering to check `d->getHasData()`.

#### [MODIFY] [WebHelper.h](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h)
- Ensure device setup (`setup=1`) correctly sets/persists `DT` port and triggers `saveNetwork()`.

---

## Verification Plan

### Automated Build Verification
- Execute `pio run` in `Gateway/LoRaNetGateway` to confirm error-free compilation and RAM/Flash usage.

### Manual Verification
- Deploy firmware to ESP32 Gateway.
- Set up a device and restart/reload gateway. Verify category association persists.
- View Homepage Mini View for a node with no payload messages; verify orange pending "No Data Yet" icon displays.
