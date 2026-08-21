# Feature Specification: Dynamic Add / Pair Device Page & Remote Router Discovery

**Feature Branch**: `009-new-add-device-pairing-page`  
**Created**: 2026-08-21  
**Last Updated**: 2026-08-22  
**Status**: Draft  
**Input**: GitHub Issue #9 ("New add device page shows list of received items not setup dynamically show devices as messages arrive")

---

## Overview
Re-architect the device discovery and pairing experience on the Gateway. The new **Add / Pair Device** screen (`/adddevice`) replaces static lists with an interactive real-time stream (Server-Sent Events) that dynamically pops in new unconfigured devices as their LoRa packets arrive over the air. It also enables querying farm Routers for remotely heard unconfigured nodes from their lightweight `PotentialList`, highlights the safest physical routes based on RSSI comparisons, presents expected-message countdown timers, and provides single-click router assignment.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Real-Time Live Discovery & Dynamic Card Streaming (Priority: P1 🎯 MVP)

When setting up new hardware in the farm field or workshop, the user opens the "Add Device" screen on a phone or laptop. The screen displays an active discovery indicator ("Listening for LoRa devices..."). As soon as a field node powers up and broadcasts its first telemetry packet, the Gateway captures it and pushes it over SSE, dynamically rendering the full device card (`sendDevice()`) on screen without page reloads.

**Why this priority**: Core value of Issue #9—provides instant visual feedback to installers in the field that their new device is communicating with the Gateway.

**Independent Test**:
- Open `/adddevice` in a browser.
- Inject a synthetic or live LoRa packet from an unknown MAC (`DeviceType::New`).
- Verify the device card dynamically appears in the DOM with live RSSI, port values, and Setup/Ignore buttons.

**Acceptance Scenarios**:
1. **Given** a user is viewing `/adddevice`, **When** a LoRa packet from an unconfigured device is received by the Gateway, **Then** the device card appears immediately at the top of the list via Server-Sent Events (SSE).
2. **Given** a device card is rendered on `/adddevice`, **When** the user clicks "Configure / Setup", **Then** the standard template/entity setup workflow is triggered.
3. **Given** a device card is rendered on `/adddevice`, **When** the user clicks "Ignore", **Then** the device is set to `DeviceType::Ignore` and removed from the active new list.

---

### User Story 2 - Historical 24-Hour Buffer & Filter Toggles (Priority: P1 🎯 MVP)

Field nodes often transmit periodically (e.g. every 15 minutes or hour). If a node transmitted before the user opened `/adddevice`, it should still be discoverable. The page checks `d->getTimeLastMsg()` and presents unconfigured devices heard in the last 24 hours as "Older Unsetup Devices".

**Why this priority**: Prevents technicians from having to wait up to an hour for sleeping sensor nodes to transmit their next periodic heartbeat.

**Independent Test**:
- Ensure an unconfigured device exists in memory with `timeLastMsg` within the last 24 hours.
- Open `/adddevice`.
- Toggle "Older Unsetup Devices" ON/OFF and verify the device card toggles visibility.

**Acceptance Scenarios**:
1. **Given** an unconfigured device (`DeviceType::New`) transmitted within the last 24 hours, **When** `/adddevice` loads, **Then** it is cataloged under "Older Unsetup Devices".
2. **Given** filter toggles at the top of `/adddevice`, **When** "Older Unsetup Devices" is toggled, **Then** historical 24-hour devices show/hide accordingly.
3. **Given** an ignored unconfigured device (`DeviceType::Ignore`), **When** "Ignored Unsetup Devices" is toggled ON, **Then** it appears with an "Unignore / Add" option.
4. **Given** any device with `DeviceType::Configured`, **When** `/adddevice` renders, **Then** it is strictly filtered out.

---

### User Story 3 - Remote Router Query & Multi-Route Smart Evaluation (Priority: P2)

On large farms, remote nodes may be out of direct Gateway radio range and only heard by one or more field Routers. The user can click "Query Router" for specific routers. The Gateway sends a LoRa `QN: 1` query to that Router, which replies from its in-memory `PotentialList` using individual Option A messages (`M1`, `M2`, `M3`, `DT`, `RS`, `SM`, `MM` minutes elapsed, and `QN: 255` on the final item or empty list). The UI aggregates routes heard across direct Gateway and Routers, displays expected-packet countdowns ("Next packet due in ~X mins"), and highlights the safest route.

**Why this priority**: Solves multi-hop farm pairing where devices cannot reach the Gateway directly.

**Independent Test**:
- Inject a router query response (`From: Router1, M1..M3: TargetMAC, DT: 2, RS: 95, SM: 15, MM: 5, QN: 255`).
- Query direct Gateway (`RS: 118`).
- Verify Router 1 is badged with green **"Safest Route (-95 dBm) — Due in ~10 mins"** and Gateway direct is badged in amber **"Weak Link (-118 dBm)"**.

**Acceptance Scenarios**:
1. **Given** one or more configured Routers on the network, **When** `/adddevice` renders, **Then** a "Query Router" toggle button is displayed for each router.
2. **Given** a user clicks "Query Router", **When** the Gateway transmits `QN: 1` to the router:
   - If empty, the Router responds with `QN: 255, QC: 0`, immediately concluding search.
   - If items exist, the Router streams Option A frames (`M1..M3`, `DT`, `RS`, `SM`, `MM`) with the final frame containing `QN: 255`.
3. **Given** LoRa packet loss on a stream, **When** the Gateway receives any router frame, **Then** it arms a 2-second idle timeout to automatically conclude the query if the final `QN: 255` frame is missed.
4. **Given** a device heard across multiple paths (Gateway direct and/or Routers), **When** evaluated on `/adddevice`, **Then** the path with the lowest attenuation / best RSSI is marked with a **Green "Safest Route"** badge.
5. **Given** any route with RSSI weaker than -115 dBm (> 115 dBm attenuation), **When** displayed, **Then** its RSSI badge is highlighted in **Amber / Warning**.

---

### User Story 4 - Single-Click Router Assignment & Next-Packet Promotion (Priority: P2)

When a node is discovered via a Router, the UI shows its MAC, Device Type (`DT`), RSSI, next-packet countdown ("Due in ~X mins"), and an "Add via Router" button. Clicking this button sends an `addRoutedNode` command to that Router. The Router saves the rule into `routedNodes[10]`. When the node's next scheduled transmission arrives at the Router, it forwards it to the Gateway with `MR: <routerID>`, promoting it to a fully-populated New Device card on the Gateway screen.

**Why this priority**: Seamlessly closes the loop on remote node pairing without requiring manual routing ID configuration.

**Independent Test**:
- On a router-discovered node, click "Add via Router".
- Verify `addRoutedNode` command is transmitted to the Router.
- Verify that when the next packet arrives with `MR: routerID`, the Gateway renders the full `DeviceType::New` card.

**Acceptance Scenarios**:
1. **Given** a node discovered via Router query, **When** the user clicks "Add via Router", **Then** the Gateway executes `d->addRoutedNode(mac, routerId)` and queues the configuration packet to the Router.
2. **Given** the Router accepts the routing rule, **When** the node transmits its next scheduled packet, **Then** the Router forwards it with `MR: <routerId>` and the Gateway renders the full `DeviceType::New` card on `/adddevice`.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Gateway MUST expose a dedicated Web UI at `/adddevice` (and alias `/pair`) accessible via the navigation bar "Add Device" icon button (`fa-circle-plus`).
- **FR-002**: Gateway MUST provide a Server-Sent Events (SSE) stream at `/api/pair/events` that pushes real-time JSON payloads whenever an unconfigured (`DeviceType::New`) LoRa packet is processed.
- **FR-003**: The Add Device web page MUST use `EventSource` to listen to `/api/pair/events` and dynamically insert or update device cards in the DOM in real time.
- **FR-004**: Gateway MUST filter all configured devices (`DeviceType::Configured`) out of the Add Device display list.
- **FR-005**: Gateway MUST evaluate `now - d->getTimeLastMsg() <= 86400` to categorize devices heard within the past 24 hours as "Older Unsetup Devices".
- **FR-006**: Add Device UI MUST provide interactive toggle filters:
  - `Toggle Older Unsetup Devices` (Default: OFF)
  - `Toggle Ignored Unsetup Devices` (Default: OFF)
  - Individual `Query Router [Router Name]` buttons for all registered routers on the farm network.
- **FR-007**: Gateway MUST support LoRa command port `QN: 1` (`Query New Devices`) sent to Routers to solicit their list of unconfigured devices from their `PotentialList`.
- **FR-008**: Routers MUST respond to `QN` using Option A individual messages (`M1..M3` for MAC, `DT` for Device Type, `RS` for RSSI, `SM` for sleep minutes, `MM` for elapsed minutes since last heard).
- **FR-009**: Routers MUST include `QN: 255` in the final message of the response stream, or send an immediate `QN: 255, QC: 0` frame if `PotentialList` is empty.
- **FR-010**: Gateway MUST arm a 2-second idle timeout upon receiving any router response frame to conclude the query gracefully if the final `QN: 255` ACK is lost.
- **FR-011**: Gateway MUST calculate `dueInMinutes = max(0, SM - MM)` to present an expected-packet arrival countdown on router-discovered devices.
- **FR-012**: Add Device UI MUST dynamically display a **Green "Safest Route"** badge on the path with the strongest RSSI.
- **FR-013**: Add Device UI MUST display an **Amber Warning** badge on any route with RSSI weaker than -115 dBm.
- **FR-014**: Router-discovered items MUST render with an "Add via Router" button that issues `addRoutedNode` to the target Router.
- **FR-015**: Navigation bar across all Gateway pages MUST include the "Add Device" link (`/adddevice`).

---

## REST & SSE Endpoint Contracts

### 1. SSE Event Stream: `GET /api/pair/events`
- **Protocol**: `text/event-stream`
- **Events**:
  - `event: new_device` &rarr; `data: {"id":"fc0fe71463c5","name":"Device 63C5","type":"New","rssi":-105,"timeLastMsg":1724230000,"ports":{"IV":325,"SM":15,"DT":0}}`
  - `event: router_device` &rarr; `data: {"routerId":101,"routerName":"North Hill Router","id":"fc0fe71454d8","dt":2,"rssi":-92,"sm":15,"minutesSinceLast":5,"dueInMinutes":10}`
  - `event: router_query_done` &rarr; `data: {"routerId":101,"count":2}`
  - `event: heartbeat` &rarr; `data: {"uptime":12345}`

### 2. Router Query Trigger: `POST /api/pair/query_router?routerId=<id>`
- **Response**: `{"success": true, "routerId": 101, "status": "Query QN transmitted"}`

### 3. Setup Route Assignment: `POST /api/pair/add_route?routerId=<id>&deviceId=<hex>`
- **Response**: `{"success": true, "routerId": 101, "deviceId": "fc0fe71454d8", "status": "Routing command queued"}`

---

## Success Criteria

1. **Zero Refresh Dynamic Addition**: An unconfigured LoRa field node powering up appears on `/adddevice` within < 1 second of Gateway packet reception without manual browser refresh.
2. **24-Hour Memory**: Unconfigured devices heard earlier in the day appear under "Older Unsetup Devices" when toggled ON.
3. **Multi-Router Safest Path**: Discovered devices heard across multiple routers or gateway directly display the Green "Safest Route" badge on the optimal path.
4. **Resilient Router Query**: Router responses stream into the UI with instant empty-list acknowledgment or 2-second idle timeout fallbacks.
5. **Automated Verification**: End-to-end injection and SSE event reception verified using `test_gateway_harness.py`.
