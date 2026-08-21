# Tasks: Dynamic Add / Pair Device Page & Remote Router Discovery

**Input**: Design documents from `specs/009-dynamic-add-device-pairing/` (`spec.md`, `plan.md`)  
**Feature Branch**: `009-new-add-device-pairing-page`

---

## Phase 1: Setup & Foundational Infrastructure

**Purpose**: Establish SSE (`/api/pair/events`), navbar links, and base routing.

- [ ] T001 Add "Add Device" link (`/adddevice`) to navbar in `Gateway/LoRaNetGateway/src/WebHelper.h` (`sendPageHeader`).
- [ ] T002 Attach `AsyncEventSource pairEvents("/api/pair/events")` to web server in `Gateway/LoRaNetGateway/src/main.cpp` and `Gateway/LoRaNetGateway/src/WebHelper.h`.
- [ ] T003 Implement `GET /api/pair/list` in `Gateway/LoRaNetGateway/src/WebHelper.h` returning in-memory unconfigured devices (`New` and `Ignore`) with 24-hour timestamp categorization.

---

## Phase 2: User Story 1 (Priority: P1 🎯 MVP) - Real-Time Dynamic Card Streaming

**Goal**: As unconfigured LoRa packets arrive at Gateway, broadcast SSE `new_device` events and dynamically render full device cards on `/adddevice` without page reloads.

- [ ] T004 Implement `new_device` SSE broadcast in `FarmNetwork::processNewMessage()` in `Gateway/LoRaNetGateway/src/FarmNetwork.cpp` when `d->getType() == DeviceType::New`.
- [ ] T005 Create `sendAddDevicePage()` in `Gateway/LoRaNetGateway/src/WebHelper.h` rendering the radar listening animation and card stream container.
- [ ] T006 Implement JavaScript client on `/adddevice` connecting to `EventSource('/api/pair/events')` to insert/update full device cards dynamically on incoming `new_device` events.
- [ ] T007 Add "Configure / Setup" and "Ignore" action buttons to dynamically rendered device cards.

---

## Phase 3: User Story 2 (Priority: P1 🎯 MVP) - Historical 24-Hour Buffer & Filter Toggles

**Goal**: Display unconfigured devices heard in the last 24h as "Older Unsetup Devices" with top filter toggles.

- [ ] T008 Implement top filter toggles on `/adddevice`:
  - `Toggle Older Unsetup Devices` (Default: OFF)
  - `Toggle Ignored Unsetup Devices` (Default: OFF)
- [ ] T009 Filter out all configured devices (`DeviceType::Configured`) from the discovery list.
- [ ] T010 Implement client-side filtering logic showing/hiding older 24h devices and ignored devices upon toggle click.

---

## Phase 4: User Story 3 (Priority: P2) - Remote Router Query & Safest Route Evaluation

**Goal**: Query Routers for candidate devices from their `PotentialList`, aggregate multi-path RSSI, and highlight Safest Route and packet countdowns.

- [ ] T011 Render individual `Query Router [Router Name]` buttons for all registered routers on `/adddevice`.
- [ ] T012 Implement `POST /api/pair/query_router?routerId=<id>` in `Gateway/LoRaNetGateway/src/WebHelper.h` transmitting `QN: 1` command to target router.
- [ ] T013 Parse incoming `QN` response stream in `FarmNetwork::processNewMessage()` (`M1..M3`, `DT`, `RS`, `SM`, `MM`, `QN: 255`) and broadcast `router_device` and `router_query_done` events.
- [ ] T014 Implement multi-route evaluation on `/adddevice`:
  - Aggregate paths heard across Direct Gateway and Routers for each unique MAC.
  - Apply **Green "Safest Route"** badge to the path with the strongest RSSI.
  - Apply **Amber Warning** badge to any link weaker than -115 dBm.
- [ ] T015 Render next-packet arrival countdown timer ("Next packet expected in ~X mins") for router-discovered nodes using `dueInMinutes = max(0, SM - MM)`.

---

## Phase 5: User Story 4 (Priority: P2) - Single-Click Router Assignment

**Goal**: Single-click "Add via Router" action issuing `CA: 1` to target router and promoting node to full card on next packet.

- [ ] T016 Implement `POST /api/pair/add_route?routerId=<id>&deviceId=<hex>` in `Gateway/LoRaNetGateway/src/WebHelper.h` queueing `CA: 1` routing command to the router.
- [ ] T017 Attach "Add via Router" button to router candidate cards triggering `POST /api/pair/add_route`.
- [ ] T018 Automatically promote device to a full `DeviceType::New` card when the router forwards the next scheduled packet with `MR: <routerID>`.

---

## Phase 6: Automated Dual-Surface Verification & Quality Gates

**Goal**: End-to-end automated testing, live hardware flashing, and manual testing pause.

- [ ] T019 Update `Gateway/LoRaNetGateway/scripts/test_gateway_harness.py` with test methods for `/adddevice` HTML validation, synthetic LoRa discovery, and `QN` router response simulation.
- [ ] T020 Compile firmware with `pio run` and verify zero errors.
- [ ] T021 Flash firmware to Gateway hardware (`pio run -t upload`) and run automated dual-surface regression suite.
- [ ] T022 Pause for user manual device testing and verification before requesting commit and Pull Request creation.
