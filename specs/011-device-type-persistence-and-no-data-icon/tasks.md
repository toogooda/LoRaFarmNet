# Tasks: Device Type Persistence and No Data Icon

**Input**: Design documents from `/specs/011-device-type-persistence-and-no-data-icon/` (`spec.md`, `plan.md`)

---

## Phase 1: User Story 1 - Persist Device Type (DT) to Disk Upon Setup (Priority: P1) 🎯 MVP

**Goal**: Ensure setting up a device persists the Device Type (DT) port value to disk (`network.dat`) so category mappings survive reboots.

**Independent Test**: Set up a device under a category, trigger reload/reboot, and verify the DT port and category mapping persist.

- [x] T001 [US1] Audit `SDHelper.h` and `WebHelper.h` to verify DT port serialization and ensure `saveNetwork()` is called on device setup in `Gateway/LoRaNetGateway/src/WebHelper.h`
- [x] T002 [US1] Verify local build using `pio run` in `Gateway/LoRaNetGateway`

---

## Phase 2: User Story 2 - Mini View Icon for Devices with "No Data Yet" (Priority: P2)

**Goal**: Render an orange pending icon badge in Homepage Mini View for devices that have `!d->getHasData()`.

**Independent Test**: View the Homepage Mini View for an unconfigured or newly added node awaiting data, and verify the orange pending icon and "No data received yet" tooltip display.

- [x] T003 [US2] Update `HomePageChunkedResponse.cpp` mini view renderer to check `!d->getHasData()` and output the orange pending "No Data Yet" icon badge in `Gateway/LoRaNetGateway/src/HomePageChunkedResponse.cpp`
- [x] T004 [US2] Verify local build using `pio run` in `Gateway/LoRaNetGateway`

---

## Phase 3: Polish & Verification

- [x] T005 Upload firmware to Gateway on COM3, wait 30s, and fetch live webpage at `http://192.168.68.104/` using `read_url_content` to verify mini view icon and DT persistence
- [x] T006 Hand over for user manual verification on hardware
- [x] T007 Create Pull Request for `Gateway/LoRaNetGateway` and update GitHub Issue #11 upon user approval
