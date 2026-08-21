# Implementation Plan: Dynamic Add / Pair Device Page & Remote Router Discovery

**Branch**: `009-new-add-device-pairing-page` | **Date**: 2026-08-22 | **Spec**: [specs/009-dynamic-add-device-pairing/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/009-dynamic-add-device-pairing/spec.md)

**Input**: Feature specification for GitHub Issue #9 ("New add device page shows list of received items not setup dynamically show devices as messages arrive")

---

## Summary
Implement a real-time, interactive **Add / Pair Device** experience (`/adddevice`) on the LoRa Gateway. The page connects to a Server-Sent Events (SSE) stream (`/api/pair/events`), instantly streaming in new unconfigured devices as live LoRa packets arrive over the air. It integrates historical 24-hour unsetup memory, remote router discovery (`QN: 1` query protocol), multi-route RSSI smart evaluation (highlighting the **Green "Safest Route"** and **Amber** for >115 dBm attenuation), next-packet arrival countdowns, and single-click router route assignment.

---

## Technical Context

- **Platform & Hardware**: ESP32-WROVER with 8MB PSRAM on Arduino framework (PlatformIO).
- **Web Server Architecture**: `ESPAsyncWebServer` with `AsyncEventSource` (`/api/pair/events`) for non-blocking SSE streaming.
- **LoRa Protocol Additions**:
  - `QN: 1`: Gateway &rarr; Router Query New Devices.
  - `M1..M3`, `DT`, `RS`, `SM`, `MM`, `QN: 255`: Router &rarr; Gateway PotentialList stream response.
  - `CA: 1`: Gateway &rarr; Router Add Node routing rule.
- **Memory & Storage**: In-memory `Device*` linked list in PSRAM (`FarmNetwork`), baseline snapshots in `/lfm/data/test_snapshot.bak`, clean reset via `FarmNetwork::clearDevices()`.
- **Testing & Quality Assurance**: Dual-Surface (API + Web UI) automated regression verification using `scripts/test_gateway_harness.py`, PlatformIO builds (`pio run`), and manual device testing.

---

## Proposed Changes

### Component 1: Gateway SSE Event Stream & REST Endpoints
Implement Server-Sent Events infrastructure and pairing REST endpoints.

#### [MODIFY] [AITestHarness.h](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/AITestHarness.h) / [WebHelper.h](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h)
- Create and attach `AsyncEventSource pairEvents("/api/pair/events")` to `server`.
- Implement `GET /api/pair/list` returning unconfigured devices (`New` and `Ignore`) with `timeLastMsg` timestamps and 24h age categorization.
- Implement `POST /api/pair/query_router?routerId=<id>` to transmit `QN: 1` packet to the router.
- Implement `POST /api/pair/add_route?routerId=<id>&deviceId=<hex>` to queue `CA: 1` routing command to the router.

#### [MODIFY] [FarmNetwork.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/FarmNetwork.cpp) & [FarmNetwork.h](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/FarmNetwork.h)
- In `FarmNetwork::processNewMessage()`:
  - When an unconfigured packet arrives (`d->getType() == DeviceType::New`):
    - Broadcast `new_device` JSON event over `pairEvents`.
  - When a `QN` response packet arrives from a Router (`M1`, `M2`, `M3`, `DT`, `RS`, `SM`, `MM`):
    - Parse target MAC, Device Type, RSSI, sleep interval (`SM`), elapsed minutes (`MM`), and compute `dueInMinutes = max(0, SM - MM)`.
    - Broadcast `router_device` JSON event over `pairEvents`.
    - If `QN: 255` is present, broadcast `router_query_done` JSON event.

---

### Component 2: Dynamic Web UI (`/adddevice` & `/pair`)
Build the modern, dynamic Add Device interface.

#### [MODIFY] [WebHelper.h](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h)
- Update `sendPageHeader()` navbar: add **"Add Device"** link (`<li class='nav-item'><a class='nav-link active' href='/adddevice'><i class='fa-solid fa-circle-plus'></i><span class='d-none d-sm-inline'> Add Device</span></a></li>`).
- Implement `sendAddDevicePage(request)`:
  - Top listening radar / pulse animation ("Listening for new LoRa devices...").
  - Filter toggles:
    - `Older Unsetup Devices` (toggles historical 24h unsetup devices, default OFF).
    - `Ignored Unsetup Devices` (toggles `DeviceType::Ignore`, default OFF).
    - `Query Router [Name]` buttons for each router registered on the network.
  - Interactive JavaScript client:
    - Connects to `EventSource('/api/pair/events')`.
    - Dynamically renders new device cards on incoming packets without page refresh.
    - Aggregates multi-route links (Direct Gateway vs Routers) and applies the **Green "Safest Route"** badge on the strongest RSSI, and **Amber** on links weaker than -115 dBm.
    - Displays live countdown timers ("Next packet expected in ~X mins") for router candidates.
    - "Add via Router" action button sending `POST /api/pair/add_route`.

---

### Component 3: Automated Dual-Surface Verification Harness
Extend `test_gateway_harness.py` to validate all real-time SSE discovery, router query parsing, and Web UI interactions.

#### [MODIFY] [test_gateway_harness.py](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/scripts/test_gateway_harness.py)
- Add test helpers for:
  - Fetching `/adddevice` and validating HTML structure (toggles, badges, router buttons).
  - Injecting synthetic unconfigured device frames and verifying state ingestion.
  - Injecting synthetic Router `QN` response frames (`M1..M3`, `DT`, `RS`, `SM`, `MM`, `QN: 255`) and validating route comparison / Safest Route badging.
  - Asserting 100% clean snapshot restore and inventory preservation.

---

## Verification Plan

### Automated Tests
1. **Compilation Check**:
   ```powershell
   & "C:\Users\USER\.platformio\penv\Scripts\platformio.exe" run
   ```
2. **Flash & Upload**:
   ```powershell
   & "C:\Users\USER\.platformio\penv\Scripts\platformio.exe" run -t upload
   ```
3. **Automated Dual-Surface Regression Suite**:
   ```powershell
   python scripts/test_gateway_harness.py --ip 192.168.68.104 --run-suite
   ```

### Manual Verification
1. Open `/adddevice` in a browser.
2. Confirm the navbar shows the "Add Device" button.
3. Test toggling "Older Unsetup Devices" and "Ignored Unsetup Devices".
4. Inject a packet and verify the device card pops up on screen dynamically in real time without refreshing.
5. Pause for user manual device testing before creating commits or Pull Requests.
