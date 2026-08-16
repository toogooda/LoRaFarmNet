# Feature Specification: Network Telemetry & Health Dashboard

**Feature Branch**: `010-telemetry-and-health`

**Created**: 2026-08-16

**Status**: Draft

**Input**: User description from GitHub Issue #10:
> "Change Network Page (/map) to become Network Health and display the following: Server up time, Current Software version, % memory used, % PSRAM used, Total number of devices new vs setup vs ignored, show devices that are late sending result i.e. heartbeat > SM port value, total Wifi drops since reboot, total MQTT drops since last boot (hide if toggled off), Number of page requests, include the network diagram but white background so change white colored items to be visible with background change zoom doesn't work well so remove it and hard code it at 80px"

---

## User Scenarios & Testing

### User Story 1 - System & Network Health Dashboard (Priority: P1)

As a farm network operator, I want to visit the Network page (`/map`) and immediately see key gateway telemetry, memory usage, uptime, connection drops, and request statistics so that I can monitor the real-time health and stability of the gateway without connecting via USB serial.

**Why this priority**: Core monitoring requirement giving operational visibility into ESP32 system resources, uptime, and communication stability.

**Independent Test**: Navigate to `/map` on a browser and verify that all metric cards render accurate live telemetry data (uptime, version, memory %, PSRAM %, WiFi drops, MQTT drops, page requests).

**Acceptance Scenarios**:
1. **Given** the gateway is running, **When** visiting `/map`, **Then** the page title and header display "Network Health" instead of "Network Map".
2. **Given** the gateway has been running for a period, **When** viewing the System Health card, **Then** it shows formatted Server Uptime (e.g. `Xd Xh Xm Xs`), Current Software Version (e.g. `1.0.0`), % Internal DRAM used (e.g. `24.5%`), and % PSRAM used (e.g. `12.1%`).
3. **Given** WiFi disconnect events occur, **When** viewing the Network Health card, **Then** "WiFi Drops Since Reboot" displays the exact count of disconnect events.
4. **Given** MQTT is enabled (`getUseMQTT() == true`), **When** viewing the card, **Then** "MQTT Drops Since Reboot" displays the disconnect/reconnect failure count.
5. **Given** MQTT is disabled (`getUseMQTT() == false`), **When** viewing the card, **Then** the MQTT drops metric is completely hidden.
6. **Given** web requests are handled by the gateway, **When** viewing the card, **Then** "Page Requests Served" displays the cumulative HTTP request counter.

---

### User Story 2 - Device Status Summary & Overdue Heartbeat Monitor (Priority: P2)

As an operator, I want to see an overview of all devices in the network breakdown (New vs Setup/Configured vs Ignored) and a highlighted list of devices that are overdue (have not reported within their expected sleep window based on their `SM` heartbeat interval) so that I can quickly detect dead supercapacitors, hardware faults, or out-of-range field nodes.

**Why this priority**: Critical for farm reliability and proactive maintenance of battery-free field nodes.

**Independent Test**: Simulate a missed message from a field node with `SM = 15` (elapsed time > 30 min) and verify that the device is flagged in the "Overdue Devices" table with its last seen time and expected interval.

**Acceptance Scenarios**:
1. **Given** devices exist in the network, **When** viewing the Device Status section on `/map`, **Then** it displays counts for:
   - Total Devices
   - Configured (Setup)
   - New
   - Ignored
2. **Given** a periodic device with an `SM` sensor value has not reported in `(SM * 60 * 2)` seconds, **When** viewing `/map`, **Then** it is listed in the "Late / Overdue Devices" warning section with device name, ID, expected interval, and elapsed time since last contact.
3. **Given** all devices are reporting on schedule, **When** viewing `/map`, **Then** the Overdue section displays a reassuring green badge: "All devices reporting on schedule".

---

### User Story 3 - Clean White-Background Network Diagram with Fixed 80px Spacing (Priority: P3)

As an operator, I want the network topology visualization on `/map` to have a clean, high-contrast white background with dark, legible labels and lines, and a fixed 80px ring spacing without the unstable zoom slider so that the tree diagram is easy to inspect on both mobile and desktop screens.

**Why this priority**: Improves visual clarity and accessibility while simplifying the UI by removing buggy slider interactions.

**Independent Test**: Load `/map` and observe the SVG network diagram rendering against a `#ffffff` background with crisp dark labels, clear colored status badges, fixed 80px node spacing, and no slider controls.

**Acceptance Scenarios**:
1. **Given** `/map` is loaded, **When** viewing the SVG diagram, **Then** the background is clean white (`#ffffff` / `#f8f9fa`) with dark node borders, dark text labels, and contrasting connection lines.
2. **Given** the diagram is rendered, **When** checking controls, **Then** the ring spacing zoom slider is removed and ring depth spacing is fixed at `80px`.
3. **Given** nodes in the diagram are hovered/tapped, **When** the tooltip appears, **Then** it displays device telemetry in a high-contrast dark popup with crisp text.

---

## Edge Cases

- **WiFi Disconnect during boot**: Counter should only increment after initial connection is established, avoiding false initial increment.
- **PSRAM not detected or 0MB**: Guard against division by zero if PSRAM size is unavailable.
- **Device has no SM port**: Devices without an `SM` port (or always-on nodes like repeaters/gateways) should not be flagged as overdue based on sleep cycles unless a separate timeout applies.
- **Empty Network**: If 0 devices are enrolled, render clean empty state cards without NaN/errors.
- **Long Uptime Overflow**: Ensure `millis()` rollover (every ~49 days) is handled gracefully using 64-bit seconds or delta calculation.

---

## Requirements

### Functional Requirements

- **FR-001**: Rename the page and navigation item from "Network Map" / "Network" to "Network Health" with a diagnostic health icon (`fa-heart-pulse` / `fa-stethoscope` / `fa-network-wired`).
- **FR-002**: Track and display Gateway **Server Uptime** in human-readable format (`Dd Hh Mm Ss`).
- **FR-003**: Display current **Software Version** matching `GATEWAY_VERSION_STRING` (`src/Version.h`).
- **FR-004**: Calculate and display **% Internal RAM Used** (`(total_heap - free_heap) / total_heap * 100`) and **% PSRAM Used** (`(total_psram - free_psram) / total_psram * 100`).
- **FR-005**: Maintain an atomic counter for **Total HTTP Page Requests** served since boot across all web request endpoints.
- **FR-006**: Register a WiFi event listener or monitor `WiFi.onEvent` to track **WiFi Disconnect Events** since boot.
- **FR-007**: Track **MQTT Disconnect / Failure Events** and display only when `farmNet->getUseMQTT()` is enabled.
- **FR-008**: Calculate device counts: Total, Configured (`DeviceType::Configured`), New (`DeviceType::New`), and Ignored (`DeviceType::Ignore`).
- **FR-009**: Detect and display **Overdue Devices** where `difftime(now, d->getTimeLastMsg()) > (sleepMins * 60.0 * 2.0)` for devices with an `SM` port.
- **FR-010**: Render the SVG network topology tree on a **white background** (`#ffffff`), updating node circles, text labels, and connector lines to dark theme colors (`#212529`, `#495057`).
- **FR-011**: Hardcode the tree layout ring spacing to **80px** and remove the slider input UI.

---

## Success Criteria

- **SC-001**: `/map` renders all 9 telemetry metrics cleanly on desktop and mobile viewports in under 150ms.
- **SC-002**: MQTT drops card is completely hidden when MQTT is disabled, and appears automatically when enabled.
- **SC-003**: Network diagram renders on white background with 100% text readability and fixed 80px ring geometry.
- **SC-004**: Overdue devices list updates in real time based on `timeLastMsg` and `SM` sleep intervals.
- **SC-005**: All code compiles without errors or warnings via PlatformIO (`pio run`).
