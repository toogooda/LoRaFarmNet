# Implementation Plan: Network Telemetry & Health Dashboard

**Branch**: `010-telemetry-and-health` | **Date**: 2026-08-16 | **Spec**: [specs/010-telemetry-and-health/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/010-telemetry-and-health/spec.md)

**Input**: Feature specification from `specs/010-telemetry-and-health/spec.md` (derived from GitHub Issue #10).

---

## Summary

Transform the current Network Map page (`/map`) into a comprehensive **Network Health & Telemetry Dashboard**. The page will aggregate real-time ESP32 hardware telemetry (uptime, memory %, PSRAM %, software version), network link reliability (WiFi drops, conditional MQTT drops), web traffic (HTTP request counter), device status breakdown (total, new, configured, ignored), and an overdue heartbeat detection table for tardy field nodes. In addition, the embedded SVG network topology diagram will be restyled with a clean white background, high-contrast dark elements, and a hardcoded 80px ring spacing (removing the legacy zoom slider).

---

## Technical Context

**Language/Framework**: C++ / Arduino Framework for ESP32 (PlatformIO)
**Target MCU**: ESP32-WROVER (8MB PSRAM mandatory)
**Web Framework**: ESPAsyncWebServer / AsyncResponseStream
**Libraries**: WiFi, PubSubClient, ArduinoJson, LoRaNetLibrary
**Storage**: SD Card (FAT32) via HSPI (`network.dat`, `registry.dat`, `categories.dat`)
**Build Verification**: PlatformIO CLI (`pio run`)

---

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- [x] **MCU Target Invariant**: Explicitly targets ESP32-WROVER with PSRAM utilization metrics.
- [x] **Power Topology Invariant**: Gateway is an Always-On node that never enters deep sleep.
- [x] **No Code Duplication / No Unrequested Refactoring**: Preserves existing network map graph layout algorithms while updating SVG rendering attributes.
- [x] **PR & Verification Policy**: Branching and PR creation required; AI will not merge locally.
- [x] **GitHub Issue Lifecycle**: Tracking GitHub Issue #10 and closing upon verified merge.

---

## Project Structure & File Layout

### Documentation (`specs/010-telemetry-and-health/`)
```text
specs/010-telemetry-and-health/
├── spec.md              # Feature specification
├── plan.md              # Implementation plan (this document)
└── tasks.md             # Actionable task list
```

### Source Code Targets (`Gateway/LoRaNetGateway/`)
```text
src/
├── TelemetryHelper.h    # [NEW] Telemetry tracking state (counters, uptime, memory, drops)
├── WebHelper.h          # [MODIFY] Render Network Health dashboard & updated SVG diagram
├── MQTTHelper.cpp       # [MODIFY] Hook MQTT disconnect/drop counter
├── main.cpp             # [MODIFY] Initialize WiFi disconnect event listener & request counter
└── Version.h            # [READ] Reference GATEWAY_VERSION_STRING
```

---

## Technical Architecture & Design

```
+--------------------------------------------------------------------------------+
|                         Network Health Dashboard (/map)                        |
+--------------------------------------------------------------------------------+
|  +---------------------------+  +--------------------------+  +-------------+  |
|  |     System Telemetry      |  |     Network Stability    |  | Web Traffic |  |
|  |---------------------------|  |--------------------------|  |-------------|  |
|  | - Uptime: 4d 12h 30m 15s  |  | - WiFi Drops: 0          |  | - Requests: |  |
|  | - Version: v1.0.0         |  | - MQTT Drops: 1 (if on)  |  |   1,420     |  |
|  | - DRAM Used: 19.2%        |  +--------------------------+  +-------------+  |
|  | - PSRAM Used: 8.4%        |                                                 |
|  +---------------------------+                                                 |
|                                                                                |
|  +--------------------------------------------------------------------------+  |
|  |                            Device Breakdown                              |  |
|  |--------------------------------------------------------------------------|  |
|  | Total: 12  |  Configured: 8  |  New (Pending): 2  |  Ignored: 2          |  |
|  +--------------------------------------------------------------------------+  |
|                                                                                |
|  +--------------------------------------------------------------------------+  |
|  |                       Overdue / Late Field Nodes                         |  |
|  |--------------------------------------------------------------------------|  |
|  | [!] Remote Tank 1 (FC0F...): Expected 15m interval, last seen 45m ago    |  |
|  +--------------------------------------------------------------------------+  |
|                                                                                |
|  +--------------------------------------------------------------------------+  |
|  |                Network Topology Tree (White BG, 80px)                    |  |
|  |--------------------------------------------------------------------------|  |
|  | [SVG Canvas: #ffffff BG, dark nodes & labels, high-contrast links]       |  |
|  +--------------------------------------------------------------------------+  |
+--------------------------------------------------------------------------------+
```

### 1. Telemetry Tracking Engine (`src/TelemetryHelper.h`)
- **Uptime Tracking**: `int64_t esp_timer_get_time() / 1000000ULL` formats days, hours, minutes, seconds.
- **Memory Metrics**:
  - DRAM: `heap_caps_get_total_size(MALLOC_CAP_INTERNAL)` and `heap_caps_get_free_size(MALLOC_CAP_INTERNAL)`.
  - PSRAM: `ESP.getPsramSize()` and `ESP.getFreePsram()`.
- **WiFi Disconnect Counter**: Incremented via `WiFi.onEvent(WiFiEvent_t::ARDUINO_EVENT_WIFI_STA_DISCONNECTED)`.
- **MQTT Drop Counter**: Incremented on reconnect attempts/disconnects in `MQTTHelper.cpp`.
- **HTTP Request Counter**: Incremented in `sendPageHeader()` on each page render.

### 2. Device Breakdown & Overdue Detection
- Device Status Counter: Iterates `farmNet->getFirstDevice()`, summing `Configured`, `New`, `Ignore`.
- Overdue Detection: Checks devices with `SM` port where `difftime(now, d->getTimeLastMsg()) > (sleepMins * 60.0 * 2.0)`.

### 3. SVG Network Topology Tree Overhaul
- Background: Set to `#ffffff` with a subtle `#dee2e6` container border.
- Elements: SVG line stroke `#6c757d`, node text labels `#212529`, router nodes `#198754`, leaf nodes `#0d6efd`.
- Geometry: Fixed `zoom = 80;` ring spacing; remove `zoomSlider` and `zoomVal` elements.

---

## Phase Breakdown & Implementation Plan

### Phase 1: Telemetry State & Global Event Hooks
- Create `src/TelemetryHelper.h` with thread-safe counters and utility functions (`getFormattedUptime()`, `getDramPercent()`, `getPsramPercent()`, etc.).
- Hook WiFi disconnect event in `main.cpp` (`setupWiFiEvents()`).
- Hook MQTT drop counter in `MQTTHelper.cpp`.
- Hook request counter in `WebHelper.h`.

### Phase 2: Device Health & Overdue Monitor Logic
- Add helper methods to calculate device category counts (New vs Configured vs Ignored).
- Implement overdue device scanner returning list of tardy devices.

### Phase 3: Dashboard UI & White SVG Diagram
- Update `sendNetworkMapPage()` in `WebHelper.h`:
  - Rename title and header to **Network Health**.
  - Render telemetry metric cards (System Health, Network Stability, Web Traffic).
  - Render Device Status breakdown and Overdue Devices table/alert.
  - Render SVG tree on `#ffffff` canvas with fixed 80px spacing and dark typography.
- Update top navigation link text to **Health** (`<i class='fa-solid'>&#xf21e;</i> Health`).

### Phase 4: Verification & GitHub Issue Closure
- Compile with `pio run`.
- Upload to ESP32 Gateway and verify telemetry, overdue alerts, and SVG rendering on browser.
- Push branch, create PR, and close Issue #10 upon merge.
