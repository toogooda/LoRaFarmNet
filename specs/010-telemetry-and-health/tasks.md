# Tasks: Network Telemetry & Health Dashboard

**Feature Branch**: `010-telemetry-and-health`
**Input**: Specification and Plan from `specs/010-telemetry-and-health/`

---

## Phase 1: Setup & Foundational Telemetry State

**Purpose**: Create telemetry engine and event listeners in the Gateway codebase.

- [x] T001 [P] Create `src/TelemetryHelper.h` with thread-safe counters (WiFi drops, MQTT drops, page requests), uptime calculation, and memory usage utilities (DRAM % and PSRAM %).
- [x] T002 Hook WiFi disconnect events in `src/main.cpp` using `WiFi.onEvent()` to increment `wifiDisconnectCount`.
- [x] T003 Hook MQTT drop counters in `src/MQTTHelper.cpp` (`reconnecting()`) to track connection drop events.
- [x] T004 Hook page request counter in `src/WebHelper.h` (`sendPageHeader()`).

---

## Phase 2: User Story 1 - System & Network Health Dashboard (Priority: P1)

**Purpose**: Render system telemetry and network stability metric cards on `/map`.

- [x] T005 Update `sendNetworkMapPage()` in `src/WebHelper.h` to render the **System Telemetry** card (Uptime, Software Version, % DRAM used, % PSRAM used).
- [x] T006 Add **Network Stability** card to `sendNetworkMapPage()` in `src/WebHelper.h` displaying WiFi Drops, Page Requests, and conditional MQTT Drops (hidden when `farmNet->getUseMQTT()` is disabled).
- [x] T007 Update main navigation bar in `src/WebHelper.h` to change "Network" to "Health" with an updated icon.

---

## Phase 3: User Story 2 - Device Status Breakdown & Overdue Monitor (Priority: P2)

**Purpose**: Calculate device counts and detect tardy/overdue field nodes based on `SM` sleep intervals.

- [x] T008 Implement device status aggregation in `src/WebHelper.h` (Total, Configured, New, Ignored counts).
- [x] T009 Implement overdue detection logic comparing `difftime(now, d->getTimeLastMsg())` against `(SM * 60 * 2)` seconds for active periodic nodes.
- [x] T010 Render the Device Breakdown summary cards and the Overdue Devices alert table in `sendNetworkMapPage()`.

---

## Phase 4: User Story 3 - White-Background Network Diagram & Fixed 80px Spacing (Priority: P3)

**Purpose**: Redesign SVG tree diagram with white background, high-contrast typography, and hardcoded 80px ring geometry.

- [x] T011 Update SVG canvas styling in `src/WebHelper.h` to `#ffffff` background with subtle borders and update connector line strokes to `#198754` / `#fd7e14`.
- [x] T012 Update SVG node labels and tooltip styles to dark accessible typography (`#212529` with white halo).
- [x] T013 Remove `zoomSlider` and `zoomVal` elements from `sendNetworkMapPage()` in `src/WebHelper.h` and hardcode ring spacing to `80px`.

---

## Phase 5: Verification & Testing

**Purpose**: Build verification, deployment, and hardware validation.

- [x] T014 Run PlatformIO build verification (`pio run`) in `Gateway/LoRaNetGateway`.
- [x] T015 Deploy to ESP32 Gateway hardware and test all dashboard metrics and SVG rendering.
- [x] T016 Pause for user hardware testing and verification.

---

## Phase 6: Pull Request & Issue Closure

**Purpose**: Finalize feature branch, create PR, and close GitHub Issue #10.

- [x] T017 Push branch `010-telemetry-and-health` and create Pull Request on GitHub.
- [ ] T018 Update and close GitHub Issue #10 upon PR approval/merge.
