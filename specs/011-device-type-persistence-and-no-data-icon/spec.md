# Feature Specification: Device Type Persistence and No Data Icon

**Feature Branch**: `011-device-type-persistence-and-no-data-icon`

**Created**: 2026-07-29

**Status**: Draft

**Input**: User description from GitHub Issue #11 (`toogooda/LoRaFarmNet`): "Devices once setup should save DT to disk so they show in correct category. Mini view needs an icon for no data yet."

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Persist Device Type (DT) to Disk Upon Setup (Priority: P1) 🎯 MVP

When a user completes the setup process for a discovered or new device on the Gateway web server, the assigned Device Type (DT) port value MUST be persisted to non-volatile storage (disk/SD/NVS). Upon Gateway reboot or network reload, the device MUST load its saved DT value and appear in its assigned category instead of reverting to Device Type 0 / Uncategorized.

**Why this priority**: Without DT persistence, setting up a device is lost when the gateway reboots, requiring re-configuration and causing devices to lose their category association.

**Independent Test**: Set up a device under a specific category, restart the Gateway (or reload the network model from disk), and verify the device remains in its assigned category with its DT port preserved.

**Acceptance Scenarios**:

1. **Given** a new or unconfigured device, **When** the user completes setup and assigns a Device Type (DT), **Then** the DT sensor port value is written to disk storage.
2. **Given** a Gateway system restart or network reload from disk, **When** the network model is loaded into memory, **Then** all devices restore their persisted DT values and render under their correct categories.

---

### User Story 2 - Mini View Icon for Devices with "No Data Yet" (Priority: P2)

When a device is listed in the Homepage Mini View mode, but has not yet transmitted any sensor payload messages (`getHasData() == false`), the UI MUST display a clear "No Data Yet" icon badge (e.g., orange warning icon `&#xf119;` or pending data status) instead of an empty space or broken visual element.

**Why this priority**: Provides instant visual feedback on the home page mini view so users can distinguish active field nodes from nodes awaiting their first transmission.

**Independent Test**: View the Homepage Mini View for a newly added device that has not sent data, and verify the "No Data Yet" icon badge is displayed in the mini card container.

**Acceptance Scenarios**:

1. **Given** a device with `getHasData() == false`, **When** rendered in Homepage Mini View, **Then** a prominent "No Data Yet" icon/tooltip is displayed.
2. **Given** a device that receives its first sensor payload, **When** the homepage refreshes, **Then** the "No Data Yet" icon is replaced by the node's active entity status indicators.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The Gateway MUST persist the Device Type (DT) port value to disk (`network.dat` / SD card) whenever a device setup operation occurs.
- **FR-002**: The Gateway network loader MUST restore the persisted DT value for each device during boot and assign the device to its correct `DeviceCategory`.
- **FR-003**: The Homepage Chunked Renderer (`HomePageChunkedResponse.cpp`) MUST display a "No Data Yet" icon badge in Mini View mode for devices where `getHasData() == false`.
- **FR-004**: Hovering or inspecting a "No Data Yet" mini view icon MUST clearly indicate to the user that no transmission has been received from the device.

### Key Entities

- **Device**: Contains Unique ID, Name, Type, and linked `SensorValue` ports (including the `DT` port).
- **SensorValue (DT)**: SensorValue port with ASCII key `"DT"` holding the numerical Device Type ID.
- **DeviceCategory**: Category mapping that groups devices by their Device Type (DT).

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Device Type (DT) assignments survive Gateway reboots and network reload operations 100% of the time.
- **SC-002**: Homepage Mini View renders the "No Data Yet" icon for all inactive/new devices without heap allocation or rendering delays.

---

## Assumptions

- Device Type persistence uses existing SD card / NVS serialization mechanisms (`SDHelper.h` / `FarmNetwork.cpp`).
- Icon for "No Data Yet" utilizes existing FontAwesome icon set (`&#xf119;` or equivalent).
