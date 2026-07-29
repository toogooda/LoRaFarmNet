# Tasks: Display Device Entities in Selected Order and Empty Ports at End

**Input**: Design documents from `/specs/007-display-device-entities-in-order/` (`spec.md`, `plan.md`)

---

## Phase 1: Foundational / State Machine Extensions

- [x] T001 Update `DevicePageChunkedResponse.h` state enum and internal tracking members for entity ordering and empty port filtering in `Gateway/LoRaNetGateway/src/DevicePageChunkedResponse.h`

---

## Phase 2: User Story 1 - Display Configured Entities in Selected Order Below Options Card (Priority: P1) 🎯 MVP

**Goal**: Render all configured entity cards ordered by `DisplayOrder` (or list order) directly below the Options card on `/device`.

**Independent Test**: Load `/device?deviceid=fc0fe71454d8` and verify configured entity cards render in ascending `DisplayOrder` sequence directly below the Options card.

- [x] T002 [US1] Implement entity collection and sorting logic in `DevicePageChunkedResponse.cpp` to iterate through configured entities in `DisplayOrder` sequence in `Gateway/LoRaNetGateway/src/DevicePageChunkedResponse.cpp`
- [x] T003 [US1] Refactor State 1 chunk renderer to output sorted configured entity cards cleanly below the Options card in `Gateway/LoRaNetGateway/src/DevicePageChunkedResponse.cpp`
- [x] T004 [US1] Verify local build using `pio run` in `Gateway/LoRaNetGateway`

---

## Phase 3: User Story 2 - Render Unconfigured/Empty Sensor Ports at the End (Priority: P2)

**Goal**: Render all unconfigured / empty sensor ports (ports without attached entities/endpoints) grouped together after all configured entities at the end of the `/device` page.

**Independent Test**: Load `/device?deviceid=fc0fe71454d8` and verify empty ports (such as `CS`, `SM`, `OR`, `FW`, `PD`, `P1`) appear grouped at the bottom after all configured entity cards.

- [x] T005 [US2] Implement empty/unconfigured port filtering and iteration phase in `DevicePageChunkedResponse.cpp` to collect remaining raw sensor ports without entities in `Gateway/LoRaNetGateway/src/DevicePageChunkedResponse.cpp`
- [x] T006 [US2] Refactor State 2 chunk renderer to output empty/unconfigured port cards grouped at the bottom of the page in `Gateway/LoRaNetGateway/src/DevicePageChunkedResponse.cpp`
- [x] T007 [US2] Verify local build using `pio run` in `Gateway/LoRaNetGateway`

---

## Phase 4: Polish & Live Verification

- [x] T008 [P] Test live webpage at `http://192.168.68.104/device?deviceid=fc0fe71454d8` using `read_url_content` to verify entity ordering and empty port placement
- [x] T009 Hand over for user manual verification on hardware
- [x] T010 Create Pull Request for `Gateway/LoRaNetGateway` and update GitHub Issue #7 upon user approval
