# Feature Specification: Display Device Entities in Selected Order and Empty Ports at End

**Feature Branch**: `007-display-device-entities-in-order`  
**Created**: 2026-07-29  
**Status**: Draft  
**Input**: GitHub Issue #7: "Every entity has a display order used to build the device view sendDevice() in webhelper.h use this same order for /device page to show ports in order below the Options card."  

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Display Configured Entities in Selected Order Below Options Card (Priority: P1)

As a Gateway Web Dashboard user, I want the `/device` details page to render configured entities in their assigned `DisplayOrder` (or user-selected ordering) directly below the Options card, so that sensor values and controls appear in a predictable, logical hierarchy.

**Why this priority**: Display order customization is essential for clean dashboard UX and prioritizing critical sensor feeds (e.g. Tank Level, Battery Voltage) over secondary metrics.

**Independent Test**: Configure entity display orders on a device via the web UI (or load a network with custom `DisplayOrder` attributes) and load `/device?mac=...`. Verify that cards render in ascending order of `DisplayOrder` below the Options section.

**Acceptance Scenarios**:
1. **Given** a device with entities configured with display orders 1, 2, 3, **When** a user opens `/device?mac=<MAC>`, **Then** the entity cards appear below the Options card in order 1, 2, 3.
2. **Given** a device with multiple entities, **When** their display orders are modified, **Then** reloading `/device` reflects the updated rendering sequence immediately.

---

### User Story 2 - Render Unconfigured/Empty Sensor Ports at the End (Priority: P2)

As a Gateway Web Dashboard user, I want any unassigned or raw unconfigured sensor ports to be grouped and rendered at the bottom of the `/device` page (after all configured entities), so that the primary configured entities are not cluttered by raw unmapped ports.

**Why this priority**: Unconfigured ports provide diagnostic utility for pairing and debugging, but should not obscure configured entities.

**Independent Test**: Open `/device` for a node with 2 configured entities and 3 raw/unmapped sensor ports. Confirm configured entities render first in order, followed by the empty/unmapped ports at the end.

**Acceptance Scenarios**:
1. **Given** a device with 2 configured entities and 2 raw unmapped ports, **When** the page renders, **Then** the 2 configured entities display first in order, followed by the 2 raw ports at the bottom.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The Gateway `/device` webpage (`sendDevice()` / `DevicePageChunkedResponse`) MUST sort and display configured entities by their assigned `DisplayOrder` (or position in entity list).
- **FR-002**: Entity cards MUST be positioned below the Options card in the HTML layout structure.
- **FR-003**: Unconfigured / empty sensor ports without associated entities MUST be rendered after all configured entities at the end of the page.
- **FR-004**: Ordering logic MUST maintain high performance and low memory overhead on ESP32-WROVER without blocking web server response threads.

### Key Entities

- **Device (`Device`)**: Represents a physical node on the network (identified by 6-byte MAC). Contains linked lists of `SensorValue` and `Entity`.
- **Entity (`Entity`)**: Represents a user-configured logical entity mapped to sensor ports. Contains attributes `DisplayOrder`, `Name`, `PortName`, etc.
- **SensorValue (`SensorValue`)**: Represents raw telemetry key-value pairs (ports) received over LoRa.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Page render order on `/device` strictly matches `DisplayOrder` for 100% of configured entities.
- **SC-002**: 100% of unconfigured ports render after all configured entities.
- **SC-003**: HTTP response latency for `/device` remains under 100ms on ESP32.

---

## Assumptions

- Entity `DisplayOrder` is stored in the network data structure and populated during network load.
- The web interface uses chunked HTTP response streaming (`DevicePageChunkedResponse`) to render the page cleanly on ESP32.
