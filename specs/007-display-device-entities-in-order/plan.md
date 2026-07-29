# Implementation Plan: Display Device Entities in Selected Order and Empty Ports at End

**Branch**: `007-display-device-entities-in-order` | **Date**: 2026-07-29 | **Spec**: [specs/007-display-device-entities-in-order/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/007-display-device-entities-in-order/spec.md)

**Input**: Feature specification from `/specs/007-display-device-entities-in-order/spec.md` and GitHub Issue #7.

---

## Summary

Update the Gateway `/device` details webpage renderer (`DevicePageChunkedResponse.cpp` and `WebHelper.h`) to render configured `Entity` cards ordered by `DisplayOrder` (or position in entity list) directly below the Options card. Following the configured entity cards, group and render all unconfigured/empty `SensorValue` ports (ports with no entities attached) at the bottom of the page.

---

## Technical Context

**Language/Version**: C++11 / Arduino framework on ESP32-WROVER  
**Primary Dependencies**: `AsyncTCP`, `ESPAsyncWebServer`, `FarmNetwork` core model  
**Storage**: NVS / PSRAM fixed chunked response buffer  
**Testing**: Build verification (`pio run`) + Web UI manual verification on `/device?deviceid=<MAC>`  
**Target Platform**: ESP32-WROVER (Gateway)  
**Project Type**: Embedded Web Application / Gateway Firmware  
**Performance Goals**: Page chunk rendering overhead <50ms without heap fragmentation (utilizing PSRAM chunk buffer)  
**Constraints**: Zero dynamic allocation per chunk, maintain chunked response streaming without blocking main loop  

---

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **Hardware Target Invariant**: ESP32-WROVER Gateway (PSRAM enabled). **PASS**
- **Always-On Device**: Gateway never sleeps. **PASS**
- **Core Shared Libraries**: Built on standard `FarmNetwork` models. **PASS**
- **Build Verification & PR Workflow**: `pio run` verification required; PR created without auto-merging. **PASS**

---

## Project Structure

### Documentation (this feature)

```text
specs/007-display-device-entities-in-order/
├── spec.md              # Feature specification
└── plan.md              # This technical implementation plan
```

### Source Code (Gateway/LoRaNetGateway repository)

```text
Gateway/LoRaNetGateway/src/
├── DevicePageChunkedResponse.h     # State machine definitions for chunked response
├── DevicePageChunkedResponse.cpp   # Chunked renderer state machine implementation
└── WebHelper.h                     # Web helper functions (sendDevice, sendEntity, etc.)
```

---

## Detailed Technical Approach

### Phase 1: Entity Sorting & Iteration State Machine

Currently, `DevicePageChunkedResponse` iterates linearly over `Device::getFirstSensorValue()`. 

To support rendering configured entities in order followed by unconfigured ports at the end, the chunked response state machine in `DevicePageChunkedResponse.cpp` will be refactored into distinct phases:

1. **State 0 (Options & Header)**: Renders the global page header, notification alerts, device summary, Options card, and Router Configuration card (unchanged).
2. **State 1 (Configured Entities Phase)**:
   - Collect or iterate through all configured `Entity` instances across the device's sensor values.
   - Sort entities by `DisplayOrder` (ascending).
   - Render each `Entity` card cleanly with its associated port value and action buttons (Edit / Remove).
3. **State 2 (Unconfigured / Empty Ports Phase)**:
   - Iterate through `SensorValue` list on `Device`.
   - Filter for ports that have **no attached entities** (`getFirstEntity() == nullptr`) or endpoints.
   - Render these remaining ports in a collapsed/grouped "Unconfigured Ports" section at the end of the page.
4. **State 3 (Page Footer)**: Render closing HTML tags and scripts (unchanged).

---

## Planned Code Changes

#### [MODIFY] [DevicePageChunkedResponse.h](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/DevicePageChunkedResponse.h)
- Add state enum states for Configured Entities iteration and Unconfigured Ports iteration.
- Add pointer array/vector or index tracking for sorted entity iteration.

#### [MODIFY] [DevicePageChunkedResponse.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/DevicePageChunkedResponse.cpp)
- Update state machine `switch(state)` to render configured entities first ordered by `DisplayOrder`.
- Render unconfigured ports in the final phase prior to page footer.

#### [MODIFY] [WebHelper.h](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h)
- Update helper functions if needed for rendering entity cards and unconfigured port badges cleanly.

---

## Verification Plan

### Automated Build Verification
- Execute `pio run` in `Gateway/LoRaNetGateway` to confirm error-free compilation and flash/RAM usage.

### Manual Verification
- Deploy firmware to ESP32 Gateway and allow ~30 seconds for startup.
- Fetch live rendered device page at `http://192.168.68.104/device?deviceid=fc0fe71454d8`.
- Verify configured entities display in ascending `DisplayOrder` below the Options card.
- Verify empty/unconfigured ports (such as `CS`, `SM`, `OR`, `FW`, `PD`, `P1`) render grouped after all configured entities at the bottom of the page.
- Then hand over for Manual checks.

---

## GitHub Issue Lifecycle Task

- **Final Step**: Upon successful manual testing and PR merge, close GitHub Issue #7 on `toogooda/LoRaFarmNet` (and `toogooda/LoRaNetGateway`).
